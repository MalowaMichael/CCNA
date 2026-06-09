# Comprehensive Enterprise Network Deployment Challenge ( Jeremy's Day 1–15 Capstone)

## 📌 Project Overview
This repository contains the architecture, configuration files, and implementation details for a highly secure, scalable, multi-router enterprise network deployment. The scenario encompasses hierarchical Variable Length Subnet Masking (VLSM) across Class A, B, and C private spaces, advanced layer 2 switch security, inter-network traffic redirection, and a resilient routing fabric utilizing Static, Default, and Floating Static paths.

---

## 🛠️ Physical Topology & Hardware Mapping

Deploy the following inventory within Cisco Packet Tracer. Cabling must align exactly with the assigned interfaces.

### 🏢 HQ Data Center (Class A Infrastructure)
*   **1x Cisco 2911 Router:** Name it `HQ-Core`.
*   **3x Cisco 2960 Switches:** Name them `SW-DC-Prod`, `SW-DC-Dev`, and `SW-DC-Mgmt`.
*   **Connections:**
    *   `HQ-Core` `G0/0` ➡️ `SW-DC-Prod` `F0/24`
    *   `HQ-Core` `G0/1` ➡️ `SW-DC-Dev` `F0/24`
    *   `HQ-Core` `G0/2` ➡️ `SW-DC-Mgmt` `F0/24`
*   **End Devices:** Attach **1x Server** to each switch on interface `F0/1`. Name them `Prod-Server-1`, `Dev-Server-1`, and `Mgmt-Server-1`.

### 🏫 Campus Core (Class B Infrastructure)
*   **1x Cisco 2911 Router:** Name it `Campus-Edge`.
*   **2x Cisco 2960 Switches:** Name them `SW-Camp-WiFI` and `SW-Camp-Labs`.
*   **Connections:**
    *   `Campus-Edge` `G0/1` ➡️ `SW-Camp-WiFI` `F0/24`
    *   `Campus-Edge` `G0/2` ➡️ `SW-Camp-Labs` `F0/24`
*   **End Devices:** Attach **1x PC** to each switch on interface `F0/1`. Name them `WiFi-Host-1` and `Lab-Host-1`.

### 🏘️ Regional Branch (Class C Infrastructure)
*   **1x Cisco 2911 Router:** Name it `Branch-R1`.
*   **2x Cisco 2960 Switches:** Name them `SW-Br-Sales` and `SW-Br-Ops`.
*   **Connections:**
    *   `Branch-R1` `G0/1` ➡️ `SW-Br-Sales` `F0/24`
    *   `Branch-R1` `G0/2` ➡️ `SW-Br-Ops` `F0/24`
*   **End Devices:** Attach **1x PC** to each switch on interface `F0/1`. Name them `Sales-Host-1` and `Ops-Host-1`.

### 🌐 WAN Backbone Mesh
To enforce path redundancy, interconnect the routing engine using the following interfaces:
*   **Primary Link (HQ to Campus):** `HQ-Core` `S0/0/0` ➡️ `Campus-Edge` `S0/0/0` (Serial DCE)
*   **Primary Link (Campus to Branch):** `Campus-Edge` `S0/0/1` ➡️ `Branch-R1` `S0/0/1` (Serial DCE)
*   **Backup Path (HQ to Branch Link):** `HQ-Core` `G0/0/0` (using HWIC-1GE expansion module if needed) or `S0/1/0` ➡️ `Branch-R1` `S0/1/0`

---

## 🔢 Phase 1: Hierarchical VLSM Allocations
You are issued three independent base address blocks. Subnetting must be calculated sequentially by host requirements from **largest to smallest** with zero gaps.

### 📦 Allocation Block 1: Class A Space (`10.0.0.0/8`)
*   **VLAN 10 (Production Data):** 150,000 required hosts.
*   **VLAN 20 (Development Hub):** 60,000 required hosts.
*   **VLAN 30 (Data Center Operations):** 25,000 required hosts.
*   **VLAN 40 (Storage Network):** 8,000 required hosts.
*   **VLAN 50 (In-Band Management):** 500 required hosts.
*   *Rule:* Apply the **first usable IP** of each subnet to the `HQ-Core` LAN interface.

### 📦 Allocation Block 2: Class B Space (`172.16.0.0/16`)
*   **VLAN 110 (Student Wi-Fi):** 2,000 required hosts.
*   **VLAN 120 (Engineering Labs):** 950 required hosts.
*   **VLAN 130 (Faculty Staff):** 450 required hosts.
*   **VLAN 140 (VoIP Infra):** 200 required hosts.
*   **VLAN 150 (Campus Admin):** 60 required hosts.
*   *Rule:* Apply the **first usable IP** of each subnet to the `Campus-Edge` LAN interface.

### 📦 Allocation Block 3: Class C Space (`192.168.100.0/24`)
*   **VLAN 210 (Sales Force):** 60 required hosts.
*   **VLAN 220 (Branch Operations):** 28 required hosts.
*   **VLAN 230 (Guest Portal):** 14 required hosts.
*   **VLAN 240 (Retail POS):** 6 required hosts.
*   **VLAN 250 (Branch Transit/Mgmt):** 2 required hosts.
*   *Rule:* Apply the **first usable IP** of each subnet to the `Branch-R1` LAN interface.

