# ARP Handling in SDN using POX Controller

## Overview
This project demonstrates ARP (Address Resolution Protocol) handling in Software Defined Networking (SDN) using the POX controller and Mininet simulation.

It includes ARP request and reply handling, proxy ARP, MAC learning, and flow rule installation for efficient forwarding.

## Author
Name: Sagar K  
SRN: PES2UG24CS424  

## Project Structure

├── controller/
│ └── arp_handler.py
├── topology/
│ └── topology.py
├── tests/
│ └── test_arp_handler.py
└── README.md


## Technologies Used
- Python 3  
- POX Controller  
- Mininet  
- OpenFlow 1.0  
- unittest  

## Network Topology

h1 ─┐
h2 ─┤
S1 ─── Controller (127.0.0.1:6633)
h3 ─┤
h4 ─┘


## Host Details

| Host | IP Address | MAC Address |
|------|-----------|------------|
| h1 | 10.0.0.1 | 00:00:00:00:00:01 |
| h2 | 10.0.0.2 | 00:00:00:00:00:02 |
| h3 | 10.0.0.3 | 00:00:00:00:00:03 |
| h4 | 10.0.0.4 | 00:00:00:00:00:04 |

## How to Run

Start the POX controller:
```bash
cd ~/pox
python3 pox.py log.level --DEBUG arp_handler

Run the Mininet topology:

sudo python3 topology/topology.py

Run automated test mode:

sudo python3 topology/topology.py --test

Run unit tests:

python3 tests/test_arp_handler.py
Features
Learns IP to MAC to port mapping dynamically
Supports proxy ARP to reduce broadcast traffic
Floods only when destination is unknown
Installs flow rules for faster forwarding
Test Scenarios

Normal communication is tested using ping between hosts.
First ping results in ARP flooding, while subsequent pings use learned entries or proxy ARP.

Throughput is measured using iperf for both TCP and UDP traffic.

Sample Output
[ARP REQ] Who has 10.0.0.2 ?
[ARP FLOOD] unknown — flooding request
[ARP REPLY] 10.0.0.2 is at 00:00:00:00:00:02
[IP FWD] 10.0.0.1 -> 10.0.0.2 via port 2
[PROXY ARP] Answering directly
Notes
Start the controller before running Mininet
Uses OpenFlow 1.0
autoStaticArp is disabled to allow controller-based ARP handling
License

This project is for academic use only.
