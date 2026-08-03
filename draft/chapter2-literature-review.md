# 第二章 文獻探討

本章從三個軸線回顧相關文獻：SDN 安全威脅與現有檢測方法的優劣、通用網路量測 Sketch 的設計原理與適用場景、以及多特徵融合在攻擊分類中的應用。三條軸線的交會點，就是本研究提出的多 Sketch 協同框架的理論基礎。

---

## 2.1 SDN 攻擊與檢測方法

### 2.1.1 SDN 的安全威脅

SDN 將控制平面集中到控制器，交換機僅負責資料轉送。Kreutz et al.（IEEE S&P, 2015）系統分析了 SDN 的七大攻擊向量，其中三類對本研究的方法設計有直接影響：

**Flow Table 溢位攻擊**：SDN 交換機使用容量有限的 TCAM 儲存 flow entry（通常數千到數萬條）。攻擊者高速產生大量短命 flow，迫使交換機不斷安裝新規則直到 TCAM 耗盡。特徵是 flow 數量暴增、每個 flow 封包數極少、flow 存活時間短。從量測角度看，這類攻擊會大幅改變 flow 大小分佈和 cardinality。

**控制層飽和攻擊**：攻擊者發送大量無法匹配既有規則的封包，產生密集的 Packet-In 請求耗盡控制器資源。與 flow table 溢位不同，這類攻擊的關鍵特徵是 Packet-In 速率異常和來源集中度——攻擊流量通常來自少數交換機和攻擊主機。

**拓撲毒化攻擊**：攻擊者偽造 LLDP 封包欺騙控制器，建立錯誤的網路拓撲。流量層面上主要表現為特定協定的異常使用模式。

一個重要的觀察是：這三種攻擊在流量層面上「看起來完全不一樣」——它們改變的是不同的統計維度。這意味著檢測系統需要能同時捕捉多種流量特徵，而非只依賴單一信號。

### 2.1.2 現有檢測方法

基於規則的 IDS（如 Snort）依賴已知攻擊特徵庫進行比對，對未知攻擊無效。基於機器學習的方法（Tang et al., 2016; Niyaz et al., 2016）從流量統計中學習正常與異常的邊界，但需要大量標記資料且模型缺乏可解釋性。

基於 Sketch 的方法走第三條路。Sketch 是一種機率式摘要結構，以可控的誤差換取極低的記憶體和計算成本。核心優勢是：不需要儲存每個 flow 的詳細資訊，只需要一個遠小於 flow 數量的固定大小計數器陣列，就能回答關於流量分佈的查詢。這使得 Sketch 特別適合部署在資源受限的交換機上。

目前文獻中的 Sketch-based 攻擊檢測多使用單一 Sketch。Mousavi & St-Hilaire（2015）在 SDN 控制器上使用目的地 IP 的熵值來檢測 DDoS。Giotis et al.（2014）結合 sFlow 取樣和 flow 統計進行聯合判斷。這些方法能檢測攻擊的存在，但不具備分類能力。其根本限制不在於 Sketch 的精確度，而在於單一 Sketch 只能提供有限視角的流量摘要——你需要多個視角才能看出攻擊的全貌。

---

## 2.2 通用網路量測 Sketch

以下逐一分析本框架中納入的四種 Sketch。每種 Sketch 的設計目標不同，因此它們捕捉的流量特徵互補。

### 2.2.1 Count-Min Sketch

Count-Min Sketch（Cormode & Muthukrishnan, 2005）是最基礎的 Sketch 結構。它使用一個 $d \times w$ 的二維計數器陣列，每個封包經過 $d$ 個獨立的 hash 函數對應到不同列的計數器並遞增。查詢某個 flow 的大小時，取對應的 $d$ 個計數器中的最小值。

CM Sketch 的核心性質是**只會高估，不會低估**。這項單邊誤差保證使其在頻率估計任務中表現可靠。在本框架中，CM Sketch 負責提供**頻率分佈**特徵：哪些 flow 的頻率相對於正常基準出現異常變化。

CM Sketch 的限制是對 heavy hitter 不敏感——當少數 flow 佔據大量封包時，CM Sketch 的計數器碰撞會加劇，頻率估計誤差增大。這正是為什麼需要 Elastic Sketch 來互補。

### 2.2.2 Elastic Sketch

Elastic Sketch（Yang et al., SIGCOMM 2018）在 CM Sketch 基礎上引入兩層分離架構：heavy part 以 hash table 精確追蹤大象流（elephant flow），light part 以 CM Sketch 壓縮儲存老鼠流（mouse flow）。分離的邊界由自適應 λ 閾值控制，能根據當前流量狀況動態調整。

這個結構讓 Elastic Sketch 同時產出兩種資訊：heavy part 給出**heavy hitter 清單與精確流量**，light part 的計數器分佈可計算**熵值和 cardinality**。更重要的是，同一個資料結構可以支援六種查詢類型（flow size、heavy hitter、heavy change、分佈、熵值、cardinality）——這是「單結構多任務」的特性，並非本框架的核心依賴。

在本框架中，Elastic Sketch 負責提供與流量**結構**相關的特徵：哪些 flow 是 elephant、flow 大小分佈的形狀、以及分佈的熵值。這些資訊對區分 DDoS（少數 heavy flow）和 Port Scan（大量 light flow）特別有用。

### 2.2.3 UnivMon

UnivMon（Liu et al., SIGCOMM 2016）的設計理念與前兩者不同。它不是針對特定查詢類型最佳化，而是從一個通用摘要結構中**派生**出多種查詢的答案。UnivMon 使用一個精簡的 Sketch 結構記錄封包，在查詢階段透過後處理邏輯來回答 heavy hitter、entropy、cardinality、分位數等不同問題。

