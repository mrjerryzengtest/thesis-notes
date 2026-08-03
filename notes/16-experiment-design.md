# 第四章 實驗設計（規劃版）

## 4.1 研究問題（Revised）

| RQ | 問題 | 對應指標 |
|-----|------|---------|
| RQ1 | 多 Sketch 協同框架在攻擊**檢測**上是否優於單一 Sketch？ | Detection Rate, FPR |
| RQ2 | 聯合特徵向量能否有效**分類**不同攻擊類型？ | Classification Accuracy, Per-class F1 |
| RQ3 | 每個 Sketch 對不同攻擊類型的**貢獻**有多大？（消融實驗） | Δ Detection Rate（移除該 Sketch 後的降幅） |
| RQ4 | 框架的資源消耗是否在實際部署可接受範圍內？ | Memory, CPU, Detection Latency |

---

## 4.2 實驗環境

| 項目 | 規格 |
|------|------|
| 模擬器 | Mininet 2.3.0 |
| SDN 控制器 | Ryu 4.34 |
| 交換機 | Open vSwitch 2.17（OpenFlow 1.3） |
| 流量產生 | iPerf3（正常流量）、Scapy（攻擊流量） |
| 背景流量 | CAIDA Anonymized Internet Traces（或 D-ITG 合成流量） |
| 主機 | Ubuntu 22.04 LTS, 8GB RAM, 4-core CPU |
| 拓撲 | 3 switches × 15 hosts（fat-tree） |
| 正常流量速率 | 100-500 Mbps |
| 實驗總時長 | 300 秒（150s normal + 150s mixed） |

---

## 4.3 攻擊場景

四種攻擊類型 × 四種強度 = 16 個場景，每個場景重複 10 次。

| 攻擊 | 工具 | 強度等級 |
|------|------|:--:|
| DDoS（偽造來源 IP） | Scapy TCP SYN flood, 隨機 srcIP | L / M / H / E |
| DDoS（殭屍網路） | hping3 UDP flood, 10-20 固定來源 | L / M / H / E |
| Port Scan | Scapy / nmap, 1-2 來源掃 1000 埠 | L / M / H / E |
| Flash Crowd | ApacheBench, 50+ HTTP 並發 | L / M / H / E |

強度定義（相對於正常流量基準）：

| 等級 | 攻擊封包速率 | 攻擊 flow 佔比 |
|:----:|:----------:|:------------:|
| L | 1.2× | 15% |
| M | 2× | 30% |
| H | 5× | 50% |
| E | 10× | 70% |

---

## 4.4 對照組（Baseline Methods）

對照組設計的核心原則是：**逐一比較單一 Sketch 與多 Sketch 組合的效果差異**。

### 4.4.1 單一 Sketch Baseline（RQ1）

| Baseline | 使用 Sketch | 特徵維度 |
|----------|-----------|:--:|
| **CM-only** | CM Sketch 頻率統計 + 簡單閾值檢測 | 3 維（$F_{freq}$） |
| **Elastic-only** | Elastic Sketch 六維熵值 + heavy hitter 檢測 | 8 維（$F_{entropy} + F_{hh}$） |
| **UnivMon-only** | UnivMon 通用查詢結果 | 2 維（$F_{univ}$） |
| **UCL-only** | UCL-Sketch 頻率估計 | 3 維（$F_{ucl}$） |

### 4.4.2 多 Sketch 組合（RQ1, RQ2）

| 組合 | 包含 Sketch | 特徵維度 |
|------|-----------|:--:|
| **2-Sketch** | CM + Elastic | 11 維 |
| **3-Sketch** | CM + Elastic + UnivMon | 13 維 |
| **Full (4-Sketch)** | CM + Elastic + UnivMon + UCL | 17 維 |

### 4.4.3 消融實驗（RQ3）

從 Full 組合中逐一移除一個 Sketch，觀察各攻擊類型的檢測率變化：

