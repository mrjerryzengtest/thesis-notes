# 11 — 文獻詳細整理（完整版）

> 整理日期：2026-07-06
> 涵蓋：教授指定全部 11 篇文獻之詳細技術分析

---

# 第一部分：原始 6 項參考資料（2026-05-01）

---

## ❶ Elastic Sketch — SIGCOMM 2018 ⭐ Baseline

**完整引用：** Tong Yang, Jie Jiang, Peng Liu, Qun Huang, Junzhi Gong, Yang Zhou, Rui Miao, Xiaoming Li, Steve Uhlig. "Elastic Sketch: Adaptive and Fast Network-wide Measurements." *ACM SIGCOMM 2018*, pp. 561-575. DOI: 10.1145/3230543.3230544. 引用 539+。

### 問題陳述

網路壅塞、掃描攻擊、DDoS 發生時流量特徵劇烈變化——此時測量反而最重要，但傳統 sketch 方法效能會崩潰。需要一種能在流量動態變化時仍保持高效能的**自適應**網路測量 sketch。

### 核心方法：Heavy Part + Light Part 分離架構

```
┌─────────────────────────────────────────┐
│              Elastic Sketch              │
├────────────────┬────────────────────────┤
│  Heavy Part    │     Light Part         │
│  (hash table)  │   (CM Sketch)          │
│                │                        │
│  Elephant      │   Mouse flows          │
│  flows         │   (no flow ID stored)  │
│  (store ID +   │                        │
│   votes + flag)│   8-bit counters       │
└────────────────┴────────────────────────┘
```

#### Ostracism 分離機制（靈感：古希臘陶片放逐制）

每個 Heavy bucket 儲存 `(flow_ID, vote⁺, flag, vote⁻)`：
- 當 incoming packet 與 bucket 中 flow 不同 → vote⁻++
- 當 vote⁻ / vote⁺ ≥ λ（λ=8）→ 驅逐原 flow，新 flow 當選
- 被驅逐的 flow → 進入 Light Part（CM Sketch）

#### 三種自適應能力

| 類型 | 機制 | 效果 |
|------|------|------|
| 頻寬自適應 | Maximum Compression（MC）與 Sum Compression（SC） | 壓縮 sketch 適應可用頻寬 |
| 封包速率自適應 | 高速率時僅 1 次記憶體存取（只用 Heavy Part） | 保證線速處理 |
| Flow 分佈自適應 | 動態倍增 Heavy Part（copy-and-double） | 處理 flow 數量變化 |

#### 六種測量任務（單一 600KB 結構）

| # | 任務 | 方法 |
|---|------|------|
| 1 | Flow Size | Heavy Part 直接統計 + Light Part MRAC 估計 |
| 2 | Heavy Hitter | Heavy Part 中超過閾值的 flow |
| 3 | Heavy Change | 相鄰時間窗 Heavy Part 比對 |
| 4 | Distribution | Light Part counter distribution |
| 5 | **Entropy** | 從 Light Part counter distribution 計算 Shannon entropy |
| 6 | Cardinality | Light Part 中非零 counter 數量 |

#### 熵值估計方法

```
從 Light Part 收集 counter distribution array (n₀, n₁, ..., n₂₅₅)

H = -Σ (i × nᵢ / m × log(nᵢ / m))
其中 m = Σ nᵢ
```

- Heavy Part flow 資訊直接獲取
- Light Part 使用 MRAC（Multi-Resolution Array of Counters）估計 flow size distribution
- 結合兩者計算整體熵值

#### 實驗結果（熵值部分）

- memory ≥ 400KB 時，準確度超越所有 state-of-the-art（UnivMon、Sieving）
- 評估指標：相對誤差（RE）
- 部署平台：P4、FPGA、GPU、CPU、multi-core CPU、OVS 共六種

#### ⚠️ 對你的論文最關鍵的一句話

§6.1 作者承認：
> *"For other tasks (e.g., DDoS, SuperSpreader, and more), we will study how to apply Elastic in the future work."*

→ **你的論文直接接這個棒子。**

### 論文中的角色：Baseline 測量工具

---

## ❷ PINT — SIGCOMM 2020 ⭐ Motivation

**完整引用：** Ran Ben Basat, Sivaramakrishnan Ramanathan, Yuliang Li, Gianni Antichi, Minlan Yu, Michael Mitzenmacher. "PINT: Probabilistic In-band Network Telemetry." *ACM SIGCOMM 2020*, pp. 662-680. DOI: 10.1145/3387514.3405894. 引用 248+。

