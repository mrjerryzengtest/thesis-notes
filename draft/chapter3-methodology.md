# 第三章 方法論

本章描述多維度熵值攻擊檢測與分類框架的設計。系統由三個模組構成：部署於交換機端的資料收集層、運行於控制器的熵值計算引擎、以及攻擊分類器。

## 3.1 系統架構

```
┌─────────────────────────────────────────────────┐
│                  SDN Controller                  │
│  ┌───────────┐  ┌──────────────┐  ┌──────────┐ │
│  │  Elastic   │  │ Multi-Dim    │  │ Attack    │ │
│  │  Sketch    │→│ Entropy      │→│ Classifier│ │
│  │  (Flow     │  │ Calculator   │  │           │ │
│  │   Stats)   │  │              │  │           │ │
│  └───────────┘  └──────────────┘  └──────────┘ │
│                                                  │
│  輸出：Normal / DDoS_spoofed / DDoS_botnet /     │
│        Port_Scan / Flash_Crowd                   │
└─────────────────────────────────────────────────┘
         ↑  flow statistics
    ┌─────────┐     ┌─────────┐
    │ Switch 1│ ... │ Switch N│
    └─────────┘     └─────────┘
```

**資料收集層**在每台 OpenFlow 交換機上維護一個 Elastic Sketch 實例。Elastic Sketch 將流量分為 heavy part（精確追蹤大象流）和 light part（以 CM Sketch 摘要老鼠流）。本方法只使用 light part 的 CM Sketch 計數器陣列進行熵值計算——heavy part 的 flow-level 資訊在此不需要，因為熵值只需要統計分佈，不需要個別 flow 的細節。

選擇 Elastic Sketch 而非獨立的 CM Sketch 有兩個原因。一是實務考量：交換機上已經有 Elastic Sketch 處理其他量測任務（heavy hitter 檢測、heavy change 檢測），熵值計算可以直接複用現有的 light part 結構，不引入額外負擔。二是當 flow 從 light part 晉升到 heavy part 時，計數器會被清除，相當於自動過濾了極端 outlier，對熵值估計反而有正面影響。

**熵值計算引擎**定期（預設每 5 秒）從 light part 的 CM Sketch 中讀取六個維度的計數器分佈，各自計算歸一化熵值，形成一個六維向量。

**攻擊分類器**基於預先建立的攻擊指紋模型進行兩階段判斷：先檢測是否有異常，再比對異常的偏移模式來分類攻擊類型。

---

## 3.2 多維度熵值計算

### 3.2.1 六個監測維度

每個封包經過交換機時，Elastic Sketch 根據封包的五個 header 欄位進行 hash 更新。本研究監測以下六個維度：

| 符號 | 維度 | Hash 對象 |
|------|------|----------|
| $H_{srcIP}$ | 來源 IP 熵值 | srcIP（32-bit） |
| $H_{dstIP}$ | 目的地 IP 熵值 | dstIP（32-bit） |
| $H_{srcPort}$ | 來源埠熵值 | srcPort（16-bit） |
| $H_{dstPort}$ | 目的地埠熵值 | dstPort（16-bit） |
| $H_{flowSize}$ | 流量大小熵值 | 5-tuple flow key（104-bit） |
| $H_{proto}$ | 協定熵值 | protocol（8-bit） |

前五個維度各自對應 CM Sketch 的一組獨立 hash 函數——同一個封包會同時更新六個維度的計數器。第六個維度 $H_{proto}$ 較為特殊：因為協定類型只有少數幾種（TCP=6, UDP=17, ICMP=1），其熵值變化範圍有限，但對於檢測特定攻擊（如 ICMP flood）仍有價值。

### 3.2.2 熵值計算與歸一化

每個時間窗口結束時，從 CM Sketch 讀取計數器陣列。令 $n_i$ 表示計數值為 $i$ 的計數器數量（跨所有列合併），$N = \sum_i i \cdot n_i$ 為總封包數，$m$ 為最大計數值。

該維度的原始熵值為：

$$H = -\sum_{i=1}^{m} \frac{i \cdot n_i}{N} \cdot \log_2\left(\frac{i \cdot n_i}{N}\right)$$

歸一化到 $[0, 1]$：

