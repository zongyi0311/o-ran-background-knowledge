<img width="486" height="649" alt="image" src="https://github.com/user-attachments/assets/78285513-be1d-4d42-89aa-d42c4e98ce2a" /># O-RAN Alliance Technical Report on Network Energy Savings Use Cases (NESUC-R003 v02.00, 2023)
## 5 Carrier and Cell Switch Off/On
### 5.1 Problem Statement, Solution and Value Proposition
- 1.Problem Context
  - Mobile networks often deploy **multiple frequency layers(carriers)** to cover the same area.
  - At low traffic load, energy can be saved by switching off carriers or cells without hurtign user experience.
  - **UEs are offloaded** to other acive cells/carriers before switch-off.
 
- 2.Challengers
  - Decision complexity:balancing system performance vs. energy saving.
  - Traffic load is **dynamic**, and offloaded carriers/cells must handle th extra traffic.
  - In some cases, swithching off cells saves locally but might increase overall network energy comsumption.
 
- 3.Proposed Solution
  - Use **RIC** to make network-wide optimized decisions, not just ;local.
  - Two deployment options for decision-making:
    - 1.Non-RT RIC(slower, centralized, policy-driven).
    - 2.Near-RT RIC(faster, real-time decisions).
   
- 4.AI/ML Role
  - AI/ML models can assist with prediction and optimization, including:
    - Future traffic demand.
    - User mobility patterns.
    - Resource usage forecasts.
    - Expected energy efficiency improvements under different switch-off states.
   
- 5. Operational Considerations
- Before switching off/on carriers or cells, E2 Nodes must perform preparation steps:
- For switching off:
  - Check ongoing emergency calls/warning messages.
  - Update or modify Carrier Aggregation/Dual Connectivity.
  - Trigger handover (HO) to move UEs to other cells/carriers.
  - Inform neighbor nodes via X2/Xn interface.
- For switching on:
  - Perform cell probing.
  - Inform neighbor nodes via X2/Xn interface.

### 5.2 Architecture/Deployment Options
- Option 1: Non-RT RIC Deployment
- Table 5.2.1.1-1: Carrier and Cell Switch Off/On: AI/ML inference via Non-RT RIC
- Goal
  - Enable carrier/cell switch-off for energy savign.
  - Non-RT RIC controls it(with optional AI/ML solutions).
  - Actions are ennforced on E2 Nodes and O-RUs.
 
- Actors and Roles
  - Non-RT RIC-> interence host (makes energy-saving decisions).
  - E2 Node & O-RU -> enforce configuration changes.
 
- Begins When
  - Operator enables optimization functions.
  - E2 Node & O-RU are operational.
 
- Main Steps
  - SMO requests measurement data from E2 Node & O-RU.
  - E2 Node & O-RU send data (periodic/event-based) to SMO.
  - Non-RT RIC retrieves data for processing.
  - Non-RT RIC trains AI/ML models, deploys them, and continuously monitors:
  - E2 Node performance & energy.
  - O-RU energy consumption.
  - Based on AI/ML inference, Non-RT RIC may ask SMO to configure E2 Node for switch off/on.
  - SMO instructs E2 Node, which updates O-RU configuration. Notification is sent back after completion.
  - Non-RT RIC analyzes performance; if savings not achieved → fallback, retraining, or model update.

- End When
  - E2 Node becomes non-operational, or
  - operator disable the optimization function.
 
- O1 = SMO ↔ E2 Node.
- FH (Fronthaul M-Plane) = E2 Node ↔ O-RU.
- Non-RT RIC
  - Data collector & Analyzer
    - Gets configurations, KPIs, measurement reports.
  - AI/ML Role
    - Can train or retrain models.
    - Can deploy or updata AI/ML model inside itself.
  - Decision Making
    - Uses data(and AI/ML if applied) to decide which carriers/cells should be switched off/on.
  - control signaling
    - Sends updated optimization configureations to E2 Nodes(O-CU) via R1/O1 interface.
   
