# 05 — 教授線上討論準備資料

> 整理日期：2026-05-06
> 用途：與教授線上討論時的議題清單與參考素材

---

## 一、目前進度摘要（教授已知的部分）

- ✅ 研讀完教授提供的 6 項參考資料（Elastic Sketch、PINT、CM Sketch 等）
- ✅ 完成文獻整理與資源關係脈絡圖
- ✅ 提出 4 個題目方案（A/B/C/D），選定 **方案 C：多維度熵值分析**
- ✅ 暫定題目：**SDN 環境下基於多維度熵值分析之網路攻擊精確檢測與分類**
- ✅ 已寄出郵件說明初步構想，教授回信約線上討論

---

## 二、核心討論議題（建議依序討論）

### 🔴 議題 1：題目方向確認與可行性

**向教授說明的主軸：**

> 不依賴單一熵值，而是利用 Elastic Sketch 同時支援多任務的特性，計算多維度熵值向量（H_srcIP, H_dstIP, H_srcPort, H_dstPort, H_flowSize），利用不同攻擊在熵值空間中的分佈差異，實現攻擊檢測 + 攻擊分類。

**提問：**

- ❓ 這個方向有沒有您認為需要調整的地方？（範圍太大？太窄？創新性夠嗎？）
- ❓ 方案 C（多維度）和方案 A（直接優化 Elastic entropy estimation）之間，教授認為哪個更適合碩士層級？
- ❓ 是否需要再加入一個維度（如 protocol entropy、packet size entropy）？還是五個已經足夠？

**準備說明的論據：**

- Elastic 作者在 §6.1 明確提到 DDoS detection 是 future work → 我們正好填補這個空白
- Elastic 的 genericity（一個結構支援六種任務）是多維度方法的天然基礎，不是硬湊的
- PINT 指出 subsampling 損害 entropy → 我們用 sketch-based（而非 sampling-based）方法正面回應

---

### 🔴 議題 2：研究貢獻層級與碩士論文定位

**提問：**

- ❓ 碩士論文的貢獻層級應該抓在哪裡？主要貢獻應該是理論分析（CM Sketch bias 對 entropy 的 error bound）、方法設計（multi-dimensional entropy vector）、還是系統實作（SDN prototype）？
- ❓ 四個預期貢獻（理論分析 + 方法設計 + 系統實作 + 實驗比較），哪一項教授認為最重要？
- ❓ 需要做到 **formal proof**（如推導 debiased entropy error bound）嗎？還是實驗驗證為主即可？

---

### 🔴 議題 3：文獻回顧範圍

**目前規劃的 related work 方向：**

| 面向 | 預計搜尋關鍵字 | 核心文獻 |
|------|--------------|---------|
| Sketch-based measurement | Elastic Sketch, UnivMon, SketchVisor | SIGCOMM 2018-2020 |
| Entropy-based attack detection | entropy DDoS detection SDN | 待搜尋 |
| P4/INT-based telemetry | PINT, P4 entropy tracking | Ding et al. NOMS 2020 |
| CM Sketch theory | Count-Min, Count Sketch | Cormode 2005 |

**提問：**

- ❓ 除了 entropy-based DDoS detection，還有哪些子領域的 related work 應該涵蓋？（如：anomaly detection in network traffic、sketch-based heavy hitter detection？）
- ❓ 教授有沒有推薦必讀的論文或 survey？（尤其 SDN entropy detection 領域）
- ❓ Related work 應該追溯到多遠？（2000 年早期 entropy 在網路安全中的應用？還是聚焦 2018 以後？）

---

### 🔴 議題 4：實驗設計與可行性

**目前規劃的實驗架構：**

```
Mininet 模擬 SDN 拓樸
    ├── 3 台交換機 + 6 台主機
    ├── Ryu/ONOS 控制器
    ├── 正常背景流量 + 攻擊流量注入
    └── Elastic Sketch 部署在交換機上收集 counter distribution
```

**攻擊場景規劃：**

| 攻擊類型 | 流量工具 | 熵值特徵（預期） |
|---------|---------|----------------|
| DDoS (SYN flood) | hping3 / MHDDoS | H_srcIP ↑, H_dstIP ↓ |
| DDoS (botnet) | 自訂腳本 | H_srcIP →, H_flowSize ↑ |
| Port Scan | nmap | H_dstPort ↑↑ |
| Low-rate DoS | 自訂腳本 | 熵值微幅變化 |

**提問：**

- ❓ 這個實驗規模對碩士論文足夠嗎？還是需要更大規模的拓樸或更多攻擊類型？
- ❓ 是否需要引入真實流量 trace（如 CAIDA、MAWI），還是合成流量即可？
- ❓ Elastic Sketch 在交換機上的部署方式：應該在 OVS 層面實作，還是用 P4 軟體交換機（BMv2）？哪個方向更可行？
- ❓ Baseline 比較對象除了原始 Elastic Sketch，還應該跟哪些方法比？（UnivMon？單純 CM Sketch entropy？ML-based detector？）

---

### 🔴 議題 5：時間規劃與里程碑

**目前規劃（總計約 8-12 個月）：**

