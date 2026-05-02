# Mininet + ONOS/Ryu SDN 實驗環境搭建指南

> 建立日期：2026-05-02  
> 適用對象：碩士論文 — SDN 環境下基於多維度熵值分析之網路攻擊檢測  
> 目前主機規格：Ubuntu 26.04 LTS (VMware VM)、4 CPU、7.2 GB RAM、20 GB 磁碟

---

## 一、硬體與軟體需求總覽

### 最低需求（可運作基本拓樸）
| 項目 | 規格 |
|------|------|
| CPU | 2 core 以上 |
| RAM | 4 GB（Mininet）+ 2 GB（控制器） |
| 磁碟 | 15 GB 以上（含 Docker image） |
| OS | Ubuntu 20.04+ / Debian 11+（**強烈建議 Ubuntu**） |
| 網路 | 可連外（下載套件、Docker image） |

### 建議需求（論文實驗用）
| 項目 | 規格 |
|------|------|
| CPU | 4 core 以上 |
| RAM | 8 GB+（Mininet 模擬百台主機時記憶體用量大） |
| 磁碟 | 30 GB+（含實驗數據儲存） |
| 虛擬化 | VMware Workstation / VirtualBox / KVM |

### ⚠️ 目前你的主機情況
- **RAM 7.2 GB**：足夠執行中等規模拓樸（~50-100 nodes），但不建議同時跑 ONOS + Mininet 大規模壓力測試
- **磁碟僅剩 8.1 GB**：**空間吃緊**，建議先清理或擴充。ONOS Docker image 約 1 GB，加上 Mininet 相依套件，實驗數據很快就會爆

---

## 二、Mininet 安裝

Mininet 是 SDN 網路模擬器，使用 Linux network namespace 實現輕量級虛擬化。可在一台機器上模擬數百台主機與交換機（實測約 2 Gbps 總頻寬）。

### 選項 A：從原始碼安裝（推薦，最新版本）

```bash
# 1. 安裝相依套件
sudo apt update
sudo apt install -y git python3 python3-pip

# 2. 複製 Mininet 原始碼
git clone https://github.com/mininet/mininet.git
cd mininet

# 3. 選擇版本（建議最新穩定版）
git tag                          # 列出可用版本
git checkout -b mininet-2.3.0 2.3.0   # 或最新 tag

# 4. 安裝（三種層級可選）
# 層級 1：最小安裝（Mininet + User Switch + Open vSwitch）
sudo util/install.sh -nfv

# 層級 2：完整安裝（含 Wireshark dissector、POX 等）
sudo util/install.sh -a

# 5. 測試是否安裝成功
sudo mn --switch ovsbr --test pingall
# 預期輸出：所有 ping 都成功（0% dropped）
```

### 選項 B：用 apt 安裝（快速但版本較舊）

```bash
# Ubuntu 22.04+ / Debian 11+
sudo apt update
sudo apt install -y mininet

# 停掉內建 controller（我們要用 ONOS/Ryu）
sudo systemctl stop openvswitch-controller 2>/dev/null
sudo systemctl disable openvswitch-controller 2>/dev/null

# 測試
sudo mn --test pingall
```

### Mininet 基礎指令

```bash
# 啟動最簡單拓樸（1 switch + 2 hosts）
sudo mn

# 啟動自訂拓樸
sudo mn --topo single,4      # 1 switch + 4 hosts
sudo mn --topo linear,4      # 4 switches + 4 hosts（鏈狀）
sudo mn --topo tree,depth=3,fanout=3  # 樹狀拓樸

# Mininet CLI 常用指令
mininet> nodes          # 列出所有節點
mininet> net            # 顯示網路連接
mininet> h1 ping h2     # host 之間 ping
mininet> iperf h1 h2    # 頻寬測試
mininet> xterm h1 h2    # 打開 host terminal
mininet> exit           # 離開並清除
```

---

## 三、ONOS 控制器安裝

ONOS（Open Network Operating System）是電信級 SDN 控制器，支援 HA 叢集、REST API、Web UI。

### 選項 A：Docker 安裝（最簡單，建議）

```bash
# 1. 拉取 ONOS Docker image
docker pull onosproject/onos:latest

# 2. 啟動 ONOS 容器
docker run -t -d \
  --name onos \
  -p 8181:8181 \      # Web UI
  -p 8101:8101 \      # CLI (SSH)
  -p 6653:6653 \      # OpenFlow channel
  -p 6633:6633 \      # OpenFlow (legacy)
  onosproject/onos:latest

# 3. 確認 ONOS 運作中
docker logs -f onos    # 看到 "Started..." 即完成啟動

# 4. 存取 ONOS
# Web UI：http://localhost:8181/onos/ui
# 預設帳密：onos / rocks
```

