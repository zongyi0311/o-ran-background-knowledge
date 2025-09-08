<img width="749" height="373" alt="image" src="https://github.com/user-attachments/assets/9105d6f5-5471-4f21-a39d-63b952b94dbf" />
- Vendor Config Files : Each vendor has its own config files, with different formats and parameter naming, increasing integration complexity.
- sh Script / UI Control: : Configuration and management can be done via scripts (sh script) or UI control.

<img width="409" height="767" alt="image" src="https://github.com/user-attachments/assets/d22e886f-9751-4ada-8659-937cf9a57b62" />
- Communication between CU and CN is via the NG interface, using the NGAP protocol.
- One of the key pieces of information in the NG Setup procedure is the PLMN ID
- PLMN ID = MCC + MNC
  - MCC: Mobile Country Code → identifies the country.
  - MNC: Mobile Network Code → identifies the mobile network operator within the country.
 
- <img width="677" height="399" alt="image" src="https://github.com/user-attachments/assets/9dcf8177-0681-4ce9-a338-8dd23ed857da" />
- Communication between DU and RU is via the **Fronthaul Interface**, divided into C/U/S/M planes, using the **eCPRI protocol**.
- S-Plane (Synchronization Plane): Provides time and frequency synchronization to maintain signal accuracy and system performance. Typically uses IEEE 1588 PTP and SyncE.
- C-Plane (Control Plane): Delivers control messages, instructing the RU when and where to transmit/receive user data.
- U-Plane (User Plane): Carries user data and transports PRACH signals for UE initial and random access.
- M-Plane (Management Plane): Handles configuration, fault management, performance monitoring, and software management. Uses NETCONF/YANG over SSH, enabling DU/SMO to remotely manage the RU.

- <img width="673" height="955" alt="image" src="https://github.com/user-attachments/assets/8b89f44b-8a5d-418a-8a7c-2f5c309ae37b" />
- Need for Timing Window
- Fronthaul split imposes strict bandwidth and latency requirements, requiring timing window parameters.
- RU timing parameters are fixed by hardware, differing across vendors → DU must adjust accordingly.
- O-DU side
  - R1 = Receive interface
  - R4 = Transmit interface
- O-RU side
  - R2 = Receive interface
  - R3 = Transmit interface
  - RA = Antenna interface

- Delay Parameters
  - T12 (Uplink Delay): O-RU → O-DU
  - T34 (Downlink Delay): O-DU → O-RU
- T12 and T34 are not fixed → affected by switching latency and physical distance, defined with min/max bounds.

- DL Packet Types
- Downlink involves DL C-Plane, UL C-Plane, and DL U-Plane packets, all subject to timing constraints.

- Window Definition
  - O-DU transmission window: T1amin ~ T1amax (configurable).
  - O-RU reception window: T2amin ~ T2amax (fixed by hardware, not configurable).
- Reference Time
  - t = 0 → The moment RU transmits the wireless signal (fixed reference).
- Packet Classification
  - Early: Arrives too early
  - Late: Arrives too late
  - On-time: Within the expected window
  - RU counters:
    - C-Plane → RX_EARLY_C
    - U-Plane → RX_EARLY

- 1.3 End-to-end testing flow
- <img width="637" height="961" alt="image" src="https://github.com/user-attachments/assets/b0ae276f-dd03-491b-8372-407ff1843f3a" />

- Before bringing up the E2E system, the CU, DU, and RU must be configured to ensure correct startup settings.

<img width="846" height="465" alt="image" src="https://github.com/user-attachments/assets/6f44edf4-ae7d-4c73-8338-d9e2f570667b" />
- The O-DU T1a window must align with the O-RU T2a window, otherwise packets will be classified as Early / Late.
- The O-DU Ta4 window must cover the O-RU Ta3 transmit range, otherwise reception failure occurs.
- Packets are classified as Early / On-time / Late based on arrival time.

