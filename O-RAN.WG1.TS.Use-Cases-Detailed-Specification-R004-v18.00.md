- [4.8 QoS-based Resource Optimization](https://github.com/zongyi0311/o-ran-background-knowledge/blob/main/O-RAN.WG1.TS.Use-Cases-Detailed-Specification-R004-v18.00.md#48-qos-based-resource-optimization)
  - [4.8.1 Background Information](4.8.1-Background-Information)
    - [1. Definition & Purpose](https://github.com/zongyi0311/o-ran-background-knowledge/edit/main/O-RAN.WG1.TS.Use-Cases-Detailed-Specification-R004-v18.00.md#1-definition--purpose)
    - [2. Role of the RAN Layer](https://github.com/zongyi0311/o-ran-background-knowledge/edit/main/O-RAN.WG1.TS.Use-Cases-Detailed-Specification-R004-v18.00.md#2-role-of-the-ran-layer)
    - [3. RAN Sub-slice Instantiation](https://github.com/zongyi0311/o-ran-background-knowledge/edit/main/O-RAN.WG1.TS.Use-Cases-Detailed-Specification-R004-v18.00.md#ran-sub-slice-instantiation)
    - [4. Handling Resource Shortages](https://github.com/zongyi0311/o-ran-background-knowledge/edit/main/O-RAN.WG1.TS.Use-Cases-Detailed-Specification-R004-v18.00.md#4-handling-resource-shortages)
    - [5. NR NRM](https://github.com/zongyi0311/o-ran-background-knowledge/edit/main/O-RAN.WG1.TS.Use-Cases-Detailed-Specification-R004-v18.00.md#5-nr-nrm)
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
