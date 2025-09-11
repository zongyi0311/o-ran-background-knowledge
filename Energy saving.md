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













