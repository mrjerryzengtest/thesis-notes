# 02 — 論文方向分析與題目擬定

> 你的研究方向：利用「熵值（Entropy）」判斷 SDN 網路是否遭受攻擊，針對「熵值」部分做優化，以更精確判斷攻擊

---

## 一、研究定位

```
SDN 攻擊檢測
    │
    ├── 方法一：Rule-based / Signature-based（傳統 IDS）
    ├── 方法二：ML-based（機器學習分類）
    └── 方法三：Entropy-based（熵值分析）  ← 你的方向
            │
            ├── 現有問題：熵值估計不夠精確（尤其高流量/攻擊情境下）
            └── 你的貢獻：優化熵值估計方法
```

---

## 二、研究缺口（Research Gap）

### 2.1 Elastic Sketch 熵值估計的可優化環節

```
封包流 → Heavy Part (elephant) + Light Part (CM Sketch, mouse)
                    │                        │
                    ▼                        ▼
            直接獲取 flow info    收集 counter distribution
                                        (n₀, n₁, ..., n₂₅₅)
                    │                        │
                    └───────────┬────────────┘
                                ▼
                   H = -Σ (i·nᵢ/m · log(nᵢ/m))
```

可優化點：
1. **CM Sketch over-estimation bias** 扭曲 counter distribution → 熵值偏差
2. **固定 λ=8** 在攻擊情境下未必最優
3. **Maximum Compression** 對 entropy 的影響未被充分研究
4. Elastic 是**通用框架**，未針對攻擊檢測做特別調校

### 2.2 PINT 揭示的根本張力

> "Entropy is poorly approximable from subsampled streams." — PINT §4.1

研究機會：設計在低 overhead 條件下仍保持高精度的 entropy 估計方法

### 2.3 攻擊情境下的特殊挑戰

| 攻擊類型 | 流量特徵 | 對 entropy 的影響 |
|----------|---------|------------------|
| DDoS | 封包速率暴增 | CM Sketch counter overflow → 失準 |
| Port Scan | 大量新 flow | Light part 過載 → 難以區分攻擊類型 |
| Low-rate Attack | 熵值變化微小 | 需要更靈敏的檢測 |

---

## 三、論文題目方案

### 方案 A：直接優化 Elastic Sketch 的熵值估計

> 基於改良式 Elastic Sketch 之 SDN 網路攻擊熵值檢測方法
> *An Enhanced Elastic Sketch-based Entropy Estimation Method for SDN Attack Detection*

- 針對 Elastic entropy estimation 模組改良
- 改進 counter distribution 估計精度
- Attack-aware 自適應 λ 閾值
- 優點：baseline 明確，改進點清晰

### 方案 B：Sketch + Telemetry 混合方法

> 結合機率式網路遙測與 Sketch 之 SDN 攻擊熵值檢測優化
> *Optimizing Entropy-based SDN Attack Detection via Probabilistic Network Telemetry and Sketch Hybridization*

- PINT-style low overhead telemetry + Elastic Sketch
- 解決 "subsampling 損害 entropy" 問題
- 優點：創新性高（結合兩個 SIGCOMM 概念）
- 挑戰：需要設計混合架構

### 🏆 方案 C（選定）：多維度熵值分析

> **SDN 環境下基於多維度熵值分析之網路攻擊精確檢測與分類**
> *Multi-Dimensional Entropy Analysis for Accurate Network Attack Detection and Classification in SDN*

#### 核心構想

不依賴單一熵值，而是利用 Elastic Sketch 同時支援多任務的特性，計算**多維度熵值向量**：

| 維度 | 說明 | 對應攻擊 |
|------|------|---------|
| H_srcIP | Source IP entropy | DDoS（多 source → 高熵） |
| H_dstIP | Destination IP entropy | Port Scan（多 destination → 高熵） |
| H_srcPort | Source Port entropy | 特定攻擊模式 |
| H_dstPort | Destination Port entropy | Port Scan |
| H_flowSize | Flow Size entropy | 流量分佈異常 |
| H_proto | Protocol entropy | 異常協定使用 |

#### 攻擊分類邏輯

```
                     H_srcIP  H_dstIP  H_dstPort  H_flowSize
DDoS (spoofed)        HIGH     LOW      LOW        MEDIUM
DDoS (botnet)         MEDIUM   LOW      LOW        HIGH
Port Scan             LOW      HIGH     HIGH       LOW
Flash Crowd           MEDIUM   HIGH     MEDIUM     HIGH
Normal                MEDIUM   MEDIUM   MEDIUM     MEDIUM
```

#### 優點
- 與 Elastic Sketch「一個結構支援多任務」的設計完美契合
- 可區分攻擊類型（不只檢測有無攻擊，還能分類）
- 多維度特徵適合後續 ML 分類器

### 方案 D：熵值計算的去偏優化

> 基於計數器分佈修正之 SDN 網路流量熵值精確估計方法
> *Accurate Entropy Estimation via Counter Distribution Correction in Count-Min Sketch*

- 聚焦 CM Sketch over-estimation bias
- Debiasing counter distribution
- 理論 error bound 分析

---

## 四、建議研究步驟（方案 C）

### Phase 1：文獻深入（1-2 個月）
- 精讀 Elastic Sketch technical report [47]
- 研讀 PINT 引用 [20]：Ding et al., P4 Entropy Tracking（NOMS 2020）
- 搜尋 entropy-based DDoS detection in SDN 相關論文
- CM Sketch error bound 對 entropy 的理論影響

### Phase 2：方法設計（2-3 個月）
- 分析 Elastic entropy estimation 誤差來源
- 設計多維度熵值融合策略
- 推導多維度熵值向量的 error bound
- 設計攻擊分類決策邏輯

### Phase 3：實作與實驗（3-4 個月）
- Mininet + ONOS/Ryu SDN 環境
- 復現 Elastic Sketch entropy baseline
- 實作多維度熵值方法
- CAIDA trace + 攻擊流量產生
- 評估指標：RE、檢測延遲、precision/recall、F1 score

### Phase 4：論文撰寫（2-3 個月）

---

## 五、可能投稿目標

- IEEE/IFIP NOMS/IM（網路管理）
- ACM CoNEXT（學生論文）
- IEEE TNSM（Transactions on Network and Service Management）
- Computer Networks（Elsevier）
