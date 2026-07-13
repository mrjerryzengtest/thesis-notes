# 15 — Entropy ↔ Cybersecurity 關聯文獻搜尋

> 搜尋日期：2026-07-14
> 目的：建立 entropy 作為網路安全指標的理論基礎，證明「為什麼 entropy 能偵測攻擊」

---

## 一、搜尋策略

| 層次 | 搜尋方向 | 關鍵詞 |
|------|---------|--------|
| 理論基礎 | 資訊理論 → 網路流量分析 | Shannon entropy network traffic |
| 經典實證 | entropy 用於異常偵測的開創性論文 | entropy anomaly detection seminal |
| SDN 應用 | entropy 在 SDN 中的 DDoS 偵測 | entropy DDoS detection SDN |
| 綜合回顧 | survey / review 論文 | entropy DDoS SDN comprehensive review |

---

## 二、經典奠基論文（Foundational）

### 🔑 最關鍵的 5 篇

#### 1. Feinstein et al. (2003)
> **"Statistical Approaches to DDoS Attack Detection and Response"**
> *DARPA Information Survivability Conference and Exposition*

- **核心貢獻**：最早將 entropy 用於 DDoS 偵測的文獻之一。計算 source IP 的 entropy，當 DDoS 發生時大量封包來自少數 IP → entropy 驟降。
- **熵值角色**：檢測指標 — entropy 下降 = 攻擊訊號
- **URL**：https://ieeexplore.ieee.org/abstract/document/1194894

#### 2. Lakhina, Crovella & Diot (2004)
> **"Structural Analysis of Network Traffic Flows"**
> *SIGMETRICS Performance Evaluation Review*

- **核心貢獻**：提出用 entropy 分析網路流量結構。證明不同 traffic feature（srcIP, dstIP, srcPort, dstPort）的 entropy 能捕捉 OD flow 的潛在結構，異常流量會破壞這個結構。
- **熵值角色**：結構描述子 — entropy 反映流量分佈的「形狀」
- **URL**：https://dl.acm.org/doi/10.1145/1005686.1005697

#### 3. Lakhina, Crovella & Diot (2005)
> **"Diagnosing Network-Wide Traffic Anomalies"**
> *SIGCOMM 2005*

- **核心貢獻**：將 entropy-based anomaly detection 擴展到全網路規模。提出用 PCA 對多維 entropy 時間序列做降維，分離正常與異常子空間，實現 network-wide 異常診斷。
- **熵值角色**：多維特徵向量 — 每個 OD flow 的 entropy 變化在 PCA 空間中形成異常軌跡
- **引用數**：~1,400+（SIGCOMM 經典）
- **URL**：https://dl.acm.org/doi/10.1145/1080091.1080133

#### 4. Wagner & Plattner (2005)
> **"Entropy Based Worm and Anomaly Detection in Fast IP Networks"**
> *IEEE WETICE 2005*

- **核心貢獻**：用 destination port 的 entropy 偵測蠕蟲擴散。蠕蟲掃描會使 dstPort entropy 異常升高（大量隨機 port），而 DDoS 則使 dstIP entropy 下降（集中在少數 victim）。
- **熵值角色**：攻擊分類器 — 不同攻擊類型產生不同 entropy 變化模式
- **URL**：https://ieeexplore.ieee.org/abstract/document/1524324

#### 5. Nychis et al. (2008)
> **"An Empirical Evaluation of Entropy-based Traffic Anomaly Detection"**
> *ACM IMC 2008*

- **核心貢獻**：對 entropy-based anomaly detection 做全面的經驗評估。發現：(1) entropy 在大多數異常類型中表現良好；(2) 但某些攻擊只改變流量體積而不改變分佈形狀，entropy 可能漏報；(3) 需要結合多個 feature 的 entropy。
- **熵值角色**：盲點分析 — entropy 不是萬能的，需要多維度互補
- **URL**：https://dl.acm.org/doi/10.1145/1452520.1452550

---

## 三、SDN 特定文獻

### 6. Mousavi & St-Hilaire (2015)
> **"Early Detection of DDoS Attacks Against SDN Controllers"**
> *IEEE ICC 2015*

- **核心貢獻**：將 entropy-based 方法移植到 SDN。在 OpenFlow switch 上計算 dstIP entropy 的 sliding window，當 entropy 低於動態閾值時觸發警報。
- **熵值角色**：SDN 控制層保護
- **URL**：https://ieeexplore.ieee.org/abstract/document/7249107

