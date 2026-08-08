# Mesh 網路技術簡報（軍事應用導向）

---

## 一、名詞解釋

| 名詞 | 定義 |
|------|------|
| **Mesh Network** | 網狀網路。每個節點同時是終端也是中繼，拓撲為 partial/full mesh，具自組網、自癒、多跳路由三特性 |
| **Multi-hop Routing** | 多跳路由。封包經多個中間節點轉送到目的地，每跳增加延遲但延伸覆蓋範圍 |
| **Self-Organizing Network (SON)** | 自組網。節點通電後自動發現鄰居、交換路由表、建立拓撲，無需人工設定 |
| **Self-Healing** | 自癒。任一節點失效或鏈路中斷時，網路自動重新計算路由，繞開故障點 |
| **Ad-hoc Network** | 隨建即連網路。無固定基礎設施，節點動態加入/離開，拓撲持續變化 |
| **MANET** | Mobile Ad-hoc Network，移動中的 Ad-hoc 網路，詳見第三節 |
| **Proactive Routing** | 主動式路由（如 OLSR）。定期廣播路由表，延遲低但 overhead 高 |
| **Reactive Routing** | 反應式路由（如 AODV）。有通訊需求才找路，省頻寬但初始延遲高 |
| **Throughput Degradation** | 吞吐量衰減。每多一跳，有效吞吐量約減半（同頻道收發共用），3 跳以上實用性銳減 |

---

## 二、現有 Mesh 系統

### 2.1 戰術無線電

| 系統 | 國別 | 頻段 | 特點 |
|------|:--:|------|------|
| **AN/PRC-117G (Falcon III)** | 美 | 30 MHz – 2 GHz | 寬頻 MANET，SINCGARS 相容，自動中繼 |
| **AN/PRC-163 (Falcon IV)** | 美 | HF/VHF/UHF | 雙通道同時語音+數據，TSM 波形，上傳至營級 |
| **PR4G (Thales)** | 法 | VHF 30-88 MHz | 跳頻抗干擾，內建 2 跳中繼 |
| **CNR-900 (R&S)** | 德 | VHF/UHF | IP-based MANET，最高 64 節點自組網 |

**TSM (Tactical Scalable MANET) 波形** — TrellisWare 開發，美軍 Nett Warrior 標準波形，支援數百節點、每跳 <1ms 延遲、高抗干擾。

### 2.2 Mesh Wi-Fi 系統

| 系統 | 標準 | 特點 |
|------|------|------|
| **IEEE 802.11s** | WiFi Mesh 標準 | HWMP (Hybrid Wireless Mesh Protocol)，預設路由協定 |
| **GoTenna** | 非 WiFi | VHF 151-154 MHz，點對點最遠 6-8 km，文字+GPS，無需基地台 |
| **Silvus StreamCaster** | MIMO MANET | 軍用 COTS，400+ Mbps，Spectrum Dominance 波形 |

802.11s 核心採 HWMP：混合 AODV（on-demand）和 tree-based（proactive）路由，適應固定與移動節點混合場景。

### 2.3 Meshtastic

| 項目 | 規格 |
|------|------|
| **硬體** | LoRa SX1276/SX1262 晶片（Semtech） |
| **頻段** | 868 MHz (EU) / 915 MHz (US) / 470 MHz (CN) |
| **速率** | ~1 kbps（LoRa 長距模式的代價） |
| **範圍** | 空曠地 10-20 km 單跳，城市 1-3 km |
| **路由** | 泛洪式（flooding），非真正路由協議；新版加入 MQTT 閘道上雲 |

Meshtastic 的軍事價值不在直接採用，而在證明了 **LoRa PHY + Mesh topology = 極低成本的大範圍文字通訊網**。缺點是 flooding 路由在大規模時頻譜效率極差。

### 2.4 其他路由協議

| 系統 | 類型 | 要點 |
|------|------|------|
| **Zigbee** | IoT Mesh | 802.15.4，低功耗，Zigbee PRO 支援多跳 |
| **Thread** | IoT Mesh | Google/Nest 主導，6LoWPAN，每個裝置都是 router |
| **B.A.T.M.A.N.** | 路由協議 | Better Approach To Mobile Ad-hoc Networking，Linux kernel 內建 |
| **OLSR / OLSRv2** | 路由協議 | Optimized Link State Routing，MANET proactive 標準，RFC 7181 |

---

## 三、MANET（Mobile Ad-hoc Network）

### 3.1 定義

> MANET = 一群移動中的節點，不依賴任何固定基礎設施，各自獨立移動，透過無線鏈路動態形成臨時網路。每個節點同時是 host 和 router。

與固定 Mesh 的關鍵差異：**節點移動**導致拓撲持續變化、鏈路頻繁斷連重建、路由協議必須是反應式或低延遲主動式。

### 3.2 路由協議分類

| 類型 | 代表協議 | 機制 | 適用 |
|------|---------|------|------|
| **Proactive（主動式）** | OLSR, DSDV | 定期交換路由表，拓撲常保最新 | 節點數少、移動慢 |
| **Reactive（反應式）** | AODV, DSR | 有需求才發 RREQ 找路 | 節點多、移動快 |
| **Hybrid（混合式）** | ZRP, HWMP | 區域內 proactive、跨區 reactive | 大規模分層網路 |
| **Geographic（地理式）** | GPSR, LAR | 靠 GPS 座標決定下一跳，不維護路由表 | 高機動、大規模 |

