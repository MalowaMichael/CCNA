# 🚀 Mega Enterprise Infrastructure Capstone (Day 1–18 Final Exam)

## 📌 Project Overview
This repository contains a comprehensive, multi-site enterprise network deployment modeling a distributed corporate infrastructure. The design incorporates a Corporate HQ Data Center, an Academic Campus, and a Regional Sales Branch. 

This project functions as a rigorous capstone assessment covering concepts from Days 1 to 18 of Jeremy's IT Lab. It tests your precision across back-to-back Variable Length Subnet Masking (VLSM), multi-tier layer 2 segmentation, redundant inter-VLAN routing (Router-on-a-Stick and Multilayer Switching), custom native VLAN re-alignments, physical media layer optimization, port-level security hardening, and explicit point-to-point static routing paths.

**Exam Rule:** No default routes (`0.0.0.0 0.0.0.0`) are allowed anywhere in this topology. Every path must be explicitly mapped. A single typo in a subnet mask, an mismatched native VLAN configuration, or an incorrect next-hop path will result in total routing failure across domains.

---

## 🛠️ Logical Topology & Workspace Architecture

<details>
   <summary>View my solutions and topology</summary>
   
   *** Detailed Subnetting Table ***
   
   | VLAN ID | Network Address | Subnet Mask      | First Usable Address | Last Usable Address | Broadcast Address |
   |---------|-----------------|------------------|----------------------|---------------------|-------------------|
   | VL 10   | 172.25.0.0/22   | 255.255.252.0    | 172.25.0.1           | 172.25.3.254        | 172.25.3.255      |
   | VL 20   | 172.25.4.0/23   | 255.255.254.0    | 172.25.4.1           | 172.25.5.254        | 172.25.5.255      |
   | VL 30   | 172.25.6.0/24   | 255.255.255.0    | 172.25.6.1           | 172.25.6.254        | 172.25.6.255      |
   | VL 40   | 172.25.7.0/25   | 255.255.255.128  | 172.25.7.1           | 172.25.7.126        | 172.25.7.127      |
   | VL 99   | 172.25.7.128/28 | 255.255.255.240  | 172.25.7.129         | 172.25.7.142        | 172.25.7.143      |
   | VL 110  | 10.100.0.0/18   | 255.255.192.0    | 10.100.0.1           | 10.100.63.254       | 10.100.63.255     |
   | VL 120  | 10.100.64.0/19  | 255.255.224.0    | 10.100.64.1          | 10.100.95.254       | 10.100.95.255     |
   | VL 130  | 10.100.96.0/20  | 255.255.240.0    | 10.100.96.1          | 10.100.111.254      | 10.100.111.255    |


</details>

### 🏢 1. Corporate HQ Core (Class B Address Pool) — *Inter-VLAN via ROAS*
*   **Inventory:** One router named `HQ-Core` and one access layer switch named `SW-HQ-Distribution`.
*   **Interconnect Wiring:** Cable the router's GigabitEthernet0/1 interface straight down to the switch's FastEthernet0/24 interface. This trunk line will serve as your Router-on-a-Stick engine.
*   **Workstations:** Deploy 5 separate end-user PCs on the workspace canvas. Name them `HQ-PC-1` through `HQ-PC-5`. Connect them respectively into `SW-HQ-Distribution` interfaces FastEthernet0/1, FastEthernet0/2, FastEthernet0/3, FastEthernet0/4, and FastEthernet0/5.

### 🏫 2. Academic Campus Core (Class A Address Pool) — *Inter-VLAN via Layer 3 Switch*
*   **Inventory:** One Multilayer Switch named `Campus-MLS` and two access switches named `SW-Camp-Access1` and `SW-Camp-Access2`.
*   **Interconnect Wiring:**
    *   Connect `Campus-MLS` interface FastEthernet0/23 down to `SW-Camp-Access1` interface FastEthernet0/24.
    *   Connect `Campus-MLS` interface FastEthernet0/24 down to `SW-Camp-Access2` interface FastEthernet0/24.
*   **Workstations:** Deploy 7 separate PCs on the workspace canvas. Name them `Camp-PC-1` through `Camp-PC-7`.
    *   Connect `Camp-PC-1`, `Camp-PC-2`, `Camp-PC-3`, and `Camp-PC-4` into `SW-Camp-Access1` interfaces FastEthernet0/1 through FastEthernet0/4.
    *   Connect `Camp-PC-5`, `Camp-PC-6`, and `Camp-PC-7` into `SW-Camp-Access2` interfaces FastEthernet0/1 through FastEthernet0/3.

