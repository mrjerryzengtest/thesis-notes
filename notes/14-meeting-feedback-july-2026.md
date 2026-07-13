# 14 — 教授 Meeting 結論與方向確認（2026-07-06）

## 一、會議摘要

2026-07-06（日）與教授線上討論，確認論文方向與後續工作分配。會議主要圍繞 sketch-based entropy estimation 在 SDN 攻擊檢測中的應用。

## 二、教授的核心回饋

| 項目 | 結論 |
|------|------|
| 論文方向 | 教授認可「多維度熵值 + 多 Sketch 協作」方向 |
| 題目 | 暫定沿用原題目，無需修改 |
| 文獻協助 | 教授主動協助搜尋 sketch 計算 entropy 的相關文獻 |
| 學生任務 | 自行搜尋文獻，證明 entropy 與 cybersecurity 的關聯性 |

## 三、工作分配

### 教授負責
- 🔍 搜尋 **sketch 計算 entropy** 的相關文獻（理論基礎層）

### 學生負責（Jerry）
- 🔍 搜尋文獻證明 **entropy 與 cybersecurity / network security** 的關聯
  - 目標：建立 entropy 作為網路安全指標的理論基礎
  - 方向：entropy-based DDoS detection、anomaly detection、intrusion detection
  - 需涵蓋：為什麼 entropy 能反映網路異常、有哪些成功案例

## 四、確認的論文題目

> **SDN 環境下基於多維度熵值分析之網路攻擊精確檢測與分類**
>
> *Multi-Dimensional Entropy Analysis for Accurate Network Attack Detection and Classification in SDN*

## 五、下一步清單

| # | 任務 | 負責 | 優先級 |
|---|------|:----:|:------:|
| 1 | 搜尋 entropy ↔ cybersecurity 關聯文獻 | Jerry | 🔴 高 |
| 2 | 等待教授提供 sketch-entropy 文獻 | 教授 | 🟡 中 |
| 3 | 收到教授文獻後，整合兩批文獻（sketch + entropy-security） | Jerry | 🟢 後續 |
| 4 | 建立完整文獻邏輯鏈（理論基礎 → 核心原語 → 攻擊檢測 → 部署） | Jerry | 🟢 後續 |

## 六、筆記

- 教授認可目前方向，沒有提出重大修正
- 題目維持原案，表示前期方向選擇與教授期望一致
- 文獻搜尋是現階段核心工作，分成兩個互補路徑：
  1. **由上而下**（教授）：sketch 技術如何計算 entropy
  2. **由下而上**（Jerry）：entropy 為何是有效的安全指標
- 兩條線收斂後，即可形成完整文獻基礎章節（Chapter 2）