$$\hat{H} = \frac{H}{\log_2(N)}$$

歸一化讓不同維度的熵值可以直接比較——$\hat{H} \approx 0$ 表示流量集中在極少數值（例如所有封包打到同一個 IP），$\hat{H} \approx 1$ 表示流量均勻分散在所有可能值。

### 3.2.3 時間平滑

單一窗口的熵值對瞬時抖動敏感。我們採用指數加權移動平均（EWMA）平滑：

$$H^{smooth}(t) = \alpha \cdot H(t) + (1-\alpha) \cdot H^{smooth}(t-1)$$

$\alpha$ 預設為 0.3，對應約 3 個時間窗口的衰減記憶。這個數值在靈敏度與穩定性之間取了折衷——$\alpha$ 太高會放大雜訊導致誤報，太低則攻擊開始後需要數十秒才會觸發警報。

### 3.2.4 CM Sketch 偏差修正

CM Sketch 的高估偏差在正常流量下影響有限，但在攻擊情境下——大量 elephant flow 湧入 light part，計數器被反覆碰撞推高——會顯著扭曲計數器分佈，使熵值被低估。

修正方法：利用 CM Sketch 的理論誤差上界 $\mathbb{E}[\text{error}] \leq N / w$（$w$ 為每列計數器數），對每個計數器進行保守折減：

$$c_j^{corrected} = \max(0, c_j - \lceil N / w \rceil)$$

折減後重新計算有效封包總數和計數器分佈，再代入熵值公式。這個方法的保守性在於「可能折減過頭」——部分合法的小計數值可能被歸零。因此在極低流量情境下（$N$ 很小、$N/w < 1$），跳過修正步驟。

---

## 3.3 熵值向量與攻擊指紋

經過上述計算後，每個時間窗口產出一個六維歸一化熵值向量：

$$\mathbf{H}(t) = [\hat{H}_{srcIP}, \hat{H}_{dstIP}, \hat{H}_{srcPort}, \hat{H}_{dstPort}, \hat{H}_{flowSize}, \hat{H}_{proto}]$$

### 3.3.1 攻擊的預期熵值模式

不同網路事件預期會在六維向量上產生不同的偏移。以下是基於理論分析的預期模式——請注意這些是假設（hypotheses），第四章將透過實驗進行驗證：

| 事件 | H_srcIP | H_dstIP | H_dstPort | H_flowSize | H_proto |
|------|:-------:|:-------:|:---------:|:----------:|:-------:|
| 正常流量 | 中 | 中 | 中 | 中 | 中 |
| DDoS（偽造來源） | ↑↑ | → | → | ↑ | → |
| DDoS（殭屍網路） | ↑ | → | → | ↑↑ | → |
| Port Scan | → | ↑↑ | ↑↑ | ↓ | → |
| Flash Crowd | ↓ | ↑↑ | ↑ | ↑↑ | → |

↑ 代表熵值預期高於正常基準、↓ 代表低於基準。箭頭數量表示預期偏移幅度。

這個指紋表的核心邏輯不在於絕對數值，而在於**相對關係**。DDoS（偽造來源）和 Flash Crowd 在 $H_{flowSize}$ 上都偏高，但前者 $H_{srcIP}$ 會飆升（隨機偽造 IP），後者 $H_{srcIP}$ 反而可能略低於正常（真實使用者群有限）。$H_{dstIP}$ 的差異是另一個關鍵——DDoS 打單一目標所以正常或偏低，Flash Crowd 可能打多台伺服器所以偏高。

**DDoS 的兩種子類型**需要特別區分：偽造來源 IP 的 DDoS 會產生極高的 $H_{srcIP}$（每個封包都是新的來源 IP），殭屍網路的 $H_{srcIP}$ 只會略高（殭屍主機數量有限），但後者的 $H_{flowSize}$ 更高（每台殭屍發送大量封包，類似 elephant flow）。這兩個指標的組合——高 $H_{srcIP}$ + 中等 $H_{flowSize}$ vs 中等 $H_{srcIP}$ + 高 $H_{flowSize}$——是區分兩種 DDoS 的主要依據。

