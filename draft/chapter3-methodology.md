# 第三章 多 Sketch 協同攻擊檢測框架

本章描述框架的設計。核心命題是：不依賴任何單一 Sketch 的完美性，而是讓多個互補的 Sketch 同時運行，各自產出所擅長的特徵，再將這些異質特徵融合成分類決策的依據。

## 3.1 框架總覽

多 Sketch 協同框架包含四個層次：Sketch 層、特徵擷取層、融合層、以及分類層。

```
┌─────────────────────────────────────────────────────────┐
│                    SDN Controller                       │
│  ┌─────────────────────────────────────────────────┐   │
│  │              融合層 + 分類層                       │   │
│  │  ┌──────────────────────────────────────────┐   │   │
│  │  │         Joint Feature Vector             │   │   │
│  │  │  [freq_dist | entropy | heavy_hitters |  │   │   │
│  │  │   cardinality | quantiles | ...]         │   │   │
│  │  └────┬──────────┬──────────┬──────────────┘   │   │
│  │       │          │          │                   │   │
│  └───────┼──────────┼──────────┼───────────────────┘   │
│          │          │          │                       │
│  ┌───────┴────┐ ┌───┴────┐ ┌──┴───────┐               │
│  │ CM Sketch  │ │Elastic │ │ UnivMon  │  UCL-Sketch  │
│  │ (frequency)│ │ Sketch │ │(universal│  (learned)    │
│  └────────────┘ └────────┘ └──────────┘               │
│                                                         │
│  輸出：Normal / DDoS_spoofed / DDoS_botnet /            │
│        Port_Scan / Flash_Crowd / Unknown                │
└─────────────────────────────────────────────────────────┘
         ↑  per-packet updates (OpenFlow / P4)
    ┌─────────┐     ┌─────────┐
    │ Switch 1│ ... │ Switch N│
    └─────────┘     └─────────┘
```

四個 Sketch 在交換機端並行運行，每個封包同時更新所有 Sketch。特徵擷取層定期（預設每 5 秒）從每個 Sketch 讀取其產出，彙整成一個聯合特徵向量。融合層對向量進行正規化與加權。分類層根據聯合向量判斷是否發生攻擊以及攻擊類型。

### 3.1.1 為什麼是這四種 Sketch

Sketch 的選擇不是隨意的，而是基於**互補性**和**部署可行性**兩個原則。

**CM Sketch** 是最基礎也最成熟的頻率估計 Sketch，幾乎所有可程式化交換機平台都支援。它產出頻率分佈特徵——這是最通用的流量信號，任何攻擊都會先反映在頻率變化上。

**Elastic Sketch** 補足了 CM Sketch 在結構性資訊上的盲點。它的 heavy/light 分離機制能清楚區分「少數大 flow」和「大量小 flow」，這兩種流量結構對應完全不同的攻擊類型（DDoS vs Port Scan）。

**UnivMon** 提供的是靈活性。前兩者部署時就決定了要量測什麼（頻率、熵值、heavy hitter），UnivMon 的通用摘要可以在事後回答部署時沒預先設定的查詢——這在面對未知攻擊時有實務價值。

**UCL-Sketch** 代表精度和效率的權衡選項。在記憶體極度受限的場景下，UCL-Sketch 可以在相同記憶體預算下達到比 CM Sketch 更高的頻率估計精度，讓框架在低成本硬體上也能運行。

這四種 Sketch 的選擇不是固定不變的——框架的設計允許根據部署環境增減 Sketch 種類。第三章的重點不是「這四種一定要用」，而是「如何系統性地選擇和組合 Sketch」。

---

## 3.2 Sketch 層：並行部署與資料收集

### 3.2.1 部署模式

框架支援兩種部署模式：

**控制器端集中模式**：四個 Sketch 實例運行在控制器上。交換機透過 OpenFlow 的 flow stats 定期回報流量統計，控制器端更新 Sketch。優點是部署簡單，不需要修改交換機硬體或韌體。缺點是延遲較高——從攻擊發生到控制器感知，中間有 flow stats 的 polling 間隔。適合原型驗證和實驗階段。

**資料平面分散模式**：Sketch 直接在交換機的資料平面上運行。對 P4 可程式化交換機（如 Barefoot Tofino），CM Sketch 和 Elastic Sketch 已有成熟的硬體實作（Lai et al., 2022）；UnivMon 和 UCL-Sketch 的硬體移植則需要額外工程。各交換機獨立維護自己的 Sketch，控制器定期彙整。優點是延遲低、吞吐量高。缺點是部署門檻較高。

本研究的實驗以控制器端集中模式為主，但第三章的方法設計與部署模式無關——聯合特徵的定義和分類邏輯在兩種模式下完全相同。

### 3.2.2 封包處理流程

每個封包經過交換機時，同時觸發四個 Sketch 的更新：

