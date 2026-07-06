# Thesis Notes — SDN 熵值攻擊檢測

> 研究生：Jerry Zeng
> 建立日期：2026-05-01
> GitHub：[mrjerryzengtest/thesis-notes](https://github.com/mrjerryzengtest/thesis-notes)

---

## 2026-05-01 進度總結

### 背景

論文方向：利用「熵值（Entropy）」判斷 SDN 網路是否遭受攻擊，針對其中的「熵值」計算部分做優化，以更精確判斷攻擊。

### 完成事項

| # | 事項 | 狀態 |
|---|------|:----:|
| 1 | 研讀教授提供的 6 項參考資料 | ✅ |
| 2 | 撰寫 **文獻詳細整理**（含資源關係圖） | ✅ |
| 3 | 撰寫 **論文方向分析**（4 個題目方案 + 研究步驟） | ✅ |
| 4 | 選定論文題目（方案 C：多維度熵值分析） | ✅ |
| 5 | 擬定給教授的進度回報郵件 | ✅ |
| 6 | 萃取 **8 個核心技術洞察** | ✅ |
| 7 | 建立 Git 儲存庫並推送至 GitHub | ✅ |

### 論文題目（暫定）

> **SDN 環境下基於多維度熵值分析之網路攻擊精確檢測與分類**
>
> *Multi-Dimensional Entropy Analysis for Accurate Network Attack Detection and Classification in SDN*

### 教授提供的參考資料

| # | 資源 | 類型 | 核心要點 |
|---|------|------|---------|
| 1 | Elastic Sketch（SIGCOMM 2018） | PDF 論文 | Baseline：自適應 sketch，已內建 entropy estimation |
| 2 | PINT（SIGCOMM 2020） | PDF 論文 | Motivation：subsampling 損害 entropy 估計 |
| 3 | Network Telemetry Top-Down（Minlan Yu, 2019） | ACM 論文 | 研究脈絡：top-down telemetry 架構 |
| 4 | Count-Min Sketch（Wikipedia + 原始論文） | 基礎文獻 | Elastic light part 的數學基礎 |
| 5 | Splunk Network Telemetry Blog | 網路文章 | 產業背景與 telemetry 框架 |
| 6 | INT P4 Video + CM Sketch Video | YouTube | 技術視覺化理解 |

### 核心研究構想

不依賴單一熵值，而是利用 **Elastic Sketch 同時支援多種測量任務**的特性，計算**多維度熵值向量**：

| 維度 | 說明 |
|------|------|
| H_srcIP | Source IP entropy |
| H_dstIP | Destination IP entropy |
| H_srcPort | Source Port entropy |
| H_dstPort | Destination Port entropy |
| H_flowSize | Flow Size entropy |

藉由不同攻擊類型在熵值空間中的分佈差異，達到**攻擊檢測 + 攻擊分類**的雙重目標。

### 2026-05-06 進度

| # | 事項 | 狀態 |
|---|------|:----:|
| 1 | 教授回信，約線上討論 | ✅ |
| 2 | 整理 **教授討論準備資料**（7 大議題 + 提問清單 + 視覺素材） | ✅ |
| 3 | 撰寫 SDN 實驗環境搭建指南（Mininet + ONOS/Ryu） | ✅ |

### 2026-05-25 進度

| # | 事項 | 狀態 |
|---|------|:----:|
| 1 | 搜尋 entropy-based DDoS detection in SDN 相關文獻（17 篇整理） | ✅ |
| 2 | 擬定出差返國後給教授的追蹤信草稿 | ✅ |

### 2026-05-31 進度

| # | 事項 | 狀態 |
|---|------|:----:|
| 1 | 整理 Meeting 快速參考 — 論文 vs. Elastic Sketch 差異對照 + 三論據 | ✅ |
| 2 | 與教授線上討論完成 — 方向調整：聚焦通用 Sketch 做攻擊識別 | ✅ |
| 3 | 整理教授 5 篇新指定文獻 + 新方向架構（notes/09-*.md） | ✅ |

### 2026-07-06 進度

| # | 事項 | 狀態 |
|---|------|:----:|
| 1 | 回顧論文進度 + 整理 Meeting 準備資料 | ✅ |
| 2 | 整理教授指定全部 11 篇文獻總覽（第一批 6 篇 + 第二批 5 篇） | ✅ |
| 3 | 撰寫 Meeting 討論議題清單（5 大議題） | ✅ |
| 4 | 建立 notes/10-meeting-prep-july-2026.md 會議準備文件 | ✅ |
| 5 | 撰寫 11 篇文獻詳細技術整理（notes/11-*.md，~19K 字） | ✅ |

### 下一步

1. 🎯 ~~與教授線上討論~~ → ✅ 已完成（2026-05-31）
2. 🎯 ~~今日與教授討論~~ → 今日進行中
3. 📖 研讀教授新指定的 5 篇文獻
4. 🔍 調查更多通用網路測量 Sketch（UnivMon、NitroSketch、SketchLearn 等）
5. 🖥️ 設計「不同攻擊 → 不同 Sketch 選擇策略」的方法框架
6. 🧪 確認實驗平台：P4 BMv2 or Mininet + OVS？

---

## 目錄結構

```
thesis-notes/
├── README.md                          ← 本檔案
├── docs/
│   └── sdn-lab-setup-guide.md         ← SDN 實驗環境搭建指南
└── notes/
    ├── 01-resource-summary.md         ← 參考資料詳細整理
    ├── 02-thesis-direction.md         ← 論文方向分析 + 4 個題目方案
    ├── 03-email-to-professor.md       ← 給教授的郵件草稿
    ├── 04-key-insights.md             ← 8 個核心技術洞察
    ├── 05-professor-discussion-prep.md ← 教授線上討論準備資料
    ├── 06-literature-search-entropy-ddos.md ← Entropy DDoS 文獻搜尋整理
    ├── 07-follow-up-email-after-trip.md ← 出差返國追蹤信草稿
    ├── 08-meeting-cheat-sheet.md        ← Meeting 快速參考（論文 vs. Elastic Sketch）
    └── 09-meeting-feedback-new-direction.md ← 教授 Meeting 結論 + 新方向 + 5 篇新文獻
    └── 10-meeting-prep-july-2026.md     ← 2026-07-06 Meeting 準備資料（文獻總覽 + 議題）
    └── 11-detailed-literature-review.md ← 11 篇文獻詳細技術整理（完整版，~19K 字）
```