| Phase | 內容 | 預估時間 |
|-------|------|:--------:|
| Phase 1 | 文獻深入 | 1-2 個月 |
| Phase 2 | 方法設計 | 2-3 個月 |
| Phase 3 | 實作與實驗 | 3-4 個月 |
| Phase 4 | 論文撰寫 | 2-3 個月 |

**提問：**

- ❓ 這個時間表合理嗎？有沒有哪個 phase 應該給更多/更少時間？
- ❓ 教授的期望畢業時間線是什麼？有沒有 conference/journal 投稿的 deadline 需要配合？
- ❓ 是否需要先產出一篇 workshop paper 或 poster 做為中期檢核點？

---

### 🔴 議題 6：技術難點預警（與教授確認解法方向）

**列舉目前能預見的挑戰：**

1. **CM Sketch over-estimation bias 的 debiasing**
   - Elastic 使用 d=1 的 CM Sketch（只有一列），無法用 median trick 去偏
   - 問題：如何在 d=1 的限制下去偏？需要方法層面的創新

2. **多維度熵值融合策略**
   - 五個維度如何組合成單一檢測分數？加權和？向量距離？ML classifier？
   - 若用 ML classifier 是否會偏離「熵值分析」的核心？

3. **攻擊分類的正確標記（Ground Truth）**
   - 實驗中如何確保攻擊流量的 label 正確？
   - 混合攻擊場景（DDoS + Port Scan 同時發生）如何處理？

4. **Entropy 的即時計算**
   - 在線（online）vs 離線（offline）entropy estimation？
   - 時間窗長度的選擇對檢測延遲的影響

**提問：**

- ❓ 這些挑戰中，教授認為哪一個最關鍵？哪一個可能成為論文的主要貢獻？
- ❓ 對於 debiasing 方法，教授有沒有建議的理論框架或參考文獻？
- ❓ Ground truth labeling 的部分，教授實驗室是否有現成的攻擊流量 dataset 可以使用？

---

### 🔴 議題 7：投稿目標

**目前考慮的選項：**

| venue | 類型 | 接受率 | 適合程度 |
|-------|------|:-----:|:--------:|
| IEEE/IFIP NOMS/IM | Conference | ~25% | ⭐⭐⭐ 網路管理領域直接相關 |
| ACM CoNEXT (student) | Conference | ~20% | ⭐⭐ 學生論文 track |
| IEEE TNSM | Journal | ~20% | ⭐⭐⭐ 適合完整系統論文 |
| Computer Networks (Elsevier) | Journal | ~20% | ⭐⭐ 篇幅較長 |
| APNet | Conference | ~25% | ⭐⭐⭐ 亞太區網路研討會 |

**提問：**

- ❓ 教授有沒有建議的投稿目標？國內外研討會優先？
- ❓ 碩士論文是「先寫完整論文 → 再投稿」，還是「以投稿為目標 → 論文基於投稿版本擴充」？

---

## 三、可展示給教授的視覺素材

### 素材 1：研究定位圖

```
SDN 攻擊檢測方法
    │
    ├── Rule-based（特徵碼）    → 無法應對新型攻擊
    ├── ML-based（機器學習）    → 需大量標記資料、可解釋性低
    └── Entropy-based（熵值）   ← 我們的方向
            │
            ├── 現有問題：單一熵值難以區分攻擊類型
            └── 我們的解法：多維度熵值向量 + Elastic Sketch
```

### 素材 2：多維度熵值攻擊分類概念

```
                     H_srcIP  H_dstIP  H_dstPort  H_flowSize
DDoS (spoofed)        HIGH     LOW      LOW        MEDIUM
DDoS (botnet)         MEDIUM   LOW      LOW        HIGH
Port Scan             LOW      HIGH     HIGH       LOW
Flash Crowd           MEDIUM   HIGH     MEDIUM     HIGH
Normal                MEDIUM   MEDIUM   MEDIUM     MEDIUM
```

### 素材 3：研究貢獻金字塔

```
            ┌─────────────────┐
            │  實驗貢獻         │  ← 系統性比較不同攻擊的熵值行為
            │  (實驗驗證)       │
            ├──────────────────┤
            │  系統貢獻         │  ← SDN 環境中 prototype 實作
            │  (實作可行性)     │
            ├──────────────────┤
            │  方法貢獻         │  ← 多維度熵值向量 + debiased sketch
            │  (核心創新)       │
            ├──────────────────┤
            │  理論貢獻         │  ← CM Sketch bias → entropy error bound
            │  (理論分析)       │
            └─────────────────┘
```

---

## 四、討論策略建議

1. **開場（2 分鐘）**：簡述目前進度 + 選定方案 C 的理由（不超過 3 個點）
2. **核心討論（15 分鐘）**：先讓教授對方向給回饋 → 再提出你的具體問題
3. **難點討論（10 分鐘）**：針對 debiasing 和多維度融合這兩個最關鍵的挑戰，請教教授看法
4. **下一步確認（3 分鐘）**：明確會議後的 action items（讀哪些 paper、先做什麼實驗）

---

## 五、會議後待辦（會後填寫）

- [ ] 教授對題目的具體回饋：
- [ ] 教授建議的額外參考文獻：
- [ ] 下一步優先事項：
- [ ] 下次會議時間／milestone：