### 問題陳述

INT（In-band Network Telemetry）讓交換機在每個封包上附加 telemetry 資訊，但 overhead 過大：
- 5 跳路徑 + 2 個值/跳 = 48 bytes/packet overhead
- → Flow completion time 增加 25%
- → Goodput 下降 20%

### 核心方法：概率式編碼

將 telemetry 資訊概率性地編碼到**多個封包**上：
- Per-packet overhead 可低至 **1 bit**
- 16 bits/packet 即可同時支援 congestion control、path tracing、latency estimation

#### 三種聚合模式

| 模式 | 說明 | 應用 |
|------|------|------|
| Per-packet | 路徑上各跳值聚合（max/min/sum） | HPCC 擁塞控制 |
| Static per-flow | Flow 路徑上固定值 | Path Tracing |
| Dynamic per-flow | 每個 (flow, switch) 統計分佈 | Latency 分位數 |

### ⚠️ 最關鍵的洞察（§4.1）

> *"Aggregation functions like the number of distinct values or the value-frequency distribution entropy are poorly approximable from subsampled streams."*

→ 概率性子採樣會嚴重損害熵值估計準確性
→ **這直接是你研究的 motivation**：用 sketch-based（非 sampling）方法解決此限制

### 論文中的角色：Motivation 來源

---

## ❸ Minlan Yu — Top-Down Telemetry（CCRV 2019）

**完整引用：** Minlan Yu. "Network Telemetry: Towards a Top-Down Approach." *ACM SIGCOMM Computer Communication Review*, Vol. 49, No. 1, 2019, pp. 11-17. DOI: 10.1145/3314212.3314215。

### 核心論點

反對傳統 Bottom-up 方法（先做硬體再想要量什麼），主張 **Top-down**：
1. 營運商先宣告「想要量什麼」
2. 系統自動轉譯成交換機上的可程式化測量原語
3. Runtime 動態執行

→ PINT（2020）是此願景的具體實現

### 論文中的角色：研究脈絡

---

## ❹ Count-Min Sketch（Cormode & Muthukrishnan, 2005）

**完整引用：** Graham Cormode, S. Muthukrishnan. "An Improved Data Stream Summary: The Count-Min Sketch and its Applications." *Journal of Algorithms*, Vol. 55, No. 1, 2005, pp. 58-75. DOI: S0196677403001913。

### 資料結構

- 二維陣列：**d rows × w columns**
- 參數設定：w = ⌈e/ε⌉，d = ⌈ln(1/δ)⌉
- 查詢：âᵢ = minⱼ count[j][hⱼ(i)]

### 核心特性

- âᵢ ≥ aᵢ（**只有 over-estimation，沒有 under-estimation**）
- Error bound：âᵢ ≤ aᵢ + ε‖f‖₁（機率 1-δ）
- 用 median trick（取 d 列最小值）抑制 over-estimation

### 在 Elastic Sketch 中的角色

CM Sketch = Elastic 的 **Light Part**。但 Elastic 只用 d=1（單列），**無法用 median trick 去偏**。

→ 這是原方向中的關鍵技術挑戰：counter over-estimation bias 直接污染 entropy，debiasing 為核心貢獻

### 論文中的角色：數學基礎

---

## ❺ Splunk Blog: Network Telemetry（2023）

**作者：** Laiba Siddiqui
**URL：** https://www.splunk.com/en_us/blog/learn/network-telemetry.html

### 內容

Telemetry 框架四大模組：
1. Management Plane（SNMP、syslog）
2. Control Plane（路由協定健康狀態）
3. Forwarding Plane（即時轉發資訊）
4. Data Analytics（AI/ML 自動化分析）

### 論文中的角色：產業背景 — entropy-based 攻擊檢測在 Data Analytics 層

---

## ❻ 影音資源

| 影片 | 作者 | 內容 |
|------|------|------|
| INT P4 Video（FOOL5BeHNVY） | P4 Language Consortium | INT 在 P4 交換機上的實作展示 |
| Count-Min Sketch（KRaSkSzwCkE） | Raphael De Lio @ Devoxx UK | CM Sketch 機率式資料結構原理 |

---

# 第二部分：新指定 5 篇（2026-05-31）

---

## ❶ Clifford & Cosma — AISTATS 2013 📐 理論基礎

**完整引用：** Peter Clifford, Ioana Ada Cosma. "A Simple Sketching Algorithm for Entropy Estimation over Streaming Data." *Proceedings of the 16th International Conference on Artificial Intelligence and Statistics (AISTATS)*, JMLR W&CP Vol. 31, 2013, pp. 196-206.

