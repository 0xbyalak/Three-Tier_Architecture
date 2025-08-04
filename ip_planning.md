## Topology
<img width="1919" height="999" alt="image" src="https://github.com/user-attachments/assets/9d871d5c-79e3-43c6-a274-198669720e12" />

## 1. Network Overview
- **Total Users**: 1000 customers (PPPoE + Hotspot)
- **Topology**: Hierarchical with RA as edge router, R1 as core, R2 for customers, R3 for servers
- **Segmentation**: Per-kecamatan dengan VLAN
- **VPN Services**: Site-to-site, Remote Access
- **Management**: Centralized monitoring dan bandwidth control

## 2. IP Address Allocation Structure

### 2.1 Core Infrastructure & Interconnects
```
Public Network: 203.128.10.0/29 (ISP Block - 6 usable IPs)

WAN & External Links:
- Cloud1 - WAN: 203.128.10.1/30
- R1 Public Interface: 203.128.10.2/29

Core Router Interconnects:
- R1-R2 Link: 10.0.0.0/30 (R1: 10.0.0.1, R2: 10.0.0.2)
- R1-R3 Link: 10.0.0.4/30 (R1: 10.0.0.5, R3: 10.0.0.6)

Area Backbone (Redundant Path):
- R2-R3 Link: 10.0.0.8/30 (R2: 10.0.0.9, R3: 10.0.0.10)

Router Loopbacks:
- R1 Loopback: 10.255.255.1/32
- R2 Loopback: 10.255.255.2/32
- R3 Loopback: 10.255.255.3/32
```

### 2.2 VPN & Remote Access Network
```
VPN Network: 172.16.0.0/24 (256 addresses untuk VPN services)

VPN Pool Allocation:
- Site-to-Site VPN: 172.16.0.1-172.16.0.49 (50 sites)
- Remote Access Pool: 172.16.0.100-172.16.0.199 (100 concurrent users)
- Management VPN: 172.16.0.200-172.16.0.220 (20 admin connections)
- Reserved/Future: 172.16.0.221-172.16.0.254 (Expansion)

```

### 2.3 Customer Networks (R2 Distribution)
```
Customer Pool: 192.168.0.0/20 (4094 total customer addresses)

Customer VLANs Distribution:
- VLAN 100: Kecamatan A - 192.168.0.0/23 (510 hosts, Gateway: 192.168.0.1)
- VLAN 101: Kecamatan B - 192.168.2.0/23 (510 hosts, Gateway: 192.168.2.1)
- VLAN 102: Kecamatan C - 192.168.4.0/23 (510 hosts, Gateway: 192.168.4.1)
- VLAN 103: Kecamatan D - 192.168.6.0/23 (510 hosts, Gateway: 192.168.6.1)
```

#### Customer Device Assignments
```
- PC2 (SW1 - VLAN 100): 192.168.0.10/23 - Kecamatan A Customer
- PC3 (SW1 - VLAN 101): 192.168.2.10/23 - Kecamatan B Customer
- PC4 (SW2 - VLAN 102): 192.168.4.10/23 - Kecamatan C Customer
- PC5 (SW2 - VLAN 103): 192.168.6.10/23 - Kecamatan D Customer
```

#### PPPoE Pool Configuration
```
PPPoE Dynamic Pools (per Kecamatan):
- Pool A (VLAN 100): 192.168.0.20 - 192.168.0.250 (230 users)
- Pool B (VLAN 101): 192.168.2.20 - 192.168.2.250 (230 users)
- Pool C (VLAN 102): 192.168.4.20 - 192.168.4.250 (230 users)
- Pool D (VLAN 103): 192.168.6.20 - 192.168.6.250 (230 users)

PPPoE Configuration:
- Authentication: Via RADIUS server
- DNS Servers: 8.8.8.8, 8.8.4.4
- MTU: 1492 bytes
- Session Timeout: 24 hours
```

#### Hotspot Pool Configuration
```
Hotspot Networks:
- Hotspot A: 192.168.1.0/25 (126 users) - VLAN 100 coverage
- Hotspot B: 192.168.3.0/25 (126 users) - VLAN 101 coverage
- Hotspot C: 192.168.5.0/25 (126 users) - VLAN 102 coverage
- Hotspot D: 192.168.7.0/25 (126 users) - VLAN 103 coverage
```

### 2.4 Server & Admin Network (R3 Distribution)  
```
Server Network: 10.1.0.0/24 (254 hosts untuk servers dan admin)

Critical Server Assignments:
- Main Server : 10.1.0.2/24 - NMS/Billing/RADIUS
- PC_Admin    : 10.1.0.100/24 - Network Administrator
```

#### Department VLAN Configuration (via SW3)
```
Departmental Networks:
VLAN 200: IT Department     - 10.1.1.0/26 (62 hosts, Gateway: 10.1.1.1)
VLAN 201: Finance          - 10.1.1.64/26 (62 hosts, Gateway: 10.1.1.65)
VLAN 202: Operations       - 10.1.1.128/26 (62 hosts, Gateway: 10.1.1.129)
VLAN 203: Customer Service - 10.1.1.192/26 (62 hosts, Gateway: 10.1.1.193)
VLAN 204: Management       - 10.1.2.0/26 (62 hosts, Gateway: 10.1.2.1)

Department Access Control:
- Full internet access for all departments
- Restricted server access based on department
- Inter-department communication via firewall rules
```