**Port Scan 和 Flash Crowd** 都表現為 $H_{dstIP}$ 和 $H_{dstPort}$ 偏高，容易混淆。區分點在 $H_{flowSize}$：Port Scan 的每個 flow 極短（通常 1-2 個封包），$H_{flowSize}$ 會顯著低於正常；Flash Crowd 是大量真實使用者產生實質流量，$H_{flowSize}$ 反而偏高。

### 3.3.2 基準向量建立

正常流量的基準向量 $\mathbf{H}_{baseline}$ 透過在無攻擊時段採集平均值獲得：

$$\mathbf{H}_{baseline} = \frac{1}{T}\sum_{t=1}^{T} \mathbf{H}(t) \quad \text{（無攻擊時段）}$$

向量中每個維度的標準差 $\sigma_d$ 也同時記錄，用於後續的偏移量化。

實務上，基準向量需要定期更新以適應網路的正常變化（例如白天和晚上的流量模式不同）。目前的設計假設基準在校準期間內是穩定的；更複雜的動態基準更新機制留待後續研究。

---

## 3.4 分類策略

### 3.4.1 第一階段：異常檢測

計算當前向量與基準向量的歐氏距離：

$$D(t) = \|\mathbf{H}(t) - \mathbf{H}_{baseline}\|_2$$

當 $D(t) > \tau$ 時觸發異常警報。$\tau$ 根據校準期間距離分佈的第 99 百分位數設定——這意味著在正常情況下，誤報率預期低於 1%。

選擇歐氏距離而非更複雜的 Mahalanobis 距離，是出於計算成本和可解釋性的考量。六維歐氏距離的物理意義直觀——就是「當前流量模式與正常模式差了多遠」。Correlation 在六維度之間確實存在（例如 $H_{srcIP}$ 和 $H_{flowSize}$ 在某些攻擊中會一起變動），但加入 covariance 矩陣會增加計算複雜度和對基準資料量的需求。

### 3.4.2 第二階段：攻擊分類

異常觸發後，計算偏移向量：

$$\Delta\mathbf{H} = \mathbf{H}(t) - \mathbf{H}_{baseline}$$

每個維度的偏移量化為三個等級：

- 若 $\Delta H_d > \sigma_d$：↑（偏高）
- 若 $\Delta H_d < -\sigma_d$：↓（偏低）
- 否則：→（正常）

將量化後的偏移向量與 3.3.1 節的預期指紋進行比對。比對採用加權相似度：為每種攻擊定義一個權重向量，對區分該攻擊最有鑑別力的維度給予較高權重。

範例：對 DDoS（偽造來源）的比對中，$H_{srcIP}$ 權重最高（因為這是此類攻擊最顯著的特徵），$H_{proto}$ 權重最低（通常沒有異常）。若量化向量在所有高權重維度上都與 DDoS（偽造來源）的預期方向一致，總分就會最高，系統輸出該分類。

如果沒有任何攻擊類型的相似度達到門檻，輸出「Unknown Anomaly」——這保留了發現未知攻擊模式的可能性。

### 3.4.3 替代方案

當網路環境穩定、攻擊模式明確時，閾值規則可以滿足需求。當面對更複雜的混合攻擊或邊界案例時，可以改用輕量級分類器處理六維熵值向量。決策樹（Decision Tree）和 KNN（$k=3$）是兩種適合的候選方案：前者規則透明、可追溯判決邏輯；後者不需要訓練階段、適合流式資料的線上分類。

---

## 3.5 計算與記憶體成本

每個時間窗口的主要操作：讀取 CM Sketch 計數器陣列（$O(d \cdot w)$）、偏差修正（$O(d \cdot w)$）、六次熵值計算（各 $O(d \cdot w)$）、一次歐氏距離 + 分類比對（$O(1)$）。總時間複雜度 $O(d \cdot w)$。

在典型配置下（$d=3$ 列、$w=2^{12}=4096$ 行，每計數器 4 bytes），六個維度的總記憶體約 $6 \times 3 \times 4096 \times 4 \approx 288$ KB。加上 Elastic Sketch heavy part 的 hash table，整體記憶體在 1-2 MB 以內。

這個數字對交換機而言可行——即使是硬體資源受限的場景，也可以只在控制器端集中維護一個 Elastic Sketch（透過 OpenFlow 的 flow stats 定期收集），犧牲部分即時性換取部署彈性。
