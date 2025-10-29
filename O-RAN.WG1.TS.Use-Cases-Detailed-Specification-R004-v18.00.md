- [4.8 QoS-based Resource Optimization](#48-qos-based-resource-optimization)
  - [4.8.1 Background Information](#481-background-information)
    - [1. Definition \& Purpose](#1-definition--purpose)
    - [2. Role of the RAN Layer](#2-role-of-the-ran-layer)
    - [3. RAN Sub-slice Instantiation](#3-ran-sub-slice-instantiation)
    - [4. Handling Resource Shortages](#4-handling-resource-shortages)
    - [5. NR NRM](#5-nr-nrm)
  - [4.8.2 Motivation](#482-motivation)
    - [1. Example Use Case](#1-example-use-case)
    - [2. Problem Scenario](#2-problem-scenario)
    - [3. Dynamic Optimization via RIC](#3-dynamic-optimization-via-ric)
    - [4. Use Case Summary](#4-use-case-summary)
  - [4.8.3 Proposed Solution](#483-proposed-solution)
<!-- TOC -->
<!-- /TOC -->
- [4.8 QoS-based Resource Optimization](https://github.com/zongyi0311/o-ran-background-knowledge/blob/main/O-RAN.WG1.TS.Use-Cases-Detailed-Specification-R004-v18.00.md#48-qos-based-resource-optimization)
  - [4.8.1 Background Information](https://github.com/zongyi0311/o-ran-background-knowledge/blob/main/O-RAN.WG1.TS.Use-Cases-Detailed-Specification-R004-v18.00.md#481-background-information)
    - [1. Definition & Purpose](https://github.com/zongyi0311/o-ran-background-knowledge/blob/main/O-RAN.WG1.TS.Use-Cases-Detailed-Specification-R004-v18.00.md#1-definition--purpose)
    - [2. Role of the RAN Layer](https://github.com/zongyi0311/o-ran-background-knowledge/blob/main/O-RAN.WG1.TS.Use-Cases-Detailed-Specification-R004-v18.00.md#2-role-of-the-ran-layer)
    - [3. RAN Sub-slice Instantiation](https://github.com/zongyi0311/o-ran-background-knowledge/blob/main/O-RAN.WG1.TS.Use-Cases-Detailed-Specification-R004-v18.00.md#3-ran-sub-slice-instantiation)
    - [4. Handling Resource Shortages](https://github.com/zongyi0311/o-ran-background-knowledge/blob/main/O-RAN.WG1.TS.Use-Cases-Detailed-Specification-R004-v18.00.md#4-handling-resource-shortages)
    - [5. NR NRM](https://github.com/zongyi0311/o-ran-background-knowledge/blob/main/O-RAN.WG1.TS.Use-Cases-Detailed-Specification-R004-v18.00.md#5-nr-nrm)
   
- [4.8.2 Motivation]()
- 
# 4.8 QoS-based Resource Optimization
## 4.8.1 Background Information
### 1. Definition & Purpose
- QoS-based resource optimization is used to provide preferential Quality of Service (QoS) for certain users.
- A common use case is in networks supporting end-to-end (e2e) slicing.
- Its main functions are:
  - Ensuring resource isolation between slices.
  - Monitoring Service Level Specifications (SLS) for each slice.
 
### 2. Role of the RAN Layer
- In RAN, the scheduler manages the allocation of Physical Resource Blocks (PRBs):
  - Ensures PRB isolation between slices
  - Optimizes PRB utilization to fulfill SLS for different slices
  - Default RAN behavior is configured via O1 interface (e.g., PRB ratio reserved for each slice)
  - QoS guides the scheduler to allocate PRBs dynamically in real time to meet specific slice SLS
  - In NR NRM, this is described by the resource partition attribute
 
### 3. RAN Sub-slice Instantiation
- Instantiation of a RAN sub-slice requires careful planning to ensure deployed RAN resources can support the sub-slice SLS
- Part of this process involves configuring RAN functionalities via the O1 interface, providing a default behavior capable of satisfying most SLS conditions

### 4. Handling Resource Shortages
- Even with rigorous planning, there may be cases where RAN resources are insufficient to meet SLS requirements
- To handle this:
  - The SMO (Service Management and Orchestration) continuously monitors RAN slice performance.
  - When SMO detects that SLS cannot be fulfilled:
    - The Non-RT RIC can use A1 policies to improve the situation
    - The Non-RT RIC also leverages additional information available from SMO for decision-making and optimization
   
|  Topic              | English Summary                                                                 |
| --------------------- |------------------------------------------------------------------------------- |
|  Purpose          | Provides preferential QoS, ensures slice isolation, and monitors SLS            |
|  Implementation   | Scheduler manages PRB allocation; O1 interface configures slice resource ratios |
|  Failure Handling | SMO monitors; if SLS unmet → Non-RT RIC applies A1 policies                     |
|  Components       | RAN, O1, SMO, Non-RT RIC, NR NRM                                                |

### 5. NR NRM
- NR NRM (NR Network Resource Model) is an information model defined by 3GPP to describe the network resources, relationships, and attributes within a 5G NR (New Radio) network.
- It provides a standardized data model that allows management and orchestration systems (like SMO or RIC) to:
  - Monitor the state of RAN components
  - Configure parameters such as power, bandwidth, or PRB allocation
  - Represent the network topology and resource hierarchy
 
## 4.8.2 Motivation
### 1. Example Use Case
- To illustrate the use case, the document uses an emergency service as a slice tenant.
It is assumed that 50% of PRBs in an area are reserved for emergency users
- This ensures that, even under high network load, emergency communications maintain service quality.
- For instance, emergency personnel’s video cameras maintain a minimum bitrate of 500 kbps.

### 2. Problem Scenario
- During an emergency, such as a fire, multiple personnel stream live videos, leading to high traffic within the slice.
- Even with 50% PRBs reserved, resources may still be insufficient to handle all video streams.
- In this case:
  - The operator may reconfigure PRB allocation to favor emergency slices.
  - However, this could degrade the SLA of other slices.
  - Despite this, some video streams may still lack sufficient resolution due to physical limitations.
 
### 3. Dynamic Optimization via RIC
- The emergency control command selects one critical video feed and requests higher resolution.
- Steps:
  - The emergency control system determines which video stream needs higher resolution.
  - It reports this to the end-to-end slice assurance function (e.g., via an Edge API).
  - The Non-RT RIC uses this information to define an A1 policy influencing resource allocation.
  - The Near-RT RIC applies the modified QoS target over the E2 interface, ensuring sufficient bandwidth for the selected stream.
 
### 4. Use Case Summary
| Item                      | Description                                                                                      |
| ------------------------- | ------------------------------------------------------------------------------------------------ |
| **Purpose**               | Dynamically adjust QoS to ensure service quality for critical users during emergency situations. |
| **Main Components**       | Non-RT RIC, Near-RT RIC, SMO, Edge API                                                           |
| **Main Interfaces**       | A1 (policy control), E2 (QoS enforcement and execution)                                          |
| **Application Scenarios** | Emergency response, disaster monitoring, public safety slice                                     |
| **Core Mechanism**        | Non-RT RIC issues A1 policy → Near-RT RIC executes E2 control → dynamic PRB / QoS adjustment     |

## 4.8.3 Proposed Solution
### 1. Main Objective
- This section describes how O-RAN components collaborate to achieve intelligent QoS-based resource optimization.
- The main objective is to enable dynamic and efficient QoS adjustments at the RAN level by leveraging SMO, Non-RT RIC, Near-RT RIC, O-CU, and O-DU functions.

### 2. Component Responsibilities

| Component           | English Description                                                                                                                             |
| ------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| **Non-RT RIC**      | - Provides **A1 policies** to Near-RT RIC to guide RAN-level QoS optimization.<br>- Monitors QoS-related metrics from both the network and SMO. |
| **Near-RT RIC**     | - Executes QoS enforcement decisions received from Non-RT RIC via A1.<br>- Applies these controls on E2 nodes influencing RRM behavior.         |
| **O-CU / O-DU**     | - Provide UE-level performance metrics with defined granularity to SMO via **O1 interface**.                                                    |
| **SMO**             | - Collects and aggregates RAN and external performance data.<br>- Supports decision-making for resource allocation.                             |
| **External Source** | - May provide non-RAN data, such as UE priority or application-level information, to SMO.                                                       |

### 3. System Flow Summary
- 1 .O-CU / O-DU report UE performance metrics to SMO.
- 2. SMO aggregates RAN and external data (e.g., UE priorities).
- 3. Non-RT RIC generates A1 policies to guide Near-RT RIC optimization.
- 4. Near-RT RIC sends QoS control instructions to E2 nodes.
- 5. E2 nodes adjust Radio Resource Management (RRM) behavior accordingly.

# 4.21 Network Energy Saving
## 4.21.1 Background Information
- Energy Consumption (EC) of the RAN is a critical issue for 5G network operators.
- RAN Energy Saving (ES) depends on careful planning and configuration.
- Common ES Techniques:
  - Deep Sleep Mode: shutting down base stations, carriers, or RF channels.
  - Short-term ES: symbol-, subframe-, and frame-level mechanisms known as Advanced Sleep Modes (ASM).
- Main EC Contributors:
- O-RU contributes the largest portion of RAN energy usage.

| Item                       | English Description                                                              |
| -------------------------- | -------------------------------------------------------------------------------- |
| **Goal**                   | Reduce RAN energy consumption through advanced energy-saving mechanisms.         |
| **Main Focus**             | O-RU energy optimization, VNF power estimation, dynamic sleep control.           |
| **Challenges**             | Varying traffic patterns, lack of standardized VNF EC models (until Rel-18).     |
| **Techniques**             | Deep Sleep Mode, Carrier Shutdown, RF On/Off, Advanced Sleep Mode (ASM).         |
| **Standardization Status** | EC measurement for VNFs still under study; will be addressed in 3GPP Release 18. |

## 4.21.2 Motivation
- 3GPP defines both centralized and distributed ES features, mainly targeting cell on/off switching within or across RATs and in legacy and 5G networks, Energy Saving (ES) can be achieved through manual configuration or SON functions.
- The motivation for O-RAN ES is to leverage AI/ML-based services and open interfaces to enable smarter ES control.

## 4.21.3 Proposed Solution

| Item                | English Description                                                                        |
| ------------------- | ------------------------------------------------------------------------------------------ |
| **Purpose**         | Optimize network energy consumption using AI/ML-driven O-RAN energy-saving mechanisms.     |
| **Main Methods**    | ① Carrier/Cell on-off switching ② RF Channel on-off switching ③ Advanced Sleep Mode (ASM). |
| **Main Actors**     | Non-RT RIC, Near-RT RIC, SMO, O-CU, O-DU, O-RU.                                            |
| **Interfaces Used** | O1, E2, and Open Fronthaul M-plane.                                                        |
| **Impact**          | Enables dynamic energy saving across RAN components at different time scales.              |
