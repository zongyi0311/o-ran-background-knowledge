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

1. Scan and synchronize.
2. Receive MIB (Master Information Blocks) and SIB (System Information Blocks).
3. Cell selection and random access (RACH).
4. RRC connection.
5 .NAS registration.
6. PDU session establishment.
7. Send/Receive data.

- step 1: Scan and Synchronize
  - At the beginning, there is no dedicated channel for the UE since it is not yet connected.
  - The UE searches for nearby 5G cells and acquires: PSS, SSS to synchronize with gNB timing.
 
- Step 2：Receive MIB and SIB
  - MIB and SIB messages are generated by the RRC layer, UE must detect MIB and SIB1 before connecting in NR SA.
  - MIB contains reference subcarrier spacing and control channel info.
  - SIB1 provides the necessary information for the initial attach and schedules other SIBs.

- step 3: Cell Selection & Contention-Based Random Access
  - After selecting a 5G cell, the UE performs a Contention-Based Random Access (CBRA) procedure with the gNB, exchanging four key messages (Msg1–Msg4) to establish initial uplink synchronization.
```
 UE → gNB : Msg1 - RA Preamble Transmission  
gNB → UE : Msg2 - RA Response  
UE → gNB : Msg3 - Scheduled PUSCH Transmission  
gNB → UE : Msg4 - Contention Resolution
```

| Message  | Direction | Channel | Content                          | Purpose                               |
| -------- | --------- | ------- | -------------------------------- | ------------------------------------- |
| **Msg1** | UE → gNB  | PRACH   | RA Preamble                      | Start random access                   |
| **Msg2** | gNB → UE  | PDSCH   | TA, RAPID, Uplink Grant, TC-RNTI | Send Random Access Response           |
| **Msg3** | UE → gNB  | PUSCH   | Temp ID, RRC Request             | Send UE ID / connection request       |
| **Msg4** | gNB → UE  | PDSCH   | Contention Resolution            | Confirm UE identity and assign C-RNTI |

- Step 4：RRC Connection
- After completing the random access procedure, the UE establishes an RRC (Radio Resource Control) connection with the gNB.
This step transitions the UE from IDLE mode to RRC Connected mode, enabling control-plane communication.

```
UE (IDLE) → gNB : RRC Connection Request   //The UE sends an RRC Connection Request message to the gNB containing
gNB → UE : RRC Connection Setup //The gNB responds with RRC Connection Setup, configuring radio parameters, control channels (e.g., SRB/RLC/MAC), and assigning identifiers to the UE.
UE → gNB : RRC Connection Setup Complete   //UE sends RRC Connection Setup Complete to confirm successful setup and may 
gNB → UE : RRC Connection Reconfiguration include a NAS message (e.g., Registration Request). //The gNB sends RRC Connection Reconfiguration to modify or enhance configurations (e.g., handover, measurement activation, DRB setup).
The UE replies with RRC Connection Reconfiguration Complete to confirm.
UE → gNB : RRC Connection Reconfiguration Complete 
```
- Step 5：NAS Registration
  - After RRC connection, the UE sends a NAS Registration Request to the AMF to register with the 5G network.
  - The message includes the UE’s security credentials and network capabilities.
  - The network performs authentication and security key exchange for secure communication.
  - Once verified, the AMF replies with a Registration Accept, which includes the 5G-GUTI and other configuration details.

- Step 6：PDU Session Establishment
  - To start a data session, the UE sends a PDU Session Establishment Request to the SMF, The message includes session requirements, SSC Mode, and the 5G-GUTI.
  - The SMF processes the request, allocates resources, and replies with a PDU Session Establishment Accept containing QoS parameters and the IP address.
 
- Step 7：Send/Receive Data
  - After the PDU session is established, the UE can now send and receive data through the 5G network (e.g., internet access, streaming, or voice communication).


# xApp Development Guide
## xApp Lifecycle

| stage            | description                      |
| --------------   | -------------------------------- |
| **Init**         | E42 Setup, E2 State              |
| **Indication**   | SM Subscription, Data Collection |
| **Logic**        | Data Analysis, Decision Making   |
| **Control**      | RAN Control, Slicing, RRM        |
| **Exit**         | SM Unsubscribe, E42 Teardown     |

<img width="860" height="785" alt="image" src="https://github.com/user-attachments/assets/498680f4-cb5c-4bd7-9737-be889f143f89" />

### Init
- The first step of every xApp is to initialize it. This includes:
  1. Parsing the input arguments from folders or files.
  2. Setting up the E42 interface towards FlexRIC
  3. Requesting and reading the current E2 state, which contains the available E2 Nodes and RAN functions
 
### Indication — Data Subscription & Handling
- In this step, we define how to handle incoming indication messages from the selected elements.
We write a custom indication callback function to:
1. Process incoming messages
2. Parse desired metrics
3. Store or calculate new values

