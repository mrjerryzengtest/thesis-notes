# 10 — Meeting 準備資料（2026-07-06）

> 距上次 Meeting（2026-05-31）約 5 週，準備與教授討論論文進度

---

## 一、教授指定文獻總覽（11 篇）

### 第一批｜原始 6 項（2026-05-01）

#### 核心論文

**❶ Elastic Sketch** — SIGCOMM 2018（Tong Yang 等，539+ citations）
- 自適應網路測量 sketch：Heavy Part（hash table）+ Light Part（CM Sketch）
- 600KB 單一結構同時支援 6 種任務，含 entropy estimation
- 作者 §6.1 承認 DDoS 檢測為 future work
- 🎯 **Baseline 工具** — 拿它的量測能力，做它沒做的事（攻擊檢測 + 分類）

**❷ PINT** — SIGCOMM 2020（Ran Ben Basat 等，248+ citations）
- 概率式 In-band Network Telemetry，per-packet overhead 低至 1 bit
- §4.1：*"Entropy is poorly approximable from subsampled streams"*
- 🎯 **Motivation** — 用 sketch-based（非 sampling）方法正面解決

#### 綜述與基礎

**❸ Minlan Yu — Top-Down Telemetry**（CCRV 2019）
- 主張從營運商需求出發，宣告式轉譯成交換機測量原語
- 🎯 **研究脈絡** — 你的工作在此框架中

**❹ Count-Min Sketch**（Cormode & Muthukrishnan, 2005）
- d rows × w columns，âᵢ ≥ aᵢ（零 under-estimation）
- Elastic 的 Light Part（但 d=1，無法用 median trick 去偏）
- 🎯 **數學基礎** — counter bias 污染 entropy，debiasing 為核心挑戰

**❺ Splunk Blog: Network Telemetry**（2023）
- Telemetry 四大模組：Management / Control / Forwarding / Data Analytics
- 🎯 **產業定位** — entropy-based 檢測在 Data Analytics 層

**❻ INT P4 影片 + CM Sketch 原理影片**
- 🎯 視覺理解素材

---

### 第二批｜新指定 5 篇（2026-05-31 Meeting 後）

**❶ Clifford & Cosma — AISTATS 2013** 📐 理論基礎
- *A Simple Sketching Algorithm for Entropy Estimation over Streaming Data*
- α-stable sketch + compressed counting → Shannon entropy
- ζ=1 估計器：指數衰減誤差尾界，漸近相對效率 0.932
- 🔗 https://proceedings.mlr.press/v31/clifford13a.html
- 🎯 Sketch entropy 的**嚴格理論保證** — 論文方法論根基

**❷ Yuan et al. — IEEE TKDE 2026** ⚡ 核心原語
- *Learning-Based Sketches for Frequency Estimation Without Ground Truth*
- UCL-sketch：壓縮感知 + 邏輯結構化估計桶
- 解碼速度比既有方程式型 Sketch 快 **~500 倍**
- 🔗 https://ieeexplore.ieee.org/abstract/document/11436112
- 🎯 頻率估計是攻擊偵測最基礎的測量原語

**❸ Soto et al. — IEEE Access 2021** 🔧 硬體加速
- *A High-Throughput Hardware Accelerator for Network Entropy Estimation Using Sketches*
- Xilinx UltraScale+ FPGA：**204 Gbps**、21 μs 延遲、誤差 < **1.5%**
- 🔗 https://ieeexplore.ieee.org/abstract/document/9452148
- 🎯 證明 sketch entropy 線速部署可行性

**❹ Wang et al. — ICPP '25** 🧠 領域適配思維
- *FP-Sketch for Measuring Low-Entropy Flows in Distributed AI Training*
- 針對低熵流量設計，flow 誤差降 **38.6%**，插入吞吐量 +**49.4%**
- 🔗 https://dl.acm.org/doi/full/10.1145/3754598.3754643
- 🎯 **核心洞察可遷移**：不同流量模式 → 不同 Sketch 策略 → 攻擊類型同理