### 選項 B：原始碼編譯（進階，需 Bazel）

```bash
# 需要 Java 17+ 和 Bazel 1.0.0+
git clone https://gerrit.onosproject.org/onos
cd onos
bazel build onos
bazel run onos-local -- clean
```

### ONOS 基礎設定

```bash
# 進入 ONOS CLI
docker exec -it onos /bin/bash   # 進入容器
onos localhost                    # 連接 CLI

# 或直接從 host 用 SSH
ssh -p 8101 onos@localhost   # 密碼：rocks

# ONOS CLI 常用指令
onos> apps -a -s            # 列出已啟用應用程式
onos> app activate org.onosproject.openflow        # 啟用 OpenFlow
onos> app activate org.onosproject.fwd             # 啟用基本轉發
onos> devices               # 檢視已連接的交換機
onos> flows                 # 檢視 flow entries
onos> ports                 # 檢視連接埠狀態
```

### ONOS 推薦啟用的 Apps（論文實驗用）

```bash
app activate org.onosproject.openflow          # OpenFlow 協定支援
app activate org.onosproject.fwd               # Reactive forwarding
app activate org.onosproject.hostprovider      # 主機發現
app activate org.onosproject.lldpprovider      # 鏈路發現
app activate org.onosproject.restsb            # REST API
```

---

## 四、Ryu 控制器安裝

Ryu 是純 Python 的 SDN 框架，適合快速原型開發與自訂應用程式。

