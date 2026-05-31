# 09 — 教授 Meeting 結論與方向調整（2026-05-31）

> 本次 meeting 結果：研究方向調整 + 5 篇新指定文獻

---

## 一、教授的核心回饋

### 方向調整

| 原方向 | 新建議 |
|--------|--------|
| 優化 Elastic Sketch 的熵值估計 | 使用**通用網路測量 Sketch** 來識別不同攻擊 |
| 聚焦「如何讓熵值更準」 | 聚焦「如何用 Sketch 區分不同攻擊」 |

### 教授的邏輯

> 「針對熵值其實也沒什麼好優化的。應該是找一些通用的網路測量 sketch，來去針對不同的攻擊做識別，這樣會是比較好的切入點。」

→ **重點從「改良測量工具」轉向「用測量工具做診斷」**

→ 不需要自己發明新的 sketch 或改良 entropy formula，而是：
  1. 調查現有的網路測量 sketch（Elastic Sketch、UnivMon、CM Sketch 等）
  2. 針對不同攻擊類型的流量特徵，選擇/組合合適的 sketch
  3. 在 SDN 環境中部署並評估攻擊識別能力
  4. 必要時針對攻擊情境做輕量適配

---

## 二、新指定文獻

### ❶ 理論基礎：Sketch 熵值估計的數學原理

- **標題**：*A Simple Sketching Algorithm for Entropy Estimation over Streaming Data*
- **作者**：Peter Clifford, Ioana Cosma
- **出處**：AISTATS 2013（PMLR Vol. 31, pp. 196–206）
- **URL**：https://proceedings.mlr.press/v31/clifford13a.html

**核心貢獻**：單趟 Sketch 演算法近似高速資料流的 Shannon 熵。透過 α-stable 資料 Sketch 與 compressed counting，從 Rényi 熵取極限 α→1 推導出 Shannon 熵的漸近無偏 log-mean 估計器。推薦的 ζ=1 估計器具有**指數衰減的誤差機率尾界**，漸近相對效率達 0.932。

**與新方向的關聯**：提供 Sketch 估算熵值的嚴格理論保證——這是建立可證明正確性的攻擊偵測方法的理論基礎。

---

### ❷ 核心測量原語：學習型頻率估計 Sketch

- **標題**：*Learning-Based Sketches for Frequency Estimation in Data Streams Without Ground Truth*
- **作者**：Xinyu Yuan, Yan Qiao, Meng Li 等
- **出處**：IEEE TKDE, Vol. 38, No. 6, pp. 3722–3736（2026 年 6 月出刊）
- **URL**：https://ieeexplore.ieee.org/abstract/document/11436112

**核心貢獻**：提出 **UCL-sketch**——基於線上等效學習的頻率估計 Sketch，完全不需要 ground truth。利用壓縮感知技術 + 邏輯結構化估計桶，在極小記憶體預算下接近 omniscient oracle 精度，解碼速度比既有方程式型 Sketch 快近 **500 倍**。

**與新方向的關聯**：頻率估計是最基礎的通用測量原語。精確識別哪些 flow 出現異常頻率變化（如 DDoS 放大流量）是攻擊偵測的基礎。

---

### ❸ 硬體加速：FPGA 線速熵估計

- **標題**：*A High-Throughput Hardware Accelerator for Network Entropy Estimation Using Sketches*
- **作者**：Javier E. Soto, Paulo Ubisse 等
- **出處**：IEEE Access, Vol. 9, pp. 85823–85838（2021）
- **URL**：https://ieeexplore.ieee.org/abstract/document/9452148

**核心貢獻**：在 Xilinx UltraScale+ ZCU102 FPGA 上實作 Sketch 熵估計加速器，僅用 50% 以內晶片資源實現 **204 Gbps** 吞吐量、21 μs 延遲、平均相對誤差低於 **1.5%**。在 1.2 億封包真實流量上完成驗證。

**與新方向的關聯**：展示 Sketch 熵估計在可程式化硬體上以線速部署的可行性，是 SDN 資料平面攻擊偵測的硬體實作參考。

---

### ❹ 領域特化設計：針對低熵流量的 Sketch

