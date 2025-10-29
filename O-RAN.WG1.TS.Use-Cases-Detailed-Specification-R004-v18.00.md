<!-- TOC -->
- [4.8 QoS-based Resource Optimization](https://github.com/zongyi0311/o-ran-background-knowledge/blob/main/O-RAN.WG1.TS.Use-Cases-Detailed-Specification-R004-v18.00.md#48-qos-based-resource-optimization)
  - [4.8.1 Background Information](https://github.com/zongyi0311/o-ran-background-knowledge/blob/main/O-RAN.WG1.TS.Use-Cases-Detailed-Specification-R004-v18.00.md#481-background-information)
    - [1. Definition & Purpose](https://github.com/zongyi0311/o-ran-background-knowledge/blob/main/O-RAN.WG1.TS.Use-Cases-Detailed-Specification-R004-v18.00.md#1-definition--purpose)
    - [2. Role of the RAN Layer](https://github.com/zongyi0311/o-ran-background-knowledge/blob/main/O-RAN.WG1.TS.Use-Cases-Detailed-Specification-R004-v18.00.md#2-role-of-the-ran-layer)
    - [3. RAN Sub-slice Instantiation](https://github.com/zongyi0311/o-ran-background-knowledge/blob/main/O-RAN.WG1.TS.Use-Cases-Detailed-Specification-R004-v18.00.md#3-ran-sub-slice-instantiation)
    - [4. Handling Resource Shortages](https://github.com/zongyi0311/o-ran-background-knowledge/blob/main/O-RAN.WG1.TS.Use-Cases-Detailed-Specification-R004-v18.00.md#4-handling-resource-shortages)
    - [5. NR NRM](https://github.com/zongyi0311/o-ran-background-knowledge/blob/main/O-RAN.WG1.TS.Use-Cases-Detailed-Specification-R004-v18.00.md#5-nr-nrm)
   
- [4.8.2 Motivation]()

<!-- /TOC -->
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
