- <img width="749" height="373" alt="image" src="https://github.com/user-attachments/assets/9105d6f5-5471-4f21-a39d-63b952b94dbf" />
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
- Parameters like l2_mtu, iq-bitwidth, compression-type, compression-method are defined once in JSON and auto-applied to both DU & RU.

### 3.2.2 MAC & VLAN Automation
- Manual process: retrieve RU MAC → edit config → create VF → bind to DPDK → update config.
- Automated rApp: detects/creates VF, sets MAC/VLAN, binds to DPDK, auto-fills gNB config.

- **JSON**
- JavaScript Object Notation:
	- A lightweight data exchange format, commonly used in configuration files, APIs, and data transmission.
 	- Features: Human-readable and easy for programs to parse.
    - Structure: composed of key-value pairs, supporting nested structures (like dictionaries/objects + arrays).
 
- **SFTP**
- A secure file transfer method based on SSH (Secure Shell)
- Unlike traditional FTP (File Transfer Protocol), SFTP encrypts the entire connection (account, password, and data content), so it is more secure.

- <img width="658" height="577" alt="image" src="https://github.com/user-attachments/assets/ed607bcd-b7ef-45ab-a936-63744c5c33f0" />
- NIC (Network Interface Card): Network card, physical network interface.
- PF (Physical Function): Physical function of the NIC, representing actual hardware resources.
- VF (Virtual Function): Virtual function carved out of the PF, which can be assigned a MAC address and VLAN.
- Bus Info: Identification information of the VF on the PCI bus, which is later provided to the DPDK for binding.
- DPDK (Data Plane Development Kit): High-performance packet processing framework required by gNBs to accelerate fronthaul traffic.
- OAI DU config: The OAI gNB configuration file, which contains the VF's MAC address, VLAN, bus information, and other information.

### 3.2.3 MIMO Adjustment
- To support flexible DL transmission configurations, rApp automates MIMO setting adjustments based on the test case.
- Reads antenna config and desired MIMO layers from JSON.
- Generates eAxC IDs, checks RU antenna capability, and limits layers if needed.
- Maps eAxC IDs to RU antennas for consistent repeatable MIMO testing.

- **eAxC ID**
- eAxC = eCPRI AxC Container, In the eCPRI (Common Public Radio Interface over Ethernet) protocol, AxC (Antenna Carrier) stands for "a combination of an antenna and a carrier."
- In O-RAN fronthaul (RU ↔ DU) transmission, each antenna carrier is assigned an ID, which is the eAxC ID.
- ex "maxMIMO_layers": 4 ,rApp will generate eAxC ID = 0,1,2,3
- Mapping relationship:

eAxC ID 0 → RU Antenna Port 0

eAxC ID 1 → RU Antenna Port 1

eAxC ID 2 → RU Antenna Port 2

eAxC ID 3 → RU Antenna Port 3
- <img width="532" height="792" alt="image" src="https://github.com/user-attachments/assets/b2547bba-8d0b-4179-a486-0ecfb50b9fe7" />

### 3.2.4 Frequency Automation
- **SubCarrier Spacing (SCS) Conversion**
	- Input subCarrierSpacing (kHz) → converted to:
		- Numerology μ (0, 1, 2, 3)
		- SCS string (scs) for DU/RU config
 
- **Bandwidth Conversion**
- Input bSChannelBw (MHz) → converted to Resource Blocks (RBs) using 3GPP TS 38.101-1 Table 5.3.2-1.

- **SSB Frequency Selection**
- input:arfcn (Absolute Radio Frequency Channel Number)
- <img width="711" height="387" alt="image" src="https://github.com/user-attachments/assets/2ffa8747-04aa-470c-be1e-c6c71592b180" />
- **RU Center Frequency Calculation**
- Input: number of PRBs (number-of-prb) instead of ARFCN.
- <img width="759" height="195" alt="image" src="https://github.com/user-attachments/assets/18652f1c-5be7-4074-a20a-24500796b041" />
- **BWP (Bandwidth Part) Location and Size**
- Encode the initial BWP (location + size) in a compact format using RIV encoding.
- <img width="782" height="342" alt="image" src="https://github.com/user-attachments/assets/f24014d5-992c-4925-8393-02cfde1d3a77" />

- <img width="712" height="474" alt="image" src="https://github.com/user-attachments/assets/27d33ec8-f795-4b67-9aca-2558fe4cbb74" />

## 3.3 Start End-to-end component
- <img width="736" height="544" alt="image" src="https://github.com/user-attachments/assets/dec1a2b9-fc65-4880-aed9-235db7314d58" />
- <img width="667" height="468" alt="image" src="https://github.com/user-attachments/assets/3f9819b5-6a68-44f3-9010-673a7d970c75" />
- GPS Status Check

	rApp queries Fronthaul Gateway (FHG) for GPS status.
	
	FHG receives GPS signal → reports back → rApp verifies GPS availability.

- DU/RU PTP Sync Check

	rApp queries PTP sync state from gNB (DU).
	
	rApp queries RU via M-plane for PTP status.
	
	Results stored in Test Execution Interface (TEV).

- Periodic Monitoring

	Every 30 seconds, rApp repeats the PTP status check.
	
	Key metrics: offset, delay, frequency (small/consistent values indicate sync).

- Sync Confirmation

	Once DU and RU both report stable sync (within ±1 μs), rApp marks gNB/RU as “got PTP sync”.


<img width="667" height="468" alt="image" src="https://github.com/user-attachments/assets/16a25df1-2ff4-4486-a539-ebf1daa5dc16" />
- Start CN

rApp launches the Core Network (e.g., Free5GC/Open5GS).

- Start gNB

	rApp launches the OAI gNB (DU/CU).

- Capture NG Setup PCAP

	rApp uses tcpdump to capture NG interface packets.
	
	Stores them as PCAP files.

- NGAP Message Exchange

	gNB → CN: NGSetupRequest
	
	CN → gNB: NGSetupResponse
	
	rApp analyzes PCAP with pyshark to verify proper NGAP exchange.

- NG Setup Success

	Presence of NGSetupResponse confirms NG setup.
	
	rApp reports NG setup success.


## 3.4Checking Fronthaul Packet Status and Tim-ing Window Adjustment
- Ensure reliable and timely packet transmission between gNB and RU over the Open Fronthaul(FH) interface.
- rApp process:

	Collect RU’s packet reception statistics via configuration service.
	
	Verify whether C/U-plane packets fall within the configured timing window.
	
		If inside → UE proceeds with attachment.
		
		If outside → Adjust timing window parameters and restart gNB.

- Timing window adjustment logic
	- Monitor counts of early packets (RX_EARLY) and late packets (RX_LATE).
	- If trends increase → update corresponding T1a parameters (T1a_cp, T1a_up) to re-center packet timing.

- Packet classification & visualization
	- Packets categorized as early / on-time / late.
	- Statistics guide whether to shift or widen the timing window.

- <img width="667" height="688" alt="image" src="https://github.com/user-attachments/assets/ed8456d1-b9bf-42ea-b34a-b45f297b9e6b" />
- UE attach is allowed only when the RU packet timing is correct.
- If a packet is sent too early or too late, the rApp automatically adjusts the time window and restarts the gNB.

## 3.5 UE Auto Attach and iperf
- **iperf3**
- iperf3 is an open-source network performance testing tool.
- Measures end-to-end throughput between two nodes.
- Supports TCP and UDP tests, evaluates:
	- Bandwidth
	- Latency
	- Jitter
	- Packet loss

- **ADB** Android Debug Bridge
- A command-line tool that lets your computer communicate with an Android device.