### 3.3 核心挑戰

| 挑戰 | 說明 |
|------|------|
| **拓撲動態性** | 節點以 30-60 km/h 移動，鏈路平均存活 <10 秒 |
| **隱蔽終端問題** | A → B ← C，A 和 C 彼此聽不到但同時發給 B，碰撞 |
| **頻寬有限** | 戰術無線電通常 25 kHz – 1 MHz 窄頻，routing overhead 可能吃掉 30% 頻寬 |
| **能量受限** | 單兵攜行裝備電池 8-12 小時，proactive routing 持續耗電 |
| **安全** | 無中央權威，身分認證和金鑰分發困難；黑洞攻擊和蟲洞攻擊是經典威脅 |

---

## 四、作戰運用

### 4.1 戰術通訊層級對應

| 層級 | 通訊需求 | Mesh/MANET 角色 |
|------|---------|----------------|
| **班/排級** | 語音 + 位置（< 2 km） | 單兵無線電 Mesh，每員一個節點，自動中繼 |
| **連級** | 語音 + 數據 + 影片（< 10 km） | MANET 車載台，移動中保持連通 |
| **營級** | 態勢圖 + 火力協調（< 40 km） | Mesh 骨幹 + SATCOM 上行 |
| **旅級以上** | 大量數據 + 跨軍種 | Mesh 退為接取層，主幹走 SATCOM/光纖 |

### 4.2 可用的 Mesh 硬體（非傳統無線電）

| 類型 | 代表硬體 | 頻段/距離 | 軍事價值 |
|------|---------|----------|---------|
| **軍用無線電** | AN/PRC-163 (Falcon IV) | HF/VHF/UHF, 數 km | TSM 波形，數百節點 MANET，語音+數據，已實戰驗證 |
| **SDR** | Ettus USRP B210 | 70 MHz – 6 GHz, 視功率 | 自訂波形，同一硬體白天通訊、夜間切 SIGINT |
| **Wi-Fi 改裝** | OpenWrt + 802.11s | 2.4/5 GHz, 100-300m | $50 一顆，可拋棄式，固守陣地內部高頻寬覆蓋 |
| **LoRa 節點** | Heltec LoRa32 V3 | 868/915 MHz, 3-8 km | $30 一顆，文字+GPS，感測器網，佈了不管 |
| **手機** | goTenna | VHF 151-154 MHz, 6-8 km | 藍牙連手機，無需基地台，文字+GPS，特戰適用 |
| **UAV 中繼** | 繫留無人機 + Wi-Fi AP | 50-100m 高度 | 24h 不間斷，補地形遮擋，2D Mesh 變 3D |

### 4.3 作戰場景 × 硬體對應

| 場景 | 用什麼 | 為什麼 |
|------|--------|--------|
| **城鎮戰** | AN/PRC-163 + UAV 中繼 | 建築遮擋衛星，多跳繞障礙，空中有節點補死角 |
| **機動突擊** | AN/PRC-163（車載） | 車隊 >40 km/h 移動中 MANET，脫隊自動重組 |
| **特戰滲透** | goTenna | 低功率不易被偵測，文字+GPS 夠用，無需攜帶大型無線電 |
| **前進基地** | OpenWrt 802.11s | 便宜高頻寬，內部 VoIP + 影片，被炸不心疼 |
| **戰場感測器網** | LoRa32 V3 | 空投 50 顆沿補給線，車輛經過震動自動回傳，太陽能供電 |
| **反登陸/灘岸** | 全部混合 | 第一線預置 LoRa 感測器，二線 OpenWrt 固守，UAV 補地形，無線電保語音 |

### 4.4 美軍 Nett Warrior 案例

Nett Warrior 是美軍現役單兵系統，核心通訊層使用 TrellisWare TSM 波形：

- 每班 9-13 人各持 AN/PRC-163，自動形成 MANET
- 班長終端顯示全隊即時位置（無 GPS 時靠慣性導航 + Mesh 相對定位）
- 語音走 mesh 中繼，不需排級中繼台
- 失去任何 1-2 員不影響網路，自癒時間 <1 秒

### 4.5 反制作為考量

| 優勢 | 弱點 |
|------|------|
| 分散式架構，無單點失效 | Routing overhead 本身就是 RF 指紋 |
| 跳頻展頻抗窄頻干擾 | 全頻段攔截式干擾仍有效 |
| 多跳可降低每跳發射功率，不易被偵測 | 節點密度不足時（<3 鄰居），Mesh 退化為星狀 |
| 自癒繞路對抗局部干擾 | 黑洞攻擊可系統性破壞路由 |

---

## 五、總結

Mesh / MANET 的軍事價值不在於取代 SATCOM 或傳統戰鬥無線電，而在於填補兩者之間的空白：SATCOM 太高（延遲）、傳統無線電太僵（單點失效）、Mesh/MANET 剛好填了中距離高機動的抗毀通訊需求。

未來方向：低軌衛星（Starlink/OneWeb）已具備星間鏈路（ISL），本質上就是太空中的 Mesh。地面 MANET + 空中 Mesh（UAV）+ 太空 Mesh（LEO ISL）的三層整合，會是下一代戰術通訊的核心架構。
