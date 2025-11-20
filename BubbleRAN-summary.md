- [BubbleRAN Architecture](#bubbleran-architecture)
  - [1. Operator Plane and Vendor Plane](#1-operator-plane-and-vendor-plane)
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

# BubbleRAN Architecture

## 1. Operator Plane and Vendor Plane

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

| 項目                | **BubbleRAN**                                           | **O-RAN（標準）**                               | **對應關係與解釋**                         |
| ----------------- | ------------------------------------------------------- | ------------------------------------------- | ----------------------------------- |
| **系統定位**          | 完整可執行的 **Cloud-Native RAN 平台**（K8s 上的 gNB + RIC + Core） | **開放 RAN 標準架構**（定義功能模組與介面）                  | BubbleRAN = O-RAN 標準的 **實作平台**      |
| **Non-RT 控制**     | **ODIN（odin-control-manager）**，策略決策、AI/ML、高層控制          | **Non-RT RIC**                              | 100% 等價：ODIN = Non-RT RIC           |
| **SMO / OAM**     | **Athena（athena-base-operator）**，負責部署/監控/更新/網路模型        | **SMO（Service Management & Orchestration）** | Athena = SMO（負責 OAM）                |
| **策略應用（rApps）**   | rApps + Operators（Kubernetes Operators）                 | rApps                                       | 概念相同；BubbleRAN 用 K8s Operator 實作    |
| **Near-RT 控制**    | 功能由 **FlexRIC + xApps** 於 Vendor Plane 提供               | Near-RT RIC + xApps                         | BubbleRAN 透過 FlexRIC 實現 Near-RT RIC |
| **xApps 執行環境**    | FlexRIC E2 Agent + Containerized xApps                  | Near-RT RIC Service Models (E2SM)           | BubbleRAN 完整支援標準 E2 SM              |
| **E2 介面**         | 由 FlexRIC 提供完整 E2：Setup / Indication / Control          | 標準 E2AP + E2SM                              | 完全遵循 O-RAN                          |
| **A1 介面**         | ODIN ↔ xApps 的策略下發                                      | Non-RT RIC ↔ Near-RT RIC                    | BubbleRAN 實作完整 A1                   |
| **R1 介面**         | rApp ↔ ODIN（policy input）                               | rApp ↔ Non-RT RIC                           | 完全一致                                |
| **RAN 實作**        | Containerized OAI gNB（DU/CU）                            | O-DU / O-CU                                 | BubbleRAN 使用 OAI 作為白盒 gNB           |
| **Core 實作**       | Containerized 5GC（AMF/SMF/UPF）                          | 5GC NFs                                     | BubbleRAN = 可部署的實際 RAN + Core       |
| **執行平台（O-Cloud）** | **Telco-Optimized Kubernetes Cluster**                  | O-Cloud                                     | BubbleRAN = 100% cloud-native RAN   |
| **功能邏輯位置**        | Operator Plane + Vendor Plane                           | SMO + RIC（Non-RT / Near-RT）+ RAN            | 完整對應整套 O-RAN 架構                     |
| **開發支援**          | SDK、Operator API、xApp SDK、rApp SDK                      | 標準，只定義 API，不提供 SDK                          | BubbleRAN = 實驗/研究最佳環境               |
| **目的**            | 快速實作、研究、AI/ML、自動化、RAN 測試                                | 建立業界一致的開放標準                                 | BubbleRAN = 把 O-RAN 落地到可跑的系統        |

| O-RAN 架構區塊        | BubbleRAN 元件           | O-RAN 名稱        | 對應說明                             |
| ----------------- | ---------------------- | --------------- | -------------------------------- |
| **SMO 層**         | **Athena**             | SMO/OAM         | 部署、監控、生命週期、Network/Element Model |
| **Non-RT RIC 層**  | **ODIN**               | Non-RT RIC      | AI/ML、策略、目標下發（R1/A1）             |
| **Near-RT RIC 層** | **FlexRIC + xApps**    | Near-RT RIC     | 準即時控制（E2）                        |
| **RAN 層**         | **OAI gNB（DU/CU）**     | O-DU / O-CU     | PHY, MAC, RLC, PDCP 執行           |
| **Core 層**        | Containerized 5GC      | 5GC AMF/SMF/UPF | 完整 end-to-end 支援                 |
| **Cloud 層**       | **Kubernetes Cluster** | O-Cloud         | 調度、容器管理、自動化                      |