### 🏘️ 3. Regional Sales Branch (Class C Address Pool) — *Legacy ROAS Infrastructure*
*   **Inventory:** One router named `Branch-ROAS` and one access switch named `SW-Branch-Access`.
*   **Interconnect Wiring:** Cable the router's GigabitEthernet0/1 interface straight down to the switch's FastEthernet0/24 interface.
*   **Workstations:** Deploy 8 separate PCs on the workspace canvas. Name them `Br-PC-1` through `Br-PC-8`. Connect them respectively into `SW-Branch-Access` interfaces FastEthernet0/1 through FastEthernet0/8.

### 🌐 4. WAN Interconnect Network
*   **Primary WAN Link 1 (HQ to Campus):** Connect `HQ-Core` GigabitEthernet0/0 to `Campus-MLS` GigabitEthernet0/1. 
*   **Primary WAN Link 2 (Campus to Branch):** Connect `Campus-MLS` GigabitEthernet0/2 to `Branch-ROAS` GigabitEthernet0/0/0.
*   **Floating Redundant WAN Link (HQ to Branch):** Connect `HQ-Core` Serial0/0/0 straight to `Branch-ROAS` Serial0/0/0. Use a serial DCE cable here

---

## 🔢 Phase 1: High-Density Back-to-Back VLSM Calculation

You are issued three strict master container blocks. You must sort the following departments based on **largest host requirement to smallest host requirement** within each respective zone. Slice the prefixes tightly down to their absolute power-of-two boundaries with **zero address padding or gaps**.

### 📦 Container Zone A: Class B Allocation (`172.25.0.0/16`) — HQ Domain (5 VLANs)
Calculate subnets sequentially for these five distinct corporate divisions:
*   **VLAN 10 (HQ Corporate Data):** Requires 1,000 usable host IPs.
*   **VLAN 20 (HQ Operations Support):** Requires 480 usable host IPs.
*   **VLAN 30 (HQ Systems Development):** Requires 250 usable host IPs.
*   **VLAN 40 (HQ Executive Telephony):** Requires 115 usable host IPs.
*   **VLAN 99 (HQ Management & Provisioning):** Requires 14 usable host IPs.
*   *Assignment Rule:* The **first usable IP address** of each calculated subnet block must be assigned to the corresponding `HQ-Core` sub-interface.

### 📦 Container Zone B: Class A Allocation (`10.100.0.0/8`) — Campus Domain (7 VLANs)
Calculate subnets sequentially for these seven campus academic divisions:
*   **VLAN 110 (Campus Student Wireless):** Requires 15,000 usable host IPs.
*   **VLAN 120 (Campus Lecture Hall LAN):** Requires 7,000 usable host IPs.
*   **VLAN 130 (Campus Engineering Labs):** Requires 3,500 usable host IPs.
*   **VLAN 140 (Campus Administration):** Requires 1,800 usable host IPs.
*   **VLAN 150 (Campus Faculty Offices):** Requires 800 usable host IPs.
*   **VLAN 160 (Campus Security & CCTV Control):** Requires 120 usable host IPs.
*   **VLAN 199 (Campus Native Management Platform):** Requires 50 usable host IPs.
*   *Assignment Rule:* The **first usable IP address** of each calculated subnet block must be assigned to the corresponding Switch Virtual Interface (SVI) on `Campus-MLS`.

### 📦 Container Zone C: Class C Allocation (`192.168.15.0/24`) — Branch Domain (8 VLANs)
Calculate subnets sequentially inside this restrictive space for eight branch operational units:
*   **VLAN 210 (Branch Sales Force):** Requires 55 usable host IPs.
*   **VLAN 220 (Branch Retail Counter):** Requires 26 usable host IPs.
*   **VLAN 230 (Branch Back-Office Support):** Requires 12 usable host IPs.
*   **VLAN 240 (Branch Inventory Warehousing):** Requires 12 usable host IPs.
*   **VLAN 250 (Branch Human Resources):** Requires 6 usable host IPs.
*   **VLAN 260 (Branch Secure POS Terminal Array):** Requires 6 usable host IPs.
*   **VLAN 270 (Branch Guest Portal Sandbox):** Requires 2 usable host IPs.
*   **VLAN 299 (Branch Native Infrastructure Control):** Requires 2 usable host IPs.
*   *Assignment Rule:* The **first usable IP address** of each calculated subnet block must be assigned to the corresponding `Branch-ROAS` sub-interface.