- E2 Node (O-DU / O-CU)
  - Reporter
    - Sends cell configuration, KPIs, traffic/energy reports to SMO via O1 interface
  - Action Executor
    - Executes energy-saving actions:
      - Before switch-off:
        - Check for emergency calls/messages.
        - adjust carrier Aggregation/Dual Connectivity.
        - Trigger handover for UEs.
        - Inform neighbors(via X2/Xn interface).
      - Before switch-on:
        - Perform call probing.
        - Inform neighbors(via X2/Xn).
    - Makes the final decision on switch-off/on and notifies SMO of the action.
   
- O-RU(Radio Unit)
  - Reporter
    - Provides energy consumption (EC) and efficiency (EE) data via Open FH M-Plane (to O-DU or SMO).

#### 5.2.1.3 Input/Output Data Requirements
- Inputs:The system collects traffic load, radio quality, mobility info, and energy/power reports from E2 Nodes and O-RUs.
- Output:The SMO sends switch-off/on commands to E2 Nodes for specific cells/carriers.

- Table 5.2.1.3-1: Initialization
- Before the system can decide which carriers/cells to switch off/on, it needs:
  - Targets/metrics for energy efficiency (bit/J).
  - Detailed characteristics of carriers and cells (location, frequency, power, mapping).

- Table 5.2.1.3-2: AI/ML Model Training
- To decide whether to switch off/on carriers or cells, the system needs traffic data + radio quality metrics + energy/power data:
  - Traffic volume: DL/UL PDCP data.
  - Radio quality: RSRP, RSRQ, SINR (per cell/beam).
  - Energy usage: kWh consumption, power per hardware, transmit power.
 
- Table 5.2.1.3-3: Input Decision Making / AI/ML Inference
- For AI/ML-based decision making, the system continuously gathers:
  - Traffic load data (UL/DL PDCP volume).
  - Radio quality metrics (RSRQ, RSRP, SINR).
  - Energy/power consumption data (total energy use, hardware power, transmit power).

- Option 2: Near-RT RIC Deployment

| **Stage**                      | **Non-RT RIC (Table 5.2.1.1-1)**                                                               | **Near-RT RIC (Table 5.2.2.1-1)**                                                                                                                                    |
| ------------------------------ | ---------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Goal**                       | Carrier/Cell switch off/on controlled by **Non-RT RIC** with AI/ML inference.                  | Carrier/Cell switch off/on controlled by **Near-RT RIC** with AI/ML inference.                                                                                       |
| **Actors & Roles**             | - Non-RT RIC = inference host (decision making). <br>- E2 Node + O-RU = execute configuration. | - Non-RT RIC = AI/ML **model manager**. <br>- Near-RT RIC = inference host (decision making). <br>- E2 Node + O-RU = execute configuration.                          |
| **Assumptions**                | - **O1**: SMO ↔ E2 Node/O-RU. <br>- **FH M-Plane**: E2 Node ↔ O-RU.                            | - **O1**: SMO ↔ E2 Node, Near-RT RIC, Non-RT RIC. <br>- **E2**: E2 Node ↔ Near-RT RIC. <br>- **A1**: Non-RT RIC ↔ Near-RT RIC. <br>- **FH M-Plane**: E2 Node ↔ O-RU. |
| **Pre-conditions**             | Operator sets energy-saving targets in **Non-RT RIC**.                                         | Same: Operator sets targets in **Non-RT RIC**.                                                                                                                       |
| **Begins When**                | Operator enables optimization functions; E2 Node + O-RU operational.                           | Same.                                                                                                                                                                |
| **Step 1–3 (Data Collection)** | SMO → E2 Node/O-RU for measurements → Data flows to Non-RT RIC.                                | Same, but training data ends up at **Non-RT RIC**, then models pushed to Near-RT RIC.                                                                                |
| **Step 4 (Training)**          | **Non-RT RIC trains & runs inference** using AI/ML.                                            | **Non-RT RIC trains AI/ML models**, but inference runs in **Near-RT RIC**.                                                                                           |
| **Step 5 (Policies/Trigger)**  | Non-RT RIC may request SMO to configure E2 Node.                                               | SMO may send policies to Near-RT RIC (via O1 or A1).                                                                                                                 |
| **Step 6 (Execution)**         | - Non-RT RIC → SMO → E2 Node. <br>- E2 Node → O-RU.                                            | - **Near-RT RIC directly controls E2 Node** via E2. <br>- E2 Node → O-RU.                                                                                            |
| **Step 7 (Monitoring)**        | Non-RT RIC monitors model performance, retrains if needed.                                     | Near-RT RIC monitors energy/performance and decides new actions (loop).                                                                                              |
| **Ends When**                  | E2 Node non-operational, or operator disables optimization.                                    | Same.                                                                                                                                                                |
| **Post Conditions**            | Non-RT RIC continues closed-loop monitoring at E2 Node & O-RU.                                 | Near-RT RIC continues closed-loop monitoring at E2 Node & O-RU.                                                                                                      |

