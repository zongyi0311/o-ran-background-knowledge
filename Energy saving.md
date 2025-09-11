# O-RAN Alliance Technical Report on Network Energy Savings Use Cases (NESUC-R003 v02.00, 2023)
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
