**URL：** https://proceedings.mlr.press/v31/clifford13a.html

### 問題陳述

在高頻率資料串流中近似計算 Shannon 熵。當串流中不重複元素數量極大時，無法在主記憶體中儲存完整的累積向量。需在 strict-turnstile model（允許 deletion，但保證任何時刻累積量 ≥ 0）下以 single-pass 完成。

### 核心方法

#### 1. α-stable Data Sketching

將每個 distinct stream element 映射到 **α-stable distribution** 的隨機變數：

- 使用 maximally skewed stable distribution：F(x; 1, -1, π/2, 0)
- 模擬方法：Zolotarev (1986) 的演算法（見 Table 1）
- 將 stream element 的累積量與對應的 α-stable 變數做加權線性組合
- 獨立複製 k 次 → k-dimensional synopsis（即 data sketch）

#### 2. 與既有方法的關鍵差異

既有方法使用 **α ≈ 1 但 α ≠ 1** 的 stable distribution，從 Rényi entropy H_α 取極限來近似 Shannon entropy —— 這引入額外誤差。

本文核心創新：**直接使用 α = 1 的 stable distribution**，避免了 α 參數校準問題。

#### 3. Log-Mean Estimator Family

提出由 ζ > 0 索引的**漸近無偏 log-mean 估計器族**：

- 計算 k 個獨立 sketch 的實數值
- 使用 log-mean 運算：先取 log 再取平均，最後取 exp
- 推薦 **ζ = 1** 的估計器

### 關鍵結果

| 指標 | 表現 |
|------|------|
| 誤差機率尾界 | **指數衰減**（exponentially decreasing tail bounds） |
| 漸近相對效率 | **0.932** |
| 計算複雜度 | 近最優（near-optimal） |
| 單趟處理 | 只需一次 pass over the data stream |
| 空間需求 | O(k)，k 由所需精度決定 |

#### 理論保證細節

對於 ζ ≤ 1 的估計器 δ̂_lm(ζ)：

```
P(|δ̂_lm(ζ) - δ| ≥ ε) < exp(-kε²/G_R)
```

其中 G_R 由 ζ 決定，當 ε → 0 時 G_R → 2(4ζ-1)/ζ²。

### 與你論文的關聯

這是 sketch-based entropy estimation 的**理論根基**。提供：
1. 嚴格的誤差界限分析 → 可以為你的攻擊檢測方法建立可證明的 detection guarantee
2. Single-pass 演算法 → 對應線速處理需求
3. Strict-turnstile model → 可處理流量變化（含 deletion）

---

## ❷ Yuan et al. — IEEE TKDE 2026 ⚡ 頻率估計原語

**完整引用：** Xinyu Yuan, Yan Qiao, Meng Li, Zhenchun Wei, Cuiying Feng, Zonghui Wang, Wenzhi Chen. "Learning-Based Sketches for Frequency Estimation in Data Streams Without Ground Truth." *IEEE Transactions on Knowledge and Data Engineering (TKDE)*, Vol. 38, No. 6, 2026, pp. 3722-3736.

**arXiv：** https://arxiv.org/abs/2412.03611
**程式碼：** https://github.com/Y-debug-sys/UCL-sketch
**DOI：** https://ieeexplore.ieee.org/abstract/document/11436112

### 問題陳述

既有 learning-based sketch 的三大瓶頸：
1. 依賴真實頻率標籤（ground truth）進行離線訓練 → 標籤往往不可得
2. 更新速度慢 → 不適合即時串流處理
3. 資料分佈變化時需頻繁重新訓練 → 無法適應動態環境

### 核心方法：UCL-sketch

**UCL = Unsupervised Compressive Learning Sketch**

#### 創新 1：等效學習（Equivalent Learning）線上訓練

- **完全不需要 ground truth**：使用 sketch 計數器本身作為自監督訊號
- 線上訓練：模型即時適應串流資料的分佈變化
- 結合壓縮感知（Compressive Sensing）與線上學習

#### 創新 2：邏輯桶（Logical Buckets）架構

- 將大規模串流分割為多個邏輯桶
- 每個桶使用共享參數學習映射關係
- 高度可擴展

#### 技術流程

1. 雜湊函數將資料項映射到計數器陣列
2. 學習解碼器從計數器值恢復每個項目的頻率
3. 等效學習策略：解碼器訓練僅需計數器值，無需真實頻率

### 關鍵結果

