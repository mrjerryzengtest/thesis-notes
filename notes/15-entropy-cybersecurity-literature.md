# 15 — 為什麼 Entropy 能抓網路攻擊？（白話版）

> 給教授看的正式版在 `notes/12`，這份是給你自己看懂用的。

---

## 0. 先說結論：Entropy 為什麼能抓攻擊？

用教室比喻：

```
正常上課 → 每個人小聲聊天 → 聲音來源很分散 → Entropy 高（亂，但正常）
老師突襲考試 → 全班同時哀號 → 聲音集中在同一件事 → Entropy 暴跌（異常！）
火災警報響 → 所有人往門口擠 → 動作集中在同一方向 → Entropy 暴跌（異常！）
```

網路也一樣：
- **正常時**：流量來自四面八方、去四面八方 → entropy 穩定在一個範圍
- **被 DDoS**：所有流量打同一個目標 → dstIP entropy 暴跌
- **被掃 port**：一個 IP 到處敲門 → dstPort entropy 暴漲

**一句話：攻擊會改變流量分佈的「亂度」，entropy 就是測量這個亂度的尺。**

---

## 1. 這五篇是地基（經典中的經典）

### 論文 1：Feinstein (2003) — 最早用 entropy 抓 DDoS 的人

> 就像第一個發現「教室突然安靜 = 要出事了」的人

**做什麼**：算 source IP 的 entropy。正常時每個人都在講話（IP 分散），entropy 高。DDoS 發生時一堆假 IP 打同一個目標，來源突然變得很單一 → entropy 驟降。

**對你論文的重要性**：⭐⭐⭐ — 證明「entropy 降 = 攻擊」這個概念是可行的，你是站在巨人的肩膀上。


### 論文 2：Lakhina (2004) — entropy 不只一個，要看好幾個

> 不只聽「誰在講話」，還要聽「在跟誰講話」、「從哪個門進來」

**做什麼**：第一次證明不同維度的 entropy 各有用途：
- srcIP entropy → 看出誰在發動攻擊
- dstIP entropy → 看出誰在被攻擊  
- srcPort entropy → 看出攻擊者用什麼手法
- dstPort entropy → 看出目標是哪種服務

**對你論文的重要性**：⭐⭐⭐⭐⭐ — 這是你論文「多維度熵值」的直接祖宗。


### 論文 3：Lakhina (2005) — SIGCOMM 神級論文（引用 1400+）

> 同時看好幾個 entropy 指標，像看股票大盤一樣，正常波動沒事，異常一起飆就知道出事了

**做什麼**：把多個 entropy 數值同時監控，用 PCA（一種數學技巧）自動分開「正常波動」和「真正異常」。不用人盯著看，機器自己判斷。

**對你論文的重要性**：⭐⭐⭐⭐⭐ — 展示「多維度 + 自動判斷」可行。你論文要做的 attack classification 就是這個的延伸。


### 論文 4：Wagner (2005) — 不同攻擊，entropy 變化不一樣！

> DDoS 像全班安靜（entropy 降）、蠕蟲擴散像全班亂叫（entropy 升）

**做什麼**：發現不同攻擊類型會產生不同的 entropy 變化模式：
- **DDoS 打同一個目標** → dstIP entropy 下降（集中在少數 victim）
- **蠕蟲掃描** → dstPort entropy 上升（大量隨機 port 被敲門）

**對你論文的重要性**：⭐⭐⭐⭐⭐ — 這是你論文「攻擊分類」的核心依據：不同攻擊 → 不同熵值模式 → 可以分類！


### 論文 5：Nychis (2008) — entropy 的盲點

> 有時候教室看起來正常（聲音分散），但其實大家都在傳紙條作弊。光聽聲音不夠。

**做什麼**：誠實檢討 entropy 的弱點：
- 大部分異常抓得到 ✅
- 但某些攻擊只改變「流量大小」不改變「分佈形狀」→ entropy 看不出來 ❌
- 解法：**不能只用一個 entropy，要結合多個維度 + 其他指標**

**對你論文的重要性**：⭐⭐⭐⭐ — 告訴你 entropy 不是萬能的，你的論文要說明覆蓋範圍和限制。