> ⚠️ **重要提醒**：Ryu 上游已停止維護。建議同步關注其維護分支 [os-ken](https://github.com/openstack/os-ken)（OpenStack 維護）。

### Ryu 安裝

```bash
# 1. 安裝系統相依
sudo apt update
sudo apt install -y gcc python3-dev python3-pip \
  libffi-dev libssl-dev libxml2-dev libxslt1-dev zlib1g-dev

# 2. 安裝 Ryu（pip）
pip install ryu

# 或從原始碼安裝（最新）
git clone https://github.com/faucetsdn/ryu.git
cd ryu && pip install .

# 3. 安裝可選相依（OF-Config、BGP 等）
pip install -r ryu/tools/optional-requires 2>/dev/null
```

### Ryu 基礎使用

```bash
# 啟動內建 simple_switch 應用
ryu-manager ryu.app.simple_switch

# 啟動多個應用
ryu-manager ryu.app.simple_switch ryu.app.ofctl_rest

# 使用 REST API（需先啟動 ofctl_rest）
curl http://localhost:8080/stats/switches
curl http://localhost:8080/stats/flow/1
```

### Ryu 常用內建應用

| 應用 | 說明 |
|------|------|
| `ryu.app.simple_switch` | 基本 L2 learning switch |
| `ryu.app.simple_switch_13` | OpenFlow 1.3 版本 |
| `ryu.app.ofctl_rest` | REST API for flow table control |
| `ryu.app.rest_topology` | 拓樸資訊 REST API |
| `ryu.app.gui_topology` | Web 拓樸視覺化 |

---

## 五、連接 Mininet 到控制器

### 連接 ONOS

```bash
# 1. 先確保 ONOS 容器運作中
docker ps | grep onos

# 2. 啟動 Mininet 並指向 ONOS
sudo mn \
  --topo tree,depth=3,fanout=3 \
  --controller remote,ip=127.0.0.1,port=6653 \
  --switch ovs,protocols=OpenFlow13

# 3. 或使用自訂 Python 腳本
sudo python3 my_topo.py
```

Python 腳本範例（`topo_onos.py`）：
```python
from mininet.net import Mininet
from mininet.node import RemoteController, OVSSwitch
from mininet.topo import Topo

class MyTopo(Topo):
    def build(self):
        # 建立 6 台主機 + 3 台交換機的簡單拓樸
        switches = [self.addSwitch(f's{i}') for i in range(1, 4)]
        hosts = [self.addHost(f'h{i}') for i in range(1, 7)]
        
        # h1, h2 → s1；h3, h4 → s2；h5, h6 → s3
        self.addLink(hosts[0], switches[0])
        self.addLink(hosts[1], switches[0])
        self.addLink(hosts[2], switches[1])
        self.addLink(hosts[3], switches[1])
        self.addLink(hosts[4], switches[2])
        self.addLink(hosts[5], switches[2])
        # 交換機互連
        self.addLink(switches[0], switches[1])
        self.addLink(switches[1], switches[2])

net = Mininet(
    topo=MyTopo(),
    controller=lambda name: RemoteController(name, ip='127.0.0.1', port=6653),
    switch=OVSSwitch,
    autoSetMacs=True
)
net.start()
net.pingAll()       # 測試全網連通
net.iperf()         # 頻寬測試
net.stop()
```

### 連接 Ryu

```bash
# 1. 啟動 Ryu（另一個 terminal）
ryu-manager ryu.app.simple_switch_13 --ofp-tcp-listen-port 6653

# 2. 啟動 Mininet 並指向 Ryu
sudo mn \
  --topo tree,depth=3,fanout=3 \
  --controller remote,ip=127.0.0.1,port=6653 \
  --switch ovs,protocols=OpenFlow13
```

---

## 六、論文實驗常用拓樸

### 適合熵值攻擊檢測實驗的拓樸設計建議

```
        [ONOS/Ryu Controller]
               |
    ┌──────────┼──────────┐
   s1 ─────── s2 ─────── s3
   / \        / \        / \
  h1 h2     h3 h4      h5 h6
 (正常)    (正常+DDoS)  (正常)
```

**建議架構特點**：
- 至少 3 台交換機 + 6 台主機：可模擬不同區段的攻擊流量
- 一台主機專門發送攻擊流量（如 DDoS），其他主機作為背景流量
- 使用 OpenFlow 1.3 以上（支援 meter table、group table）
- 交換機之間設定頻寬限制（用 `TCLink` 加入延遲/頻寬參數）

### 加入流量控制的拓樸

```python
from mininet.link import TCLink
from functools import partial

link = partial(TCLink, bw=10, delay='5ms')   # 10 Mbps, 5ms 延遲
net = Mininet(topo=topo, link=link, controller=RemoteController)
```

---

## 七、ONOS vs Ryu 選擇對比

| 面向 | ONOS | Ryu |
|------|------|-----|
| 語言 | Java | Python |
| 部署方式 | Docker / Bazel 編譯 | pip install |
| 效能 | 高（電信級） | 中（適合原型） |
| 學習曲線 | 陡峭 | 平緩 |
| 論文常見度 | 高（業界採用） | 高（學術常用） |
| 維護狀態 | 活躍維護 | **已停止維護** ⚠️ |
| REST API | 完整（Northbound/Southbound） | 部分（需附加 app） |
| Web UI | 有（拓樸視覺化） | 有（gui_topology） |
| 自訂應用 | 需寫 Java app | 純 Python，開發快速 |
| 適合論文用途 | 大規模實驗、效能測試 | 演算法原型、流量分析 |

### 我的建議

- **先從 Ryu 開始**：Python 上手快，適合快速驗證熵值檢測演算法。自訂流量收集 app 開發容易
- **後期用 ONOS 驗證**：確認演算法在電信級控制器上也能運作，提升論文完整性
- **不要同時跑兩個控制器**：資源有限的 VM 上專注一個

---

## 八、常見問題排除

### Mininet 無法啟動
```bash
# 確認 Open vSwitch 服務執行中
sudo systemctl status openvswitch-switch
sudo systemctl start openvswitch-switch

# 清理殘留的 network namespace
sudo mn -c
```

### 連不上 ONOS 控制器
```bash
# 確認 ONOS OpenFlow port 有監聽
docker exec onos netstat -tlnp | grep 6653

# 確認 ONOS 已啟用 OpenFlow app
docker exec onos /bin/bash -c "echo 'app activate org.onosproject.openflow' | onos localhost"
```

### Docker 權限問題
```bash
# 將自己加入 docker 群組
sudo usermod -aG docker $USER
# 重新登入後生效
```

### 磁碟空間不足
```bash
# 檢查 docker 佔用
docker system df
# 清理未使用資源
docker system prune -a
```

---

## 九、建議安裝順序

```
Day 1: Mininet 安裝 + 基礎拓樸測試（30 分鐘）
Day 1: Ryu pip install + simple_switch 測試（20 分鐘）
Day 1: Mininet ↔ Ryu 整合測試（30 分鐘）
Day 2: 撰寫自訂拓樸腳本，加入流量控制（1-2 小時）
Day 3: ONOS Docker 安裝 + Mininet ↔ ONOS 測試（1 小時）
Day 4: 開始進行攻擊流量模擬與熵值收集實驗
```

---

## 參考資料

- Mininet 官方：http://mininet.org
- Mininet GitHub：https://github.com/mininet/mininet
- ONOS 官方：https://onosproject.org
- ONOS Docker Hub：https://hub.docker.com/r/onosproject/onos
- Ryu SDN Framework：https://ryu-sdn.org
- Ryu (os-ken fork)：https://github.com/openstack/os-ken