- **標題**：*A High-Accuracy Sketch for Measuring Low-Entropy Flows in Distributed AI Training*
- **作者**：Jin Wang, Chenye Zhu, Jinbin Hu
- **出處**：ICPP '25, pp. 53–62, ACM
- **URL**：https://dl.acm.org/doi/full/10.1145/3754598.3754643

**核心貢獻**：提出 **FP-Sketch**——針對分散式 AI 訓練的**低熵流量特徵**設計的高精度 Sketch。利用暫存佇列預測與分類不同大小的 flow，結合分層儲存結構。相比現有最佳方法，flow 估計誤差降低 **38.6%**，插入吞吐量提升 **49.4%**。

**與新方向的關聯**：核心洞察——**不同流量模式需要不同設計策略的 Sketch**。可遷移到攻擊偵測：不同攻擊（DDoS、Port Scan、慢速攻擊）產生不同特徵的流量模式，應針對性地選擇/設計合適的 Sketch 來區分。

---

### ❺ SDN 資料平面部署：P4 交換機上的 Sketch 熵估計

- **標題**：*Sketch-Based Entropy Estimation: A Tabular Interpolation Approach Using P4*
- **作者**：Yu-Kuen Lai, Se-Young Yu, Iek-Seng Chan 等
- **出處**：EuroP4@CoNEXT '22, pp. 57–60, ACM
- **URL**：https://dl.acm.org/doi/abs/10.1145/3565475.3569082

**核心貢獻**：在 **P4 可程式化 ASIC**（Barefoot Tofino2）上實作 Sketch 熵估計，透過查表插值法將複雜運算轉換為 match-action pipeline 中的快速查表。實現 **400 Gbps** 線速熵估計。

**與新方向的關聯**：**最直接對應新方向的一篇。** Sketch 熵估計可在 P4 交換機上以線速執行，攻擊偵測邏輯直接嵌入資料平面，無需鏡像流量到控制器。此架構可直接作為論文的系統設計參考。

---

## 三、新方向的邏輯架構

```
                     攻擊流量
                        │
                        ▼
              ┌─────────────────┐
              │  SDN 可程式化    │
              │  資料平面 (P4)   │  ← 參考文獻 ❺
              └────────┬────────┘
                        │
          ┌─────────────┼─────────────┐
          ▼             ▼             ▼
    ┌──────────┐ ┌──────────┐ ┌──────────┐
    │ CM Sketch│ │ Elastic  │ │ UCL-     │  ← 參考文獻 ❶❷
    │ (頻率)   │ │ Sketch   │ │ Sketch   │
    └────┬─────┘ └────┬─────┘ └────┬─────┘
         │            │            │
         └────────────┼────────────┘
                      ▼
              ┌─────────────────┐
              │  特徵向量提取    │
              │  (entropy,      │
              │   frequency,    │  ← 參考文獻 ❹（分類思維）
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

## 四、修改後的論文定位

| | 原方向 | 新方向 |
|---|--------|--------|
| **核心問題** | 如何優化熵值估計精度？ | 如何用通用 Sketch 區分不同攻擊？ |
| **技術手段** | 改良 Elastic Sketch 熵值模組 | 調查/組合多種 Sketch 做攻擊特徵提取 |
| **貢獻** | Debiasing + 多維度熵值向量 | Sketch-based 攻擊識別框架 + SDN 部署 |
| **Baseline** | Elastic Sketch | CM Sketch、Elastic、UnivMon、UCL-Sketch |
| **部署** | Mininet 模擬 | P4 交換機（BMv2 / Tofino） |

---

## 五、建議的論文題目（更新版）

> **基於通用網路測量 Sketch 之 SDN 攻擊偵測與分類**
>
> *Network-Wide Measurement Sketch-Based Attack Detection and Classification in SDN*

或保留多維度概念但調整重心：

> **SDN 環境下基於多 Sketch 協同之網路攻擊識別方法**
>
> *Multi-Sketch Cooperative Framework for Network Attack Identification in SDN*

---

## 六、下一步

1. [ ] 與教授確認新方向的理解是否正確
2. [ ] 研讀 5 篇新文獻，產出詳細筆記
3. [ ] 調查更多通用網路測量 Sketch（UnivMon、NitroSketch、SketchLearn 等）
4. [ ] 設計「不同攻擊 → 不同 Sketch 選擇策略」的方法框架
5. [ ] 確認實驗平台：P4 BMv2 軟體交換機 or Mininet + OVS？
