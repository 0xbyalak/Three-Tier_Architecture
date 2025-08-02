# ISP Network IP Planning & Addressing Scheme

## 1. Network Overview
- **Total Users**: 1000 customers (PPPoE + Hotspot)
- **Segmentation**: Per-kecamatan dengan VLAN
- **VPN Sites**: Remote branches
- **Internal Departments**: 5 departments
- **Management**: Bandwidth control dan monitoring

## 2. IP Address Allocation Structure

### 2.1 Core Infrastructure (R1 - Core Router)
```
Network: 10.0.0.0/16 (Private Internal)
Public: 203.128.10.0/24 (Example ISP Block)

Core Links:
- R1-Internet: 203.128.10.1/30 (WAN)
- R1-R2 Link: 10.0.1.0/30
- R1-R3 Link: 10.0.1.4/30
- R1-R4 Link: 10.0.1.8/30
- Loopback R1: 10.255.255.1/32
```

### 2.2 Customer Networks (R2 - Distribution Router)
```
Customer Pool: 192.168.0.0/16
Subnet per Kecamatan: /20 (4094 hosts per area)

VLAN Structure:
VLAN 100: Kecamatan A - 192.168.0.0/20    (192.168.0.1 - 192.168.15.254)
VLAN 101: Kecamatan B - 192.168.16.0/20   (192.168.16.1 - 192.168.31.254)
VLAN 102: Kecamatan C - 192.168.32.0/20   (192.168.32.1 - 192.168.47.254)
VLAN 103: Kecamatan D - 192.168.48.0/20   (192.168.48.1 - 192.168.63.254)
VLAN 104: Kecamatan E - 192.168.64.0/20   (192.168.64.1 - 192.168.79.254)
```

#### PPPoE Pool Configuration
```
PPPoE Pools per Kecamatan:
- Pool A (VLAN 100): 192.168.0.10 - 192.168.0.210 (200 users)
- Pool B (VLAN 101): 192.168.16.10 - 192.168.16.210 (200 users)
- Pool C (VLAN 102): 192.168.32.10 - 192.168.32.210 (200 users)
- Pool D (VLAN 103): 192.168.48.10 - 192.168.48.210 (200 users)
- Pool E (VLAN 104): 192.168.64.10 - 192.168.64.210 (200 users)
```

#### Hotspot Pool Configuration
```
Hotspot Pools per Kecamatan:
- Hotspot A: 192.168.1.0/24 (254 users)
- Hotspot B: 192.168.17.0/24 (254 users)
- Hotspot C: 192.168.33.0/24 (254 users)
- Hotspot D: 192.168.49.0/24 (254 users)
- Hotspot E: 192.168.65.0/24 (254 users)
```

### 2.3 Server Network (R3 - Server Distribution)
```
Server Network: 10.10.0.0/16

Department VLANs:
VLAN 200: IT Department     - 10.10.1.0/24
VLAN 201: Finance          - 10.10.2.0/24
VLAN 202: Operations       - 10.10.3.0/24
VLAN 203: Customer Service - 10.10.4.0/24
VLAN 204: Management       - 10.10.5.0/24
```

#### Critical Servers
```
NMS Server:        10.10.1.10/24
AAA/Radius:        10.10.1.11/24
Billing Server:    10.10.1.12/24
DNS Primary:       10.10.1.13/24
DNS Secondary:     10.10.1.14/24
DHCP Server:       10.10.1.15/24
File Server:       10.10.1.16/24
Backup Server:     10.10.1.17/24
```

### 2.4 VPN Network (R4 - VPN Gateway)
```
VPN Network: 172.16.0.0/16

Site-to-Site VPN:
Branch A: 172.16.1.0/24
Branch B: 172.16.2.0/24
Branch C: 172.16.3.0/24
Branch D: 172.16.4.0/24

VPN Pool (Remote Users): 172.16.100.0/24
```

### 2.5 Management Network
```
Management VLAN: 10.254.0.0/24
VLAN 999: Management Network

Device Management IPs:
R1 Management: 10.254.0.1/24
R2 Management: 10.254.0.2/24
R3 Management: 10.254.0.3/24
R4 Management: 10.254.0.4/24
Switch Mgmt:   10.254.0.10-20/24
AP Management: 10.254.0.30-50/24
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

## 7. Implementation Notes

### 7.1 Scalability Considerations
- Each kecamatan can expand to 4000+ users
- VPN network allows 65,000 sites
- Server network can accommodate growth
- Management network separate for security

### 7.2 Redundancy Planning
- Dual-homed internet connections
- Server clustering for critical services
- Backup power and cooling systems
- Configuration backups automated

### 7.3 Documentation Standards
- All changes logged in change management
- Network diagrams updated real-time
- IP allocation tracked in IPAM system
- Bandwidth utilization monitored 24/7