### 7. Giotis, Argyropoulos, Androulidakis, Kalogeras & Maglaris (2014)
> **"Combining OpenFlow and sFlow for an Effective and Scalable Anomaly Detection and Mitigation Mechanism on SDN Environments"**
> *Computer Networks*

- **核心貢獻**：結合 OpenFlow 統計 + sFlow 做 entropy-based anomaly detection，在 SDN 環境中實現 scalable 的攻擊偵測與緩解。
- **URL**：https://www.sciencedirect.com/science/article/pii/S1389128614000176

### 8. Kalkan, Zeadally & Al-Bayatti (2018)
> **"A Novel Renyi Entropy-Based DDoS Detection and Mitigation Scheme for SDN"**
> *IEEE Systems Journal*

- **核心貢獻**：用 Rényi entropy（Shannon entropy 的廣義化，參數 α 可調）取代 Shannon entropy，提高對不同攻擊模式的適應性。
- **URL**：https://ieeexplore.ieee.org/abstract/document/8401538

---

## 四、Survey / Review 論文（最佳出發點）

### 9. "Entropy Based DDoS Detection and Mitigation Methods in SDN: A Comprehensive Review"
> 2023–2024 年發表，系統性回顧 SDN 中所有 entropy-based DDoS 偵測方法

- **價值**：提供完整的 method taxonomy（Shannon / Rényi / Tsallis / Generalized entropy 比較表）
- **URL**：搜尋 Google Scholar `"Entropy Based DDoS Detection and Mitigation Methods in SDN"`

### 10. Fernandes et al. (2019)
> **"A Comprehensive Survey on Network Anomaly Detection"**
> *Telecommunication Systems*

- **價值**：將 entropy-based 方法放在整體 network anomaly detection 框架中比較
- **URL**：https://link.springer.com/article/10.1007/s11235-018-0475-8

---

## 五、Entropy ↔ Cybersecurity 邏輯鏈

用國中生也能懂的比喻說明：

### 類比：教室裡的噪音

```
正常上課 → 每個人小聲討論 → 聲音來源分散 → Entropy 高（混亂但正常）
老師宣布考試 → 全班同時驚呼 → 聲音集中在一個事件 → Entropy 驟降（異常！）
火災警報 → 所有人往門口跑 → 動作集中在一個方向 → Entropy 驟降（異常！）
有人鬧事 → 某個人突然大吼 → 音量集中在一個人 → Entropy 驟降（異常！）
```

### 對應到網路：

| 教室情境 | 網路情境 | Entropy 變化 |
|---------|---------|:-----------:|
| 正常討論 | 正常流量（IP 分散） | 高 entropy |
| 全班驚呼 | DDoS 攻擊（集中打同一 target） | dstIP entropy ↓ |
| 火災逃跑 | Port scan（大量隨機 port） | dstPort entropy ↑ |
| 有人鬧事 | Botnet C&C（少數 IP 大量流量） | srcIP entropy ↓ |

### 邏輯鏈（可用於論文 Chapter 2）：

```
Shannon 資訊理論（1948）
  └→ Entropy 度量隨機性/不確定性
      └→ 網路流量可視為機率分佈（srcIP, dstIP, port...）
          └→ 正常流量：分佈穩定，entropy 在可預測範圍
          └→ 攻擊流量：分佈偏離，entropy 出現顯著變化
              └→ 不同攻擊類型產生不同 entropy 變化模式（Wagner 2005）
                  └→ 多維度 entropy 可分類攻擊（Lakhina 2005）
                      └→ SDN 控制層可即時讀取 switch 統計計算 entropy（Mousavi 2015）
```

---

## 六、下一步建議

1. **先讀 Survey**：從 #9（SDN entropy DDoS review）和 #10（general anomaly detection survey）入手，快速建立全景
2. **再讀基石**：#2–#5 是 entropy-for-security 的經典，引用數高、方法被大量引用
3. **SDN 應用**：#6–#8 展示如何在 SDN 中落地
4. **整合到 Chapter 2**：以上邏輯鏈即為文獻回顧章的 backbone

---

## 相關搜尋關鍵詞（供後續擴充）

- `"entropy based" "DDoS detection" "SDN" survey`
- `"Shannon entropy" "network anomaly" Lakhina`
- `"Renyi entropy" OR "Tsallis entropy" DDoS SDN`
- `"generalized entropy" network attack detection`
- `"information theoretic" network security anomaly`