- Key Differences
- 1. Decision Location
  - Non-RT RIC case: Decisions & inference centralized in Non-RT RIC.
  - Near-RT RIC case: AI/ML inference is closer to the RAN (Near-RT RIC), improving reaction time.
- 2. Role of Non-RT RIC
  - Non-RT RIC = main controller in first case.
  - Non-RT RIC = AI/ML trainer + policy provider in second case.
- 3. Interfaces
  - Non-RT RIC case: Uses O1 + Fronthaul.
  - Near-RT RIC case: Adds E2 + A1 interfaces, making it richer and more distributed.
 
- Figure 5.2.1.1-1 (Non-RT RIC inference)
- Figure 5.2.2.1-1 (Near-RT RIC inference)

| **Aspect**                 | **Non-RT RIC (Figure 5.2.1.1-1)**                                                                        | **Near-RT RIC (Figure 5.2.2.1-1)**                                                                                                               |
| -------------------------- | -------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| **AI/ML Workflow**         | Training **and inference** both happen in the **Non-RT RIC**.                                            | **Training** happens in Non-RT RIC, but **inference** runs in the **Near-RT RIC**.                                                               |
| **Interfaces Used**        | - **O1**: SMO ↔ E2 Node <br>- **FH (M-Plane)**: E2 Node ↔ O-RU                                           | - **O1**: SMO ↔ E2 Node / RICs <br>- **E2**: Near-RT RIC ↔ E2 Node <br>- **A1**: Non-RT RIC ↔ Near-RT RIC <br>- **FH (M-Plane)**: E2 Node ↔ O-RU |
| **SMO Role**               | Collects measurement data, forwards to Non-RT RIC, then relays RIC’s configuration decisions to E2 Node. | Collects measurement data, forwards to Non-RT RIC for training, may also provide optimization triggers/policies to Near-RT RIC.                  |
| **Decision Making**        | **Non-RT RIC decides** which carriers/cells to switch off/on and tells SMO → E2 Node → O-RU.             | **Near-RT RIC decides** based on AI/ML inference + policies from Non-RT RIC, then directly controls E2 Node via E2 interface.                    |
| **Policy Handling**        | Not explicit (Non-RT RIC makes the decision).                                                            | Explicit policy exchange: SMO or Non-RT RIC can provide **optimization triggers/policies** to Near-RT RIC.                                       |
| **Execution**              | - SMO sends configuration changes to E2 Node. <br>- E2 Node applies changes to O-RU.                     | - Near-RT RIC instructs E2 Node directly. <br>- E2 Node applies changes to O-RU.                                                                 |
| **Closed-Loop Monitoring** | Non-RT RIC monitors outcomes, retrains AI/ML if needed.                                                  | Near-RT RIC monitors outcomes in near-real-time, while Non-RT RIC may update/retrain models and redeploy them.                                   |

- Non-RT RIC Case (Centralized)
  - Simpler, but slower.
  - Good for long-term optimization and training-heavy AI/ML.
