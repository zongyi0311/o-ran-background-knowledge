# Webinar
## 1.  Network Automation
<img width="810" height="454" alt="image" src="https://github.com/user-attachments/assets/0bf6a877-2d3d-4d05-bfe5-84983a48f732" />

1. **Objectives / Policies / Application Abstraction**: Provides the APIs and tools to define high-level network intents, policies, and applications.
2. **Executive Operators & Controllers**: Implements executive logic that translates high-level intents from rApps into actionable network control commands.
3. **Network Entities**: Represents the physical and virtual network components controlled by Athena and ODIN.
4. **Infrastructure Layer**: Provides container orchestration, scalability, and isolation for all operator components;

### Zero-touch Network Automation
- The network can deploy, configure, and optimize itself automatically — without manual operations.
- Concept: The desired state of your network is defined in a file as an intent.

| Feature               | English Description                                                       |                                                    
| :-------------------- | :------------------------------------------------------------------------ | 
| **Extendable**        | Building blocks for each 5G domain (CN, RAN, Terminal) → *Modular design* | 
| **Multi-vendor**      | Compatible with various vendors and open-source components                |                                                 
| **O-RAN aligned SMO** | SMO orchestrates RAN and Core automation following O-RAN architecture     | 
| **Intent-driven**     | Files define what you want instead of how to create it → *Declarative*    | 

- <img width="329" height="282" alt="image" src="https://github.com/user-attachments/assets/122fffd4-5e65-457e-a8af-ae7ed875f409" />

- Summary
  - Automation driven by intents, enabling fully autonomous management across Radio Access Network (RAN), Core Network, and Terminal devices
  - Declarative intent files specify the desired network state, which is then realized by the Service Management and Orchestration (SMO) framework
  - The system supports multi-vendor integration and modular architecture, allowing flexible and scalable 5G Standalone (SA) deployments and Automation is applied throughout the entire network lifecycle from deployment to operation and adheres to 3GPP and O-RAN standards
 
## 2. xApp automation
- System Architecture
- <img width="306" height="240" alt="image" src="https://github.com/user-attachments/assets/701f1c0d-4f52-4665-ba0d-1b57ed61b7c6" />

- Summary
  - MX-RIC offers intergration across RATs(4G,5G), vendors and hardware
  - deploy ready-to-go xApps, or build own with the xApp SDK from the BubbleRAN catalogue.
 
| xApp Category                        | Available Features (English)                                            | 
| :----------------------------------- | :---------------------------------------------------------------------- | 
| **Key Performance Monitoring (KPM)** | Standard O-RAN KPM, RC, and custom Service Models (e.g. MAC, RLC, PDCP) | 
| **RAN Control**                      | Slicing, Handover, SLA management, Modulation and Coding Scheme (MCS)   | 
| **Low Layer Control**                | Sensing, Object Detection                                               | 
| **Cell Configuration and Control**   | Bandwidth Part (BWP)                                                    | 
## rApp automation
## Slice automation

# BubbleRAN USER guide
## 5G Control Plane

| Aspect          | Control Plane (CP)                                | User Plane (UP)           |
| --------------- | ------------------------------------------------- | ------------------------- |
| Function        | Network control, registration, session management | Actual data forwarding    |
| Core Function   | AMF                                               | UPF                       |
| Protocol Layers | NAS, RRC, PDCP, RLC, MAC, PHY                     | SDAP, PDCP, RLC, MAC, PHY |
| Interface       | NG-C                                              | NG-U                      |

- 5G architecture has two modes:
  - **Non-Standalone (NSA)**: Uses LTE for the control plane, relying on the 4G EPC core.
  - **Standalone (SA)**: Fully 5G-based, with both control and user planes on the 5G Core (5GC).
 
| eng              | full name                         |
| ---------------  | --------------------------------- |
| **EPC**          | Evolved Packet Core               |
| **5G-CN / 5GC**  | 5G Core Network                   |
| **EN-DC**        | E-UTRA NR Dual Connectivity       |
| **NE-DC**        | NR E-UTRA Dual Connectivity       |
| **NR-DC**        | NR Dual Connectivity              |
| **CUPS**         | Control and User Plane Separation |

### 5G SA
1.Scan and synchronize.
2.Receive MIB (Master Information Blocks) and SIB (System Information Blocks).
3.Cell selection and random access (RACH).
4.RRC connection.
5.NAS registration.
6.PDU session establishment.
7.Send/Receive data.

- step 1: Scan and Synchronize
  - At the beginning, there is no dedicated channel for the UE since it is not yet connected.
  - The UE searches for nearby 5G cells and acquires: PSS, SSS to synchronize with gNB timing.
 
- Step 2：Receive MIB and SIB
  - MIB and SIB messages are generated by the RRC layer, UE must detect MIB and SIB1 before connecting in NR SA.
  - MIB contains reference subcarrier spacing and control channel info.
  - SIB1 provides the necessary information for the initial attach and schedules other SIBs.


