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