- The RU must be restarted for new configs to take effect. After configuration, PTP (S-plane) synchronization must be verified.
- System Bring-up Sequence
  - Startup sequence → 5GC → CU → DU.
- NG Setup Check
  - If failed → Correct CU configuration, restart CU/DU.
  - If successful → Check if RU receives DU packets and whether packets are Early/Late.
 
- Timing Adjustment
  - If Early/Late occurs → Adjust DU timing window parameters and restart CU/DU.

- 1.4 DU/RU configuration parameters
- Configuration Consistency: DU and RU must be configured with consistent frequency and fronthaul parameters; otherwise, link failure or IQ decoding errors may occur.
- Fronthaul Parameters: DU/RU fronthaul configuration must support CUS-plane communication with precisely matched parameters.
- Ethernet Settings: 
  - MAC addresses and VLAN IDs: Ensure correct packet routing between nodes.
  - MTU (Maximum Transmission Unit): Too small → packet segmentation → added overhead and latency.


- 1.4.2 Frequency-related Parameters As shown in Figure 1.13, both DU and RU require
- DU and RU must be configured with consistent frequency-domain parameters to ensure synchronization and proper transmission.
- carrierBandwidth: Defines the overall bandwidth of the channel.
- A (absoluteFrequencyPointA): Absolute frequency at lower edge of carrier, reference for subcarrier indexing.
- centerOfChannelBandwidth: Center frequency of carrier bandwidth, aligns RU RF tuning.
- absoluteFrequencySSB: Defines the location of SSB, must be within allocated bandwidth.
- subCarrierSpacing: Determines spacing between subcarriers, impacts number of available subcarriers.
- initialBWP: Configured on DU side for RA, includes startRB and numberRB.

- 1.5 OAI split 7.2 gNB
- Split 7.2 Architecture
  - One of the functional split options between DU and RU in O-RAN.
  - This study adopts the OAI gNB implementation of Split 7.2.

- Fronthaul Library Integration
  - Requires integration of OSC Fronthaul Library from the O-RAN Software Community.
  - During OAI gNB build, the library must be compiled with OAI → enables proper DU–RU fronthaul communication.

# Chapter 2 System Architecture
- The E2E system consists of CU/DU, Core Network, commercial O-RU, and 5G UE.
- Automation & Visualization
  - A custom rApp is deployed on the SMO.
  - Test cases are fed via RESTful API.
  - rApp → interprets test cases → triggers DU/RU config → initiates E2E testing.
  - Test results (throughput) → published to Grafana dashboards.
 
- Tested free5GC and Open5GS; architecture supports any core network with NG signaling.
- Compatible with all config-file-driven setups. Primarily tested with OAI gNB.
- Requires M-plane (NETCONF/YANG) for remote config/monitoring. Used JURA RU in this study.
- Must support ADB (Android Debug Bridge) → automated access + performance testing. Used MTK UE.
- <img width="641" height="802" alt="image" src="https://github.com/user-attachments/assets/484bba09-d30d-4112-8d40-c0ee678da070" />
- Input: Test cases → REST API → Integration rApp.
- Process: rApp → Configuration service → configure DU/RU → initiate test.
- Output: Throughput results → Grafana Dashboard visualization.

# Chapter 3 Proposed Method
## 3-1 System Input Parameters for rApp Execution
- Input Parameter Categories
  - gNB Configuration
	- Test Configuration
	- Host Information

- Input Format & Workflow
  - Input is provided in JSON format.
	- rApp receives and parses JSON → verifies required fields.
	- Automation proceeds sequentially: DU/RU config → gNB/CN startup → performance & synchronization verification.

- <img width="2036" height="1327" alt="image" src="https://github.com/user-attachments/assets/93cec1b9-913f-4f0f-8ef0-ae894c785e48" />

## 3.2 Automatic Configuration mapping of gNB and RU
### 3.2.1 Repeatable parameters in DU/RU
