# MikroTik Dual-WAN Router with VLANs and PCC Load Balancing

This configuration sets up a MikroTik router to support:
- Dual-WAN connections (ISP1 and ISP2)
- VLAN trunking with separate subnets for management and users
- DHCP servers for each VLAN
- PCC (Per Connection Classifier) load balancing across both ISPs
- Policy-based routing
- Basic firewall security

---

## 📦 Features

- 🔌 **Dual-WAN failover and load balancing** using PCC (per-connection-classifier)
- 🌐 **Three VLANs**: Management (VLAN 1), User VLAN 10, User VLAN 20
- 🛜 **DHCP server** for each VLAN
- 🔒 **Basic firewall rules** for input filtering
- 🔀 **Policy-based routing** via separate routing tables
- 🧰 Designed for **SME network environments** with trunk ports

---

## 🔧 Interface Overview

| Interface | Name  | Description                  |
|-----------|-------|------------------------------|
| ether1    | ISP1  | Primary Internet Connection  |
| ether2    | ISP2  | Secondary Internet Connection|
| ether3    | LAN   | Connected to switch (Trunk)  |
| VLAN 1    | Mgmt  | Management VLAN (172.31.10.0/24) |
| VLAN 10   | V10   | User VLAN 10 (192.168.10.0/24) |
| VLAN 20   | V20   | User VLAN 20 (192.168.20.0/24) |

---

## 🌍 IP Addressing

| VLAN  | Subnet            | Gateway IP      | DHCP Pool                  |
|-------|-------------------|------------------|----------------------------|
| Mgmt  | 172.31.10.0/24    | 172.31.10.1      | 172.31.10.2 – .254         |
| V10   | 192.168.10.0/24   | 192.168.10.1     | 192.168.10.2 – .254        |
| V20   | 192.168.20.0/24   | 192.168.20.1     | 192.168.20.2 – .254        |
| ISP1  | 192.168.19.128/25 | 192.168.19.129   | —                          |
| ISP2  | 192.168.19.0/25   | 192.168.19.101   | —                          |

---

## 📡 Load Balancing Logic (PCC)

Traffic from the LAN is split across both WANs:
- **50/50 Load Balancing** using `per-connection-classifier=both-addresses:2/X`
- **Routing Marks** assigned to connections
- **Separate Routing Tables**

### Mangle Chain Summary

- Accept internal WAN-to-WAN traffic
- Mark incoming connections from ISP1/ISP2
- Mark new connections from LAN using PCC
- Apply routing marks based on connection marks

---

## 🔥 Firewall

### Input Filter

| Action | Chain | Description                  |
|--------|-------|------------------------------|
| Drop   | input | Drop invalid connections     |
| Accept | input | Allow established/related    |

### NAT

| Chain    | Action      | Out Interface |
|----------|-------------|---------------|
| srcnat   | Masquerade  | ISP1          |
| srcnat   | Masquerade  | ISP2          |

---
