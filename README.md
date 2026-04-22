# 🌐 SDN ARP Handling using POX Controller & Mininet

![SDN](https://img.shields.io/badge/SDN-POX%20Controller-blue)
![Python](https://img.shields.io/badge/Python-3.9-green)
![Mininet](https://img.shields.io/badge/Mininet-2.3-orange)
![OpenFlow](https://img.shields.io/badge/OpenFlow-1.0-red)

> A Software Defined Networking (SDN) project that implements dynamic ARP Handling using the POX controller and Mininet network emulator.

---

## 📌 Table of Contents
- [Overview](#overview)
- [Topology](#topology)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Project Structure](#project-structure)
- [How It Works](#how-it-works)
- [Running the Project](#running-the-project)
- [Demo Output](#demo-output)
- [Flow Table](#flow-table)
- [ARP Cache](#arp-cache)
- [Key Concepts](#key-concepts)
- [Why POX](#why-pox)
- [Mininet CLI Reference](#mininet-cli-commands-reference)

---

## 📖 Overview

This project demonstrates ARP Request & Reply Handling in an SDN environment. The POX controller acts as the **brain** of the network — it learns MAC addresses dynamically, handles ARP requests, installs flow rules into the switch, and enables direct host-to-host communication.

---

## 🗺️ Topology

```
        POX Controller
        (127.0.0.1:6633)
              |
              | OpenFlow Protocol
              |
         [S1 Switch]
        /    |    |    \
      P1    P2   P3    P4
      |      |    |     |
      H1    H2   H3    H4
  10.0.0.1  .2   .3    .4
```

| Host | IP Address | MAC Address | Switch Port |
|------|-----------|-------------|-------------|
| H1 | 10.0.0.1 | 00:00:00:00:00:01 | Port 1 |
| H2 | 10.0.0.2 | 00:00:00:00:00:02 | Port 2 |
| H3 | 10.0.0.3 | 00:00:00:00:00:03 | Port 3 |
| H4 | 10.0.0.4 | 00:00:00:00:00:04 | Port 4 |

---

## ✅ Prerequisites

- Ubuntu 20.04 / 22.04 / 24.04
- Python 3.9
- Mininet 2.3
- POX Controller (gar branch)
- Open vSwitch (OvS)

---

## ⚙️ Installation

### Step 1 — Install Python 3.9
```bash
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt update
sudo apt install python3.9 python3.9-venv python3.9-dev -y
```

### Step 2 — Verify Python 3.9
```bash
python3.9 --version
# Python 3.9.25
```

### Step 3 — Clone POX Controller
```bash
cd ~
git clone http://github.com/noxrepo/pox
cd pox
git checkout gar
```

### Step 4 — Create Virtual Environment
```bash
cd ~/pox
python3.9 -m venv venv39
source venv39/bin/activate
python --version
# Python 3.9.25
```

### Step 5 — Install Mininet
```bash
sudo apt install mininet -y
```

### Step 6 — Copy arp_handler to POX
```bash
cp arp_handler.py ~/pox/ext/
```

---

## 📁 Project Structure

```
CN_SDN/
├── topology/
│   └── topology.py        # Mininet topology — 1 switch, 4 hosts
├── arp_handler.py         # POX ARP Handler — copy to ~/pox/ext/
└── README.md
```

---

## 🔄 How It Works

### Phase 1 — LEARN
When H1 sends any packet, the controller reads the **source MAC** and stores it:
```
[ARP LEARN]  10.0.0.1  ->  00:00:00:00:00:01  (port=1)
```

### Phase 2 — REQUEST
H1 asks *"Who is 10.0.0.2?"* — controller floods to all ports:
```
[ARP REQ]   Who has 10.0.0.2 ? Tell 10.0.0.1
[ARP FLOOD] 10.0.0.2 unknown — flooding request
```

### Phase 3 — REPLY
H2 responds, controller learns H2's MAC and forwards reply to H1:
```
[ARP LEARN]  10.0.0.2  ->  00:00:00:00:00:02  (port=2)
[ARP REPLY]  10.0.0.2 is at 00:00:00:00:00:02
[ARP FWD]    Forwarding reply to 10.0.0.1 via port 1
```

### Phase 4 — FLOW INSTALL
Controller programs the switch with forwarding rules:
```
[IP FWD]  10.0.0.1 -> 10.0.0.2  via port 2  (flow installed)
[IP FWD]  10.0.0.2 -> 10.0.0.1  via port 1  (flow installed)
```

---

## 🚀 Running the Project

### Terminal 1 — Start POX Controller
```bash
cd ~/pox
source venv39/bin/activate
python pox.py log.level --DEBUG arp_handler
```

Expected output:
```
INFO:core:POX 0.7.0 (gar) is up.
DEBUG:openflow.of_01:Listening on 0.0.0.0:6633
INFO:openflow.of_01:[00-00-00-00-00-01 1] connected
INFO:arp_handler:[SWITCH 00-00-00-00-00-01] connected — installing table-miss rule
```

### Terminal 2 — Start Mininet Topology
```bash
cd ~/Desktop/CN_SDN
sudo python topology/topology.py
```

### Terminal 2 — Test Connectivity
```bash
# Ping H2 from H1
mininet> h1 ping -c 3 h2

# Ping all hosts
mininet> pingall

# Individual pings
mininet> h1 ping -c 3 h3
mininet> h2 ping -c 3 h4

# Bandwidth test
mininet> iperf h1 h2

# ARP cache
mininet> h1 arp -a

# Flow table
mininet> sh ovs-ofctl dump-flows s1
```

---

## 📊 Demo Output

### First Ping (ARP Learning Phase)
```
mininet> h1 ping -c 3 h2
PING 10.0.0.2 (10.0.0.2) 56(84) bytes of data.
64 bytes from 10.0.0.2: icmp_seq=1 ttl=64 time=20.3 ms
64 bytes from 10.0.0.2: icmp_seq=2 ttl=64 time=0.102 ms
64 bytes from 10.0.0.2: icmp_seq=3 ttl=64 time=0.098 ms
--- 10.0.0.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss
rtt min/avg/max/mdev = 0.098/6.833/20.300/9.530 ms
```

### Second Ping (Flow Already Installed)
```
mininet> h1 ping -c 3 h2
3 packets transmitted, 3 received, 0% packet loss
rtt min/avg/max/mdev = 0.098/0.101/0.104/0.003 ms
```

---

## 🔍 Flow Table

```bash
mininet> sh ovs-ofctl dump-flows s1
```

```
cookie=0x0, duration=15s, priority=1, dl_dst=00:00:00:00:00:01  actions=output:s1-eth1
cookie=0x0, duration=14s, priority=1, dl_dst=00:00:00:00:00:02  actions=output:s1-eth2
cookie=0x0, duration=38s, priority=1, dl_dst=00:00:00:00:00:03  actions=output:s1-eth3
cookie=0x0, duration=37s, priority=1, dl_dst=00:00:00:00:00:04  actions=output:s1-eth4
cookie=0x0, duration=60s, priority=0                             actions=CONTROLLER:65535
```

| Rule | Priority | Match | Action | Meaning |
|------|----------|-------|--------|---------|
| 1-4 | 1 | dl_dst = Host MAC | output:port | Forward to correct host |
| 5 | 0 | everything else | CONTROLLER | Send unknown to POX |

---

## 🖥️ ARP Cache

```bash
mininet> h1 arp -a
? (10.0.0.2) at 00:00:00:00:00:02 [ether] on h1-eth0
? (10.0.0.3) at 00:00:00:00:00:03 [ether] on h1-eth0
? (10.0.0.4) at 00:00:00:00:00:04 [ether] on h1-eth0
```

---

## 💡 Key Concepts

### Table-Miss Rule
- Installed on switch connect
- Priority 0 — matches all unknown packets
- Sends them to the controller for processing

### ARP Learning
- Controller reads source MAC from every incoming packet
- Builds a MAC-to-Port table dynamically
- No manual configuration needed

### Flow Installation
- After ARP resolution, controller installs flow rules in the switch
- `idle_timeout=30` — rule expires after 30s of inactivity
- `hard_timeout=120` — rule expires after 120s regardless
- Switch forwards future packets directly — no controller involved

### Proxy ARP
- If target IP is already known, controller answers ARP directly
- No flooding needed — reduces unnecessary broadcast traffic

---

## 🤔 Why POX?

| Controller | Language | Best For |
|------------|----------|---------|
| **POX** ✅ | Python | Learning, Labs, Prototyping |
| Ryu | Python | Research |
| ONOS | Java | Production Networks |
| OpenDaylight | Java | Enterprise Networks |
| Floodlight | Java | Commercial Use |

POX is used because:
- Written in **Python** — easy to understand and modify
- **Lightweight** — runs on a single machine
- **Perfect with Mininet** — standard SDN lab combination
- **Full DEBUG visibility** — every packet event is logged
- Ideal for understanding **OpenFlow 1.0** concepts

---

## 📝 Mininet CLI Commands Reference

```bash
mininet> h1 ping -c 3 h2          # Ping H2 from H1
mininet> h1 arping -c 2 10.0.0.2  # ARP ping
mininet> pingall                   # Ping all host combinations
mininet> sh ovs-ofctl dump-flows s1  # Show flow table
mininet> sh ovs-ofctl dump-ports s1  # Show port statistics
mininet> iperf h1 h2               # Bandwidth test
mininet> h1 arp -a                 # Show ARP cache on H1
mininet> net                       # Show network topology
mininet> nodes                     # List all nodes
mininet> exit                      # Exit Mininet
```

---

## 👨‍💻 Author

**Sagar K**
Computer Networks Lab — SDN Project
Ubuntu 24.04 | Python 3.9.25 | POX 0.7.0 | Mininet 2.3
