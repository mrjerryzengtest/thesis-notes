# 06 — Entropy-based DDoS Detection in SDN 文獻搜尋整理

> 整理日期：2026-05-25
> 用途：擴充 related work 文獻清單，為論文文獻回顧做準備

---

## A. Entropy-based DDoS 檢測（核心相關）

| # | 論文 | 作者 | 出處 | 年份 | 重點 |
|---|------|------|------|:---:|------|
| 1 | JESS: Joint Entropy-Based DDoS Defense Scheme in SDN | Kalkan, Altay, Gür, Alagöz | IEEE JSAC | 2018 | 聯合熵值防禦框架，結合 source/destination entropy |
| 2 | An Entropy-Based Network Anomaly Detection Method | Bereziński, Jasiul, Szpyrka | Entropy (MDPI) | 2015 | 熵值異常檢測方法的全面綜述（高引用） |
| 3 | DDoS Attack Detection Using Fast Entropy Approach on Flow-Based Network Traffic | David & Thomas | Procedia Computer Science | 2015 | 基於 flow 的快速熵值計算 DDoS 檢測 |
| 4 | Early Detection of DDoS Attacks Against SDN Controllers | Mousavi & St-Hilaire | IEEE ICC | 2015 | SDN controller 層級 entropy 早期檢測 |
| 5 | An Entropy-Based Distributed DDoS Detection Mechanism in SDN | Wang, Jia, Ju | IEEE TrustCom | 2015 | SDN 分散式 entropy 檢測架構 |
| 6 | Combining OpenFlow and sFlow for Anomaly Detection in SDN | Giotis et al. | Computer Networks (Elsevier) | 2014 | OpenFlow + sFlow 混合 entropy detection |

## B. 多維度特徵／熵值分類

| # | 論文 | 作者 | 出處 | 年份 | 重點 |
|---|------|------|------|:---:|------|
| 7 | A Multi-dimensional Approach for DDoS Attack Detection | Hoque, Bhattacharyya, Kalita | JNCA | 2015 | 多維度特徵融合，方法論可參考 |
| 8 | Detection and Defense of DDoS Attack Based on Deep Learning in OpenFlow-Based SDN | Li et al. | IJCS | 2018 | DL + entropy features，分類準確度對比 |

## C. Sketch-based Measurement（Baseline 與 Related Work）

| # | 論文 | 作者 | 出處 | 年份 | 狀態 |
|---|------|------|------|:---:|:----:|
| 9 | Elastic Sketch | Yang et al. | SIGCOMM | 2018 | ✅ 已研讀 |
| 10 | PINT | Ben Basat et al. | SIGCOMM | 2020 | ✅ 已研讀 |
| 11 | UnivMon | Liu et al. | SIGCOMM | 2016 | 待研讀 — universal sketch-based monitoring |
| 12 | SketchVisor | Huang et al. | SIGCOMM | 2017 | 待研讀 — robust sketch under overload |
| 13 | NitroSketch | Liu et al. | NSDI | 2019 | 待研讀 — sketch on programmable switches |
| 14 | Count-Min Sketch | Cormode & Muthukrishnan | LATIN / J. Algorithms | 2004/2005 | ✅ 已研讀 |

## D. SDN 安全綜述（建立 Related Work 全景）

| # | 論文 | 作者 | 出處 | 年份 |
|---|------|------|------|:---:|
| 15 | Security in SDN: A Comprehensive Survey | Correa Chica et al. | JNCA | 2020 |
| 16 | The Application of SDN on Securing Computer Networks: A Survey | Sahay, Meng, Jensen | JNCA | 2019 |
| 17 | Distributed Denial of Service (DDoS) Resilience in Cloud | Osanaiye, Choo, Dlodlo | JNCA | 2016 |

---

## 待搜尋關鍵字（建議用 Google Scholar / DBLP / arXiv）

| # | 關鍵字組合 | 目標 |
|---|-----------|------|
| 1 | `"joint entropy" + SDN + DDoS` | JESS 後續引用文獻，找 multi-dimensional entropy 的競爭者 |
| 2 | `"sketch-based entropy estimation"` | 核心技術理論基礎 |
| 3 | `"multi-dimensional entropy" + "network attack classification"` | 直接競爭文獻 |
| 4 | `"CM Sketch" + "debiasing" + "entropy"` | 方法論關鍵挑戰 |
| 5 | `"entropy-based" + "DDoS" + "SDN" + 2022..2026` | 近五年新論文 |
| 6 | `"P4" + "entropy" + "telemetry"` | P4/INT-based entropy 遙測方案 |

---

## 下一步

- [ ] 逐一取得上述待研讀論文 PDF
- [ ] 對 #1-#8 進行深入閱讀與摘要
- [ ] 建立文獻對照矩陣（方法、熵值維度、SDN 平台、攻擊類型）
- [ ] 與教授討論後根據回饋擴充或收斂範圍
