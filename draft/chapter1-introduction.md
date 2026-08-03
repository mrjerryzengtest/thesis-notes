# 第一章 緒論

## 1.1 研究背景

軟體定義網路（Software-Defined Networking, SDN）將控制邏輯集中到控制器，交換機只負責根據規則轉送封包。這個架構讓網路行為變得可程式化，但也讓攻擊的影響更集中——控制器本身就是高價值目標，而交換機有限的 flow table 容量容易被攻擊者利用。

網路安全領域一直有個核心難題：要在高速流量中即時發現攻擊，又不能花太多資源在監控本身。傳統的深度封包檢測（DPI）逐封包檢查、逐特徵比對，在 SDN 的線速環境中成本太高。機器學習方法能自動學習攻擊模式，但需要大量標記資料，且黑箱決策難以在資安場景中被信任——管理者需要知道「為什麼判定這是攻擊」。

網路量測領域有一類輕量級資料結構，稱為 Sketch。Sketch 以犧牲微小精確度為代價，將高速封包流壓縮成極小的統計摘要。其中兩種 Sketch 特別值得關注：Count-Min Sketch（Cormode & Muthukrishnan, 2005）是最基礎的頻率估計結構，以極簡的計數器陣列追蹤 flow 的封包數量分佈；Elastic Sketch（Yang et al., SIGCOMM 2018）在 CM Sketch 的基礎上增加了 heavy/light 分離機制，能同時產出 heavy hitter 清單、流量分佈形狀、以及熵值。這兩種 Sketch 的量測能力互補——CM Sketch 給粗粒度的頻率快照，Elastic Sketch 補上結構性的 heavy/light 資訊。

一個自然的問題出現了：如果把這兩種互補的 Sketch 放在一起運行，把各自的量測結果合起來看，能不能比單用其中任何一個更有效地識別攻擊類型？

## 1.2 問題陳述

現有的 Sketch-based 攻擊檢測方法幾乎都使用**單一 Sketch**：有人用 CM Sketch 追蹤 flow 大小變化來抓 DDoS，有人用 Elastic Sketch 的熵值來判斷流量是否異常。這些方法能回答「有沒有攻擊」，但很難回答「是什麼攻擊」。

問題的本質不是「哪個 Sketch 不夠好」，而是**單一 Sketch 的測量能力有限**。Elastic Sketch 擅長追蹤 elephant flow 和計算流量分佈，但要區分 Port Scan（大量短 flow）和 DDoS（少數 heavy flow），光靠分佈資訊不夠——你需要同時知道頻率分佈的形狀（CM Sketch 的強項）。CM Sketch 能捕捉頻率異常，但無法區分「流量集中在少數 flow」和「流量均勻分散」——這兩種模式對應完全不同的攻擊類型，而這正是 Elastic Sketch 能提供的結構性資訊。

更具體地說，兩個問題尚未被解決：

1. **互補性未被利用。** CM Sketch 和 Elastic Sketch 的量測能力是互補的——前者給頻率分佈的粗粒度快照，後者補上 heavy hitter 和熵值的結構性資訊。目前沒有人嘗試把兩者的輸出合在一起做攻擊檢測，更沒有人量化過「合起來看比單獨看強多少」。

2. **缺乏可解釋的分類邏輯。** 即使有了多個 Sketch 的特徵，還需要一套能告訴管理者「為什麼判定這是 DDoS 而不是 Port Scan」的決策規則。這套規則必須直觀到可以逐條驗證——「因為 heavy hitter ratio 偏高但 cardinality 正常，所以不是 Port Scan」。

## 1.3 研究動機與目標

本研究嘗試建立一個**雙 Sketch 協同**的攻擊檢測框架，選用互補性最強的兩種通用 Sketch——CM Sketch 和 Elastic Sketch——在 SDN 環境中並行部署，將兩者的輸出融合為聯合特徵向量，用於攻擊檢測與分類。

這個方向受到兩個觀察的啟發。第一，教授在方向討論中指出：「針對熵值其實也沒什麼好優化的。應該是找一些通用的網路測量 sketch，來去針對不同的攻擊做識別。」重點不在於改良某個 Sketch 的計算精度，而在於善用不同 Sketch 之間的量測互補性。第二，文獻中反覆出現一個訊號：不同攻擊需要不同的特徵才能有效區分——Nychis et al. (2008) 證明單一維度熵值無法覆蓋所有異常類型，FP-Sketch (2025) 顯示不同流量模式需要不同設計策略的 Sketch。

本研究具體目標：

1. **建立 CM + Elastic 雙 Sketch 協同架構**：在 SDN 交換機上同時部署 CM Sketch 和 Elastic Sketch，每個封包同時更新兩個結構，每五秒從兩者中擷取互補特徵。

2. **設計聯合特徵向量與分類規則**：將 CM Sketch 的頻率分佈統計與 Elastic Sketch 的熵值、heavy hitter 資訊、cardinality 整合為一個聯合特徵向量，基於不同攻擊在此向量空間中的偏移模式建立可解釋的分類規則。

3. **實驗驗證協同效果**：在 Mininet 模擬環境中，比較 CM-only、Elastic-only、以及 CM+Elastic 三種配置在 DDoS 和 Port Scan 檢測與分類上的效能差異，並透過消融實驗量化每個 Sketch 的貢獻。

## 1.4 研究貢獻

1. **雙 Sketch 協同檢測框架**：提出一個具體的、可在 SDN 環境中實作的雙 Sketch 協同方法，證明 CM Sketch 和 Elastic Sketch 的量測輸出具有互補性，合併使用能顯著提升攻擊檢測與分類能力。

2. **可解釋的攻擊分類邏輯**：基於聯合特徵向量中各維度的物理意義（頻率分佈形狀、heavy hitter 佔比、cardinality 變化等），設計直觀可驗證的分類規則——管理者可以清楚看到每一個判斷的依據。

3. **量化的消融分析**：透過逐一移除 Sketch 的實驗，量化 CM Sketch 和 Elastic Sketch 各自對 DDoS 和 Port Scan 檢測的邊際貢獻，為後續研究提供「哪種攻擊最需要哪種 Sketch 特徵」的實證參考。

## 1.5 論文架構

第二章回顧三個領域的文獻：SDN 攻擊檢測方法的演進、CM Sketch 與 Elastic Sketch 的設計原理與適用場景、以及多特徵融合在攻擊分類中的應用。

第三章描述雙 Sketch 協同框架的設計：並行部署架構、聯合特徵向量的組成（11 維）、以及基於聯合特徵的兩階段分類邏輯。

第四章為實驗設計與結果分析，在 Mininet 模擬的 SDN 環境中比較 CM-only、Elastic-only、CM+Elastic 三種配置在 DDoS 和 Port Scan 場景下的檢測與分類效能。

第五章總結研究成果，討論框架在更多 Sketch 種類和更多攻擊類型上的可擴充性，以及未來可延伸的方向。