### 2.5 Management Network (VLAN 999 - Out-of-Band)
```
Management VLAN: 10.254.0.0/26 (62 addresses - all network devices)
VLAN 999: Dedicated management untuk monitoring dan maintenance

Core Device Management:
- R1 Management: 10.254.0.2/26
- R2 Management: 10.254.0.3/26
- R3 Management: 10.254.0.4/26

Switch Management:
- SW1 Management: 10.254.0.10/26
- SW2 Management: 10.254.0.11/26  
- SW3 Management: 10.254.0.12/26

Server Management:
- Main Server (R3): Management via 10.1.0.2
- PC_Admin: Management station at 10.1.0.100
```

## 3. Routing Protocol Configuration

### 3.1 OSPF Area Design
```
OSPF Process ID: 1
Router ID: Menggunakan loopback addresses

Area 0 (Backbone): 
- RA-R1 link (10.0.0.0/30)
- R1-R2 link (10.0.0.4/30)
- R1-R3 link (10.0.0.8/30)
- R2-R3 link (10.0.0.12/30)
- All loopbacks

Area 1 (Customer Networks):
- R2-SW1/SW2 links
- Customer VLANs 100-103
- Customer subnets (192.168.0.0/20)

Area 2 (Server Networks):
- R3-Server link
- R3-SW3 link  
- Server networks (10.1.0.0/24)
- Department VLANs 200-204

Area 3 (VPN/Remote):
- VPN networks (172.16.0.0/24)
- Remote access pools
```

### 3.2 BGP Configuration
```
BGP AS: 65001 (Private untuk lab environment)
eBGP Neighbor: Cloud1 (ISP Upstream)
Route Advertisement: 
- Customer networks (192.168.0.0/20)
- Server networks (10.1.0.0/24)
Route Filtering: Prevent private IP leakage
```

## 4. Bandwidth Management

### 4.1 Customer Service Tiers
```
Residential Packages:
- Basic:    5 Mbps Down / 1 Mbps Up   - Pool: 40% customers
- Standard: 10 Mbps Down / 2 Mbps Up  - Pool: 35% customers  
- Premium:  20 Mbps Down / 5 Mbps Up  - Pool: 20% customers
- Ultimate: 50 Mbps Down / 10 Mbps Up - Pool: 5% customers

Corporate Packages:
- Small Office:  10 Mbps symmetric
- Medium Office: 50 Mbps symmetric
- Large Office:  100 Mbps symmetric
```

### 4.2 Per-VLAN Bandwidth Allocation
```
Aggregate Limits per Switch:
SW1 (400 customers max):
- VLAN 100: 2.5 Gbps aggregate limit
- VLAN 101: 2.5 Gbps aggregate limit

SW2 (400 customers max):
- VLAN 102: 2 Gbps aggregate limit
- VLAN 103: 2 Gbps aggregate limit

Uplink Capacities:
- Cloud1-RA: 10 Gbps (Internet uplink)
- R2-SW1: 10 Gbps fiber
- R2-SW2: 10 Gbps fiber
- R3-Server: 1 Gbps dedicated
```

## 5. Security Implementation

### 5.1 Access Control Lists
```
Customer Traffic Rules:
- Permit internet access with bandwidth limits
- Deny direct access to server networks
- Allow DNS queries to designated servers
- Block P2P during business hours (optional)

Inter-VLAN Rules:
- Management VLAN: Full access to all networks
- Department VLANs: Restricted server access
- Customer VLANs: Internet only, no inter-VLAN
- VPN Network: Controlled access based on user groups
```

### 5.2 Network Monitoring & Management
```
SNMP Configuration:
- Community Strings: "publicISP" (RO), "privateISP" (RW)
- SNMP v3: For secure management access
- OID Monitoring: Interface stats, CPU, memory

Logging:
- Syslog Server: 10.1.0.2 (Main Server)
- Log Levels: Warnings and above
- Rotation: Daily with 30-day retention

Time Synchronization:
- NTP Server: 10.1.0.2 (Main Server)
- External NTP: pool.ntp.org (backup)
```

## 6. High Availability & Redundancy

### 6.1 Link Redundancy
```
Primary Paths:
- Internet: Cloud1 → RA → R1 → R2/R3
- Customer Access: R2 → SW1/SW2
- Server Access: R3 → Server/SW3

Backup Paths:
- Area Backbone: R2 ↔ R3 (direct link)
- Load Balancing: Equal-cost multipath via OSPF
- Failover: Automatic via routing protocol convergence
```

### 6.2 Service Redundancy
```
Critical Services:
- RADIUS/AAA: Primary on Server1 (172.16.0.10), Backup on Main Server (10.1.0.2)
- DNS: Primary on Main Server, Secondary on Router (cache)
- DHCP: Main Server with backup configuration on routers
- NMS: Primary monitoring with distributed collectors
```

## 7. Implementation Checklist

### 7.1 Device Configuration Order
```
1. Configure router loopbacks and management IPs
2. Setup inter-router links and OSPF
3. Configure VLANs on switches
4. Setup customer PPPoE and hotspot services
5. Configure server services and department VLANs
6. Implement security policies and monitoring
7. Test connectivity and failover scenarios
```

### 7.2 Verification Commands
```
Router Status:
- show ip ospf neighbor
- show ip route
- show ip bgp summary

Switch Status:
- show vlan brief
- show interface status
- show spanning-tree

Service Status:
- show pppoe session
- show ip dhcp binding
- show access-lists
```

### 7.3 Scalability Notes
```
Growth Capacity:
- Each kecamatan VLAN: 510 potential customers
- Total customer capacity: 2040 customers (100% growth)
- VPN services: 100 concurrent + 30 site-to-site
- Department networks: 310 total employees
- Management overhead: <5% of total IP space

Future Expansion:
- Additional switches dapat ditambahkan ke R2/R3
- New VLANs untuk kecamatan baru
- Server clustering untuk high availability
- Multiple internet uplinks untuk redundancy
```