1. **CM Sketch**：對 flow 5-tuple 做 hash → 遞增對應的 $d$ 個計數器
2. **Elastic Sketch**：Ostracism 機制判斷 heavy/light → 更新 heavy part 或 light part 的 CM Sketch
3. **UnivMon**：對封包 header 做 universal hash → 更新通用摘要結構
4. **UCL-Sketch**：對 flow key 做 hash → 更新學習型估計桶

每個 Sketch 使用獨立的記憶體空間，互不干擾。總記憶體佔用為四個 Sketch 各自的記憶體之和。

### 3.2.3 時間窗口與資料讀取

每 $T = 5$ 秒（可配置），控制器從四個 Sketch 中讀取以下資料：

| Sketch | 讀取內容 |
|--------|---------|
| CM Sketch | 完整計數器陣列（$d \times w$ 個計數值） |
| Elastic Sketch | Heavy part 的 flow list + light part 的計數器分佈 |
| UnivMon | 通用摘要結構的 bit array |
| UCL-Sketch | 學習型估計桶的狀態向量 |

讀取後，計數器可選擇清零或保留（sliding window vs. tumbling window）。預設使用 tumbling window（每窗獨立），以簡化基準向量的建立。

---

## 3.3 特徵擷取層

### 3.3.1 從各 Sketch 擷取的特徵

每個時間窗口結束後，從四個 Sketch 中擷取以下特徵，形成聯合特徵向量：

| 特徵 | 來源 Sketch | 說明 | 維度 |
|------|-----------|------|:--:|
| $F_{freq}$ | CM Sketch | 頻率分佈的統計量（均值、變異數、最大值） | 3 |
| $F_{entropy}$ | Elastic Sketch | 六個流量維度的歸一化熵值 | 6 |
| $F_{hh}$ | Elastic Sketch | Heavy hitter 數量及佔總流量比例 | 2 |
| $F_{card}$ | Elastic Sketch | Light part 的 cardinality 估計（非零計數器數） | 1 |
| $F_{univ}$ | UnivMon | 通用摘要的查詢結果（預設查 entropy 和分位數） | 2 |
| $F_{ucl}$ | UCL-Sketch | 高精度頻率估計的統計摘要 | 3 |

**聯合特徵向量維度**：$3 + 6 + 2 + 1 + 2 + 3 = 17$ 維。

維度不高（17 維），避免了高維特徵空間帶來的維數災難，同時也比任何單一 Sketch 產出的特徵（最多 6 維的 Elastic Sketch 熵值向量）豐富得多。

### 3.3.2 各特徵的計算方式

**頻率分佈統計（$F_{freq}$）**：從 CM Sketch 的計數器陣列中計算三個統計量——平均計數值 $\mu$、計數值的標準差 $\sigma$、以及最大計數值 $\max(c)$。這三個數字捕捉了 flow 頻率分佈的核心形狀：$\mu$ 反映整體流量水準、$\sigma$ 反映 flow 間的不均勻度、$\max(c)$ 反映是否存在極端的 elephant flow。

**六維度熵值（$F_{entropy}$）**：從 Elastic Sketch 的 light part CM Sketch 中，分別對來源 IP、目的地 IP、來源埠、目的地埠、5-tuple flow key、協定編號等六個 hash 目標計算 Shannon 熵值，歸一化到 $[0, 1]$。計算公式與之前版本相同：

$$H = -\sum_{i=1}^{m} \frac{i \cdot n_i}{N} \cdot \log_2\left(\frac{i \cdot n_i}{N}\right), \quad \hat{H} = H / \log_2(N)$$

**Heavy hitter 資訊（$F_{hh}$）**：直接從 Elastic Sketch 的 heavy part 讀取——heavy part 中的 flow 數量、這些 flow 的總流量佔全部流量的比例。這兩個數字直觀地區分「少數 flow 佔據大部分流量」和「流量均勻分散」兩種極端情境。

**Cardinality（$F_{card}$）**：Elastic Sketch light part 的 CM Sketch 中非零計數器的數量。這個數字近似於活躍 flow 的數量，對 Port Scan（大量新 flow）特別敏感。

**通用摘要查詢（$F_{univ}$）**：從 UnivMon 的通用摘要中查詢兩個預設指標——整體流量的 Shannon 熵值估計、以及 flow 大小的 95 百分位數。後者提供了流量分佈尾端的資訊。

**UCL 頻率摘要（$F_{ucl}$）**：從 UCL-Sketch 的學習型估計桶中提取 flow 頻率的均值、變異數、以及估計精度的信心指標。

### 3.3.3 特徵正規化

各特徵的數值範圍差異很大——$\max(c)$ 可能達到數十萬，而 $\hat{H}$ 永遠在 $[0, 1]$。為了讓分類器公平對待所有特徵，使用 Z-score 正規化：

