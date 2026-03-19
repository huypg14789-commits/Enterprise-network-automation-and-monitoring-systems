# Enterprise Network Automation & Monitoring Infrastructure

## Project Overview
This project demonstrates the design, deployment, and operation of a Highly Available (HA) Enterprise Network Architecture leveraging modern **NetDevOps** methodologies. Built upon the Cisco hierarchical 3-tier model (Core, Distribution, Access) and integrating a dedicated **Out-of-Band (OOB) Management** plane, the infrastructure ensures **Zero Single Point of Failure (SPoF)** through dual-homed connectivity at the Server Farm and DMZ zones.

The project integrates Python-based automation scripts for scalable mass provisioning (Infrastructure as Code) and deploys a centralized monitoring stack for real-time network telemetry and proactive fault management.

## Architecture & Core Technologies
The infrastructure is a hybrid Campus and Data Center network simulated on EVE-NG, utilizing the following foundational planes:

* **Automation Controller:** An Ubuntu Linux server acts as the central execution node, utilizing the `Netmiko` library to push configurations programmatically.
* **Monitoring Stack:** A Zabbix server aggregates SNMP telemetry data, securely integrated via the isolated OOB network to ensure separation from the production Data Plane.
* **Device Inventory:** Virtualized Cisco IOS/IOL instances acting as Edge Routers, Multi-layer Switches (Core/Dist), and Layer 2 Access Switches.

### 1. Switching (Layer 2 Foundation)
* **Spanning Tree Protocol:** `Rapid-PVST+` (802.1w) implemented for sub-second Layer 2 convergence and loop prevention.
* **Link Aggregation:** `LACP` (802.3ad) EtherChannel deployed between Core switches and Server Farm switches to maximize throughput and provide link redundancy.
* **VLAN Trunking:** `802.1Q` encapsulation for Inter-Switch Links (ISL).

### 2. Routing (Layer 3 & FHRP)
* **Interior Gateway Protocol (IGP):** `OSPFv2` (Multi-area) deployed as the core routing protocol for dynamic, fast-converging internal reachability.
* **Exterior Gateway Protocol (EGP):** `eBGP` configured at the Enterprise Edge for redundant ISP peering (Primary/Backup).
* **First Hop Redundancy Protocol (FHRP):** `HSRP` configured at the Distribution layer to provide resilient default gateways (Virtual IP) for end-user VLANs, optimized with IP SLA tracking.

### 3. Network Services
* **Centralized DHCP:** Scopes configured to dynamically allocate IP addresses to Access layer clients.
* **DNS & Web Services:** Hosted internally and within the DMZ for public-facing access.
* **NTP & Syslog:** Centralized time synchronization and logging to the Ubuntu server for accurate event correlation and forensic analysis.

### 4. Network Assurance & Security
* **Layer 2 Security Framework:** Enforced `DHCP Snooping`, `Dynamic ARP Inspection (DAI)`, and `Port Security` on Access interfaces to mitigate rogue DHCP servers and ARP spoofing attacks.
* **Spanning Tree Protection:** `BPDU Guard` and `Root Guard` applied to edge and downlink ports to protect the STP topology.
* **Telemetry & Visibility:** `SNMPv2c` enabled globally for Zabbix polling, complemented by `SPAN` (Port Mirroring) and `NetFlow` for deep traffic analysis.

---

## Network Topology

### 1. Physical Topology
*Physical cabling layout, Main Distribution Frame (MDF), and Intermediate Distribution Frame (IDF) placement.*
![Physical Topology](./images/physical_topology.png)

### 2. Logical Topology
*Routing domains, data plane traffic flow, VLAN assignment, and OOB management overlays.*
![Logical Topology](./images/logical_topology.png)

---

## IP Addressing & Routing Plan