| 指標 | 表現 |
|------|------|
| 誤差界限 | 理論證明遠低於先前方法 |
| 極端記憶體限制 | 品質幾乎匹配 omniscient oracle |
| 解碼速度 | 相比既有方程式型 sketch 平均加速 **~500 倍** |
| 適應性 | 無標籤線上學習，自動適應分佈變化 |

### 與你論文的關聯

頻率估計是攻擊偵測最基礎的測量原語。UCL-sketch 的優勢：
- 精確識別 heavy hitter（哪個 IP/flow 異常？）
- 檢測 DDoS 流量突增
- 無標籤線上學習 → 適合動態攻擊場景
- 極低記憶體需求 → 適合交換機部署

---

## ❸ Soto et al. — IEEE Access 2021 🔧 硬體加速

**完整引用：** Javier E. Soto, Paulo Ubisse, Yaime Fernandez, Cecilia Hernandez, Miguel Figueroa. "A High-Throughput Hardware Accelerator for Network Entropy Estimation Using Sketches." *IEEE Access*, Vol. 9, 2021, pp. 85823-85838.

**DOI：** https://ieeexplore.ieee.org/abstract/document/9452148
**引用：** 14 次（OpenAlex 分類：Network Security and Intrusion Detection）

### 問題陳述

經驗熵是檢測 DDoS、埠掃描、蠕蟲傳播的關鍵指標，但高速網路中精確計算熵極困難：
- 軟體實現在高吞吐量下無法滿足即時性
- 資料平面硬體記憶體資源極有限

### 核心方法：FPGA 加速器

在 **Xilinx UltraScale+ ZCU102 FPGA** 上實現三級架構：

#### 1. 基數估計（Cardinality Estimation）
- 使用 sketch 在線計算不重複元素數量

#### 2. Top-K 頻率估計
- 使用 sketch 追蹤頻率最高的前 K 個元素
- 精確計算它們對熵的貢獻

#### 3. 剩餘元素熵估計
- 對於不在 Top-K 中的元素
- 假設服從均勻分佈（uniform distribution）
- 估計它們對總熵的貢獻

> **關鍵洞察**：Top-K 元素對熵貢獻最大且需精確計算，其餘大量低頻元素可用統計假設近似 → 大幅降低計算與記憶體需求

#### 硬體實現
- 全晶片上記憶體（on-chip memory only）
- FPGA 資源使用率 < 50%
- 無需外部 DRAM

### 關鍵結果

| 指標 | 表現 |
|------|------|
| 平均相對誤差 | **< 1.5%** |
| 延遲 | **21 μs** |
| 最小吞吐量 | **204 Gbps** |
| 測試規模 | 1.2 億封包、超過 500 萬條流 |

### 與你論文的關聯

- 證明 sketch entropy 在硬體上以線速部署的可行性
- 21 μs 延遲對即時攻擊回應至關重要
- 低資源使用率意味著可在不影響正常交換功能的同時進行安全監控
- 此文獻直接以攻擊偵測為應用場景

---

## ❹ Wang et al. — ICPP '25 🧠 領域適配思維

**完整引用：** Jin Wang, Chenye Zhu, Jinbin Hu. "A High-Accuracy Sketch for Measuring Low-Entropy Flows in Distributed AI Training." *Proceedings of the 54th International Conference on Parallel Processing (ICPP)*, 2025, pp. 53-62.

**DOI：** https://dl.acm.org/doi/full/10.1145/3754598.3754643

### 問題陳述

分散式 AI 訓練產生的網路流量具有獨特的**低熵特性**：
- 流量模式可預測（predictable）
- 具單一性（singular）和重複性（repetitive）
- 與傳統網路流量的重尾分佈（heavy-tailed）有本質差異

既有 sketch 都是為傳統流量設計，無法利用低熵流量的特徵 → 在測量 AI 訓練流量時準確度顯著下降。

### 核心方法：FP-Sketch

#### 1. 分段佇列（Staging Queue）進行流量預測與分類
- 在 sketch 前端設置 staging queue
- 利用低熵 AI 流量的可預測性
- 根據流量大小對不同大小的流進行預測和分類

#### 2. 分層儲存（Hierarchical Storage）
- 大流和小流使用不同層級的儲存策略
- 最大化記憶體效率
- 提供嚴格的理論誤差界限

### 關鍵結果

| 指標 | 相比現有最佳方法 |
|------|-----------------|
| 流量估計誤差 | 降低 **38.6%** |
| 插入吞吐量 | 提升 **49.4%** |

### 與你論文的關聯

**核心洞察可遷移**：不同流量模式需要不同設計策略的 Sketch。

