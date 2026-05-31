# 08 — Meeting 快速參考：你的論文 vs. Elastic Sketch

> 整理日期：2026-05-31
> 用途：與教授討論時，快速說明你的論文與 Elastic Sketch 的差異

---

## 一句話講完

> **Elastic Sketch 是「量測工具」，我的論文是用這個工具來「診斷攻擊」——而且是多維度診斷，不是只看一個數字。**

---

## 核心區別對照表

| | **Elastic Sketch (SIGCOMM '18)** | **我的論文** |
|---|---|---|
| **做什麼** | 發明新的 sketch 資料結構 | 用 sketch 輸出做攻擊檢測與分類 |
| **核心問題** | 「如何高效量測網路？」 | 「如何判斷攻擊 + 是什麼攻擊？」 |
| **輸出** | 流量統計值 | 攻擊警報 + 攻擊類型標籤 |
| **熵值** | **1 個** aggregate entropy | **5 維** 熵值向量 |
| **攻擊分類** | ❌ 沒有 | ✅ DDoS / Port Scan / Botnet 分類 |
| **SDN** | 通用框架 | 針對 SDN 環境設計與評估 |

---

## 最關鍵的技術差異：單一熵值 → 多維度熵值向量

**Elastic Sketch 給你：**

```
H_total = 所有 counter 混在一起算 Shannon entropy → 一顆數字
```

**我的方法需要：**

```
H_srcIP    ─┐
H_dstIP     │
H_srcPort   ├── 五維熵值向量 → 攻擊分類
H_dstPort   │
H_flowSize ─┘
```

這不是 trivial extension。Elastic 的 counter 是混在一起的，我需要讓它們**按維度分解**——這是原 paper 沒做的事。

---

## 教授提供的 6 項參考資料 — 論文內容說明與角色

---

### ❶ Elastic Sketch（SIGCOMM 2018）

- **作者**：Tong Yang 等（北京大學、中科院、阿里巴巴、QMUL）
- **引用**：539+，頂會最佳論文候選

**這篇在講什麼？**

當網路發生壅塞、DDoS、掃描攻擊時，流量特徵劇烈變化——此時測量最重要，但傳統方法效能會崩潰。Elastic Sketch 是第一個**自適應**的網路測量 sketch。

核心設計：**Heavy Part + Light Part 分離架構**
- Heavy Part（hash table）：記錄 elephant flow 的完整 ID + 計數
- Light Part（CM Sketch）：用 8-bit counter 處理 mouse flow，不存 flow ID
- 「Ostracism 陶片放逐制」：動態決定哪些 flow 留 Heavy、哪些踢到 Light
- 三種自適應：頻寬自適應（Maximum Compression）、封包速率自適應、flow 分佈自適應

一個 600KB 的結構同時支援六種任務（flow size、heavy hitter、heavy change、distribution、**entropy**、cardinality）。

**熵值估計**：從 Light Part 收集 counter distribution，套 Shannon 公式。作者在 §6.1 承認這是初步方案，DDoS 檢測列為 future work。

> 🔧 **在我論文中的角色**：Baseline 工具 — 拿它的量測能力來用，但做它沒做的事（攻擊檢測 + 分類）

---

### ❷ PINT: Probabilistic In-band Network Telemetry（SIGCOMM 2020）

- **作者**：Ran Ben Basat 等（Harvard、USC、QMUL），Minlan Yu 為共同作者
- **引用**：248+

**這篇在講什麼？**

INT（In-band Network Telemetry）讓交換機在每個封包上附加 telemetry 資訊，但 overhead 過大——5 跳路徑 + 2 個值/跳 = 48 bytes/packet，導致 goodput 下降 20%。PINT 提出**概率式** telemetry：將資訊概率性地編碼到多個封包上，per-packet overhead 可低至 1 bit。

**關鍵洞察（§4.1）**：
> *"Entropy is poorly approximable from subsampled streams."*

概率性子採樣會嚴重損害熵值估計的準確性。這是 PINT 自我揭露的限制，也是你論文的切入點。

> 💡 **在我論文中的角色**：Motivation 來源 — PINT 說 subsampling 不行，我用 sketch-based（非 sampling）方法正面解決

---

### ❸ Network Telemetry: Towards a Top-Down Approach（Minlan Yu, CCRV 2019）

- **作者**：Minlan Yu（Harvard，PINT 共同作者）

**這篇在講什麼？**

反對傳統 Bottom-up 方法（先做硬體支援再想要量什麼），主張 Top-down：
1. 營運商先宣告「想要量什麼」
2. 系統自動轉譯成交換機上的可程式化測量原語
3. Runtime 動態執行

這篇是 Minlan Yu 的 telemetry 願景宣言，PINT（2020）是這個願景的具體實現。

> 🗺️ **在我論文中的角色**：研究脈絡 — 理解 telemetry 領域的大方向，你的工作也在這個脈絡中

---

### ❹ Count-Min Sketch（Cormode & Muthukrishnan, Journal of Algorithms 2005）

- **作者**：Graham Cormode、S. Muthukrishnan
- **結構**：d rows × w columns 二維陣列，d = ⌈ln(1/δ)⌉，w = ⌈e/ε⌉

**這篇在講什麼？**

Count-Min Sketch 是機率式資料結構的經典，用 sub-linear 空間估計資料流中各項目的頻率。核心特性：
- âᵢ ≥ aᵢ（**只有 over-estimation，沒有 under-estimation**）
- Error bound：âᵢ ≤ aᵢ + ε‖f‖₁（機率 1-δ）
- 查詢時取 d 列的最小值（median trick）來抑制 over-estimation

**在 Elastic Sketch 中的角色**：CM Sketch = Elastic 的 Light Part。但 Elastic 只用 d=1（單列），**無法用 median trick 去偏**——這是你的方法需要解決的關鍵技術挑戰。

> 📐 **在我論文中的角色**：數學基礎 — counter over-estimation bias 直接污染 entropy，debiasing 是核心貢獻

---

### ❺ Splunk Blog: What is Network Telemetry?（Laiba Siddiqui, 2023）

- **來源**：Splunk 官方技術部落格

**這篇在講什麼？**

從產業角度整理 telemetry 四大模組：Management Plane（SNMP/syslog）、Control Plane（路由協定）、Forwarding Plane（即時轉發資訊）、Data Analytics（AI/ML 自動化分析）。

> 🏭 **在我論文中的角色**：產業背景 — 你的 entropy-based 攻擊檢測落在 Data Analytics 層，有明確的實務定位

---

### ❻ INT P4 實作影片 + Count-Min Sketch 原理影片

- INT P4 Video：P4 Language Consortium，展示 INT 在 P4 交換機上的實作
- CM Sketch Video：Raphael De Lio @ Devoxx UK，機率式資料結構原理講解

> 🎬 **在我論文中的角色**：視覺理解素材 — 幫助快速掌握技術的實際運作方式

---

## 三個最強論據

### ❶ Elastic 作者自己說 DDoS 是 future work

> Elastic Sketch §6.1：*"For other tasks (e.g., DDoS, SuperSpreader, and more), we will study how to apply Elastic in the future work."*

→ 我不是重複做 Elastic，我是**接它留下的棒子**。

### ❷ 單一熵值無法分類攻擊

DDoS 和 Port Scan 都會讓 entropy 升高——但**升的是不同維度**：

```
              H_srcIP   H_dstIP   H_dstPort
DDoS          ↑↑        →         →
Port Scan     →         →         ↑↑
```

只看 H_total 分不出來，多維度才分得出來。

### ❸ PINT 的悲觀結論 = 我的 motivation

> PINT §4.1：*"Entropy is poorly approximable from subsampled streams."*

→ 我用 **sketch-based** 方法（非 sampling），正面回應這個挑戰。

---

## 類比（幫助教授快速理解）

```
Elastic Sketch  = 發明「體溫計」
我的論文        = 用體溫計數據，透過多維度體溫模式，診斷「是什麼病」

不是重複發明溫度計，是用溫度計做診斷。
```

---

## 論文題目

> **SDN 環境下基於多維度熵值分析之網路攻擊精確檢測與分類**
>
> *Multi-Dimensional Entropy Analysis for Accurate Network Attack Detection and Classification in SDN*

---

## 會議討論快速索引

完整討論清單 → [`notes/05-professor-discussion-prep.md`](05-professor-discussion-prep.md)

| 議題 | 核心問題 |
|------|---------|
| ❶ 方向確認 | 多維度熵值方向可行嗎？創新性夠嗎？ |
| ❷ 貢獻層級 | 碩士論文該抓理論/方法/系統哪個層級？ |
| ❸ 文獻範圍 | 還需要涵蓋哪些子領域？ |
| ❹ 實驗設計 | Mininet 規模夠嗎？用 OVS 還是 P4 BMv2？ |
| ❺ 時間規劃 | 8-12 個月合理嗎？ |
| ❻ 技術難點 | CM Sketch bias 如何 debias？多維度如何融合？ |
| ❼ 投稿目標 | NOMS/IM？TNSM？APNet？ |