| ID | Device Name | Interface | IP Address | Routing (Destination) | Routing (Next-Hop/Gateway) | Remarks |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **1** | **ISP_Viettel** | e0/0 | 203.162.1.1/30 | 10.28.0.0/16 | 203.162.1.2 | eBGP Peering |
| **2** | **ISP_Mobi** | e0/0 | 222.252.1.1/30 | 10.28.0.0/16 | 222.252.1.2 | eBGP Peering |
| **3** | **HCM_EDGE** | e0/2 | 203.162.1.2/30 | 0.0.0.0/0 (Default) | 203.162.1.1 | Viettel (Primary Link) |
| | | e0/3 | 222.252.1.2/30 | 0.0.0.0/0 (Default) | 222.252.1.1 | MobiFone (Backup Link) |
| | | e0/1 | 10.28.1.1/30 | 10.28.0.0/16 | Learned via OSPF | Link to HCM_CORE_1 |
| | | e0/0 | 10.28.1.5/30 | | Learned via OSPF | Link to HCM_CORE_2 |
| | | mgmt0 | 10.28.99.254/24 | | | OOB Management IP |
| **4** | **HCM_CORE_1** | e0/1 | 10.28.1.2/30 | 0.0.0.0/0 (Default) | 10.28.1.1 (EDGE) | OSPF Area 0 |
| | | Po2 | 10.28.1.9/30 | | | LACP Inter-link to CORE_2 |
| | | e1/1 | 10.28.1.13/30 | | | Link to HCM_DIST_1 |
| | | e1/3 | 10.28.1.17/30 | | | Link to HCM_DIST_2 |
| | | e0/2 | 10.28.1.29/30 | | | Link to HCM_DMZ |
| | | e1/2 | 10.28.1.37/30 | | | Link to HCM_DIST_SRV |
| | | mgmt0 | 10.28.99.11/24 | | | OOB Management IP |
| **5** | **HCM_CORE_2** | e0/1 | 10.28.1.6/30 | 0.0.0.0/0 (Default) | 10.28.1.5 (EDGE) | OSPF Area 0 |
| | | Po2 | 10.28.1.10/30 | | | LACP Inter-link to CORE_1 |
| | | e0/3 | 10.28.1.21/30 | | | Link to HCM_DIST_1 |
| | | e0/2 | 10.28.1.25/30 | | | Link to HCM_DIST_2 |
| | | e2/0 | 10.28.1.33/30 | | | Link to HCM_DMZ |
| | | Po1 | 10.28.1.41/30 | | | Link to HCM_DIST_SRV |
| | | mgmt0 | 10.28.99.12/24 | | | OOB Management IP |
| **6** | **HCM_DIST_1** | Vlan 10 | 10.28.10.252/24 | 0.0.0.0/0 | Learned via OSPF | HSRP Active Gateway: .254 |
| | | Vlan 20 | 10.28.20.252/24 | | | HSRP Active Gateway: .254 |
| | | mgmt0 | 10.28.99.21/24 | | | OOB Management IP |
| **7** | **HCM_DIST_2** | Vlan 30 | 10.28.30.253/24 | 0.0.0.0/0 | Learned via OSPF | HSRP Active Gateway: .254 |
| | | Vlan 40 | 10.28.40.253/24 | | | HSRP Active Gateway: .254 |
| | | mgmt0 | 10.28.99.22/24 | | | OOB Management IP |
| **8** | **HCM_DMZ** | Vlan 100| 10.28.100.254/24| 0.0.0.0/0 | Learned via OSPF | Public Web Gateway |
| | | mgmt0 | 10.28.99.26/24 | | | OOB Management IP |
| **9** | **HCM_DIST_SRV**| Vlan 50 | 10.28.50.254/24 | 0.0.0.0/0 | Learned via OSPF | Internal SRV Gateway |
| | | mgmt0 | 10.28.99.25/24 | | | OOB Management IP |
| **10**| **WEB Server** | eth0 | 10.28.100.10/24 | 0.0.0.0/0 | 10.28.100.254 | Public Service (DMZ Zone) |
| **11**| **Database** | eth0 | 10.28.50.10/24 | 0.0.0.0/0 | 10.28.50.254 | Internal Service |
| **12**| **DNS** | eth0 | 10.28.50.11/24 | 0.0.0.0/0 | 10.28.50.254 | Internal Service |
| **13**| **DHCP** | eth0 | 10.28.50.12/24 | 0.0.0.0/0 | 10.28.50.254 | Dynamic IP Allocation |
| **14**| **Zabbix** | eth0 | 10.28.99.101/24 | N/A | N/A | OOB Network Polling |

---

## Repository Structure

```text
Enterprise-Network-Automation/
├── configs/                    # Device configuration backups and templates
│   ├── initial_configs/        # Day-0 base configurations for EVE-NG nodes
│   └── backups/                # Automated configuration backups pulled via Python
├── docs/                       # Additional project documentation (HLD/LLD)
├── images/                     # Network topology diagrams
│   ├── physical_topology.png
│   └── logical_topology.png
├── scripts/                    # Core Python automation scripts
│   ├── deploy_ultimate_layer2.py # L2, Security, FHRP provisioning
│   ├── deploy_core_routing.py  # OSPF & BGP provisioning (Phase 2)
│   └── monitor_telemetry.py    # SNMP/Syslog validation scripts
├── .gitignore                  # Git ignore rules
├── README.md                   # Main project documentation
└── requirements.txt            # Python dependencies list (netmiko, etc.)