應用到攻擊偵測：
- DDoS → 大量重複封包 → 類似低熵模式 → 可借鑑 FP-Sketch 的分段預測策略
- Port Scan → 不同 dest port 高頻率 → 需要不同策略
- 慢速攻擊 → 低頻率但持續 → 又需要另一種策略

→ 你的「不同攻擊 → 不同 Sketch 選擇策略」方向直接與此文的設計哲學呼應

---

## ❺ Lai et al. — EuroP4 '22 ⭐ 最直接對應

**完整引用：** Yu-Kuen Lai, Se-Young Yu, Iek-Seng Chan, Bo-Hsun Huang, Che-Hao Chang, Jim Hao Chen, Joe Mambretti. "Sketch-Based Entropy Estimation: A Tabular Interpolation Approach Using P4." *Proceedings of the 5th International Workshop on P4 in Europe (EuroP4 @ CoNEXT)*, 2022, pp. 57-60.

**DOI：** https://dl.acm.org/doi/abs/10.1145/3565475.3569082
**引用：** 7 次（OpenAlex 分類：Anomaly Detection Techniques and Applications）

### 問題陳述

在 P4 交換機上直接計算 Shannon 熵的兩大挑戰：
1. 熵計算涉及複雜對數運算 → P4 match-action pipeline 不支援
2. ASIC 記憶體與計算資源極有限

### 核心方法：表格插值法

#### 1. 隨機投影 → 查表轉換
- 將 sketch 中的 random projection 預先計算
- 存入 match-action pipeline 的查找表（lookup table）
- Runtime 僅需快速查表，無需複雜數學運算

#### 2. 插值啟發式（Interpolation Heuristic）
- 使用插值技術大幅縮減預計算表大小
- 使表可放入有限的 ASIC 記憶體
- 更多表空間 → 容納更多 sketch 陣列 → 提高估計精度

#### 3. 真實部署驗證
- 部署平台：**Barefoot Tofino2** 可編程交換機
- 連接至 iCAIR（International Center for Advanced Internet Research）國家級測試平台
- 使用真實網路流量進行驗證

### 關鍵結果

| 指標 | 表現 |
|------|------|
| 吞吐量 | **400 Gbps** 線速 |
| 準確度 | 真實流量 trace 驗證 |
| 部署平台 | Barefoot Tofino2 P4 可編程 ASIC |

### 與你論文的關聯（最強）

這篇是**最直接對應你新方向的論文**：
1. P4 交換機 + sketch entropy + 線速 = 你的系統架構藍圖
2. 攻擊偵測邏輯可直接嵌入資料平面，無需鏡像流量到控制器
3. 展示 P4 限制下的創新方法（查表替代計算）
4. 400 Gbps 意味著可在核心網路邊緣做第一線分散式探測器

---

# 第三部分：綜合對照

## 技術層次對應

```
理論層 ──── ❶ Clifford (entropy 估計數學保證)
                │
演算法層 ── ❷ UCL-sketch (頻率估計新原語)
                │     ❹ FP-Sketch (領域適配設計思維)
                │
硬體層 ──── ❸ Soto FPGA (硬體加速可行性)
                │     ❺ Lai P4 (P4 ASIC 部署實作)
                │
應用層 ──── Elastic Sketch (baseline 工具)
                PINT (motivation 來源)
```

## 論文方向演進

| | 原方向（5/1） | 新方向（5/31） |
|---|--------------|--------------|
| 核心問題 | 優化 entropy 估計精度 | 用通用 Sketch 區分不同攻擊 |
| 技術手段 | 改良 Elastic Sketch entropy 模組 | 調查/組合多種 Sketch 做特徵提取 |
| 理論基礎 | CM Sketch 數學 | ❶ Clifford 嚴格理論保證 |
| 核心原語 | Elastic (單一) | ❷ UCL-sketch + CM + Elastic |
| 部署目標 | Mininet 模擬 | ❺ P4 Tofino2 + ❸ FPGA |
| 設計思維 | 通用 entropy | ❹ 領域特化（攻擊類型 → Sketch 策略） |

## 對你的論文最有用的 3 篇（優先讀）

1. **❺ Lai et al. (EuroP4 '22)** — 你的系統架構直接參考
2. **❶ Clifford & Cosma (AISTATS '13)** — 你的方法論數學基礎
3. **❷ Yuan et al. (TKDE '26)** — 你的核心測量原語

---

> 整理日期：2026-07-06
> 資料來源：論文原文（Clifford 全文已下載分析）、arXiv、web search