### 🛣️ Container Zone D: WAN Serial/Routed Transit Blocks (`192.168.222.0/24`)
Allocate three consecutive `/30` point-to-point blocks out of this transit pool to bind the routers together:
1.  `WAN-HQ-to-Campus-Primary`
2.  `WAN-Campus-to-Branch-Primary`
3.  `WAN-HQ-to-Branch-Floating-Backup`

---

## ⚙️ Phase 2: Physical Media Layer & Port Hardening

### 🏎️ Layer 1 Interface Optimization
*   **HQ Core Interconnect:** Access the link linking `HQ-Core` to `SW-HQ-Distribution`. Turn off auto-negotiation on both sides. Forcefully lock the operational speed parameters to **100 Mbps** and bind the duplex setting to **Full** on both components.
*   **Campus Core Interconnect:** Access the two trunk links linking `Campus-MLS` down to the campus access switches. Turn off negotiation and explicitly lock their duplex parameters to **Full**.

### ⌨️ Management Plane & Privilege Level Hardening
Apply these standard administrative baseline security parameters globally across all 3 routers and 4 switches:
*   **Password Security:** Configure the devices to encrypt all plain-text passwords stored in the running configuration file.
*   **Privileged EXEC Access:** Lock privileged mode with a cryptographically secure hashing method using the password: `CiscoMegaMode77!`
*   **Console and Remote Pathways:** Secure both the physical console line and all VTY lines using the password: `SecureLineAccess44!`
*   **Inactivity Management:** Set an exact automatic inactivity disconnect timer of **6 minutes and 15 seconds** on all active VTY lines.
*   **Terminal Output Alignment:** Ensure that background system logs do not interrupt or fracture your typed lines during manual configuration sessions.

### 🛡️ Layer 2 Switchport Mitigation & Port Security
Execute the following port mitigation and locking techniques across **all 4 switches**:
*   **Client Port Lockdown:** Access all active user-facing client ports (`F0/1` through `F0/10`) and explicitly force them to operate strictly as access ports.
*   **Trunk Port Isolation:** Access interface F0/24 on all switches and lock it to run strictly as a static trunk link. Disable Dynamic Trunking Protocol (DTP) negotiations entirely on these ports.
*   **Port Security Blueprint:** Apply these structural security parameters directly onto all active user-facing client access interfaces:
*   - Activate port security on the interface.
    - Cap the maximum allowable learned MAC addresses strictly to a value of 1.
    - Configure the port to dynamically learn the device's MAC address and permanently write it directly into the running configuration.
    - Set the security violation rule to instantly shut down the interface and log an alert if an unauthorized host connects.
*   **Black-hole containment:** Build an isolated, unroutable containment VLAN named VLAN_DEAD using the designated ID number 666. Assign all unused, unassigned physical ports across the switches into this VLAN, and systematically disable the interfaces.

---

## 🔀 Phase 3: Inter-VLAN Configuration Engine

### 🏎️ Router-On-A-Stick (ROAS) Deployments

#### 🏢 HQ-Core Router Routing Setup
*   Instantiate logical sub-interfaces on interface GigabitEthernet0/1 for VLANs 10, 20, 30, and 40 using 802.1Q encapsulation framing.
*   Assign the **first usable IP address** of each respective Class B department block to these sub-interfaces to function as the client Default Gateway.
*   **Native VLAN Configuration:** Instantiate sub-interface number 99 to process traffic for VLAN 99. Explicitly define this sub-interface to pass all management traffic untagged as the native lane.

#### 🏘️ Branch-ROAS Router Routing Setup
*   Instantiate logical sub-interfaces on interface GigabitEthernet0/1 for VLANs 210, 220, 230, 240, 250, 260, and 270 using 802.1Q encapsulation. 
*   Assign the **first usable IP address** of each Class C department block as the gateway.
*   **Native VLAN Configuration:** Instantiate sub-interface number 299 and configure it to handle VLAN 299 as the hardcoded native transit lane.

