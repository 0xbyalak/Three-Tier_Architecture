## 1. Network Overview
- **Total Users**: 1000 customers (PPPoE + Hotspot)
- **Segmentation**: Per-kecamatan dengan VLAN
- **VPN Sites**: Remote branches
- **Internal Departments**: 5 departments
- **Management**: Bandwidth control dan monitoring

## 2. IP Address Allocation Structure

### 2.1 R1 - Core Router
```
Network: 10.0.0.0/22 (Private Internal - 1022 hosts)
Public: 203.128.10.0/29 (ISP Block - 6 usable)

Core Links:
- R1-Internet: 203.128.10.1/30 (WAN)
- R1-R2 Link: 10.0.0.0/30
- R2-R3 Link: 10.0.0.4/30
- R3-R1 Link: 10.0.0.8/30
- Loopback R1: 10.255.255.1/32
- Loopback R2: 10.255.255.2/32
- Loopback R3: 10.255.255.3/32
```

### 2.2 R2 - Distribution Router
```
Customer Pool: 192.168.0.0/20 (Optimized for 4094 hosts total)
Subnet per Kecamatan: /23 (510 hosts per area)

VLAN Structure:
VLAN 100: Kecamatan A - 192.168.0.0/23    (192.168.0.1 - 192.168.1.254)
VLAN 101: Kecamatan B - 192.168.2.0/23    (192.168.2.1 - 192.168.3.254)
VLAN 102: Kecamatan C - 192.168.4.0/23    (192.168.4.1 - 192.168.5.254)
VLAN 103: Kecamatan D - 192.168.6.0/23    (192.168.6.1 - 192.168.7.254)
VLAN 104: Kecamatan E - 192.168.8.0/23    (192.168.8.1 - 192.168.9.254)
```

#### PPPoE Pool Configuration
```
PPPoE Pools per Kecamatan (200 users):
- Pool A (VLAN 100): 192.168.0.10 - 192.168.0.210 
- Pool B (VLAN 101): 192.168.2.10 - 192.168.2.210 
- Pool C (VLAN 102): 192.168.4.10 - 192.168.4.210 
- Pool D (VLAN 103): 192.168.6.10 - 192.168.6.210 
- Pool E (VLAN 104): 192.168.8.10 - 192.168.8.210
```

#### Hotspot Pool Configuration
```
Hotspot Pools per Kecamatan (Subnet terpisah untuk isolasi):
- Hotspot A: 192.168.1.0/25 (126 users) - dalam range VLAN 100
- Hotspot B: 192.168.3.0/25 (126 users) - dalam range VLAN 101
- Hotspot C: 192.168.5.0/25 (126 users) - dalam range VLAN 102
- Hotspot D: 192.168.7.0/25 (126 users) - dalam range VLAN 103
- Hotspot E: 192.168.9.0/25 (126 users) - dalam range VLAN 104
```

### 2.3 R3 - Distribution Router
```
Server Network: 10.1.0.0/24 (Optimized for 254 hosts)

Department VLANs:
VLAN 200: IT Department     - 10.1.1.0/26 (62 hosts)
VLAN 201: Finance          - 10.1.1.64/26 (62 hosts)
VLAN 202: Operations       - 10.1.1.128/26 (62 hosts)
VLAN 203: Customer Service - 10.1.1.192/26 (62 hosts)
VLAN 204: Management       - 10.1.2.0/26 (62 hosts)
```

#### Critical Servers
```
NMS Server          :        10.1.0.10/24
AAA & Billing server:        10.1.0.11/24
```

### 2.4 R4 - VPN Gateway
```
VPN Network: 172.16.0.0/24 (Optimized untuk 254 hosts)

Site-to-Site VPN:
Branch A: 172.16.0.0/27   (30 hosts)
Branch B: 172.16.0.32/27  (30 hosts)
Branch C: 172.16.0.64/27  (30 hosts)
Branch D: 172.16.0.96/27  (30 hosts)

VPN Pool (Remote Users): 172.16.0.128/25 (126 users)
```

### 2.5 Management Network
```
Management VLAN: 10.254.0.0/26 (62 hosts - cukup untuk perangkat)
VLAN 999: Management Network

Device Management IPs:
R1 Management: 10.254.0.1/26
R2 Management: 10.254.0.2/26
R3 Management: 10.254.0.3/26
R4 Management: 10.254.0.4/26
Switch Mgmt:   10.254.0.10-20/26
AP Management: 10.254.0.21-30/26
```

## 3. Bandwidth Management Scheme

### 3.1 Customer Bandwidth Tiers
```
Paket Residential:
- Basic:    5 Mbps Down / 1 Mbps Up
- Standard: 10 Mbps Down / 2 Mbps Up
- Premium:  20 Mbps Down / 5 Mbps Up
- Ultimate: 50 Mbps Down / 10 Mbps Up

Paket Corporate:
- Small:   10 Mbps Down / 10 Mbps Up
- Medium:  50 Mbps Down / 25 Mbps Up
- Large:   100 Mbps Down / 50 Mbps Up
```

### 3.2 QoS Classes
```
Priority Classes:
1. Voice/Video Call (EF) - 10% bandwidth guarantee
2. Critical Apps (AF41) - 20% bandwidth guarantee
3. Business Apps (AF31) - 30% bandwidth guarantee
4. Standard Traffic (BE) - 40% bandwidth
```

### 3.3 Per-VLAN Bandwidth Allocation
```
Kecamatan Bandwidth Limits:
VLAN 100 (Kecamatan A): 2 Gbps aggregate
VLAN 101 (Kecamatan B): 2 Gbps aggregate
VLAN 102 (Kecamatan C): 1.5 Gbps aggregate
VLAN 103 (Kecamatan D): 1.5 Gbps aggregate
VLAN 104 (Kecamatan E): 1 Gbps aggregate
```

## 4. DHCP Configuration

### 4.1 Customer DHCP Pools
```
PPPoE: Static IP assignment via RADIUS
Hotspot: Dynamic DHCP with lease time 4 hours

DHCP Options:
- DNS: 8.8.8.8, 8.8.4.4
- Gateway: VLAN interface IP
- Lease Time: 4 hours (hotspot), 24 hours (corporate)
```

### 4.2 Internal DHCP
```
Department Networks:
- Lease Time: 8 hours
- DNS: Internal DNS servers
- WINS: If required for legacy systems
```

## 5. Routing Configuration

### 5.1 Interior Routing (OSPF)
```
OSPF Areas:
Area 0 (Backbone): R1, R2, R3, R4 interconnects
Area 1: Customer networks (R2)
Area 2: Server networks (R3)
Area 3: VPN networks (R4)
```

### 5.2 Exterior Routing (BGP)
```
BGP Configuration:
AS Number: 65001 (Private for lab)
Upstream Providers: 2 providers for redundancy
Route Filtering: Prevent route leaks
```

## 6. Security Implementation

### 6.1 Access Control Lists
```
Customer to Internet: Permit all (with bandwidth limit)
Customer to Servers: Deny direct access
Management VLAN: Full access for monitoring
Inter-VLAN: Controlled via firewall rules
```

### 6.2 Network Monitoring
```
SNMP Community: 
- Read: "publicISP"
- Write: "privateISP"

Syslog Server: 10.10.1.18
NTP Server: 10.10.1.19
```
