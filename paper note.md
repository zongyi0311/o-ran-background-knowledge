
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
