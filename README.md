📡ARP Handling in SDN using POX Controller
📌 Overview

This project demonstrates ARP (Address Resolution Protocol) handling in Software Defined Networking (SDN) using the POX controller and Mininet simulation.

It implements:

ARP Request & Reply handling
Proxy ARP (controller responds directly)
MAC learning & forwarding
Flow rule installation for efficient packet forwarding
👨‍💻 Author
Name: Sagar K
SRN: PES2UG24CS424
🏗️ Project Structure
├── controller/
│   └── arp_handler.py
├── topology/
│   └── topology.py
├── tests/
│   └── test_arp_handler.py
└── README.md
⚙️ Technologies Used
Python 3
POX SDN Controller
Mininet
OpenFlow 1.0
unittest
🌐 Network Topology
h1 ─┐
h2 ─┤
     S1 ─── Controller (127.0.0.1:6633)
h3 ─┤
h4 ─┘
Host	IP Address	MAC Address
h1	10.0.0.1	00:00:00:00:00:01
h2	10.0.0.2	00:00:00:00:00:02
h3	10.0.0.3	00:00:00:00:00:03
h4	10.0.0.4	00:00:00:00:00:04
🚀 How to Run
1️⃣ Start POX Controller
cd ~/pox
python3 pox.py log.level --DEBUG arp_handler
2️⃣ Run Mininet
sudo python3 topology/topology.py
3️⃣ Run Test Mode
sudo python3 topology/topology.py --test
4️⃣ Run Unit Tests
python3 tests/test_arp_handler.py
🔍 Features
✅ ARP Learning
Learns IP → MAC → Port mapping
Updates entries dynamically
✅ Proxy ARP
Controller directly responds to ARP requests
Reduces broadcast traffic
✅ Flooding
Unknown ARP requests are flooded
✅ IPv4 Forwarding
Uses MAC table
Installs flow rules
🧪 Test Scenarios
🔹 Scenario 1: ARP + Ping
First ping → ARP Flood
Second ping → Proxy ARP
Full connectivity → pingAll
🔹 Scenario 2: Throughput
TCP → iperf
UDP → iperf
📊 Sample Logs
[ARP REQ]    Who has 10.0.0.2 ?
[ARP FLOOD]  unknown — flooding request
[ARP REPLY]  10.0.0.2 is at 00:00:00:00:00:02
[IP FWD]     10.0.0.1 -> 10.0.0.2 via port 2
[PROXY ARP]  Answering directly
🧠 Key Concepts
SDN (Control plane vs Data plane)
OpenFlow protocol
ARP protocol
Proxy ARP
🎯 Learning Outcomes
Implement SDN controller logic
Understand ARP handling
Reduce network flooding
Work with Mininet + POX
⚠️ Notes
Start POX before Mininet
Uses OpenFlow 1.0
autoStaticArp=False is required
