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

### 下一步

1. 🎯 與教授線上討論（使用 notes/05-professor-discussion-prep.md）
2. 📧 寄出追蹤信給教授（notes/07-follow-up-email-after-trip.md）
3. 📖 根據教授回饋調整研究方向
4. 📄 深入閱讀文獻搜尋結果中標記「待研讀」的論文
5. 🖥️ 搭建 Mininet + ONOS/Ryu SDN 實驗環境

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
    └── 07-follow-up-email-after-trip.md ← 出差返國追蹤信草稿
```
