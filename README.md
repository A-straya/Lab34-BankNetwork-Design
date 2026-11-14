

# OXA BANK
### Enterprise Banking Network Infrastructure

**A multi-floor, segmented and redundant headquarters network built in Cisco Packet Tracer 9.0.1**

![Cisco Packet Tracer](https://img.shields.io/badge/Cisco-Packet%20Tracer%209.0.1-1BA0D7?logo=cisco&logoColor=white)
![Routing](https://img.shields.io/badge/Routing-RIPv2-2563EB)
![Redundancy](https://img.shields.io/badge/High%20Availability-HSRP-0D9488)
![Security](https://img.shields.io/badge/Security-ACL%20%7C%20NAT%20%7C%20SSH-16A34A)
![Status](https://img.shields.io/badge/Type-Lab%20%2F%20Portfolio-orange)

</div>

---

## 01 · Project Overview

**OXA BANK** is a simulated enterprise banking headquarters network designed to demonstrate practical routing, switching, network segmentation, redundancy and security controls.

The topology represents four office floors split into two distribution blocks. Each block is built around a pair of multilayer switches, with dedicated access switches for departments and service zones. A routed core connects both blocks to an edge router and an ISP simulation.

### Design objectives

- Segment departments and sensitive banking services into dedicated VLANs.
- Provide redundant default gateways using HSRP.
- Prevent Layer 2 loops with Rapid-PVST+.
- Exchange internal routes using RIPv2.
- Centralize DHCP and DNS services.
- Restrict traffic to Guest, ATM, Backup and DMZ zones using ACLs.
- Provide simulated Internet access using NAT overload (PAT).
- Harden access ports and restrict network-device management.

> **Project scope:** Educational Packet Tracer simulation. This is not a certified or production-ready banking architecture.

## 02 · Network Topology

The diagram below summarizes the logical design and the main connections defined in the device configurations.

![OXA BANK logical network topology](docs/topology.png)

### Topology at a glance

| Segment | Devices | Responsibility |
|---|---|---|
| Floor 1 | SW-MGMT, SW-FIN, SW-HR, SW-CUST | Management, Finance, HR, Customer Services |
| Floor 2 | SW-IT, SW-ACC, SW-BOPS, SW-ADM | IT, Accounting, Bank Operations, Admin |
| Left distribution | MLS1, MLS2 | Inter-VLAN routing, HSRP, STP |
| Floor 3 | SW-LOANS, SW-BKP, SW-SEC, SW-ATM | Loans, Backup, Security, ATM |
| Floor 4 | SW-VOICE, SW-GUEST, SW-DMZ, SW-SRV | Voice, Guest, DMZ, DNS/DHCP |
| Right distribution | MLS3, MLS4 | Inter-VLAN routing, HSRP, STP |
| Routed core | R1, R2, R3, R4 | Internal routed connectivity |
| Perimeter | R5 + ISP | NAT/PAT, outside filtering, ISP simulation |

**Physical design notes**

- Each access switch has two 802.1Q trunk uplinks: one to each multilayer switch in its block.
- Routed `/30` links connect the routers and distribution switches as shown in the device configurations.
- The four internal routers form a serial ring: R1–R2–R3–R4–R1.
- R3 and R4 connect to R5; R5 connects to the ISP over Gigabit Ethernet.
- The optional LACP EtherChannel between MLS1–MLS2 and MLS3–MLS4 requires additional physical links and is commented out in the base configuration.

## 03 · VLAN & IP Addressing Plan

Each department or service zone has a dedicated `/24` subnet. The HSRP virtual gateway is `.1`; the two multilayer switches use `.2` and `.3`.

### Left block · Corporate departments

| VLAN | Name | Subnet | Virtual gateway | HSRP Active |
|---:|---|---|---|---|
| 10 | MANAGEMENT | `10.10.10.0/24` | `10.10.10.1` | MLS1 |
| 20 | FINANCE | `10.10.20.0/24` | `10.10.20.1` | MLS2 |
| 30 | HR | `10.10.30.0/24` | `10.10.30.1` | MLS1 |
| 40 | CUSTOMER-SERVICES | `10.10.40.0/24` | `10.10.40.1` | MLS2 |
| 50 | IT | `10.10.50.0/24` | `10.10.50.1` | MLS1 |
| 60 | ACCOUNTING | `10.10.60.0/24` | `10.10.60.1` | MLS2 |
| 70 | BANK-OPERATIONS | `10.10.70.0/24` | `10.10.70.1` | MLS1 |
| 80 | ADMIN | `10.10.80.0/24` | `10.10.80.1` | MLS2 |

### Right block · Banking services and infrastructure

| VLAN | Name | Subnet | Virtual gateway | HSRP Active |
|---:|---|---|---|---|
| 100 | LOANS | `10.10.100.0/24` | `10.10.100.1` | MLS3 |
| 110 | BACKUP | `10.10.110.0/24` | `10.10.110.1` | MLS4 |
| 120 | SECURITY | `10.10.120.0/24` | `10.10.120.1` | MLS3 |
| 130 | ATM | `10.10.130.0/24` | `10.10.130.1` | MLS4 |
| 140 | VOICE | `10.10.140.0/24` | `10.10.140.1` | MLS3 |
| 150 | GUEST | `10.10.150.0/24` | `10.10.150.1` | MLS4 |
| 160 | DMZ | `10.10.160.0/24` | `10.10.160.1` | MLS3 |
| 170 | DNS-DHCP | `10.10.170.0/24` | `10.10.170.1` | MLS4 |
| 199 | MGMT-RIGHT | `10.10.4.0/24` | `10.10.4.1` | MLS3 |

### Reserved VLANs

| VLAN | Name | Purpose |
|---:|---|---|
| 998 | NATIVE-UNUSED | Unused native VLAN on trunks |
| 999 | BLACKHOLE | Unused access ports; ports are administratively shut down |

### Addressing conventions

- HSRP virtual IP: `10.10.X.1`
- MLS1 / MLS3 SVI address: `10.10.X.2`
- MLS2 / MLS4 SVI address: `10.10.X.3`
- DHCP clients start from `.20`.
- Router loopbacks: `10.10.255.1`–`10.10.255.5`.
- Multilayer-switch loopbacks: `10.10.255.11`–`10.10.255.14`.
- Left access-switch management addresses: `10.10.10.241`–`10.10.10.248`.
- Right access-switch management addresses: `10.10.4.241`–`10.10.4.248`.

## 04 · Routing & High Availability

### Inter-VLAN routing

The four multilayer switches provide Layer 3 switching through SVIs. Each department VLAN uses an HSRP virtual IP as its default gateway.

### HSRP

- MLS1 and MLS2 provide gateway redundancy for the left block.
- MLS3 and MLS4 provide gateway redundancy for the right block.
- Active gateway roles are distributed between the two switches.
- HSRP preemption and authentication are configured.
- HSRP hello traffic must be permitted through interface ACLs.

### Spanning Tree

Rapid-PVST+ is used for Layer 2 loop prevention. STP root placement follows the HSRP active switch for each VLAN:

- Priority `4096`: intended root bridge.
- Priority `8192`: secondary bridge.

This alignment keeps the preferred Layer 2 path consistent with the active Layer 3 gateway.

### RIPv2

RIPv2 is used across the routed infrastructure.

- Version 2.
- `no auto-summary`.
- Passive interfaces on loopbacks and non-routing-facing interfaces.
- Routed point-to-point links use `/30` subnets.
- R5 originates the default route toward the internal routing domain.

### Routed link addressing

| Connection | Subnet |
|---|---|
| R1–R2 | `10.10.1.4/30` |
| R2–R3 | `10.10.1.8/30` |
| R1–R4 | `10.10.1.12/30` |
| R3–R4 | `10.10.1.16/30` |
| R4–R5 | `10.10.1.20/30` |
| R4–MLS2 | `10.10.1.24/30` |
| R1–MLS1 | `10.10.1.28/30` |
| R1–MLS2 | `10.10.1.44/30` |
| R2–MLS3 | `10.10.1.32/30` |
| R3–MLS4 | `10.10.1.36/30` |
| R4–MLS1 | `10.10.1.40/30` |
| R2–MLS4 | `10.10.1.52/30` |
| R3–MLS3 | `10.10.1.48/30` |
| R3–R5 | `10.10.1.56/30` |
| R5–ISP | `203.0.113.0/30` |

## 05 · Network Services

### DHCP

The DHCP server is placed in VLAN 170.

| Parameter | Value |
|---|---|
| Hostname | SRV-DHCP |
| Address | `10.10.170.10/24` |
| Default gateway | `10.10.170.1` |
| DNS server | `10.10.170.11` |

DHCP pools are configured for the user and service VLANs, with client allocation starting at `.20`. The multilayer-switch SVIs relay DHCP requests using:

```cisco
ip helper-address 10.10.170.10
```

VLAN 170 and VLAN 199 use static addressing and do not have DHCP pools.

### DNS

| Parameter | Value |
|---|---|
| Hostname | SRV-DNS |
| Address | `10.10.170.11/24` |
| Default gateway | `10.10.170.1` |
| Domain | `oxabank.local` |

DNS records include:

- `dns.oxabank.local` → `10.10.170.11`
- `dhcp.oxabank.local` → `10.10.170.10`
- `www.oxabank.local` → `10.10.160.10`
- `r1`–`r5.oxabank.local` → router loopbacks
- `mls1`–`mls4.oxabank.local` → multilayer-switch loopbacks
- `isp.oxabank.local` → `203.0.113.2`

The `www` record reserves an address for a web server in the DMZ; deploy that server separately if web-service testing is required.

## 06 · Security Architecture

### Access-layer hardening

- Port Security with sticky MAC learning.
- Maximum of two secure MAC addresses per endpoint port.
- Violation mode: `restrict`.
- PortFast and BPDU Guard on endpoint ports.
- Unused ports assigned to VLAN 999 and shut down.
- VLAN 1 SVI administratively disabled.
- Native VLAN 998 on trunk links.
- DTP negotiation disabled with `switchport nonegotiate`.
- VTP configured in transparent mode.

### Management plane

- SSH version 2 configured.
- Telnet transport disabled on VTY lines.
- `SSH-ONLY` standard ACL restricts management source networks.
- Session timeouts and MOTD banners configured.

**Configuration caveat:** The supplied configuration explicitly does not create local usernames or passwords and uses `no login` on VTY lines. Therefore, SSH transport and source filtering are configured, but authenticated administrative access is not complete. Add local credentials and an appropriate VTY login method before considering management access secured.

### Inter-VLAN ACL policy

| ACL | Security objective |
|---|---|
| `GUEST-IN` | Prevent Guest clients from reaching internal bank subnets; allow intended external traffic |
| `DMZ-IN` | Prevent DMZ hosts from initiating connections into internal networks |
| `ATM-IN` | Limit ATM traffic to approved server and Bank Operations networks |
| `BACKUP-OUT` | Restrict access to the Backup VLAN to approved sources |
| `OUTSIDE-IN` | Filter inbound traffic on the ISP-facing interface |
| `SSH-ONLY` | Restrict device management access to approved source networks |

The ACLs include a permit for HSRP hello packets (`UDP/1985` to `224.0.0.2`) so gateway redundancy messages are not inadvertently blocked.

### NAT/PAT and ISP edge

R5 is the perimeter router:

- Outside interface: `203.0.113.1/30`.
- ISP interface: `203.0.113.2/30`.
- Default route points to the ISP.
- NAT overload translates internal private addresses to the outside interface address.
- `OUTSIDE-IN` filters inbound traffic.
- ISP loopback `8.8.8.8/32` is used as a simulated external ping target.

## 07 · DMZ & Server Zone

The DMZ is isolated in VLAN 160:

| Parameter | Value |
|---|---|
| VLAN | 160 |
| Subnet | `10.10.160.0/24` |
| Default gateway | `10.10.160.1` |
| Reserved web-server address | `10.10.160.10` |

The DNS and DHCP servers are kept in VLAN 170, separate from the DMZ. The DMZ ACL is designed to prevent new sessions from the DMZ toward internal banking networks while permitting the explicitly configured return traffic.

## 08 · Implementation Sequence

Follow the deployment sequence defined in the configuration notes:

1. ISP router.
2. Edge router R5.
3. Internal routers R1–R4.
4. Multilayer switches MLS1–MLS4.
5. The 16 access switches.
6. DHCP and DNS server services.
7. End devices and DHCP clients.
8. Connectivity, security and redundancy validation.

**Packet Tracer cabling notes**

- R5 connects to ISP using Gigabit Ethernet (`R5 G0/0` ↔ `ISP G0/0`).
- The DCE ends of serial links must be on the interfaces where `clock rate` is configured.
- Optional LACP requires two additional physical links between MLS1–MLS2 and two between MLS3–MLS4. The LACP configuration is commented out until those links are added.

## 09 · Verification & Testing

### Layer 2

```cisco
show vlan brief
show interfaces trunk
show spanning-tree summary
show spanning-tree vlan 20
show port-security interface fa0/1
show etherchannel summary
```

### Layer 3 and redundancy

```cisco
show ip interface brief
show ip route
show ip protocols
show ip rip database
show standby brief
```

### ACL and NAT

```cisco
show access-lists
show ip nat translations
show ip nat statistics
```

### Client tests

```text
ipconfig /renew
ipconfig /all
ping 10.10.X.1
ping 10.10.170.10
ping 8.8.8.8
ping dns.oxabank.local
nslookup www.oxabank.local
```

Replace `X` with the VLAN ID under test.

### Expected security-policy checks

| Test | Expected behavior |
|---|---|
| Guest → Finance / internal bank subnets | Blocked |
| Guest → simulated Internet | Allowed when routing and NAT are operational |
| DMZ → internal network (new connection) | Blocked |
| ATM → DNS/DHCP server VLAN | Allowed |
| ATM → Bank Operations VLAN | Allowed |
| ATM → Finance VLAN | Blocked |
| IT / Admin → Backup VLAN | Allowed |
| Finance → Backup VLAN | Blocked |

### High-availability checks

- Confirm one Active and one Standby router per HSRP group.
- Verify the expected Active switch for each VLAN.
- Test gateway failover by shutting down the active switch or its relevant SVI/uplink.
- Confirm that the standby switch takes over the virtual gateway.
- Check routing convergence after a routed-link failure.

Expected results depend on correct cabling, interface status, VLAN propagation, routing convergence and server configuration.

## 10 · Troubleshooting Guide

| Symptom | What to inspect |
|---|---|
| Client gets `169.254.x.x` | DHCP pool, helper address, VLAN assignment, DHCP ACL permits |
| Both HSRP peers show Active | HSRP hello ACL permit, group settings, VLAN trunk allowance |
| No inter-block connectivity | RIPv2 neighbors, routed `/30` addressing, interface status |
| No Internet simulation | R5 default route, NAT inside/outside, PAT rule, ISP link |
| Port becomes err-disabled | Port Security / BPDU Guard event and interface logs |
| Trunk is not forwarding expected VLAN | Native VLAN, allowed VLAN list, trunk mode, cabling |
| EtherChannel does not form | Optional physical links and matching LACP/trunk configuration |
| SSH access is unsuccessful | RSA keys, VTY ACL, credentials and login configuration |

## 11 · Technologies

| Area | Technologies |
|---|---|
| Switching | VLANs, 802.1Q, VTP Transparent |
| Layer 2 resilience | Rapid-PVST+, PortFast, BPDU Guard |
| Link aggregation | LACP (optional; requires additional cabling) |
| Layer 3 | SVIs, inter-VLAN routing, `/30` routed links |
| Gateway redundancy | HSRP |
| Dynamic routing | RIPv2 |
| Network services | DHCP, DHCP Relay, DNS |
| Security | Standard ACLs, Extended ACLs, Port Security, SSH |
| Perimeter | NAT Overload / PAT, outside ACL |
| Simulation | Cisco Packet Tracer 9.0.1 |

## 12 · Repository Structure

```text
OXA-BANK/
├── README.md
├── OXABANK.pkt
├── docs/
│   ├── topology.png
│   ├── vlan-ip-plan.md
│   └── testing-checklist.md
├── configs/
│   ├── routers/
│   │   ├── ISP.txt
│   │   ├── R1.txt
│   │   ├── R2.txt
│   │   ├── R3.txt
│   │   ├── R4.txt
│   │   └── R5.txt
│   ├── multilayer-switches/
│   │   ├── MLS1.txt
│   │   ├── MLS2.txt
│   │   ├── MLS3.txt
│   │   └── MLS4.txt
│   └── access-switches/
│       └── access-switch-configs.txt
└── screenshots/
    ├── topology-overview.png
    ├── hsrp-status.png
    ├── routing-table.png
    └── connectivity-tests.png
```

Adjust filenames to match the files actually committed to the repository.

## 13 · Skills Demonstrated

- Enterprise network segmentation and hierarchical design.
- Redundant Layer 3 gateway implementation.
- Dynamic routing and routed-core design.
- DHCP relay and centralized DNS.
- ACL-based traffic control for sensitive network zones.
- NAT/PAT and edge-router configuration.
- Layer 2 hardening and management-plane controls.
- Network validation, troubleshooting and failover testing.

---

<div align="center">

**OXA BANK · Enterprise Banking Network Infrastructure**

*Designed and simulated with Cisco Packet Tracer 9.0.1*

</div>
