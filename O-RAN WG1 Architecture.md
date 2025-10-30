# 5. O-RAN architecture
- This section provides a high-level overview of the O-RAN architecture, showing how the interfaces — A1, O1, Open Fronthaul M-plane, and O2 — connect the SMO framework to O-RAN Network Functions (NFs) and the O-Cloud.
- The O-Cloud includes an O-Cloud Notification API, enabling relevant NFs (e.g., Near-RT RIC, O-CU-CP, O-CU-UP, O-DU) to receive event notifications, all O-RAN NFs, except O-RU, are managed via the O1 interface under the SMO.
- The Open Fronthaul M-plane between SMO and O-RU supports hybrid management mode.
- The Near-RT RIC offers RAN analytics information services through the Y1 interface, as defined in Y1GAP.
  - Y1 Consumers access these services after mutual authentication and authorization.
  - If the Y1 Consumer is outside the PLMN trusted domain, secure access is ensured via exposure function

| Item                                    | English Description                                                                          |
| --------------------------------------- | -------------------------------------------------------------------------------------------- |
| **Core Framework**                      | SMO connects with O-RAN NFs via A1, O1, O2, and M-plane interfaces.                          |
| **O-Cloud Role**                        | Hosts O-RAN NFs and provides event notification APIs via the O-Cloud Notification interface. |
| **O1 Interface**                        | Main management channel between SMO and O-RAN NFs (except O-RU).                             |
| **M-plane**                             | Supports hybrid management between SMO and O-RU.                                             |
| **AAL (Accelerator Abstraction Layer)** | Provides acceleration APIs for O-RAN NFs instantiated in O-Cloud.                            |
| **Y1 Interface**                        | Used by Near-RT RIC to expose RAN analytics information services to authorized Y1 consumers. |
| **Security**                            | Y1 Consumers require mutual authentication and secure access if outside PLMN trusted domain. |

- <img width="579" height="296" alt="image" src="https://github.com/user-attachments/assets/e8ba641e-b991-4af2-9937-91643a5052c9" />
- The O-RAN architecture consists of three main layers:
  - Service Management and Orchestration (SMO) — includes the Non-RT RIC and manages global orchestration.
  - O-RAN Network Functions (NFs) — includes Near-RT RIC, O-CU-CP, O-CU-UP, O-DU, and O-RU.
  - O-Cloud — provides the virtualized and physical infrastructure to host O-RAN NFs.
 
| Interface                  | Description                                                                       |
| -------------------------- | --------------------------------------------------------------------------------- |
| **A1**                     | Connects Non-RT RIC to Near-RT RIC for policy control and AI/ML model management. |
| **O1**                     | Connects SMO to O-RAN NFs for management, monitoring, and configuration.          |
| **O2**                     | Connects SMO to O-Cloud for resource and orchestration management.                |
| **Open Fronthaul M-Plane** | Between SMO and O-RU, supports hierarchical or hybrid management.                 |
| **Y1**                     | Connects Near-RT RIC to Y1 consumers for RAN analytics information services.      |
| **NG**                     | Connects O-RAN NFs (e.g., O-CU) to the 5G Core (5GC).                             |

| Item                     | Description                                                                       |
| ------------------------ | --------------------------------------------------------------------------------- |
| **Architecture Layers**  | SMO → RIC → O-RAN NFs → O-Cloud → O-RU → 5GC                                      |
| **Control Hierarchy**    | Non-RT RIC (>1 s) → Near-RT RIC (10 ms–1 s) → O-DU/O-CU (real time).              |
| **Management Scope**     | SMO manages NFs via O1; O-Cloud resources via O2; O-RU via M-Plane.               |
| **Inter-RIC Flow**       | A1 policy exchange between Non-RT RIC and Near-RT RIC.                            |
| **External Integration** | Y1 analytics and O-Cloud notification APIs support AI/ML and data-driven control. |

<img width="698" height="462" alt="image" src="https://github.com/user-attachments/assets/2dae1f0c-624f-4b9a-96bc-3e996b3cc1c1" />

| Item                     | Description                                                                      |
| ------------------------ | -------------------------------------------------------------------------------- |
| **Purpose**              | Shows end-to-end interaction from UE through the O-RAN stack to SMO and O-Cloud. |
| **Main Control Flow**    | SMO → Non-RT RIC (A1) → Near-RT RIC (E2) → O-DU/O-CU → O-RU → UE.                |
| **LTE Integration**      | O-eNB supports LTE Uu interface separately from O-RAN NR structure.              |
| **Virtualization Layer** | O-Cloud provides compute and virtualization for O-RAN NFs.                       |
| **Interface Legend**     | Green = O-RAN interfaces; Black = 3GPP interfaces.                               |

- **Loop introduce**

| Type                       | Control Scope                                                | Time Scale  | Control Entity          | Examples                                                    |
| -------------------------- | ------------------------------------------------------------ | ----------- | ----------------------- | ----------------------------------------------------------- |
| **Non-RT Control Loop**    | Policy management, AI/ML model training, global optimization | ≥ 1 s       | **Non-RT RIC / SMO**    | Energy saving, slice policy enforcement                     |
| **Near-RT Control Loop**   | Dynamic RAN optimization, traffic balancing                  | 10 ms – 1 s | **Near-RT RIC (xApps)** | QoS control, handover optimization, interference mitigation |
| **Real-Time Control Loop** | Physical layer, instantaneous scheduling                     | < 10 ms     | **O-DU / PHY Layer**    | HARQ, beamforming, MAC scheduling                           |

- Quick note
| Item                 | Description                                                                                 |
| -------------------- | ------------------------------------------------------------------------------------------- |
| **Purpose**          | Defines control loop hierarchy (Non-RT, Near-RT, RT) within the O-RAN system.               |
| **Key Interfaces**   | A1 (Non-RT ↔ Near-RT), E2 (Near-RT ↔ O-CU/O-DU), Open FH M/CUS Planes (O-DU ↔ O-RU).        |
| **Control Flow**     | SMO → Non-RT RIC → Near-RT RIC → O-DU/O-RU → UE.                                            |
| **Execution Layers** | Each layer operates with distinct latency and scope to ensure stability and responsiveness. |

























