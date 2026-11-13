Markdown
# Atlas Bank: Enterprise Multi-Site Network Architecture & Security Infrastructure

![Cisco Packet Tracer](https://img.shields.io/badge/Simulation-Cisco%20Packet%20Tracer-1BA0D7?style=for-the-badge&logo=cisco&logoColor=white)
![Windows Server](https://img.shields.io/badge/Infrastructure-Server--PT%20%2F%20Windows%20Server-0078D6?style=for-the-badge&logo=windows&logoColor=white)
![Routing Protocol](https://img.shields.io/badge/Routing-Multi--Area%20OSPFv2-2EA44F?style=for-the-badge)
![Firewall](https://img.shields.io/badge/Perimeter-Cisco%20ASA%205506--X-D73A49?style=for-the-badge)
![Layer 2 Security](https://img.shields.io/badge/Switching-Rapid--PVST%20%7C%20DHCP%20Snooping-6F42C1?style=for-the-badge)

A comprehensive, multi-site banking network infrastructure engineered in Cisco Packet Tracer. The architecture models a production-grade financial institution connecting a centralized Headquarters (HQ) data center with regional branch offices in **Casablanca (`BR1`)** and **Rabat (`BR2`)**. The design enforces strict network segmentation, stateful perimeter defense, authenticated dynamic routing, Layer 2 port hardening, and centralized DHCP/DNS services.

---

## Table of Contents

1. [Network Architecture & Topology](#network-architecture--topology)
2. [IP Addressing & VLAN Segmentation Plan](#ip-addressing--vlan-segmentation-plan)
3. [Physical Port & Interconnect Matrix](#physical-port--interconnect-matrix)
4. [Routing & WAN Architecture](#routing--wan-architecture)
5. [Security Hardening & Access Control](#security-hardening--access-control)
6. [Centralized Services & DHCP Relay](#centralized-services--dhcp-relay)
7. [Verification & Validation Checklist](#verification--validation-checklist)
8. [Repository Structure](#repository-structure)

---

## Network Architecture & Topology

```mermaid
flowchart TB
    subgraph EXT ["External & Perimeter Edge"]
        direction TB
        ISRV["Internet-SRV<br/>198.51.100.10/24"]
        ISPR["ISP-R Router<br/>Gi0/1: 198.51.100.1 | Gi0/0: 203.0.113.1/29"]
        DMZWEB["DMZ-WEB Portal<br/>172.16.100.10/24<br/>Static NAT: 203.0.113.3"]
        DMZSW["DMZ-SW"]
        HQFW["HQ-FW (ASA 5506-X)<br/>Outside: Gi1/1 | Inside: Gi1/2 | DMZ: Gi1/3"]

        ISRV <--> ISPR
        ISPR <-->|"203.0.113.0/29"| HQFW
        DMZWEB <--> DMZSW <--> HQFW
    end

    subgraph HQ ["Headquarters Data Center & Campus (OSPF Area 0)"]
        direction TB
        HQCORE["HQ-CORE (Catalyst 3650 L3)<br/>SVIs: VLANs 10, 20, 30, 40, 50, 60"]
        SWSRV["SW-SRV (Server Access)"]
        SWHQ["SW-HQ-ACC (Campus Access)"]

        DC1["DC1 (Central Server)<br/>10.10.10.10/24 (VLAN 10)<br/>DHCP | DNS | AD"]
        SRVCORE["SRV-CORE (Core Banking)<br/>10.10.20.10/24 (VLAN 20)"]
        SRVATM["SRV-ATM (ATM Switch)<br/>10.10.20.20/24 (VLAN 20)"]

        WIN10["WIN10-CLI (VLAN 30)"]
        TELLERS["HQ Tellers (VLAN 30)"]
        PCIT["PC-IT Admin (VLAN 40)"]
        GUESTAP["Guest AP (VLAN 60)"]

        HQFW <-->|"Inside Link"| HQCORE
        HQCORE <==>|"Port-Channel 1 (LACP)"| SWSRV
        HQCORE <==>|"Port-Channel 2 (LACP)"| SWHQ

        SWSRV ---|"Fa0/3 (VLAN 10)"| DC1
        SWSRV ---|"Fa0/4 (VLAN 20)"| SRVCORE
        SWSRV ---|"Fa0/5 (VLAN 20)"| SRVATM

        SWHQ ---|"Fa0/3"| WIN10
        SWHQ ---|"Fa0/4-5"| TELLERS
        SWHQ ---|"Fa0/10"| PCIT
        SWHQ ---|"Fa0/20"| GUESTAP
    end

    subgraph WAN ["WAN & Regional Branches"]
        direction TB
        HQWAN["HQ-WAN-R (ABR)<br/>Router-ID: 10.255.255.2"]

        subgraph BR1 ["Branch 1: Casablanca (OSPF Area 1)"]
            BR1R["BR1-R<br/>Lo0: 10.255.255.3<br/>WAN: 10.255.1.2/30"]
            BR1SW["BR1-SW<br/>MGMT: 10.20.50.2/24"]
            BR1PC["Tellers (VLAN 30)<br/>10.20.30.0/24"]
            BR1ATM["Branch ATM (VLAN 70)<br/>10.20.70.0/24"]

            BR1R <-->|"802.1Q Trunk (Native 99)"| BR1SW
            BR1SW ---|"Fa0/2-5"| BR1PC
            BR1SW ---|"Fa0/10"| BR1ATM
        end

        subgraph BR2 ["Branch 2: Rabat (OSPF Area 2)"]
            BR2R["BR2-R<br/>Lo0: 10.255.255.4<br/>WAN: 10.255.2.2/30"]
            BR2SW["BR2-SW<br/>MGMT: 10.30.50.2/24"]
            BR2PC["Tellers (VLAN 30)<br/>10.30.30.0/24"]
            BR2ATM["Branch ATM (VLAN 70)<br/>10.30.70.0/24"]

            BR2R <-->|"802.1Q Trunk (Native 99)"| BR2SW
            BR2SW ---|"Fa0/2-5"| BR2PC
            BR2SW ---|"Fa0/10"| BR2ATM
        end

        HQCORE <--> HQWAN
        HQWAN <-->|"10.255.1.0/30 (Area 1)"| BR1R
        HQWAN <-->|"10.255.2.0/30 (Area 2)"| BR2R
    end
IP Addressing & VLAN Segmentation Plan
Campus & Branch VLAN Allocation
VLAN ID	VLAN Name	HQ Subnet (10.10.x.0)	Casablanca (10.20.x.0)	Rabat (10.30.x.0)	Gateway IP	IP Assignment Method
10	SRV-INFRA	10.10.10.0/24	—	—	.1	Static (DC1: .10)
20	SRV-COREBANK	10.10.20.0/24	—	—	.1	Static (SRV-CORE: .10, SRV-ATM: .20)
30	STAFF	10.10.30.0/24	10.20.30.0/24	10.30.30.0/24	.1	DHCP Relayed (ip helper-address 10.10.10.10)
40	IT-ADMIN	10.10.40.0/24	—	—	.1	DHCP Relayed (ip helper-address 10.10.10.10)
50	MGMT	10.10.50.0/24	10.20.50.0/24	10.30.50.0/24	.1	Static (Switch SVIs: .2)
60	GUEST	10.10.60.0/24	—	—	.1	Local DHCP Pool / Isolated Internet Only
70	ATM	—	10.20.70.0/24	10.30.70.0/24	.1	DHCP Relayed (ip helper-address 10.10.10.10)
99	NATIVE	Trunk Native	Trunk Native	Trunk Native	—	Untagged Trunk Carrier
999	BLACKHOLE	Unused Ports	Unused Ports	Unused Ports	—	Shutdown / Isolated
Infrastructure, WAN, DMZ & Server Addressing
Device / Host	Interface / VLAN	IP Address	Subnet Mask	Default Gateway	Role
DC1 (Server-PT)	NIC (VLAN 10)	10.10.10.10	255.255.255.0	10.10.10.1	Centralized DHCP & DNS Server
SRV-CORE	NIC (VLAN 20)	10.10.20.10	255.255.255.0	10.10.20.1	Core Banking Application Server
SRV-ATM	NIC (VLAN 20)	10.10.20.20	255.255.255.0	10.10.20.1	ATM Transaction Switch (TCP/8443)
DMZ-WEB	NIC (DMZ)	172.16.100.10	255.255.255.0	172.16.100.1	Public Web Portal (NAT: 203.0.113.3)
Internet-SRV	NIC (External)	198.51.100.10	255.255.255.0	198.51.100.1	Simulated External Web/DNS Target
BR1-R (Casablanca)	Loopback0 / Gi0/0	10.255.255.3 / 10.255.1.2	/32 & /30	OSPF Area 1	Branch Router (Router-on-a-Stick)
BR2-R (Rabat)	Loopback0 / Gi0/0	10.255.255.4 / 10.255.2.2	/32 & /30	OSPF Area 2	Branch Router (Router-on-a-Stick)
BR1-SW	Vlan50	10.20.50.2	255.255.255.0	10.20.50.1	Casablanca Access Switch
BR2-SW	Vlan50	10.30.50.2	255.255.255.0	10.30.50.1	Rabat Access Switch
Physical Port & Interconnect Matrix
Source Device	Source Port	Destination Device	Destination Port	Link Type / Trunk / VLAN Details
ISP-R	Gi0/0	HQ-FW	Gi1/1 (Outside)	Public Transit (203.0.113.0/29)
ISP-R	Gi0/1	Internet-SRV	NIC	External Network (198.51.100.0/24)
HQ-FW	Gi1/2 (Inside)	HQ-CORE	Gi1/0/1	Routed Perimeter Transit
HQ-FW	Gi1/3 (DMZ)	DMZ-SW	Fa0/1	DMZ Segment (172.16.100.0/24)
HQ-CORE	Gi1/0/2	HQ-WAN-R	Gi0/0	Core-to-WAN Backbone (OSPF Area 0)
HQ-CORE	Gi1/0/3 - 1/0/4	SW-SRV	Gi0/1 - 0/2	EtherChannel Po1 (802.1Q Trunk)
HQ-CORE	Gi1/0/5 - 1/0/6	SW-HQ-ACC	Gi0/1 - 0/2	EtherChannel Po2 (802.1Q Trunk)
SW-SRV	Fa0/3	DC1	NIC	Access Mode — VLAN 10 (SRV-INFRA)
SW-SRV	Fa0/4	SRV-CORE	NIC	Access Mode — VLAN 20 (SRV-COREBANK)
SW-SRV	Fa0/5	SRV-ATM	NIC	Access Mode — VLAN 20 (SRV-COREBANK)
SW-HQ-ACC	Fa0/3 - 0/5	HQ Staff PCs	NIC	Access Mode — VLAN 30 (STAFF)
SW-HQ-ACC	Fa0/10	PC-IT	NIC	Access Mode — VLAN 40 (IT-ADMIN)
SW-HQ-ACC	Fa0/20	Guest AP	Port 0	Access Mode — VLAN 60 (GUEST)
HQ-WAN-R	Gi0/1	BR1-R	Gi0/0	Point-to-Point WAN (10.255.1.0/30 — Area 1)
HQ-WAN-R	Gi0/2	BR2-R	Gi0/0	Point-to-Point WAN (10.255.2.0/30 — Area 2)
BR1-R	Gi0/1	BR1-SW	Fa0/1	802.1Q Trunk (Allowed: 30,50,70,99
BR1-SW	Fa0/2 - 0/5	BR1 Tellers	NIC	Access Mode — VLAN 30 (STAFF)
BR1-SW	Fa0/10	BR1 ATM	NIC	Access Mode — VLAN 70 (ATM)
BR2-R	Gi0/1	BR2-SW	Fa0/1	802.1Q Trunk (Allowed: 30,50,70,99
BR2-SW	Fa0/2 - 0/5	BR2 Tellers	NIC	Access Mode — VLAN 30 (STAFF)
BR2-SW	Fa0/10	BR2 ATM	NIC	Access Mode — VLAN 70 (ATM)
Routing & WAN Architecture
Multi-Area OSPFv2 (process 1):

Area 0 (Backbone): Encompasses HQ-CORE and HQ-WAN-R (acting as Area Border Router).

Area 1 (Casablanca): Encompasses HQ-WAN-R (Gi0/1) and BR1-R (10.255.1.0/30, 10.20.0.0/16, Loopback0).

Area 2 (Rabat): Encompasses HQ-WAN-R (Gi0/2) and BR2-R (10.255.2.0/30, 10.30.0.0/16, Loopback0).

Routing Security & Optimization:

All point-to-point WAN links utilize ip ospf network point-to-point to eliminate DR/BDR election overhead.

MD5 Neighbor Authentication (ip ospf authentication message-digest with key Atl@s0spf) prevents unauthorized route injection.

passive-interface default is enabled on all routers, restricting OSPF hello packets strictly to active transit interfaces.

Security Hardening & Access Control
1. Perimeter Defense (HQ-FW ASA 5506-X)
Security Levels: Outside (0), DMZ (50), Inside (100).

NAT Policies: Dynamic PAT for internal subnets accessing the Internet; Static NAT mapping 172.16.100.10 (DMZ-WEB) to public IP 203.0.113.3.

Traffic Inspection: Stateful inspection permits return traffic while strictly filtering inbound public access to HTTP/HTTPS on DMZ-WEB.

2. ATM Micro-Segmentation (ACL-ATM-IN)
Applied inbound on branch ATM subinterfaces (Gi0/1.70) on both BR1-R and BR2-R to enforce zero-trust isolation for ATM terminals:

Cisco CLI
ip access-list extended ACL-ATM-IN
 permit udp any eq bootpc any eq bootps
 permit tcp 10.20.70.0 0.0.0.255 host 10.10.20.20 eq 8443
 permit icmp 10.20.70.0 0.0.0.255 host 10.10.20.20 echo
 deny ip any any log
(Note: On BR2-R, source subnet is 10.30.70.0 0.0.0.255.)

3. Layer 2 Switch Hardening (BR1-SW, BR2-SW, SW-HQ-ACC)
Spanning Tree: spanning-tree mode rapid-pvst with portfast and bpduguard enable on all edge access ports.

DHCP Snooping: Enabled globally (ip dhcp snooping) on user and ATM VLANs (30, 70), with no ip dhcp snooping information option configured to ensure compatibility with Cisco IOS DHCP relay. Uplink trunks are explicitly trusted (ip dhcp snooping trust), while edge ports are rate-limited (10–15 pps).

Port Security: Sticky MAC learning enforced with maximum 2 (violation restrict) on teller ports and maximum 1 (violation shutdown) on ATM ports.

Port Shutdown & Blackholing: All unused switch ports (Fa0/6-9, Fa0/11-24) are administratively shut down and assigned to VLAN 999 (BLACKHOLE).

Centralized Services & DHCP Relay
DHCP is centralized on DC1 (10.10.10.10) using Cisco Packet Tracer Server-PT to eliminate external VM bridging overhead while preserving enterprise relay behavior:

DHCP Relay Agent: All L3 gateways (HQ-CORE SVIs and BR1-R/BR2-R subinterfaces for VLANs 30, 40, and 70) forward client broadcast DHCPDISCOVER packets via ip helper-address 10.10.10.10.

Configured DHCP Scopes on DC1:

HQ-Staff: Gateway 10.10.30.1 | Pool 10.10.30.21 – 10.10.30.200/24

HQ-IT: Gateway 10.10.40.1 | Pool 10.10.40.21 – 10.10.40.200/24

BR1-Staff: Gateway 10.20.30.1 | Pool 10.20.30.21 – 10.20.30.200/24

BR1-ATM: Gateway 10.20.70.1 | Pool 10.20.70.21 – 10.20.70.200/24

BR2-Staff: Gateway 10.30.30.1 | Pool 10.30.30.21 – 10.30.30.200/24

BR2-ATM: Gateway 10.30.70.1 | Pool 10.30.70.21 – 10.30.70.200/24

Verification & Validation Checklist
Use the following commands across devices to verify full operational convergence:

Validation Target	Device	Verification Command	Expected Result
OSPF Adjacencies	HQ-WAN-R, BR1-R, BR2-R	show ip ospf neighbor	State FULL/ - across all WAN point-to-point links
End-to-End Routing	BR1-R, BR2-R	show ip route ospf	O IA routes learned for 10.10.10.0/24 and 10.10.20.0/24
Trunking & VLANs	BR1-SW, BR2-SW	show interfaces trunk	Native VLAN 99; VLANs 30,50,70,99 forwarding
DHCP Snooping	BR1-SW, BR2-SW	show ip dhcp snooping	Uplink Fa0/1 listed as Trusted: yes; Option 82 disabled
DHCP Lease Acquisition	Branch & HQ PCs	ipconfig /renew	Valid IP in .21–.200 range with correct default gateway
ATM ACL Enforcement	Branch ATMs	ping 10.10.20.20 / ping 10.10.10.10	Ping to SRV-ATM succeeds; ping to DC1 is dropped by ACL-ATM-IN
Repository Structure
Plaintext
atlas-bank-network/
├── configs/
│   ├── firewall/
│   │   └── HQ-FW-ASA5506.cfg
│   ├── routers/
│   │   ├── ISP-R.cfg
│   │   ├── HQ-WAN-R.cfg
│   │   ├── BR1-R-Casablanca.cfg
│   │   └── BR2-R-Rabat.cfg
│   └── switches/
│       ├── HQ-CORE-3650.cfg
│       ├── SW-SRV.cfg
│       ├── SW-HQ-ACC.cfg
│       ├── BR1-SW.cfg
│       └── BR2-SW.cfg
├── topology/
│   └── Atlas-Bank-Enterprise.pkt
└── README.md