- Near-RT RIC Case (Distributed)
  - Adds A1 and E2 interfaces to allow faster, policy-guided decisions closer to the RAN.
  - Better for real-time or near-real-time energy optimization.
- Role Separation
  - Non-RT RIC = long-term training + policy manager.
  - Near-RT RIC = short-term inference + fast decision-making.

- **E2 Node**
- Definition
  - In O-RAN, an E2 Node is basically any RAN node(like gNB functions) that connects to the Near-RT RIC via the E2 interface.
  - It is the entity that executes RIC decisions(e.g., swithcing carriers/cells on or off, reconfiguring resources).
 
-  Examples of E2 Nodes
  - Depending on how a gNB is split, the E2 Node can be:
  - O-CU (Central Unit) – CU-CP (control plane) or CU-UP (user plane).
  - O-DU (Distributed Unit) – handles MAC and lower PHY.
  - Sometimes a combined CU/DU acts as a single E2 Node.

- Role in Energy Saving Use Case
- From the tables and diagrams you shared:
  - Reporting:
    - E2 Node sends cell configurations, load statistics, mobility info, and energy reports to SMO and/or Near-RT RIC.
  - Execution:
    - Prepares for switch-off/on (e.g., checks emergency calls, handles carrier aggregation, triggers handovers).
    - Updates O-RU configurations (activate/deactivate carriers)
  - Decision Support:
    - In some cases, it makes the final decision on switching actions (with guidance from RIC).

- Input/Output Data Requirements
  - Input Data
    - From E2 Node → SMO / Non-RT RIC (training) & Near-RT RIC (decision making/inference):
      - Carrier / cell characteristics
      - EE/EC (Energy Efficiency / Energy Consumption) measurement reports
      - Load statistics per cell and per carrier
      - UE mobility information (e.g., RSRP, RSRQ, SINR)
      - Energy consumption
    - From O-RU → E2 Node:
    - Power consumption metrics:
      - Mean total/per carrier power consumption
      - Mean total/per carrier transmit power
     
  - Output Data
    - From Near-RT RIC → E2 Node:
    - Carrier(s)/cell(s) recommended to be switched off/on

- <img width="680" height="824" alt="image" src="https://github.com/user-attachments/assets/091388be-8d54-4e57-9d7f-2222ae2e8ad7" />
- Simplified Summary
- WG1: Defines the use case.
- WG2: Updates Non-RT RIC (R1/A1) specs.
- WG3: Updates Near-RT RIC (E2, KPIs) specs.
- WG4: Updates fronthaul M-Plane/CUS-Plane for energy KPIs.
- WG5: Updates O1 management models (SMO↔O-DU/CU).
- WG10: Updates information/data models to support the new functionality.

#### 5.4.2 Ongoing Work in 3GPP Rel-18 RAN
- Current Study Focus
- Base station energy consumption model – define how energy use is modeled.
- Evaluation methodology & KPIs – standardize how to measure energy efficiency.
- Techniques on gNB and UE side – study methods to improve network energy savings for both transmission and reception.

#### 5.4.3 Impact on 3GPP RAN Specifications
- Non-RT or Near-RT RIC controlled Cell Switch Off/On is not expected to impact 3GPP RAN specifications.

- 3GPP: Provides baseline energy efficiency models, KPIs, and OAM procedures (TR/TS series).
- O-RAN: Uses these as foundation, but its RIC-driven Cell Switch Off/On is outside 3GPP scope (no direct impact).

### 5.5 Gain Analysis
- By switching off carriers or cells in low traffic, operators can achieve large power savings.
- Saving depend on:
  - Network deployment (multi-frequency layers, overlapping coverage).
  - Cell configuration (bandwidth, control channels, etc.).
  - Hardware design (e.g., whether power amplifiers can be fully shut down).
  - Traffic load.
 