- To receive indications, the xApp subscribes to the desired E2 Node and RAN function using the proper Service Model (SM).
During subscription, the xApp provides:
1. The ID(s) of the E2 Node(s)
2. The Service Model ID
3. The reporting interval (e.g., 5 ms)
4. The custom callback function for message handling

- Logic — Core Intelligence Phase
- This is the creative phase of the xApp, where we design the core logic to address a specific use case.
- At this stage, the xApp has already gathered and processed indication messages from E2 Nodes, it is now the ideal time to perform data analysis, extract features, and generate forecasts, turning raw data into actionable insights.

| type                                                  | description                                        |
| --------------------------------------------------    | -------------------------------------------------- |
| **Exploratory Data Analysis (EDA)**                   | Analyze data distribution and detect anomalies     |
| **Pre-processing & Feature Extraction**               | Clean and extract relevant features                |
| **Forecasting (Supervised/Unsupervised Learning)**    | Predict future trends using ML models              |
| **Reinforcement Learning (RL)**                       | Optimize actions based on feedback loops           |
| **Data Set Creation & Augmentation**                  | Generate or augment datasets for robustness        |
| **Model Training & Continuous Learning**              | Train models and adapt continuously                |
| **Model Evaluation & Tuning**                         | Evaluate performance and fine-tune hyperparameters |

- Based on data analysis and network observability, the xApp determines the appropriate control actions to apply to the RAN
to ensure that the network operates closer to the desired state.

## xApp SDK design
- The xApp SDK is designed to be simple yet flexible, enabling efficient xApp development tailored to user needs, and it provides standardized APIs that manage the entire xApp lifecycle.
- Thanks to SWIG, xApps can be developed in multiple languages, including C/C++ and Python.
The C SDK provides the fundamental functions that manage the xApp lifecycle and Service Models (SM).
These functions form the backbone of xApp development, supporting innovation and flexibility.
<img width="423" height="377" alt="image" src="https://github.com/user-attachments/assets/70f86a07-d470-4fd2-9176-07505cd63e10" />

- SWIG acts like a translator, allowing different languages ​​(such as Python) to "understand" the functionality provided by the C/C++ SDK.


# rApp Development Guide
## overview
- ODIN is BubbleRAN’s logical controller that manages and controls Network Functions (NFs).
In BubbleRAN, control means ODIN can access and modify NFs internally — like handling them as white-boxes, not black-boxes.
- Because every NF (gNB, AMF, SMF) behaves differently, ODIN must collect specific statistics, make decisions, and send back policies or configurations to adapt the network dynamically.
- Thus, ODIN requires a deep understanding of both the core network and the RAN to properly tune or reconfigure NFs to meet user requirements (like QoS, energy efficiency, load balancing, etc.).
- When focusing on the Radio Access Network (RAN) part, ODIN plays the role equivalent to the Non-Real-Time RIC (Non-RT RIC) in the O-RAN architecture — meaning it performs higher-level, slower-time-scale control and optimization functions.

## BubbleRAN Operator Plane
<img width="870" height="533" alt="image" src="https://github.com/user-attachments/assets/df4c595b-b393-47b4-956f-6f0c0870708e" />
- The Operator Plane in BubbleRAN is a cloud-native control and orchestration layer, built on the Kubernetes Operator Pattern.

| Level                                            | Name                          | Description                                                                                                                                                                                                                                                                                                                                           |
| ------------------------------------------------ | ----------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **rApps, Operators, Agents**                 | Application Abstraction Layer | Defines network **objectives and policies**, representing high-level decision logic such as **energy-saving strategies, traffic optimization, and QoS management**. This layer is usually handled by **rApps**, similar to the **Non-RT RIC** in O-RAN.                                                                                               |
| **Control Subsystem / Management Subsystem** | Executive Layer               | Executes high-level strategies:<br>• **Control Subsystem** handles the control plane (e.g., resource allocation, scheduling, interference control).<br>• **Management Subsystem** manages configuration and monitoring (e.g., automated deployment, state management).<br>This layer can be seen as the operational core for **xApps and Operators**. |
| **Network Entities**                         | Network Entities Layer        | Includes actual network functions such as **Terminal**, **Access (e.g., gNB)**, **Edge**, and **Core**. These are the components being managed and controlled.                                                                                                                                                                                        |
| **Telco-Optimized K8s Cluster**              | Infrastructure Layer          | Composed of a telecom-optimized **Kubernetes cluster** that provides deployment and container orchestration. All control modules (e.g., **ODIN**, **Athena**, **Odin Controller**) in BubbleRAN run on this layer.                                                                                                                                    |

