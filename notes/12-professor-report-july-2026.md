# 12 — 教授討論報告（2026-07-06）

> 距前次討論（2026-05-31）約五週，整理進度與待討論事項

---

## 一、前次指定文獻研讀報告

教授前次指定五篇文獻，以下回報各篇理解與對本論文的關聯：

---

### 文獻一：Clifford & Cosma（AISTATS 2013）
**"A Simple Sketching Algorithm for Entropy Estimation over Streaming Data"**

**內容摘要：** 本篇提出以 α-stable distribution 對資料串流做 sketching，再透過 log-mean estimator 近似 Shannon 熵。與先前方法不同處在於直接使用 α = 1 的 stable distribution，避免了從 Rényi 熵取極限所引入的額外誤差。文中推薦 ζ = 1 的估計器，其誤差機率尾界呈指數衰減、漸近相對效率達 0.932。

**與本研究關聯：** 提供 sketch-based entropy estimation 的嚴格理論基礎。本研究後續若需證明所提出攻擊檢測方法之誤差界限，可援引此文的理論框架。

---

### 文獻二：Yuan et al.（IEEE TKDE 2026）
**"Learning-Based Sketches for Frequency Estimation in Data Streams Without Ground Truth"**

**內容摘要：** 提出 UCL-sketch，結合壓縮感知與等效學習（Equivalent Learning），在完全不需要 ground truth 的條件下進行線上訓練，使 sketch 能即時適應串流資料的分佈變化。解碼速度較既有方法提升約 500 倍，且在極端記憶體限制下仍接近最優精度。

**與本研究關聯：** UCL-sketch 具備無標籤自適應能力，適合本研究中攻擊流量模式動態變化的場景。其精確的 per-flow 頻率估計可作為攻擊特徵提取的基礎原語。

---

### 文獻三：Soto et al.（IEEE Access 2021）
**"A High-Throughput Hardware Accelerator for Network Entropy Estimation Using Sketches"**

**內容摘要：** 於 Xilinx UltraScale+ FPGA 上實作三級架構（基數估計 → Top-K 頻率估計 → 殘餘熵近似），達成 204 Gbps 吞吐量、21 μs 延遲、平均相對誤差低於 1.5%。使用晶片上記憶體且資源利用率低於 50%。

**與本研究關聯：** 驗證 sketch 熵估計在硬體上以線速部署的可行性，可作為本研究若選用 FPGA 平台時的實作參考。

---

### 文獻四：Wang et al.（ICPP 2025）
**"A High-Accuracy Sketch for Measuring Low-Entropy Flows in Distributed AI Training"**

**內容摘要：** 針對分散式 AI 訓練流量的低熵特性，設計 FP-Sketch：前端 staging queue 預測並分類 flow 大小，後端分層儲存依流量類型使用不同策略。相較既有方法，flow 估計誤差降低 38.6%、插入吞吐量提升 49.4%。

**與本研究關聯：** 此文的核心洞察——「不同流量模式需要不同設計策略的 Sketch」——直接支持本研究的方向：DDoS、Port Scan、慢速攻擊等不同攻擊類型產生不同流量特徵，應選用或設計對應的 sketch 來做區分，而非以單一 sketch 和單一 entropy threshold 做判斷。

---

### 文獻五：Lai et al.（EuroP4 2022）
**"Sketch-Based Entropy Estimation: A Tabular Interpolation Approach Using P4"**

**內容摘要：** 於 Barefoot Tofino2 可編程交換機上，以表格插值法將 sketch 熵估計中的複雜對數運算轉化為 match-action pipeline 的查表操作，達成 400 Gbps 線速處理。使用插值啟發式縮減查找表大小以符合 ASIC 記憶體限制。

**與本研究關聯：** 本篇最直接對應本研究之系統架構。展示了在 P4 交換機上以線速執行 sketch 熵估計的可行性，攻擊檢測邏輯可嵌入資料平面，無需將流量鏡像至外部伺服器。此為本研究系統設計之主要參考藍圖。