- O-RU Power Consumption Categories
- O-RU (Radio Unit) power use can be divided into:
  - User channels’ load (uplink/downlink traffic).
  - Common signals’ load (SSB, reference signals, broadcast).
  - Regular PA (Power Amplifier) operational power (energy to keep PA running).
  - Regular operational power (energy to power the radio module).

- <img width="522" height="270" alt="image" src="https://github.com/user-attachments/assets/f0b424c0-2345-4a06-b9b5-5b28ca9546d6" />
- <img width="983" height="204" alt="image" src="https://github.com/user-attachments/assets/d7a599c1-e75a-4cbe-9812-93d4a374d1de" />

- <img width="642" height="131" alt="image" src="https://github.com/user-attachments/assets/5a3a299c-bd8a-473a-b07a-891ea31ee80e" />
- Simplified Calculation(Table 5.5-1)
- Two scenarios:
  - Case 1: 100 W saved, 2 hrs/day, 10,000 O-RUs → 730 MWh/year.
  - Case 2: 200 W saved, 4 hrs/day, 20,000 O-RUs → 5,840 MWh/year.
- Shows that small per-unit savings add up massively when scaled to thousands of O-RUs.

- <img width="407" height="349" alt="image" src="https://github.com/user-attachments/assets/b633129c-2d8f-4a6f-b059-9916fa3f91da" />
- Hardware Impact (Figure 5.5-2)
  - Major savings when RF Processing Unit and Digital Processing Unit can enter lower power states.
  - Power Unit and FH Processing Unit usually stay active, so savings depend on which blocks can be shut down.

- <img width="654" height="197" alt="image" src="https://github.com/user-attachments/assets/7e538ed1-2360-4e38-b721-76901cb070d3" />
- Main savings come from:
  - RF Processing Unit (130–145 W reduction).
  - Digital Processing Unit (20–25 W reduction).

- <img width="465" height="150" alt="image" src="https://github.com/user-attachments/assets/18ed93bc-ebea-4c14-8c26-d10a35e585a2" />
- Even just 3 hours/day of switch-off at 10,000 O-RUs = equivalent to the yearly electricity use of a small town.

## 6.RF Channel Reconfigureation
### Problem Statementm, Solution and Value Proposition
- **Problem Statement**
  - Massive MIMO (mMOMO) antennas are used for beamforming to imporve capacity and throughput.
  - O-RUs concentrate power amplifiers near the radome by combining radiating elements.
  - At low load(traffic or user count below a threshold), keeping all Tx/Rx arrays active wastes energy. 
- **Solution**
  - Achieve energy saviang(ES) by switching off part of the Tx/Rx arrays during low load.
    - Example:turn off 32 out of 64 arrays in a digital mMIMO O-RU.
  - This may require reducing:the number of spatial layers, the number of SSB beams, the O-RU transmit power.
 
- Decision Making  
  - Depends on management model:hybrid or hierachical.
  - Two options for AI/ML decisions: AI/ML inference at Non-RT RIC or Near-RT RIC
  - AI/ML models may predict:Future traffic, User mobility, Resource usage and Expected gains in energy efficiency and network performance under diffenent configurations

- **Value Proposition**
  - Main objective: Tx/Rx array selection and reconfiguration in the O-RU.
  - Possible adjustments:Maximum spatial streams, Number of SSB beams and Antenna transmit power.
  - Value:Reduce power consumption during low load

