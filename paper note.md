
-   [1. Motivation & Problem](#1-motivation--problem)
-   [2. Proposed Solution](#2-proposed-solution)
-   [3. System Model](#3-system-model)
-   [4. LSTM-based rApp](#4-lstm-based-rapp)
-   [5. DRL-based xApp](#5-drl-based-xapp)
-   [6. Algorithm (DDRL)](#6-algorithm-ddrl)
-   [7. Simulation Results](#7-simulation-results)
-   [8. Conclusion](#8-conclusion) 

# Open RAN LSTM Traffic Prediction and Slice Management using Deep Reinforcement Learning

**Authors:** Fatemeh Lotfi, Fatemeh Afghah (Clemson University)\
**Year:** 2024\
**Keywords:** Open RAN, Network Slicing, DRL, LSTM, Distributed Learning

------------------------------------------------------------------------

## 1. Motivation & Problem

5G/6G networks require **network slicing** to support different QoS
(latency, reliability, throughput).\
However, real-time **slice management** is challenging because: -
Traffic demand is **non-stationary and unpredictable**. - Multiple DUs
(Distributed Units) in O-RAN have heterogeneous environments. - Static
or offline optimization causes **SLA violations**.

------------------------------------------------------------------------

## 2. Proposed Solution

The authors propose a **Distributed Deep Reinforcement Learning
(DDRL)**-based xApp for O-RAN, **combined with an LSTM-based traffic
prediction rApp**.

-   **rApp (Non-RT RIC):** Uses **LSTM (RNN)** to predict traffic load
    (user density, data rate) for each DU based on historical data.\
-   **xApp (Near-RT RIC):** Uses **DRL (Soft Actor-Critic)** to allocate
    radio resources and manage slices dynamically using predicted data
    from rApp.

This integration improves network awareness and reduces QoS violations.

------------------------------------------------------------------------

## 3. System Model

-   **3 slices:** eMBB, URLLC, mMTC.\
-   **7 DUs**, each serving users with different service demands.\
-   **Optimization goal:** Minimize QoS violations and maximize SLA
    satisfaction.\
-   Formulated as an **MDP (Markov Decision Process)** with:
    -   **State:** QoS (Ql), traffic (Nl, Λl), previous action (at−1)
    -   **Action:** Resource allocation (e, b)
    -   **Reward:** Inverse of QoS violation using arctan mapping

------------------------------------------------------------------------

## 4. LSTM-based rApp

-   Predicts future traffic load (Nl, Λn,l) for each DU.
-   Each LSTM cell processes time-series traffic patterns.
-   Implemented in the **non-real-time RIC**, trained with past data.
-   Outputs predictions to **xApp** in near-real-time RIC.

------------------------------------------------------------------------

## 5. DRL-based xApp

-   Uses **Soft Actor-Critic (SAC)** algorithm:
    -   Multiple **actors** distributed at each DU.
    -   One **critic** network in near-RT RIC.
-   SAC helps balance **exploration vs. exploitation** via entropy
    regularization.
-   Distributed learning allows each DU to share experiences → faster
    convergence and robustness.

------------------------------------------------------------------------

## 6. Algorithm (DDRL)

**Algorithm Steps:** 1. Initialize networks (actors, critic, LSTM
predictor).\
2. For each iteration: - Each DU's actor evaluates environment →
collects experience. - LSTM predicts next-step traffic. - Update critic
and actor parameters (θv, θp) using SAC loss.\
3. Stop when policy converges or after max iterations.

------------------------------------------------------------------------

## 7. Simulation Results

-   Scenario: 3 slices, 7 DUs, 60 users, 20 MHz bandwidth (100 RBs).
-   Compared DDRL (with LSTM prediction) vs. DDRL (without prediction).

**Results:** - **+7.7% higher reward** with LSTM prediction. - **Lower
QoS violations** and smoother throughput. - LSTM-based DDRL better
captures long-term traffic trends (γ=0.99 case). - Improved per-user
throughput even though the goal was mainly QoS stabilization.

------------------------------------------------------------------------

## 8. Conclusion

This work proposes the **first integration of RNN-based rApp and
distributed DRL-based xApp** for O-RAN network slicing.
It demonstrates: - Enhanced **QoS stability**
- Reduced **SLA violations**
- Better **traffic adaptability**


<!-- TOC -->
- [An Open RAN Development Framework with Network Energy Saving rApp Implementation](#an-open-ran-development-framework-with-network-energy-saving-rapp-implementation)
  - [Abstract](#abstract)
  - [Section II – Open RAN Architecture](#section-ii--open-ran-architecture)
  - [Section III – Development Framework](#section-iii--development-framework)
  - [Section IV – AI/ML-Based Energy Saving](#section-iv--aiml-based-energy-saving)
  - [Section V – Visualization](#section-v--visualization)
  - [Section VI – Conclusion](#section-vi--conclusion)
  - [Key Insights](#key-insights)
<!-- /TOC -->


# An Open RAN Development Framework with Network Energy Saving rApp Implementation

## Abstract
This paper presents a lightweight Open RAN development framework that supports the design, testing, and visualization of rApps, focusing on AI/ML-based network energy saving. The framework includes Non-RT RIC functionalities, simplified O1 management, and an E2-Node simulator for dataset generation. The team develops an energy-saving rApp using AI/ML to determine when to switch base stations (BSs) on or off. Results demonstrate effective energy reduction with minimal QoS loss, validating the framework’s extensibility to other O-RAN use cases.

## Section II – Open RAN Architecture
The Open RAN system centers on the Service Management and Orchestration (SMO) layer, hosting the Non-RT RIC and rApps. It interacts with the Near-RT RIC (hosting xApps) and E2 Nodes (O-CU, O-DU) through standardized interfaces (A1, O1, E2). Non-RT RIC manages policies, data exposure, and long-term network optimization (energy saving, load balancing).

## Section III – Development Framework
### Components
- **Non-RT RIC Framework:** Provides R1 services for rApps, including Service, Data, and Policy Management; and simple OAM (via REST).
- **rApps:** Applications connected via R1 to implement custom functions (e.g., energy saving).
- **E2 Nodes / Simulator:** Generate 5G RAN data for AI model training.
- **Visualization (Front-End):** GUI for real-time system monitoring.

### Implementation
- OS: Ubuntu 22.04
- Container: Dockerized services
- RIC Base: O-RAN Software Community (OSC) Non-RT RIC Release I
- Database: CSV / InfluxDB
- API: Simple REST-based O1 substitute

## Section IV – AI/ML-Based Energy Saving
### Scenario
Traditional methods (e.g., greedy BS sleep mode) cannot adapt to time-varying traffic. Thus, the authors use an AI/ML-based, data-driven approach that learns when to switch BSs ON/OFF while maintaining network performance.

### Model
- Model: 3-layer LSTM network (hidden size = 256).
- Input: Sequence of past CSI (channel state information).
- Output: Binary vector αₜ₊₁ indicating ON(1)/OFF(0) for each BS.
- Activation: Sigmoid + Threshold τ=0.5.

### Dataset Generation
- Custom 5G simulator generates CSI and optimal BS ON/OFF pairs.
- UEs move randomly in 3GPP Urban Micro environments.
- Objective: Maximize \( P_{NES} = \sum R_k - λ \sum P_m \), balancing throughput and energy.
- 10,000 training samples → 450,000 sequences; 1,000 testing samples.

### Performance Results
| Method | Data Rate | Power | P_NES |
|---------|-----------|--------|--------|
| All-on | 217.0 | 45.6 | 171.4 |
| Proposed (LSTM) | 216.0 (↓0.46%) | 41.8 (↓8.33%) | **174.2 (+2.8%)** |
| Optimal | 216.2 | 41.7 | 174.5 |

The proposed model reduces power consumption by 8.3% with only 0.46% throughput loss.

## Section V – Visualization
Front-End Stack: React + Kakao Maps + Recharts + InfluxDB

### Panels
1. Cell & UE Location — displays BS/UE position.
2. Network Energy Saving — shows real-time energy saving graphs.
3. Cell Metrics — throughput, connected UEs per BS.
4. UE Metrics — UE throughput and received power.
5. System Control — manual BS ON/OFF and energy mode.

## Section VI – Conclusion
The proposed framework offers a lightweight, flexible, and visual environment for developing and testing Open RAN rApps. It integrates simulation, AI-driven control, and intuitive GUI, and can serve as a baseline for diverse O-RAN use cases. Future work targets integration with real testbeds (Colosseum, X5G) and exploration of issues like imperfect CSI, latency, scalability, and generalization.

## Key Insights
- Provides the first practical Non-RT RIC development framework for AI/ML rApps.
- Uses simulated data to overcome lack of real Open RAN datasets.
- Demonstrates an LSTM-based AI model achieving near-optimal energy saving.
- Offers modular visualization GUI for debugging and performance analysis.
- Lays groundwork for future AI-enhanced, sustainable 6G RAN systems.