---

## 二、論文方向

### 方向演進

| | 原始方向（5/1） | 調整後方向（5/31） |
|---|--------------|-----------------|
| 核心問題 | 如何優化熵值估計精度？ | 如何用通用網路測量 Sketch 區分不同攻擊？ |
| 技術手段 | 改良 Elastic Sketch 的 entropy 模組 | 調查並組合多種 Sketch 做攻擊特徵提取 |
| 貢獻定位 | Debiasing + 多維度熵值向量 | Sketch-based 攻擊識別框架 + SDN 部署 |
| 主要 baseline | Elastic Sketch（單一） | CM Sketch、Elastic、UnivMon、UCL-Sketch 等 |
| 部署平台 | Mininet 模擬 | P4 交換機（BMv2 軟體模擬 or Tofino 硬體） |

### 調整緣由

前次討論教授的建議：不應聚焦於優化 entropy 公式本身，而是利用現有的通用網路測量 sketch 來針對不同攻擊的流量特徵做識別。此調整使研究方向從「改良測量工具」轉向「用測量工具做診斷」，更具應用價值。

### 預定論文題目

> **SDN 環境下基於多 Sketch 協同之網路攻擊識別方法**
>
> *Multi-Sketch Cooperative Framework for Network Attack Identification in SDN*

備選版本：
> **基於通用網路測量 Sketch 之 SDN 攻擊偵測與分類**
>
> *Network-Wide Measurement Sketch-Based Attack Detection and Classification in SDN*

---

## 三、討論事項

### 議題一：方向確認

- 上述論文方向與題目的理解是否正確？
- 目前定位為「調查既有 sketch、針對攻擊類型選擇/組合、在 SDN 環境部署評估」，此貢獻層級對碩士論文是否足夠？
- 是否需要提出新的 sketch 設計，或者「組合既有方法」即可？

### 議題二：文獻範圍

- 除了前次指定的五篇及原始的 Elastic Sketch、PINT 等，是否還有其他建議涵蓋的子領域或關鍵文獻？
- 預計調查的通用 sketch：UnivMon、NitroSketch、SketchLearn、MRAC 等，是否有優先順序建議？

### 議題三：實驗平台

- 目前考量兩個方案：
  - **方案 A**：P4 BMv2（軟體模擬交換機），可驗證 P4 程式邏輯
  - **方案 B**：Mininet + Open vSwitch，環境搭建較簡易
- 實驗室是否有可用的 P4 硬體（如 Tofino）資源？
- 攻擊流量生成工具建議？（如 D-ITG、hping3、Scapy 等）

### 議題四：攻擊類型範圍

- 論文預計涵蓋的攻擊類型範圍建議：
  - DDoS（SYN flood、UDP flood）
  - Port Scan
  - 是否需包含慢速攻擊（Slowloris）或 Botnet C&C？
- 各攻擊類型需有足夠的流量特徵差異，以驗證多 sketch 分類的有效性

### 議題五：時間規劃

- 文獻回顧預計完成時間
- 實驗設計與初步結果時程
- 論文撰寫時程
- 是否鎖定特定投稿會議/期刊？（NOMS、IM、TNSM、APNet 等）

---

## 四、目前進度摘要

| 階段 | 事項 | 狀態 |
|------|------|:----:|
| 第一階段（5/1） | 研讀教授 6 項原始參考資料 | ✅ |
| | 選定論文方向（多維度熵值分析） | ✅ |
| 第二階段（5/31） | 與教授討論，方向調整為「通用 Sketch 攻擊識別」 | ✅ |
| | 取得 5 篇新指定文獻 | ✅ |
| 第三階段（7/6） | 五篇文獻研讀與整理 | ✅ |
| | 論文方向與題目確認 | 🔄 今日討論 |
| 下一階段 | 調查更多通用 Sketch、設計方法框架 | ⏳ 待進行 |
| | 實驗平台確認與環境搭建 | ⏳ 待進行 |

---

> 整理日期：2026-07-06