## 2. 這三篇是把熵值搬進 SDN

### 論文 6：Mousavi (2015) — 第一次在 SDN 交換機上算 entropy

> 把「聽教室聲音」這件事，直接裝在交換機（郵差）身上

**做什麼**：在 OpenFlow 交換機上實作 entropy 計算，用 sliding window（滑動窗口，看最近幾秒的流量），entropy 低於門檻就發警報。

**對你論文的重要性**：⭐⭐⭐⭐ — 證明「SDN 交換機上算 entropy」技術上可行，你的系統架構可以直接參考。


### 論文 7：Giotis (2014) — SDN 裡大規模抓攻擊

> 不只一間教室，整棟樓都在監控

**做什麼**：結合 OpenFlow + sFlow 兩種資料來源，在 SDN 環境做大規模 entropy-based 攻擊偵測。證明這個方法可以 scale（擴展到整棟大樓）。

**對你論文的重要性**：⭐⭐⭐ — 補充 scalability 的證據。


### 論文 8：Kalkan (2018) — 用 Rényi entropy 取代 Shannon entropy

> Shannon entropy 是一把固定刻度的尺，Rényi entropy 是可以調刻度的尺

**做什麼**：用 Rényi entropy（Shannon 的廣義版，多一個可調參數 α）取代傳統 Shannon entropy。不同 α 值對不同攻擊敏感度不同，可以調參數適應不同場景。

**對你論文的重要性**：⭐⭐⭐ — 你的「多維度熵值」如果想擴充，可以考慮不只 Shannon，也加入 Rényi。


## 3. 這兩篇幫你快速入門（Survey）

### 論文 9：SDN Entropy DDoS 總回顧（2023-2024）

> 有人幫你把所有相關論文整理成一張大地圖

直接搜 `"Entropy Based DDoS Detection and Mitigation Methods in SDN"`，裡面有所有 SDN + entropy 方法的比較表。讀這一篇抵十篇。

### 論文 10：Fernandes (2019) — 網路異常偵測大全

> 不只 entropy，所有抓攻擊的方法都在這

把 entropy-based 方法放在整個 anomaly detection 家族中比較，讓你論文 Chapter 2 可以說「為什麼選 entropy 而不是其他方法」。


## 4. 整體邏輯鏈（你論文 Chapter 2 的骨架）

```
Shannon 提出「資訊量可以量化」（1948）
  └→ entropy = 測量亂度的數字
      └→ 網路流量可以看成機率分佈（誰在傳、傳給誰、從哪個門）
          └→ 正常流量：分佈穩定，entropy 在可預測範圍
          └→ 攻擊流量：分佈被破壞，entropy 出現異狀
              └→ 不同攻擊破壞的方式不同（Wagner 2005）
                  └→ 所以看多個 entropy 可以分類攻擊類型（Lakhina 2005）
                      └→ SDN 交換機可以直接算（Mousavi 2015）
                          └→ 你的論文：組合多種 Sketch，在 SDN 中同時算多個 entropy，精確分類攻擊
```


## 5. 建議閱讀順序

| 順序 | 論文 | 理由 |
|:--:|------|------|
| ① | 論文 9（Survey） | 先看大地圖，知道有哪些路線 |
| ② | 論文 4（Wagner） | 最直覺：不同攻擊 → 不同 entropy 模式 |
| ③ | 論文 2+3（Lakhina x2） | 核心：多維度 + 自動判斷 |
| ④ | 論文 1（Feinstein） | 歷史根源，知道這條路怎麼開始的 |
| ⑤ | 論文 5（Nychis） | 認識盲點，論文要提限制 |
| ⑥ | 論文 6（Mousavi） | SDN 實作藍圖 |
| ⑦ | 論文 8（Kalkan） | 進階選配：Rényi entropy |


## 6. 關鍵搜尋詞（以後要擴充文獻用）

- `"entropy based" "DDoS detection" "SDN" survey`
- `"Shannon entropy" "network anomaly" Lakhina`
- `"Renyi entropy" OR "Tsallis entropy" DDoS SDN`
- `"information theoretic" network security anomaly`