**❺ Lai et al. — EuroP4 '22** ⭐ 最關鍵
- *Sketch-Based Entropy Estimation: A Tabular Interpolation Approach Using P4*
- Barefoot **Tofino2** ASIC 實作，查表插值法 → match-action pipeline
- **400 Gbps** 線速 entropy 估計
- 🔗 https://dl.acm.org/doi/abs/10.1145/3565475.3569082
- 🎯 **最直接對應新方向** — 你的系統架構藍圖

---

### 兩批文獻角色對照

```
第一批（基礎）             第二批（深化）
─────────────────────    ─────────────────────
Elastic Sketch ───────→  ❺ P4 Tofino 部署架構
PINT (motivation)         ❸ FPGA 硬體加速（替代方案）
CM Sketch (數學) ──────→  ❶ Entropy 理論保證
                           ❷ UCL-sketch（新原語）
                           ❹ 領域適配思維
```

---

## 二、論文方向演進

| | 原始方向（5/1） | 調整後方向（5/31） |
|---|--------------|-----------------|
| **核心問題** | 優化 entropy 估計精度 | 用通用 Sketch 區分不同攻擊 |
| **技術手段** | 改良 Elastic Sketch entropy 模組 | 調查/組合多種 Sketch 做特徵提取 |
| **貢獻** | Debiasing + 多維度熵值向量 | Sketch-based 攻擊識別框架 + SDN 部署 |
| **Baseline** | Elastic Sketch | CM Sketch、Elastic、UnivMon、UCL-Sketch |
| **部署** | Mininet 模擬 | P4 交換機（BMv2 / Tofino） |

### 論文題目（暫定）

> **基於通用網路測量 Sketch 之 SDN 攻擊偵測與分類**
> *Network-Wide Measurement Sketch-Based Attack Detection and Classification in SDN*

---

## 三、新方向邏輯架構

```
                    攻擊流量
                       │
                       ▼
             ┌─────────────────┐
             │  SDN 可程式化    │
             │  資料平面 (P4)   │  ← 文獻 ❺
             └────────┬────────┘
                       │
         ┌─────────────┼─────────────┐
         ▼             ▼             ▼
   ┌──────────┐ ┌──────────┐ ┌──────────┐
   │ CM Sketch│ │ Elastic  │ │ UCL-     │  ← 文獻 ❶❷
   │ (頻率)   │ │ Sketch   │ │ Sketch   │
   └────┬─────┘ └────┬─────┘ └────┬─────┘
        │            │            │
        └────────────┼────────────┘
                     ▼
             ┌─────────────────┐
             │  特徵向量提取    │
             │  (entropy,      │
             │   frequency,    │  ← 文獻 ❹（分類思維）
             │   cardinality)  │
             └────────┬────────┘
                       ▼
             ┌─────────────────┐
             │  攻擊識別/分類   │
             │  DDoS / Scan /  │
             │  Botnet / ...   │
             └─────────────────┘
```

---

## 四、今日討論議題

| # | 議題 | 核心問題 |
|---|------|---------|
| ❶ | 方向確認 | 5/31 調整的「通用 Sketch → 攻擊識別」理解正確嗎？ |
| ❷ | 文獻進度 | 回報 5 篇新文獻閱讀狀況 |
| ❸ | 實驗平台 | P4 BMv2 vs. Mininet + OVS？有無 Tofino 硬體資源？ |
| ❹ | 時間線 | 文獻回顧完成時間？何時開始實驗？ |
| ❺ | 投稿目標 | NOMS/IM？TNSM？APNet？ |

---

## 五、待辦事項（上次 Meeting 遺留）

1. ✅ 與教授線上討論（5/31 完成）
2. 📖 研讀教授新指定的 5 篇文獻
3. 🔍 調查更多通用 Sketch（UnivMon、NitroSketch、SketchLearn）
4. 🖥️ 設計「不同攻擊 → 不同 Sketch 選擇策略」方法框架
5. 🧪 確認實驗平台

---

> 整理日期：2026-07-06
