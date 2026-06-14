# Packet Tracer Lab — VLANs, EtherChannel, HSRP, DHCP Snooping & Port Security

## Quick Navigation

[![Open Lab](https://img.shields.io/badge/Open%20Lab-.pkt-blue)](packet-tracer-vlan-hsrp-lab.pkt)
[![Docs Folder](https://img.shields.io/badge/Folder-Docs%2F-lightgrey)](Docs/)
[![Validation Outputs](https://img.shields.io/badge/Folder-Validation%2F-green)](Validation/)

### Device Configurations
[![dhcpServer+GW](https://img.shields.io/badge/Config-dhcpServer_GW.txt-blue)](Docs/dhcpServer_GW.txt)
[![Router1](https://img.shields.io/badge/Config-Router1.txt-blue)](Docs/Router1.txt)
[![Router2](https://img.shields.io/badge/Config-Router2.txt-blue)](Docs/Router2.txt)
[![ISP](https://img.shields.io/badge/Config-ISP.txt-blue)](Docs/ISP.txt)
[![R_Deleg](https://img.shields.io/badge/Config-R_Deleg.txt-blue)](Docs/R_Deleg.txt)
[![SW_Sede1](https://img.shields.io/badge/Config-SW_Sede1.txt-blue)](Docs/SW_Sede1.txt)
[![SW_Sede2](https://img.shields.io/badge/Config-SW_Sede2.txt-blue)](Docs/SW_Sede2.txt)
[![Switch3](https://img.shields.io/badge/Config-Switch3.txt-blue)](Docs/Switch3.txt)
[![Switch4](https://img.shields.io/badge/Config-Switch4.txt-blue)](Docs/Switch4.txt)
[![SW_Deleg](https://img.shields.io/badge/Config-SW_Deleg.txt-blue)](Docs/SW_Deleg.txt)


## 1. Overview

This project was developed as part of UC01478 — *Computer Network Configuration*, within the CET in Cybersecurity and Network Infrastructure at IEFP (March 2026).

The lab simulates an enterprise network using Cisco Packet Tracer, implementing Layer 2 and Layer 3 switching, redundancy, segmentation, centralized DHCP, and security hardening across multiple devices.

The topology includes a Headquarters site and a Remote Branch connected through routed links.

---

## 2. Network Topology

### Headquarters (Sede)

- 1× Cisco 3560 Layer 3 Switch — **dhcpServer+GW**
  - Inter-VLAN routing
  - DHCP server
  - Default gateway for all VLANs

- 2× Cisco 2960 Access Switches — **SW_Sede1**, **SW_Sede2**
  - User access
  - EtherChannel uplinks

- 2× Cisco 2960 Distribution Switches — **Switch3**, **Switch4**

- 2× Cisco 2901 Routers — **Router1**, **Router2**
  - HSRP redundancy cluster

- 1× Cisco 2901 Router — **ISP**
  - Internet gateway

- 1× Web Server (external)

### Remote Branch (Delegação)

- 1× Cisco 2901 Router — **R_Deleg**
- 1× Cisco 2960 Switch — **SW_Deleg**
  - DHCP Snooping
  - Port Security
- 1× Laptop — DHCP client
- 1× Rogue DHCP Server — blocked by security policies

---

## 3. Technologies Implemented

| Technology | Description |
|---|---|
| VLANs | VLANs 110, 120, 130, 140 (users) and VLAN 367 (management/native) |
| EtherChannel (LACP) | dhcpServer+GW ↔ SW_Sede1 (4 links) |
| EtherChannel (PAgP) | dhcpServer+GW ↔ SW_Sede2 (4 links) |
| Inter-VLAN Routing | Performed by the L3 switch using SVIs |
| DHCP Server | Centralized DHCP for all VLANs and the branch |
| DHCP Relay | `ip helper-address` on R_Deleg |
| HSRP | Router1 (Active, priority 110) and Router2 (Standby, priority 90) |
| Static Routing | Default routes and ISP routes for internal networks |
| DHCP Snooping | Enabled on SW_Deleg; rogue server blocked |
| Port Security | Max 1 MAC, sticky, violation = shutdown |
| SSH Access | SSHv2, local authentication, domain iefp.pt |
| Security Hardening | enable secret, console password, VTY SSH-only, MOTD banner |

---

## 4. IP Addressing

### Headquarters VLANs (/27)

| VLAN | Subnet | Gateway | DHCP Range | Excluded (last 5) |
|---|---|---|---|---|
| 110 | 10.0.0.0/27 | 10.0.0.30 | 10.0.0.1–25 | 10.0.0.26–30 |
| 120 | 10.0.0.32/27 | 10.0.0.62 | 10.0.0.33–57 | 10.0.0.58–62 |
| 130 | 10.0.0.64/27 | 10.0.0.94 | 10.0.0.65–89 | 10.0.0.90–94 |
| 140 | 10.0.0.96/27 | 10.0.0.126 | 10.0.0.97–121 | 10.0.0.122–126 |
| 367 (Mgmt) | 10.0.0.128/27 | 10.0.0.158 | — | 10.0.0.154–158 |

### Remote Branch (/24)

| Subnet | Gateway | DHCP Range | Excluded |
|---|---|---|---|
| 192.168.100.0/24 | 192.168.100.254 | 192.168.100.1–249 | 192.168.100.250–254 |

---

## 5. External / WAN Networks

| Network | Purpose |
|---|---|
| 5.5.5.0/28 | Switch3 ↔ Router1/Router2 |
| 5.5.5.16/28 | Switch4 ↔ Router1/Router2 ↔ ISP |
| 5.5.5.200/30 | ISP ↔ Web Server |
| 192.168.200.0/30 | R_Deleg ↔ dhcpServer+GW |

---

## 6. HSRP Configuration Summary

| Parameter | Router1 | Router2 |
|---|---|---|
| Role | Active | Standby |
| Priority | 110 | 90 |
| Virtual IP | 5.5.5.15 | 5.5.5.15 |
| Interface | Gi0/0 | Gi0/0 |

---

## 7. Security Configuration (applied to all devices)

```
enable secret 12345

line con 0
 password 123

line vty 0 4
 transport input ssh
 login local

ip domain-name iefp.pt
username teste privilege 15 secret 1234
crypto key generate rsa 1024
ip ssh version 2

banner motd # Unauthorized access is prohibited #
```

---

## 8. Challenges and Lessons Learned

- EtherChannel requires identical configuration on both sides; mismatches silently break the bundle.
- Native VLAN must match across all trunks to avoid CDP warnings and STP inconsistencies.
- Routed ports (`no switchport`) remove interfaces from the VLAN domain entirely.
- DHCP Snooping inserts Option 82 by default, which breaks DHCP relay; disabling it resolves the issue.
- Only uplink/router-facing ports should be trusted under DHCP Snooping.
- HSRP requires consistent VLAN availability and L2 reachability for proper Active/Standby election.

---

## 9. Repository Structure

```
packet-tracer-vlan-hsrp-lab/
│
├── README.md
├── LICENSE
├── lab.pkt
│
├── docs/
│   ├── topology.png
│   └── addressing-table.md
│
├── configs/
│   ├── dhcpServer_GW.txt
│   ├── SW_Sede1.txt
│   ├── SW_Sede2.txt
│   ├── Switch3.txt
│   ├── Switch4.txt
│   ├── Router1.txt
│   ├── Router2.txt
│   ├── ISP.txt
│   ├── R_Deleg.txt
│   └── SW_Deleg.txt
│
└── validation/
    └── ping-tests.txt
```

---

## 10. Tools Used

- Cisco Packet Tracer 8.x
- Cisco IOS (2960, 3560, 2901)

---

## 11. Author

**Filipe Castanheira**
CET Cybersecurity & Network Infrastructure — IEFP
GitHub: [filipecastanheira-rgb](https://github.com/filipecastanheira-rgb)