### 🎛️ Multilayer Switch (MLS) Layer 3 Architecture
Program `Campus-MLS` to run as a high-speed multi-VLAN core routing vehicle:
*   **Core IP Routing:** Globally activate the switch's hardware routing engine capability.
*   **Trunk Framing Preparation:** Access your uplink trunk interfaces (`F0/23` and `F0/24`). Before converting them into trunks, you must explicitly declare their structural encapsulation protocol as 802.1Q.
*   **Native Alignment:** Re-align the native management transit path across both trunks to use **VLAN 199**.
*   **Switch Virtual Interfaces (SVIs):** Instantiate virtual interface frameworks for VLANs 110, 120, 130, 140, 150, 160, and 199. Assign the **first usable IP address** of each respective Class A block to these SVIs. These addresses will act as the Default Gateways for the campus end stations.

---

## 🗺️ Phase 4: Explicit Static Routing Grid

> ⚠️ **Constraint Reminder:** Do not use default routes (`0.0.0.0/0`) anywhere. You must map out individual, targeted destination paths across the WAN backbone.

### 🧩 1. HQ-Core Static Routes
*   **Campus Network Paths:** Add **7 distinct explicit static routes** pointing directly to each of the 7 individual Class A campus subnets sitting behind `Campus-MLS`. Point these routes to the next-hop IP of the primary WAN link.
*   **Branch Network Paths:** Add **8 distinct explicit static routes** pointing directly to each of the 8 individual Class C branch subnets sitting behind `Branch-ROAS`. Route these entries using the primary `Campus-MLS` transit pathway.
*   **Floating Route Resiliency:** Add **8 redundant floating static routes** targeting those same 8 Class C branch subnets. Route these backup entries directly over the point-to-point link to `Branch-ROAS`. Assign these backup paths an Administrative Distance (AD) of **150** so they stay dormant until a primary link failure occurs.

### 🧩 2. Campus-MLS Static Routes
*   **HQ Network Paths:** Add **5 distinct explicit static routes** targeting the 5 individual Class B corporate subnets, pointing directly to `HQ-Core`'s WAN interface IP.
*   **Branch Network Paths:** Add **8 distinct explicit static routes** targeting the 8 individual Class C branch subnets, pointing directly to `Branch-ROAS`'s WAN interface IP.

### 🧩 3. Branch-ROAS Static Routes
*   **HQ Network Paths:** Add **5 distinct explicit static routes** mapping out paths to each of the 5 individual Class B subnets in the HQ domain, pointing directly to the primary `Campus-MLS` link next-hop IP.
*   **Campus Network Paths:** Add **7 distinct explicit static routes** mapping out paths to each of the 7 individual Class A subnets in the Campus domain, pointing directly to the primary `Campus-MLS` link next-hop IP.
*   **Floating Route Resiliency:** Add **5 redundant floating static routes** targeting the 5 Class B HQ subnets. Route these backup entries directly over your point-to-point link to `HQ-Core`. Assign these paths an Administrative Distance (AD) of **150**.

---

## 🏆 Required Diagnostic Verification Milestone

Before pushing this network configuration to production or uploading your completed work to GitHub, you must verify your deployment calculations and configurations.

Analyze your configuration and provide your exact solutions to the following **four diagnostic questions** to prove the stability of your network design:

### 📋 Verification Checklist

#### ❓ Question 1: Class C Micro-Block Validation
What are the exact **Network IDs**, **CIDR Subnet Masks**, and **Broadcast Addresses** for **VLAN 250 (Branch HR)** and **VLAN 260 (Branch POS)**? Show your calculations to prove there are no address overlaps between these two blocks.

#### ❓ Question 2: Router-on-a-Stick (ROAS) Native VLAN Verification
Detail the specific parameters that must be set on the `HQ-Core` sub-interface number 99 to ensure it processes untagged management frames correctly, using the **first usable IP** of your calculated Class B management block.

#### ❓ Question 3: Multilayer Switch Interface Conversion Command
To bind your primary WAN link directly to `Campus-MLS` interface GigabitEthernet0/1 and assign it an explicit point-to-point IP address, what mode shift must you perform on that interface before the system will allow you to assign an IP address directly to a physical switch port?

#### ❓ Question 4: Explicit Asymmetric Traffic Inspection
If the primary WAN link between `Campus-MLS` and `Branch-ROAS` drops completely, tracing a packet sent from `Br-PC-1` (VLAN 210) to `HQ-PC-1` (VLAN 10) shows that traffic flows over the backup link. However, the reply packet from `HQ-PC-1` drops completely. Explain **exactly why** this return path asymmetric routing failure occurs based on your floating static route configuration rules.
