# Arsitektur three-tier

Dokumentasi ini merupakan bagian dari portofolio project simulasi infrastruktur dengan arsitektur three-tier, manajemen IP, autentikasi pengguna, bandwidth management, serta monitoring terpusat menggunakan GNS3 dan VMware.


## Project Summary

- **Target:** Simulasi Infrastruktur ISP Kabupaten (1000+ pelanggan)
- **Topologi:** Three-Tier Architecture (Core, Distribution, Access)
- **Teknologi:** BGP, OSPF, PPPoE, Hotspot, VLAN, VPN, FreeRADIUS, NuxxBill, Zabbix, phpIPAM
- **Tools:** GNS3 (Router Simulation), VMware (Server Deployment)


## Network Topology 

<img width="1177" height="610" alt="image" src="https://github.com/user-attachments/assets/32152a02-0684-4f7a-9c11-f8e44c5d621e" />


## Three-Tier Architecture

### 1. **Core Layer (R1 - Core Router)**
- BGP peering ke upstream ISP
- OSPF Backbone Area 0
- Firewall + NAT + Traffic Engineering
- VPN Termination Point (R4 link)

### 2. **Distribution Layer (R2, R3)**
- R2: PPPoE Server, Hotspot, VLAN segmentation per kecamatan
- R3: VLAN untuk internal department + Server Distribution (NMS, AAA, Billing)
- Queue Tree untuk bandwidth control per pelanggan

### 3. **Access Layer**
- Switches & AP untuk koneksi pelanggan akhir (RT/RW Net, Hotspot Public)
- VLAN trunk ke distribution routers
- DHCP for Hotspot, Static via RADIUS for PPPoE


## IP Address Plan Summary

| Segment | IP Range | Description |
|---------|----------|-------------|
| Core Links | 10.0.1.0/30 - 10.0.1.12/30 | R1-R2, R1-R3, R1-R4 |
| Loopback R1 | 10.255.255.1/32 | OSPF Router-ID |
| PPPoE Pool | 10.0.0.0/16 | Private IPs |
| Customer VLAN | 192.168.0.0/16 (/20 per kecamatan) | VLAN 100-104 |
| Hotspot VLAN | 192.168.X.0/24 | Dynamic Pool per kecamatan |
| Server Network | 10.10.0.0/16 | Internal servers |
| VPN Network | 172.16.0.0/16 | Site-to-site & remote users |
| Management VLAN | 10.254.0.0/24 | Device OOB Management |

**Lebih jelasnya ada di [IP Planning](https://github.com/0xbyalak/Three-Tier_Architecture/blob/9e5a4d0868f7020066e3f15ea5f9f5797e7f153f/ip_planning.md)**

## GNS3 & VMware Simulation Guide

### GNS3 (Router Simulation)
1. Import MikroTik CHR image ke GNS3.
2. Buat topologi Core (R1), Distribution (R2, R3), VPN Router (R4).
3. Konfigurasikan BGP peering di R1 simulated ISP Cloud.
4. Konfigurasikan OSPF Backbone (R1-R2-R3-R4) Area 0.
5. Buat VLAN interface di R2 untuk PPPoE dan Hotspot pelanggan.
6. Implementasi Queue Tree di R2 untuk bandwidth shaping.
7. Konfigurasi IP link sesuai IP Plan.

### VMware (Server Deployment)
1. Deploy Ubuntu Server di VMware untuk NMS, AAA/Billing, phpIPAM.
2. Install & setup:
   - Zabbix Server + Agent
   - FreeRADIUS + NuxxBill for PPPoE/Hotspot Auth & Billing
   - phpIPAM for IP Address Documentation
   - Syslog-ng for log collection
3. Hubungkan server ke R3 di VLAN server segment.
4. Setup SNMP monitoring untuk semua router di Zabbix.
5. Konfigurasikan DHCP untuk Hotspot VLAN, RADIUS untuk PPPoE user.


## Goals of this Simulation
- Demonstrate full-scale ISP design with modular architecture.
- Show practical implementation of AAA, IP Management, Bandwidth Control.
- Provide realistic NOC Monitoring dashboard (Zabbix + Grafana).
- Simulate failover BGP & OSPF scenarios (link failure, redundancy tests).