$$F_i^{norm} = \frac{F_i - \mu_i}{\sigma_i}$$

其中 $\mu_i$ 和 $\sigma_i$ 是在無攻擊校準期間計算的特徵 $i$ 的均值和標準差。正規化後，每個特徵的數值代表「相對於正常基準偏離了幾個標準差」。

---

## 3.4 融合層與分類層

### 3.4.1 異常檢測

計算當前聯合特徵向量與基準向量的加權歐氏距離：

$$D(t) = \sqrt{\sum_{i=1}^{17} w_i \cdot (F_i^{norm}(t))^2}$$

其中 $w_i$ 是特徵 $i$ 的權重。初始設定所有權重均等（$w_i = 1/17$），後續可根據消融實驗結果調整——對區分攻擊貢獻大的特徵給更高權重。

當 $D(t) > \tau$ 時觸發異常。$\tau$ 根據校準期間距離分佈的第 99 百分位數設定，對應預期的 1% 誤報率。

### 3.4.2 攻擊分類

異常觸發後，分類器分析 17 維聯合向量的偏移模式，與預先定義的攻擊指紋進行比對。指紋定義為每個攻擊類型在各特徵上的預期偏移方向（↑偏高 / →正常 / ↓偏低）：

| 特徵群 | DDoS (spoofed) | DDoS (botnet) | Port Scan | Flash Crowd |
|--------|:---:|:---:|:---:|:---:|
| $F_{freq}$: max | ↑↑ | ↑↑↑ | → | ↑↑ |
| $F_{entropy}$: srcIP | ↑↑ | ↑ | → | ↓ |
| $F_{entropy}$: dstIP | → | → | ↑↑ | ↑↑ |
| $F_{entropy}$: dstPort | → | → | ↑↑ | ↑ |
| $F_{hh}$: count | → | ↑ | ↓↓ | ↑↑ |
| $F_{hh}$: ratio | → | ↑↑ | ↓↓ | ↑↑ |
| $F_{card}$ | ↑ | ↑ | ↑↑↑ | ↑↑ |

每個攻擊類型定義一個參考偏移向量 $\mathbf{R}_k = [r_1, r_2, ..., r_{17}]$，其中 $r_i \in \{-1, 0, +1\}$ 對應 ↓/→/↑。分類時計算當前歸一化向量與各參考向量的相似度（餘弦相似度），取最高分對應的攻擊類型：

$$k^* = \arg\max_k \frac{\mathbf{F}^{norm} \cdot \mathbf{R}_k}{\|\mathbf{F}^{norm}\| \cdot \|\mathbf{R}_k\|}$$

若所有相似度低於門檻（預設 0.5），輸出 "Unknown Anomaly"——保留發現新型態攻擊的空間。

### 3.4.3 分類器設計的考量

選擇規則匹配而非 ML 分類器的原因有兩個。第一，可解釋性——當系統判定某個流量為 Port Scan，管理者可以看到「cardinality 偏高、heavy hitter ratio 偏低」，這些是直觀且可驗證的理由。第二，冷啟動——規則匹配不需要訓練資料，框架部署後可以立即運作。缺點是規則的定義依賴領域知識，當攻擊行為與預期指紋有出入時可能誤判。第四章會比較規則匹配與輕量級 ML（決策樹）在相同特徵上的效果。

---

## 3.5 運作流程總覽

```
每個時間窗口 t（5 秒）：
  1. 從四個 Sketch 中讀取原始資料
  2. 從 CM Sketch 計算 F_freq（μ, σ, max）
  3. 從 Elastic Sketch 計算 F_entropy（6 維）+ F_hh（2 維）+ F_card（1 維）
  4. 從 UnivMon 查詢 F_univ（2 維）
  5. 從 UCL-Sketch 計算 F_ucl（3 維）
  6. Z-score 正規化 → 17 維聯合向量
  7. 計算加權歐氏距離 D(t)
  8. 若 D(t) > τ：
     a. 餘弦相似度比對 4 種攻擊參考向量
     b. 輸出最佳匹配攻擊類型（或 Unknown）
  9. 否則：輸出 Normal
```

---

## 3.6 資源分析

四個 Sketch 的記憶體佔用合計：CM Sketch ~96 KB（d=3, w=4096, 4B/counter）、Elastic Sketch ~600 KB（含 heavy part hash table）、UnivMon ~200 KB、UCL-Sketch ~50 KB。總計約 1 MB，遠小於基於 flow table 的方法（可能需要數十 MB）。

每窗口的計算開銷集中在步驟 2-5，複雜度上限為 $O(d \cdot w)$（遍歷 CM Sketch 計數器陣列），在 $d=3, w=4096$ 時約 12K 次操作，在現代 CPU 上可在毫秒級完成。