| **BubbleRAN Component**             | **Corresponding O-RAN Role** | **Functional Level**                |
| ----------------------------------- | ---------------------------- | ----------------------------------- |
| **rApps / Operators / Agents**      | **Non-RT RIC**               | **Policy and Objective Definition** |
| **Control & Management Subsystems** | **Near-RT RIC**              | **Execution and Control**           |
| **Access / Edge / Core Entities**   | **O-DU / O-CU / Core NFs**   | **Physical Execution Elements**     |
| **Telco-Optimized K8s Cluster**     | **Cloud Infrastructure**     | **Execution Environment**           |

## Core Components
### OAM Component: athena-base-operator
- Implements the foundational Network, Element, and Composition Models.
- Enables full lifecycle management of RAN elements — including deploy, monitor, update, and delete.

### Non-RT RIC Component: odin-control-manager
- Acts as the control gateway inside the Operator Plane.
- Interfaces with rApps via the R1 interface and with xApps through A1.
- Handles complex orchestration, policy enforcement, and decision dissemination across the RAN.
- Ensures alignment between intent (e.g., SLA objectives) and execution (actual network actions).

<img width="864" height="632" alt="image" src="https://github.com/user-attachments/assets/1082e77a-616a-4d13-9d98-499fb6f3f2df" />

- ODIN functions as a Non-RT RIC Orchestrator, providing a central decision-making and orchestration layer within BubbleRAN.

### BubbleRAN vs O-RAN 
| **Item**                   | **BubbleRAN**                                                                                                                    | **O-RAN**                                                                                                                                     | **Correspondence and Explanation**                                                                                                         |
| -------------------------- | -------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| **Overall Positioning**    | A **cloud-native RAN platform based on Kubernetes**, where gNB, RIC, and OAM are fully containerized and automated.              | An **open RAN standardized architecture** defined by the O-RAN Alliance, focusing on interface standardization and functional modularization. | BubbleRAN can be regarded as **an implementation platform of the O-RAN architecture**, realizing modularity and automation via Kubernetes. |
| **Control Plane**          | **ODIN Controller / odin-control-manager** — responsible for policy delivery and decision control, equivalent to the Non-RT RIC. | **Non-RT RIC** — responsible for strategic decisions such as energy saving, load prediction, and AI/ML-based control.                         | Both share similar roles. **ODIN corresponds to the Non-RT RIC.**                                                                          |
| **Application Layer**      | **rApps, Operators, Agents** — define high-level intents and policies for policy-based control.                                  | **rApps** — policy applications running on the SMO / Non-RT RIC.                                                                              | Conceptually identical; BubbleRAN implements them through **Kubernetes Operators**.                                                        |
| **Execution Layer**        | **Control & Management Subsystems** — execute xApp logic and operational instructions.                                           | **Near-RT RIC + xApps** — handle real-time or near-real-time control (milliseconds to seconds).                                               | BubbleRAN integrates xApp logic into the operator plane; its functionality matches that of the Near-RT RIC.                                |
| **Network Entities Layer** | **Access / Edge / Core Entities** — actual deployed components such as gNB, Core, and UE.                                        | **O-DU / O-CU / Core NFs / UE**                                                                                                               | BubbleRAN simulates the physical components in O-RAN, all containerized and deployable.                                                    |
| **OAM / Management Layer** | **athena-base-operator** — manages the full lifecycle of components (deploy, monitor, update, delete).                           | **OAM / SMO (Service Management and Orchestration)**                                                                                          | **Athena** corresponds to the **SMO’s** core management functionality in O-RAN.                                                            |
| **Platform Foundation**    | **Telco-Optimized Kubernetes Cluster** — supports automation and scalability.                                                    | **Cloud or on-premises platforms**, depending on vendor implementation.                                                                       | BubbleRAN natively integrates Kubernetes, making it a fully cloud-native system.                                                           |
| **Control Interfaces**     | **R1** (rApp ↔ ODIN) <br> **A1** (ODIN ↔ xApp)                                                                                   | **R1** (rApp ↔ Non-RT RIC) <br> **A1** (Non-RT ↔ Near-RT RIC)                                                                                 | Identical naming and purpose — BubbleRAN strictly follows O-RAN interface design.                                                          |
| **Development Approach**   | Provides developers with **modular SDKs, Operator APIs, and rApp/xApp frameworks**, enabling customization and rapid deployment. | Focuses on **standardized interfaces and interoperability**, with applications developed by various vendors.                                  | BubbleRAN serves as an **implementation and experimental platform** for O-RAN concepts.                                                    |
| **Goal Orientation**       | Enables rapid building, simulation, and testing of **RAN automation and AI-driven control** in an end-to-end setup.              | Aims to establish **a globally standardized open RAN** framework.                                                                             | BubbleRAN represents the **research and practical extension** of the O-RAN standard.                                                       |
