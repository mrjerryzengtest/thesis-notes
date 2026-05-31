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

## 教授提供的 6 項參考資料角色

| # | 資料 | 在我的論文中的角色 |
|---|------|-------------------|
| 1 | **Elastic Sketch** (SIGCOMM '18) | 🔧 **Baseline 工具** — 我拿它的量測能力來用，但做它沒做的事 |
| 2 | **PINT** (SIGCOMM '20) | 💡 **Motivation 來源** — 它說 subsampling 損害 entropy，我用 sketch 方法解決 |
| 3 | **Top-Down Telemetry** (Minlan Yu '19) | 🗺️ **研究脈絡** — 理解 telemetry 領域的大方向 |
| 4 | **Count-Min Sketch** (Cormode '05) | 📐 **數學基礎** — Elastic 的 Light Part 就是 CM Sketch，bias 問題從這裡來 |
| 5 | **Splunk Telemetry Blog** | 🏭 **產業背景** — telemetry 的實務框架 |
| 6 | **INT P4 / CM Sketch 影片** | 🎬 **視覺理解** — 技術實作展示 |

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
