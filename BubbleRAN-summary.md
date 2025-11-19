- [BubbleRAN vs O-RAN Architecture Summary](#bubbleran-vs-o-ran-architecture-summary)
  - [1. Operator Plane vs Vendor Plane](#1-operator-plane-vs-vendor-plane)
  - [2. The Four Layers of BubbleRAN](#2-the-four-layers-of-bubbleran)
    - [Layer 1: rApps / Operators / Agents](#layer-1-rapps--operators--agents)
    - [Layer 2: Control Subsystem (ODIN)](#layer-2-control-subsystem-odin)
    - [Layer 2: Management Subsystem (Athena)](#layer-2-management-subsystem-athena)
    - [Layer 3: Network Entities](#layer-3-network-entities)
    - [Layer 4: Telco-Optimized Kubernetes](#layer-4-telco-optimized-kubernetes)
  - [3. Where is the Near-RT RIC?](#3-where-is-the-near-rt-ric)
  - [4. BubbleRAN ↔ O-RAN Mapping Table](#4-bubbleran--o-ran-mapping-table)
  - [5. What is the Vendor Plane?](#5-what-is-the-vendor-plane)
- [BubbleRAN vs O-RAN](#bubbleran-vs-o-ran)

# BubbleRAN vs O-RAN Architecture Summary

## 1. Operator Plane vs Vendor Plane

**Operator Plane**: Policies, orchestration, automation (SMO + Non-RT RIC)  
**Vendor Plane**: Real-time execution (Near-RT RIC + RAN + Core)

---

## 2. The Four Layers of BubbleRAN

![alt text](image.png)

### Layer 1: rApps / Operators / Agents

- Define policies, intents, objectives
- O-RAN equivalent: Non-RT RIC (rApps)
- Interface: R1

### Layer 2: Control Subsystem (ODIN)

- odin-control-manager = Non-RT RIC
- Receives rApp policies (R1)
- Sends A1 targets to xApps
- Decision orchestration and policy enforcement

### Layer 2: Management Subsystem (Athena)

- athena-base-operator = SMO / OAM
- Handles network model, lifecycle, monitoring, deployment, updates

### Layer 3: Network Entities

- Terminal / Access (gNB) / Edge / Core
- O-RAN equivalent: O-DU / O-CU / Core NFs

### Layer 4: Telco-Optimized Kubernetes

- Full containerization of BubbleRAN components
- O-RAN equivalent: O-Cloud

---

## 3. Where is the Near-RT RIC?

- BubbleRAN implements Near-RT RIC using **FlexRIC + xApps** (Vendor Plane)
- ODIN provides A1 policy input to Near-RT RIC

Control pipeline:

```
rApp → R1 → ODIN → A1 → xApp → E2 → RAN
```

---

## 4. BubbleRAN ↔ O-RAN Mapping Table

| BubbleRAN         | O-RAN              | Function                     |
| ----------------- | ------------------ | ---------------------------- |
| rApps / Operators | Non-RT RIC (rApps) | Policy & Intent              |
| ODIN              | Non-RT RIC         | A1 Policy Control            |
| Athena            | SMO/OAM            | Lifecycle & State Management |
| FlexRIC + xApps   | Near-RT RIC        | Real-time control            |
| gNB / Core        | O-DU / O-CU / 5GC  | Network Functions            |
| Kubernetes        | O-Cloud            | Cloud Execution Layer        |

---

## 5. What is the Vendor Plane?

- The actual execution domain: Near-RT RIC, xApps, RAN, Core
- In BubbleRAN: FlexRIC + xApps + OAI gNB + BubbleRAN Core

---

# BubbleRAN vs O-RAN

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

- [BubbleRAN vs O-RAN Architecture Summary](#bubbleran-vs-o-ran-architecture-summary)
  - [1. Operator Plane vs Vendor Plane](#1-operator-plane-vs-vendor-plane)
  - [2. The Four Layers of BubbleRAN](#2-the-four-layers-of-bubbleran)
    - [Layer 1: rApps / Operators / Agents](#layer-1-rapps--operators--agents)
    - [Layer 2: Control Subsystem (ODIN)](#layer-2-control-subsystem-odin)
    - [Layer 2: Management Subsystem (Athena)](#layer-2-management-subsystem-athena)
    - [Layer 3: Network Entities](#layer-3-network-entities)
    - [Layer 4: Telco-Optimized Kubernetes](#layer-4-telco-optimized-kubernetes)
  - [3. Where is the Near-RT RIC?](#3-where-is-the-near-rt-ric)
  - [4. BubbleRAN ↔ O-RAN Mapping Table](#4-bubbleran--o-ran-mapping-table)
  - [5. What is the Vendor Plane?](#5-what-is-the-vendor-plane)
- [BubbleRAN vs O-RAN](#bubbleran-vs-o-ran)
