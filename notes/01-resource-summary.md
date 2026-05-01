# 01 — 參考資料詳細整理

> 整理日期：2026-05-01

---

## 一、核心論文

### 1.1 Elastic Sketch: Adaptive and Fast Network-wide Measurements

- **出處**：ACM SIGCOMM 2018
- **作者**：Tong Yang, Jie Jiang, Peng Liu, Qun Huang, Junzhi Gong, Yang Zhou, Rui Miao, Xiaoming Li, Steve Uhlig（北京大學、中科院、阿里巴巴、QMUL）
- **引用數**：539+
- **DOI**：10.1145/3230543.3230544

#### 核心貢獻

當網路發生壅塞、掃描攻擊、DDoS 攻擊時，流量特徵（可用頻寬、封包速率、flow 大小分佈）會劇烈變化——此時測量反而最重要，但傳統方法在此時效能大幅下降。Elastic Sketch 是第一種**自適應的網路測量 sketch**。

#### 資料結構：Heavy Part + Light Part

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

**Ostracism 分離機制**（靈感：古希臘陶片放逐制）：
- 每個 bucket 儲存 (flow ID, vote⁺, flag, vote⁻)
- 當 incoming packet 與 bucket 中的 flow 不同 → vote⁻++
- 當 vote⁻ / vote⁺ ≥ λ（λ=8）→ 驅逐原 flow，新 flow 當選
- 被驅逐的 flow → 進入 Light Part（CM Sketch）

#### 三種自適應能力

1. **頻寬自適應**：Maximum Compression（MC）與 Sum Compression（SC）壓縮 sketch
2. **封包速率自適應**：高速率時僅 1 次記憶體存取（只訪問 Heavy Part）
3. **Flow 大小分佈自適應**：動態倍增 Heavy Part（copy-and-double）

#### 支援的六種測量任務

| # | 任務 | 說明 |
|---|------|------|
| 1 | Flow Size Estimation | 估測任意 flow 的大小 |
| 2 | Heavy Hitter Detection | 偵測超過閾值的大流 |
| 3 | Heavy Change Detection | 偵測相鄰時間窗的流量劇變 |
| 4 | Flow Size Distribution | 估測 flow 大小分佈 |
| 5 | **Entropy Estimation** | **估測 flow 大小的熵值** |
| 6 | Cardinality Estimation | 估測不重複 flow 數量 |

#### 熵值估計方法

```
從 Light Part 收集 counter distribution array (n₀, n₁, ..., n₂₅₅)

H = -Σ (i × nᵢ / m × log(nᵢ / m))

其中 m = Σ nᵢ（所有 counter 值的總和）
```

- Heavy part 的 flow 資訊直接獲取
- Light part 使用 MRAC 演算法估計 flow size distribution
- 結合兩者計算整體熵值

#### 實驗結果（熵值部分關鍵數據）

- memory ≥ 400KB 時，準確度超越所有 state-of-the-art 演算法
- 比較對象：UnivMon、Sieving
- 評估指標：相對誤差（RE）
- Elastic 只用**一個資料結構（600KB）**處理全部六種任務

#### 平台實作

P4、FPGA、GPU、CPU、multi-core CPU、OVS 共六種平台

---

### 1.2 PINT: Probabilistic In-band Network Telemetry

- **出處**：ACM SIGCOMM 2020
- **作者**：Ran Ben Basat, Sivaramakrishnan Ramanathan, Yuliang Li, Gianni Antichi, Minlan Yu, Michael Mitzenmacher（Harvard、USC、QMUL）
- **引用數**：248+
- **DOI**：10.1145/3387514.3405894

#### 核心問題

INT（In-band Network Telemetry）允許交換機在每個封包上附加 telemetry 資訊，但 **overhead 過大**：

- 5 跳路徑 + 2 個值/跳 = 48 bytes/packet overhead
- → Flow completion time **增加 25%**
- → Goodput **下降 20%**

#### PINT 方案：概率式 In-band Network Telemetry