### 🛣️ WAN Links (Point-to-Point Blocks)
From a separate container block of `192.168.200.0/24`, allocate three tight `/30` subnets back-to-back:
1.  `WAN-HQ-to-Campus` (/30)
2.  `WAN-Campus-to-Branch` (/30)
3.  `WAN-HQ-to-Branch-Backup` (/30)

---

## 🔒 Phase 2: Device Hardening & Layer 2 Security
Configure all Routers and Switches to fulfill these institutional security requirements.

### ⌨️ Console, VTY, and Privilege EXEC Hardening
*   **Hostname:** Enforce identical naming to the architecture topology layout.
*   **Global Encryption:** Encrypt all plain-text passwords stored in the running configuration.
*   **Privilege EXEC Mode:** Set the secret password to `CiscoPriv15!`.
*   **Console Access:** Protect with password `ConsoleAccess99!`, enable login, and prevent console log messages from disrupting your command-line interface entries.
*   **Remote Management:** Configure VTY lines 0-15 to require password access (`VtyRemote88!`), enforce synchronous logging, and set an automatic inactivity timeout value of **7 minutes 30 seconds**.
*   **Banner:** Implement a secure legal warning banner blocking unauthorized access.

### 🛡️ Layer 2 Switchport Mitigation (Every Switch)
*   **Trunking & Access Security:** Forcefully configure interface `F0/24` on all switches as a hardcoded static trunk link. Disable DTP negotiation on this interface.
*   **Access Port Enforcement:** Explicitly configure interface `F0/1` as an access port.
*   **Port Security Blueprint (On Interface `F0/1`):**
    *   Enable standard port security.
    *   Limit the secure MAC address capacity strictly to a maximum count of **2**.
    *   Implement **Sticky MAC** learning so the switch registers dynamically learned devices directly into the running configuration.
    *   Set the security violation mode to **Shutdown**. The port must instantly transition to an error-disabled state upon an unauthorized frame arrival.
*   **Unused Interface Mitigation:** Shift all remaining unused fast-ethernet and gigabit interfaces into a designated black-hole VLAN (VLAN 999). Forcefully **shut down** all of these unassigned ports.

---

## 🔀 Phase 3: Routing Architecture Matrix
You must design a static routing fabric that provides seamless traffic flows across all infrastructure zones, complete with path-failure protection.

### 🗺️ HQ-Core Routing Policy
1.  **Static Paths:** Establish explicit next-hop static routes toward all **five** Class B subnets located behind `Campus-Edge` via the primary point-to-point link.
2.  **Primary Path to Branch:** Establish explicit next-hop static routes toward all **five** Class C subnets located behind `Branch-R1`, routing through the primary `WAN-HQ-to-Campus` interface path.
3.  **Floating Static Route Protection:** Implement a redundant, fault-tolerant path toward the Class C Branch subnets. These backup paths must target the `WAN-HQ-to-Branch-Backup` next-hop IP and carry an Administrative Distance (AD) of **150**. This ensures they remain invisible in the active routing table until a primary serial link drops.

### 🗺️ Campus-Edge Routing Policy
1.  **Symmetric Visibility:** Establish explicit next-hop static routes to all **five** Class A subnets via `HQ-Core`.
2.  **Branch Reachability:** Establish explicit next-hop static routes to all **five** Class C subnets via `Branch-R1`'s primary serial address.

### 🗺️ Branch-R1 Routing Policy
1.  **Gateway of Last Resort:** To keep the branch configuration compact, do not write individual destination routes. Instead, configure a single **Default Route (`0.0.0.0/0`)** pointing toward the primary `Campus-Edge` interface IP.
2.  **Backup Default Route:** Implement a **Floating Default Route (`0.0.0.0/0`)** targeting the backup link interface of `HQ-Core`. Assign this path an Administrative Distance (AD) of **150**.

---

## 📝 Milestone Verification Checkpoint (Required Response)
Before uploading your complete `.pkt` file and configuration scripts to GitHub, you must verify your deployment calculations and configurations.

Analyze your configuration and provide your exact solutions to the following **four diagnostic questions** to prove the stability of your network design:

### ❓ Verification Questions
1.  **The Class A Binary Border Check:** What are the exact Subnet Masks (in standard dot-decimal notation, e.g., `255.x.x.x`) and the exact **Broadcast Addresses** for **VLAN 20 (Development Hub)** and **VLAN 30 (Data Center Operations)**?
2.  **The Class B Subnet Overlap Audit:** What is the precise network range (Network ID, First Usable, Last Usable, Broadcast Address) calculated for **VLAN 120 (Engineering Labs)**? Prove that it does not overlap with VLAN 110.
3.  **The Floating Route Convergence Scenario:** If the serial connection between `Campus-Edge` and `Branch-R1` physically fails, describe exactly what happens inside the routing table of `Branch-R1`. Which route is purged, which route takes its place, and what is its specific next-hop IP address?
4.  **Cisco IOS Security Status Validation:** Paste the exact block of Cisco IOS commands required to configure the VTY line access timeout to exactly **7 minutes and 30 seconds**, while also ensuring console logs do not split your typed command input.