UnivMon 的獨特價值在於它的**通用性**：同一個資料結構可以在事後靈活回答多種不同的查詢，而不需要在部署時就決定要量測什麼。這對於攻擊檢測場景有實務意義——你可能在部署時還不知道未來會出現什麼新攻擊，但 UnivMon 記錄的通用摘要讓你在發現新攻擊模式後仍能回頭查詢相關特徵。

在本框架中，UnivMon 作為通用特徵提取器，提供 heavy hitter 變化趨勢和分位數資訊，補充 CM Sketch 和 Elastic Sketch 的覆蓋範圍。

### 2.2.4 UCL-Sketch

UCL-Sketch（Yuan et al., IEEE TKDE 2026）是最近提出的學習型頻率估計 Sketch。它不依賴 ground truth 進行訓練，而是透過壓縮感知技術和邏輯結構化的估計桶，在極小的記憶體預算下實現高精度頻率估計。相比傳統的方程式型 Sketch，解碼速度快近 500 倍。

UCL-Sketch 對本框架的價值在於其**高精度低記憶體**的特性：在相同記憶體預算下，UCL-Sketch 的頻率估計精度顯著優於 CM Sketch。這意味著在資源極度受限的交換機上，UCL-Sketch 可以在不犧牲精度的前提下壓縮更多維度的頻率資訊。

### 2.2.5 四種 Sketch 的互補性總結

| Sketch | 主要輸出 | 對攻擊檢測的價值 |
|--------|---------|-----------------|
| CM Sketch | 頻率分佈 | 偵測 flow 頻率的異常變化（基礎指標） |
| Elastic Sketch | heavy hitter + 熵值 + cardinality | 判斷攻擊是否改變流量結構（elephant/mouse 分佈） |
| UnivMon | 通用摘要（多查詢） | 提供靈活的後續查詢能力，應對未知攻擊模式 |
| UCL-Sketch | 高精度頻率估計 | 在資源受限下壓縮更多維度，提升頻率估計精度 |

這四種 Sketch 不是競爭關係，而是**分工關係**。CM Sketch 給出頻率分佈的粗粒度快照，Elastic Sketch 補充結構性的 heavy/light 分離資訊，UnivMon 提供查詢靈活性，UCL-Sketch 在記憶體受限時提供精度保障。把它們的特徵合在一起看，能得到的資訊遠多於任何單一 Sketch。

---

## 2.3 多特徵融合與攻擊分類

### 2.3.1 熵值與網路異常

熵值在網路異常檢測中的應用始於 Lakhina et al.（SIGCOMM 2005）。他們將來源 IP 和目的地 IP 的熵值時間序列輸入 PCA，成功檢測了多種異常。Nychis et al.（IMC 2008）進一步證明不同流量特徵的熵值在不同異常類型上的表現差異很大——沒有任何單一特徵能涵蓋所有異常。

這兩篇經典研究的共同結論指向同一個方向：需要多個特徵聯合判斷。Lakhina 用的是 PCA 融合多個熵值序列，Nychis 用的是並行比較多個維度的檢測結果。本研究的聯合特徵向量方法延續了這個思路，但擴大了特徵來源——不只是多個熵值維度，還包括頻率、cardinality、heavy hitter 等異質特徵。

### 2.3.2 SDN 中的 Sketch 部署

將 Sketch 部署到 SDN 環境中有兩種路徑。一是在交換機端（資料平面）部署，利用 P4 可程式化 ASIC 的 match-action pipeline 實現線速的 Sketch 更新——Lai et al.（EuroP4 2022）在 Barefoot Tofino2 上實現了 400 Gbps 的 Sketch 熵值估計。二是在控制器端集中部署，透過 OpenFlow 的 flow stats 定期收集統計資料。

兩種路徑各有取捨：資料平面部署延遲低但硬體需求高，控制器部署靈活但受限於 Packet-In 頻寬。本研究的框架在設計上相容兩種部署模式，第四章的實驗以控制器端部署為主。

### 2.3.3 攻擊分類中的多特徵方法

Wang et al.（2019）嘗試使用三維熵值（來源 IP、目的地 IP、埠號）進行聯合檢測，但缺乏系統性的分類規則和 Sketch 選擇策略。Zhang et al.（2020）使用 SVM 處理多維度熵值特徵進行分類，但特徵來源仍然是單一資料結構（NetFlow 統計），而非多種 Sketch 的異質輸出。

本研究與這些工作的關鍵差異在於：特徵不是來自同一個量測工具的「多個維度」，而是來自**多個不同量測工具**的「不同類型的量測」。CM Sketch 的頻率分佈和 Elastic Sketch 的 heavy hitter 清單是兩種性質完全不同的資訊——前者是統計摘要，後者是精確的 flow 層級紀錄。這種異質性帶來的互補資訊量，是同質多維度方法無法提供的。

---

## 2.4 本章總結

從文獻回顧中，三個層次的結論逐漸收斂：

1. SDN 攻擊會同時改變多個流量統計維度，但現有檢測方法多依賴單一 Sketch 提供的有限視角。

2. 現有的通用網路量測 Sketch（CM Sketch、Elastic Sketch、UnivMon、UCL-Sketch）各自專精於不同的量測任務，它們之間存在自然的互補關係。

3. 多特徵融合方法在攻擊分類中有潛力，但目前缺乏一個系統性的框架來「選擇哪些 Sketch、產出哪些特徵、如何融合做決策」。

本研究的框架針對第三點提出設計方案。第三章將詳細描述這個多 Sketch 協同框架的架構與運作邏輯。