- 將 telemetry 資訊概率性地編碼到**多個封包**上
- Per-packet overhead 可低至 **1 bit**
- 用 **16 bits/packet** 即可同時支援 congestion control、path tracing、latency estimation

#### 三種聚合模式

| 模式 | 說明 | 應用 |
|------|------|------|
| Per-packet | 路徑上各跳的值做聚合（max/min/sum） | HPCC 擁塞控制 |
| Static per-flow | Flow 路徑上固定的值 | Path Tracing |
| Dynamic per-flow | 每個 (flow, switch) 的統計分佈 | Latency 分位數 |

#### ⚠️ 關鍵洞察：Entropy 與 Subsampling 的張力

PINT §4.1 明確指出：

> **"Aggregation functions like the number of distinct values or the value-frequency distribution entropy are poorly approximable from subsampled streams."**

→ 這直接說明：概率性子採樣會損害熵值估計的準確性

→ 這同時也是**你的研究 motivation**：如何在低 overhead telemetry 下仍能準確估計熵值

#### Entropy 相關引用

PINT 引用文獻 [20]：Ding et al., *"Estimating Logarithmic and Exponential Functions to Track Network Traffic Entropy in P4"*（IEEE/IFIP NOMS 2020）

---

## 二、綜述與基礎文獻

### 2.1 Network Telemetry: Towards a Top-Down Approach

- **作者**：Minlan Yu（Harvard，PINT 共同作者）
- **出處**：CCRV 2019
- **DOI**：10.1145/3314212.3314215

#### 核心論點

反對傳統 Bottom-up 方法，主張 **Top-down**：
1. 高層宣告式抽象（讓運營商宣告測量查詢）
2. 可程式化測量原語（在交換機和主機上）
3. Runtime 轉譯（高層查詢 → 低層 API）

Minlan Yu 的研究脈絡：Top-Down Telemetry 願景 → PINT 具體實現

### 2.2 Splunk Blog: What is Network Telemetry?

- **作者**：Laiba Siddiqui（2023）
- **URL**：https://www.splunk.com/en_us/blog/learn/network-telemetry.html

Telemetry 框架四大模組：
1. Management Plane（SNMP、syslog）
2. Control Plane（路由協定健康狀態）
3. Forwarding Plane（即時轉發資訊）
4. Data Analytics（AI/ML 自動化分析）

### 2.3 Count-Min Sketch

- **原始論文**：Cormode & Muthukrishnan（Journal of Algorithms, 2005）
- **DOI**：S0196677403001913

#### 資料結構
- 二維陣列：d rows × w columns
- 參數：w = ⌈e/ε⌉, d = ⌈ln(1/δ)⌉
- 查詢：âᵢ = minⱼ count[j][hⱼ(i)]（只有 over-estimation，無 under-estimation）

#### 在 Elastic Sketch 中的角色
- CM Sketch = Elastic 的 **Light Part**
- Elastic 改進：d=1、8-bit counters、Maximum Compression

---

## 三、影音資源

| 影片 | 作者 | 內容 |
|------|------|------|
| INT p4 video（FOOL5BeHNVY） | P4 Language Consortium | INT 在 P4 交換機上的實作展示 |
| Count-Min Sketch（KRaSkSzwCkE） | Raphael De Lio @ Devoxx UK | CM Sketch 的機率式資料結構原理 |

---

## 四、資源關係圖

```
Count-Min Sketch（2005，基礎資料結構）
    │
    ├──→ Elastic Sketch（SIGCOMM 2018）
    │       └── Light Part 使用 CM Sketch
    │       └── 支援 Entropy Estimation
    │       └── 自適應頻寬/速率/分佈變化
    │
    └──→ PINT（SIGCOMM 2020）
            └── 指出 subsampling 損害 entropy
            └── 引用 P4 Entropy Tracking 工作

Minlan Yu 研究脈絡：
    Top-Down Telemetry（2019）
        └──→ PINT（2020）

Splunk Blog — 產業視角與框架
```
