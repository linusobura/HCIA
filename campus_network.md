### Campus Network Configuration LAB

---

#### **1. Overview**
**Objective**: Design and implement a scalable, secure campus network for a 6-floor office building (3 floors operational).  
**Key Components**:
- **Layers**: Access, Aggregation, Core, Egress
- **Technologies**: VLANs, OSPF, WLAN, DHCP, SNMPv3, NAT, Security Policies
- **Devices**: Huawei routers, switches (S5700/S6700), AC (Wireless Controller), Fit APs

---

#### **2. Requirement Analysis**
**Critical Requirements**:
1. **Terminal Counts**:
   - Floor 1: 10 wired (servers), 100 wireless (guests)
   - Floors 2–3: 200 wired, 50 wireless (employees)
2. **Bandwidth**:
   - Servers: 1000 Mbps
   - Computers: 100 Mbps
   - Wireless clients: 2 Mbps/client
3. **Security**:
   - Guest SSID isolated from intranet
   - Internet access restricted to wireless terminals
4. **Redundancy**: Layer 3 failover capabilities
5. **Scalability**: Support future floors without hardware replacement.

---

#### **3. Network Design**
##### **Physical Topology**
- **Core Layer**: `CORE1` (First-floor equipment room)
- **Aggregation Layer**: `F2-AGG1` (Floor 2), `F3-AGG1` (Floor 3)
- **Access Layer**: 
  - Floor 1: `F1-ACC1` (servers/APs)
  - Floor 2: `F2-ACC1/2/3` (admin/general manager)
  - Floor 3: `F3-ACC1/2/3` (R&D/marketing)
- **Egress**: Router (static IP to ISP)
- **Wireless**: 3 dual-band APs per floor.

##### **Layer 2 Design (VLANs)**
| VLAN ID | Purpose                          |
|---------|----------------------------------|
| 1–3     | Device Management (per floor)    |
| 100     | Servers                          |
| 101     | General Manager's Office         |
| 102     | Administrative Department        |
| 103     | Marketing Department             |
| 104     | R&D Department                   |
| 105–107 | Wireless Terminals (per floor)   |
| 201–207 | Interconnections/WLAN Management |

##### **Layer 3 Design (IP & Routing)**
- **Addressing**: `192.168.0.0/16` subnet
- **DHCP**: 
  - Floor 1: `CORE1` assigns IPs to APs/STAs
  - Floors 2–3: Aggregation switches assign IPs
- **Routing**: OSPF across all devices for internal connectivity; default route to router for internet.

##### **WLAN Design**
| Parameter          | Floor 1       | Floor 2       | Floor 3       |
|--------------------|---------------|---------------|---------------|
| **AP Management**  | VLAN 205      | VLAN 206      | VLAN 207      |
| **Service VLAN**   | VLAN 105      | VLAN 106      | VLAN 107      |
| **SSID**           | `WLAN-F1`     | `WLAN-F2`     | `WLAN-F3`     |
| **Security**       | WPA2-PSK/AES  | WPA2-PSK/AES  | WPA2-PSK/AES  |
| **Password**       | `WLAN@Guest123` | `WLAN@Employee2` | `WLAN@Employee3` |

##### **Security & Egress**
- **Guest Isolation**: Traffic filter on `CORE1` to block intranet access.
- **Internet Access**: NAT on router for wireless subnets only.
- **Web Server**: NAT mapping (public IP `1.1.1.1:8080` → private `192.168.100.1:80`).

##### **Network Management**
- **SNMPv3**: Encrypted communication with NMS (`192.168.100.2`).
- **Trap Sources**: Management VLANs for switches; `VLANIF205` for AC.

---

#### **4. Implementation Highlights**
##### **Router Configuration**
- **NAT**: 
  ```bash
  nat server protocol tcp global 1.1.1.1 8080 inside 192.168.100.1 www
  nat outbound 2000 address-group 1
  ```
- **OSPF**: Advertises default route to internal network.
- **ACL**: Permits only wireless subnets (`192.168.105.0/24`, `106.0/24`, `107.0/24`) for internet access.

##### **Core Switch (`CORE1`)**
- **VLANs**: 100 (servers), 105/205 (WLAN), 201–202 (interconnections).
- **DHCP Pools**: For APs (`ap-f1`) and wireless clients (`sta-f1`).
- **ACL 3000**: Blocks guest VLAN (`105`) from intranet (`192.168.0.0/16`).

##### **Aggregation Switches (`F2-AGG1`/`F3-AGG1`)**
- **Role**: Gateways for their floors; DHCP for wired/wireless terminals.
- **OSPF**: Advertises floor-specific subnets.

##### **AC (Wireless Controller)**
- **CAPWAP**: Uses `VLANIF205` for AP management.
- **Profiles**: SSID, security, and VAP profiles per floor.
- **AP Groups**: Assign APs to floor-specific groups (e.g., `WLAN-F1`).

---

#### **5. Verification & O&M**
**Acceptance Tests**:
1. Wireless clients connect to SSIDs with correct passwords.
2. OSPF neighborships are established.
3. Inter-VLAN/intranet connectivity.
4. Guest VLAN cannot access internal resources.
5. NMS receives SNMP traps from all devices.

**O&M Schedule**:
| Frequency   | Check Item          | Method                     |
|-------------|---------------------|----------------------------|
| **Daily**   | Device temperature  | `display temperature`      |
| **Weekly**  | CPU/Memory usage    | `display cpu-usage`        |
| **Monthly** | Configuration backup| Manual backup              |

---


