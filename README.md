# Enterprise Network Automation & Monitoring System

## Project Overview
This project demonstrates the design, deployment, and operation of a highly available Enterprise Network Architecture using NetDevOps methodologies. Built upon a robust 3-Tier hierarchical model (Core, Distribution, Access) combined with an isolated Out-of-Band (OOB) management plane, the system ensures zero single point of failure (Dual-homed Server Farm & DMZ). The project integrates Python-based network automation for rapid provisioning and a centralized monitoring stack for real-time telemetry.

## Architecture & Technologies
* **Network Topology:** 3-Tier Enterprise Campus & Data Center hybrid, simulated on EVE-NG.
* **Automation Controller:** Ubuntu Linux executing Python scripts (Netmiko) for mass configuration deployment.
* **Monitoring Stack:** Zabbix Network Monitoring Server integrated via isolated OOB Management network.
* **Control Plane Protocols:** OSPF Area 0 (Backbone Routing), eBGP (Edge/ISP Peering), HSRP (First Hop Redundancy).
* **Data Plane Technologies:** LACP (802.3ad) for EtherChannel, Rapid-PVST+, and comprehensive L2 Security (DHCP Snooping, DAI, Port Security).

## Network Diagrams

### 1. Physical Topology (StarUML)
> *Note for author: Replace the path below with your StarUML diagram showing the 5-floor building, MDF, and IDF rack layouts.*
![Physical Topology](./images/physical_topology.png)

### 2. Logical Topology (EVE-NG)
> *Note for author: Replace the path below with the final 10/10 EVE-NG screenshot showing OOB, DMZ, and the Dual-homed Server Farm.*
![Logical Topology](./images/logical_topology.png)

## IP Addressing & Routing Plan

| STT | Device Name | Interface | IP Address | Routing (Destination) | Routing (Gateway/Nexthop) | Notes |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **1** | **ISP_Viettel** | e0/0 | 203.162.1.1/30 | 10.28.0.0/16 | 203.162.1.2 | eBGP Peering |
| **2** | **ISP_Mobi** | e0/0 | 222.252.1.1/30 | 10.28.0.0/16 | 222.252.1.2 | eBGP Peering |
| **3** | **HCM_EDGE** | e0/2 | 203.162.1.2/30 | 0.0.0.0/0 (Default) | 203.162.1.1 | Viettel (Primary) |
| | | e0/3 | 222.252.1.2/30 | 0.0.0.0/0 (Default) | 222.252.1.1 | MobiFone (Backup) |
| | | e0/1 | 10.28.1.1/30 | 10.28.0.0/16 | Learned via OSPF | Link to HCM_CORE_1 |
| | | e0/0 | 10.28.1.5/30 | | Learned via OSPF | Link to HCM_CORE_2 |
| | | mgmt0 | 10.28.99.254/24 | | | OOB Management IP |
| **4** | **HCM_CORE_1** | e0/1 | 10.28.1.2/30 | 0.0.0.0/0 (Default) | 10.28.1.1 (EDGE) | OSPF Area 0 |
| | | Po2 | 10.28.1.9/30 | | | Inter-link HCM_CORE_2 |
| | | e1/1 | 10.28.1.13/30 | | | Link to HCM_DIST_1 |
| | | e1/3 | 10.28.1.17/30 | | | Link to HCM_DIST_2 |
| | | e0/2 | 10.28.1.29/30 | | | Link to HCM_DMZ |
| | | e1/2 | 10.28.1.37/30 | | | Link to HCM_DIST_SRV |
| | | mgmt0 | 10.28.99.11/24 | | | OOB Management IP |
| **5** | **HCM_CORE_2** | e0/1 | 10.28.1.6/30 | 0.0.0.0/0 (Default) | 10.28.1.5 (EDGE) | OSPF Area 0 |
| | | Po2 | 10.28.1.10/30 | | | Inter-link HCM_CORE_1 |
| | | e0/3 | 10.28.1.21/30 | | | Link to HCM_DIST_1 |
| | | e0/2 | 10.28.1.25/30 | | | Link to HCM_DIST_2 |
| | | e2/0 | 10.28.1.33/30 | | | Link to HCM_DMZ |
| | | Po1 | 10.28.1.41/30 | | | Link to HCM_DIST_SRV |
| | | mgmt0 | 10.28.99.12/24 | | | OOB Management IP |
| **6** | **HCM_DIST_1** | Vlan 10 | 10.28.10.252/24 | 0.0.0.0/0 | Learned via OSPF | HSRP Active: .254 |
| | | Vlan 20 | 10.28.20.252/24 | | | HSRP Active: .254 |
| | | mgmt0 | 10.28.99.21/24 | | | OOB Management IP |
| **7** | **HCM_DIST_2** | Vlan 30 | 10.28.30.253/24 | 0.0.0.0/0 | Learned via OSPF | HSRP Active: .254 |
| | | Vlan 40 | 10.28.40.253/24 | | | HSRP Active: .254 |
| | | mgmt0 | 10.28.99.22/24 | | | OOB Management IP |
| **8** | **HCM_DMZ** | Vlan 100| 10.28.100.254/24| 0.0.0.0/0 | Learned via OSPF | Public Web Gateway |
| | | mgmt0 | 10.28.99.26/24 | | | OOB Management IP |
| **9** | **HCM_DIST_SRV**| Vlan 50 | 10.28.50.254/24 | 0.0.0.0/0 | Learned via OSPF | Internal SRV Gateway |
| | | mgmt0 | 10.28.99.25/24 | | | OOB Management IP |
| **10**| **WEB Server** | eth0 | 10.28.100.10/24 | 0.0.0.0/0 | 10.28.100.254 | DMZ Public Service |
| **11**| **Database** | eth0 | 10.28.50.10/24 | 0.0.0.0/0 | 10.28.50.254 | Internal Service |
| **12**| **DNS** | eth0 | 10.28.50.11/24 | 0.0.0.0/0 | 10.28.50.254 | Internal Service |
| **13**| **DHCP** | eth0 | 10.28.50.12/24 | 0.0.0.0/0 | 10.28.50.254 | IP Allocation |
| **14**| **Zabbix** | eth0 | 10.28.99.101/24 | N/A | N/A | Dedicated OOB Traffic |

## Key Features
* **Automated Provisioning:** Push configurations (VLANs, STP, Routing, LACP) across multiple devices systematically via Netmiko.
* **Resilient Routing & Switching:** Full OSPF multi-area integration with sub-second failover, backed by LACP EtherChannels to eliminate STP blocking.
* **Isolated Out-of-Band Management:** Independent physical/logical management plane ensuring administrative access during Data Plane outages.
* **Real-time Telemetry:** SNMP-based tracking of critical device metrics (CPU, Memory, Interface bandwidth) via Zabbix.

## Getting Started
To replicate this environment:
1. Import the provided `.unl` topology file into your EVE-NG instance.
2. Bridge the `OOB-Management` cloud to your local VMware VMnet interface.
3. Boot the Ubuntu Automation Server and install requirements: `pip install netmiko`.
4. Execute the provisioning scripts located in the `/scripts` directory.