| 消融組合 | 移除的 Sketch | 保留 |
|----------|:----------:|------|
| Full − CM | CM Sketch | Elastic + UnivMon + UCL |
| Full − Elastic | Elastic Sketch | CM + UnivMon + UCL |
| Full − UnivMon | UnivMon | CM + Elastic + UCL |
| Full − UCL | UCL-Sketch | CM + Elastic + UnivMon |

### 4.4.4 外部 Baseline

| 方法 | 說明 |
|------|------|
| **PINT-style** | 機率取樣 + 單一維度熵值（非 Sketch，用於驗證全流量 vs 取樣差異） |
| **Decision Tree on Full** | 使用完整的 17 維聯合向量訓練決策樹，與規則匹配比較（RQ2） |

---

## 4.5 評估指標

### 檢測指標

| 指標 | 定義 |
|------|------|
| Detection Rate (Recall) | TP / (TP + FN) |
| False Positive Rate | FP / (FP + TN) |
| Precision | TP / (TP + FP) |
| F1-score | 2PR / (P + R) |

### 分類指標

| 指標 | 定義 |
|------|------|
| Classification Accuracy | 所有分類正確的比例 |
| Per-class F1-score | 每種攻擊類型的 F1 |
| Confusion Matrix | DDoS(spoofed) vs DDoS(botnet) vs Port Scan vs Flash Crowd |

### 效能指標

| 指標 | 定義 |
|------|------|
| Detection Latency | 攻擊開始 → 警報觸發的時間 |
| Total Memory | 所有 Sketch 的記憶體合計 |
| Per-window CPU | 每時間窗口的特徵計算時間 |

---

## 4.6 參數調校

| 參數 | 範圍 | 預設值 |
|------|------|:--:|
| 時間窗口 $T$ | 1s, 3s, 5s, 10s, 30s | 5s |
| EWMA 平滑 $\alpha$ | 0.1, 0.2, 0.3, 0.5 | 0.3 |
| 檢測閾值百分位 | 95%, 99%, 99.9% | 99% |
| 特徵權重 $w_i$ | uniform / 消融導出 | uniform |

---

## 4.7 實驗步驟與時程

| Phase | 內容 | 預估時間 |
|:-----:|------|:-------:|
| 1 | 搭建 Mininet + Ryu 環境 | 1 週 |
| 2 | 實作四個 Sketch 原型（CM, Elastic, UnivMon, UCL） | 2 週 |
| 3 | 實作聯合特徵擷取 + 分類器 | 1 週 |
| 4 | 正常流量 profiling + 參數調校 | 1 週 |
| 5 | 完整實驗（16 場景 × 10 重複 × 9 組合 = 1440 次） | 2 週 |
| 6 | 消融實驗 | 1 週 |
| 7 | 結果分析 + 圖表 | 1 週 |

**總預估**：9 週

---

## 4.8 預期結果

**H1（RQ1）**：Full (4-Sketch) 組合在低強度攻擊（L 場景）下檢測率預期優於所有單一 Sketch baseline。原因：輕微攻擊可能在單一量測維度上不明顯，但跨多個互補維度的微小偏移累積後可被聯合向量捕捉。

**H2（RQ2）**：Full 組合的分類準確率在 M 強度以上預期 > 85%。最難區分的組合是 DDoS（botnet）vs Flash Crowd——兩者在 heavy hitter 相關特徵上表現相似，但來源 IP 熵值（$F_{entropy}$: srcIP）預期能提供關鍵區分信號。

**H3（RQ3）**：預期 Elastic Sketch 的貢獻最大（因其提供的結構性資訊對 Port Scan 的區分最關鍵），其次是 CM Sketch（基礎頻率資訊覆蓋所有攻擊類型）。UCL-Sketch 在 L 強度下的貢獻預期較明顯（其高精度頻率估計能捕捉微小的頻率偏移）。

**H4（RQ4）**：總記憶體 < 2 MB，Per-window CPU < 10ms，Detection Latency < 2s（包含 flow stats polling 延遲）。資源消耗不是瓶頸——真正的瓶頸是分類邏輯的設計，而不是硬體限制。
