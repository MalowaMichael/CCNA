# Cisco IOS Switch Interface Hardening & Diagnostic Lab

## 📌 Project Overview
This repository contains the architecture, verification documentation, and production configuration scripts for a multi-switch corporate infrastructure audit. The project focuses on two core objectives:
1. **Attack Surface Reduction**: Securing vacant access and uplink ports across the facility using optimal configuration workflows.
2. **Layer 1/2 Diagnostics**: Performing a Root Cause Analysis (RCA) on a degraded backbone link experiencing severe performance issues during high-volume data operations.

---

## 🏗️ Topology Design & Mapping
The infrastructure consists of two core switches interconnecting local corporate server environments over a dedicated distribution backbone.

* **Backbone Link**: `SW-CORE-01 (Gi0/1)` connects directly to `SW-CORE-02 (Gi0/1)`.
* **SW-CORE-01 Server Allocation**: `FastEthernet0/1` through `FastEthernet0/4` (Active Production).
* **SW-CORE-02 Server Allocation**: `FastEthernet0/5` through `FastEthernet0/8` (Active Production).

---

## 🔒 Task 1: Device Management & Security Hardening
To eliminate physical security vulnerabilities, all unused interfaces were isolated and shut down in a single configuration pass per switch without disrupting active production servers or the inter-switch link. System boundaries were fortified with banner restrictions and encrypted authentication.

### SW-CORE-01 Deployment Script
```text
SW-CORE-01# configure terminal
SW-CORE-01(config)# hostname SW-CORE-01
SW-CORE-01(config)# enable secret sysadmin99
SW-CORE-01(config)# banner motd $
WARNING
*************************************
 * UNAUTHORIZED ACCESS IS PROHIBITED *
*************************************
$
SW-CORE-01(config)# interface range gigabitethernet 0/2, fastethernet 0/5 - 24
SW-CORE-01(config-if-range)# shutdown
SW-CORE-01(config-if-range)# end
SW-CORE-01# write memory
```

### SW-CORE-02 Deployment Script
```text
SW-CORE-02# configure terminal
SW-CORE-02(config)# hostname SW-CORE-02
SW-CORE-02(config)# enable secret sysadmin99
SW-CORE-02(config)# banner motd $
WARNING
*************************************
 * UNAUTHORIZED ACCESS IS PROHIBITED *
*************************************
$
SW-CORE-02(config)# interface range gigabitethernet 0/2, fastethernet 0/1 - 4, fastethernet 0/9 - 24
SW-CORE-02(config-if-range)# shutdown
SW-CORE-02(config-if-range)# end
SW-CORE-02# write memory
```

---

## 🕵️‍♂️ Task 2: Root Cause Analysis (RCA) — The Backbone Mismatch Mystery

### Symptom Profile
The backbone uplink (`Gi0/1`) retains a physical link light (`connected`), but data throughput degrades heavily and experiences massive packet drops whenever server backup routines execute.

### Collected Diagnostic Data
* **`SW-CORE-01# show interfaces gigabitethernet 0/1`**
  ```text
  GigabitEthernet0/1 is up, line protocol is up (connected)
    Full-duplex, 1000Mb/s, media type is 10/100/1000BaseTX
  ```
* **`SW-CORE-02# show interfaces status`**
  ```text
  Port      Name               Status       Vlan       Duplex  Speed Type
  Gi0/1     UPLINK_TO_CORE_01  connected    1          a-full  a-100 10/100/1000BaseTX
  ```

### Technical Findings & Diagnosis
1. **The Operational `a-` Prefix**: In the `show interfaces status` command, the `a-` prefix denotes the **Actual / Operational** state of the port. It confirms what speed and duplex the hardware is currently utilizing.
2. **Layer 1 Physical Cable Degradation**: While `SW-CORE-01` is operating at full hardware capacity (`1000Mb/s`), `SW-CORE-02` indicates an operational speed of `a-100` (100 Mbps). Because Gigabit Ethernet requires all **8 internal wires (4 pairs)** to establish a 1 Gbps signal, a damaged pin inside the RJ45 jack or a broken internal copper strand forces the Cisco interface to auto-negotiate downward to 100 Mbps to prevent a total link dropout.
3. **Throughput Bottleneck**: The link stays up, but a severe speed mismatch exists. When `SW-CORE-01` transmits data at 1000 Mbps, the degraded capacity on the receiving end (`SW-CORE-02` at 100 Mbps) causes internal memory buffers to oversubscribe, resulting in massive output drops and performance degradation.

---

## 🛠️ Task 3: Infrastructure Remediation Script
To stabilize the link and maximize performance across the degraded Layer 1 medium without relying on unstable `auto` parameters, both sides must be hardcoded to an identical, stable operational baseline until the physical cable is replaced.

### SW-CORE-01 Remediation
```text
SW-CORE-01# configure terminal
SW-CORE-01(config)# interface gigabitethernet 0/1
SW-CORE-01(config-if)# description [MANUAL BACKBONE UPLINK TO CORE 02]
SW-CORE-01(config-if)# speed 100
SW-CORE-01(config-if)# duplex full
SW-CORE-01(config-if)# end
```

### SW-CORE-02 Remediation
```text
SW-CORE-02# configure terminal
SW-CORE-02(config)# interface gigabitethernet 0/1
SW-CORE-02(config-if)# description [MANUAL BACKBONE UPLINK TO CORE 01]
SW-CORE-02(config-if)# speed 100
```
*(Note: Per Cisco IOS fallback behavior, hardcoding the speed to 100 on a FastEthernet/Gigabit interface automatically drops a non-negotiating interface to Half-Duplex unless explicitly stated. Therefore, configuring `duplex full` ensures symmetry).*
```text
SW-CORE-02(config-if)# duplex full
SW-CORE-02(config-if)# end
```

### Permanent Resolution
The long-term infrastructure fix requires a physical cable replacement using standard **Cat6 UTP (23AWG, 4-pair)** cabling to support true, hardcoded `speed 1000` full-duplex traffic cleanly across the network backbone.