- Table 6.1-1: Actions in the context of RF channel Recongiguration
  - 1. O-RU Tx/Rx Array Selection: Switch off certain Tx/Rx arrays to reduce O-RU power consumption. Re-selecting arrays may affect cell coverage.
  - How it's done:
    - O-RU reports supported Tx/Rx array selections to O-DU or SMO via Open FH M-Plane.
    - Based on traffic load & user distribution, Non-RT / Near-RT RIC decides the optimal Tx/Rx array configuration.
  - 2. Modification of Number of SU/MU MIMO Spatial Streams of Data Layers: Maximum spatial streams depend on active Tx/Rx arrays. May need reduction when arrays are switched off, ex: 16 streams for 64T64R, 8 streams for 32T32R.
  - How it’s done:
    - O-RU reports max supported spatial streams.
    - Based on traffic & user distribution, O-DU/Non-RT/Near-RT RIC instructs the O-RU to reduce streams, turning off unneeded processing units.
   
  - 3. Modification of Number of SSB Beams: Number of SSB beams depends on TX/RX array selection and spatial streams.
  - How it's done:
    - O-DU controls SSB beams.
    - Non-RT / Near-RT RIC instructs O-DU to set the number of beams based on Tx/Rx array configuration.
  - 4. Modification of O-RU Antenna Transmit Power: Max transmit power depends on active TX/RX arrays. If arrays are reduced, transmit power may need to be adjusted to keep coverage.
  - How it's done:
    - O-RU power is per Tx array.
    - No change needed if selection is proportional.
    - If coverage drops, O-RU Tx power can be boosteed to compensate.

- **SSB**
  - The SSB(Synchronization signal Block) is the signal structure broadcast by the gNB to allow UEs to: detect and synchronize with the cell, identify the cell and measure beam quallity for beam management.
  - Often uses beamforming, especially in mmWave and massive MIMO deployments.
  - Each SSB carries:
    - PSS (Primary Synchronization Signal)
    - SSS (Secondary Synchronization Signal)
    - PBCH (Physical Broadcast Channel)

### 6.2.1 Option 1: Non-RT RIC Deployment
- Goal: Enable RF Channel Reconfiguration for Energy Saving(ES) by letting the Non-RT RIC handle-making with AI/ML models.
- Non-RT RIC -> inference host for energy saving decisions
- E2 Node(O-DU/CU) & O-RU -> execute the actual configuration changes.

- Flow
- Measurement Collection
  - Non-RT RIC requests measurement data(via O1 or Open FH M-plane) from E2 Node or O-RU.
  - E2 Node & O-RU send traffic load, energy consumption, and other KPIs back through SMO.
  - Non-RT RIC retrieves and aggregates these measurements.
 
- AI/ML Training & Inference
  - Train or Updata AI/ML models with the collected data.
  - Then continuously monitors KPIs like load, EE/EC reports, and geolocation. 

- Decision & Optimization
  - Based on AI/ML inference, Non-RT RIC decides on RF channel Reconfiguration actions.
  - Non-RT RIC requests SMO → E2 Node (O-DU) → O-RU to apply changes.
 
- Reconfiguration Execution
  - Possible actions applied at the O-RU:
  - Switch off certain Tx/Rx arrays.
  - Reduce number of MIMO layers.
  - Adjust number of SSB beams.
  - Modify Tx power per antenna.

- Ends at the moment Operator disables Es function, or E2 Node becomes non-operational.

- Summary: In Option 1, the Non-RT RIC is the “brain” — it both trains AI/ML models and decides optimizations, while the E2 Node & O-RU just execute reconfiguration actions.

- Input Data
- From SMO/E2 Node: Load statistics per cell/carrier:Active users, RRC connections, Scheduled users per TTI, PRB utilization, DL/UL throughput, PMI/CSI reports
- From O-RU: Power consumption metrics: Mean total/per carrier power consumption,
Mean total/per carrier transmit power.

Output Data
- From SMO to E2 Node(reconfiguration instructions)
  - O-RU Tx/Rx Array selection
  - Modify SU/MU MIMO spatial streams/data layers
  - Modify number of SSB beams
  - Modify O-RU antenna transmit power

### 6.2.2 Option 2: Near-RT RIC Deployment
- Goal
  - Enable RF Channel Reconfiguration for Energy Saving (EE/ES) using configuration changes.
  - Decisions made by Near-RT RIC (AI/ML inference).
  - AI/ML model training may still happen in Non-RT RIC.

- Roles:
  - Near-RT RIC: inference host for energy saving decisions.
  - Non-RT RIC / SMO: policy maker, may trigger or guide Near-RT RIC optimization.
  -  








