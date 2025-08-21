# II. 3GPP’S FRAMEWORK FOR UL-TDOA BASED
POSITIONING
- Uplink Time-Difference-of-Arrival: the UE sends uplink signals (e.g., SRS); multiple gNBs time-stamp the arrivals; their time differences locate the UE.
- UE ➜ gNBs (uplink signal) → gNBs send measurements over NRPPa → LMF computes the position.
- **UL-TDOA**
- 3GPP defines a UL-TDOA (Uplink Time Difference of Arrival) framework for 5G that combines several network entities and procedures so multiple base stations can time-stamp the uplink signal from a UE and trilaterate its position. TS 38.305 specifies the requirements for all 5G location services, including UL-TDOA.

| Entity                                          | Role in UL-TDOA                                                                                                                                                                                    |
| ----------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **UE (User Equipment)**                         | The device to be located; transmits uplink signals that are time-stamped at multiple sites.                                                                                                        |
| **NG-RAN (gNB / TRPs)**                         | 5G access network. One or more gNBs with one or more **Transmission Reception Points (TRPs)** receive the UE’s uplink, create precise arrival-time measurements, and forward them for positioning. |
| **AMF (Access & Mobility Management Function)** | 5G Core control-plane anchor. Manages registration, access/mobility, and helps coordinate location procedures and messaging.                                                                       |
| **LMF (Location Management Function)**          | Central 5G Core positioning engine. Collects the TDOA measurements from the RAN, performs the hyperbolic position calculation, and produces the final UE location.                                 |
| **LCS entity (Location Services)**              | Manages location-service requests and delivery to **LCS clients** (apps on the UE or external systems). Orchestrates with network elements to obtain the UE’s position.                            |

- <img width="581" height="231" alt="image" src="https://github.com/user-attachments/assets/83d86465-059a-4f02-a669-8cccc5f9ec69" />
- NR-Uu – air interface between UE ↔ gNB (uplink positioning signals travel here).
- NG-C (NG-AP over N2) – control-plane link gNB ↔ AMF; transports positioning requests/results related to the gNB.
- Nls (NRPPa) – LMF ↔ AMF (and onward to gNB via AMF); carries positioning control and measurement messages.
- (Often used as needed) LPP – LMF ↔ UE user-plane protocol for assistance/config (e.g., SRS settings or timing aid).
- How a UL-TDoA session runs (high level)
  - Request – LCS client asks the LMF to locate a UE.
  - Control setup – LMF instructs the RAN (via Nls→AMF→NG-C→gNB) to perform UL-TDoA; optionally sends LPP assistance to the UE.
  - Sounding – UE transmits uplink positioning signals (e.g., SRS/PRACH/PUSCH) over NR-Uu.
  - Time-stamping – Multiple TRPs/gNBs capture precise receive times for the same UE burst.
  - Reporting – gNB sends those time stamps/quality metrics back to LMF via NG-C → Nls (NRPPa).
  - Location fix – LMF forms TDoA measurements, solves for the UE position, and returns results to the LCS client.
 
- Requires tight network time synchronization across TRPs.
- Accuracy hinges on timestamp precision, TRP geometry, bandwidth, and multipath mitigation.
- UE changes are minimal (it just transmits UL), which makes UL-TDoA attractive for network-centric positioning.

## A. Protocol Required for UL-TDoA Positioning Method in OAI
- Core Measurements: Each gNB/TRP measures UL-RToA of UE uplink signals; the LMF converts them to TDoA and estimates the UE position.
- Signal Used: SRS serves as the uplink sounding signal; its characteristics must remain consistent across periodic transmissions.
- Control Flow:
  - LMF instructs the serving gNB to command the UE to transmit SRS;
  - Serving gNB allocates resources and reports the SRS configuration to LMF;
  - LMF relays the configuration to neighboring gNBs/TRPs for cooperative measurements.
- Protocol: Procedure per 3GPP TS 38.305 §8.13; NRPPa PDUs carry positioning configs, measurement requests, and results.
- Message Path: No direct LMF–gNB link; all NRPPa messages traverse the AMF (control plane transport).

# III. CONTRIBUTION TO THE OAI AND IMPLEMENTATION STATUS OF UL-TDOA
- Initiation of Location Request:
  - End-to-end procedure
  - External API → LMF determine-location (inputs: SUPI/IMSI, NCGI, Periodic Event Info).
  - TRP info exchange / capability transfer (incl. optional LPP capability).
  - LMF→gNB via AMF: NRPPa POSITIONING INFORMATION REQUEST → gNB allocates UL SRS (and configures UE if needed).
  - SRS activation: LMF asks gNB to activate; UE starts sending SRS.
  - MEASUREMENT REQUEST: LMF→all participating TRPs (via AMF).
  - TRPs measure UL SRS ToA/RToA.
  - MEASUREMENT RESPONSE: TRPs/gNB→LMF (via AMF) with measurements/quality.
  - Positioning: LMF computes UE position from TDoA and returns results (e.g., to LCS/API).

## Fig. 2
- <img width="790" height="677" alt="image" src="https://github.com/user-attachments/assets/f30bcc50-44ac-476b-9726-58fe209c1bb1" />
  - UE — transmits UL SRS according to RRC/DCI.
  - Serving gNB / TRP — coordinates the session, configures the UE.
  - Neighbour gNB / TRPs — additional receive points for TDoA.
  - LMF (Location Management Function) — orchestrates and solves the position.
  - Interfaces: LPP (UE↔LMF, CP positioning protocol); NRPPa (LMF↔(S)gNB/TRPs); RRC/DCI (gNB↔UE).

- **0.NRPPa TRP Configuration Information Exchange**
  - Serving gNB ↔ Neighbour gNB/TRPs (via LMF or peer NRPPa).
  - LMF must know where each TRP is and how well their clocks are aligned; TDoA needs (near-)synchronous receivers or explicit clock biases.

- **1. LPP Capability Transfer**
  - UE ↔ LMF (over NG-RAN).
  - Ensures the plan matches the UE’s radio/positioning capabilities.
 
- **2. NRPPa POSITIONING INFORMATION REQUEST**
  - LMF → Serving gNB.
  - Ask the serving cell to prepare UE resources and line up neighbour TRPs.
 
- **3. gNB determines UL SRS resources**
  - Serving gNB (internal).
  - 38.211 §6.4.1.4 (SRS mapping/sequence), 38.213 §9 (UL power/activation), 38.331 (RRC SRS-Config IEs).
  - UE SRS configuration :Serving gNB → UE (RRCReconfiguration with SRS-ResourceSet/SRS-Resource).

- **4.NRPPa POSITIONING INFORMATION RESPONSE**
  - Serving gNB → LMF.
  - Confirms readiness; lets LMF schedule measurement collection.
 
- **5a. NRPPa Request UE SRS activation**
  - Who: LMF → Serving gNB.
- **5b. Activate UE SRS transmission**
  - Serving gNB → UE.
  - Aperiodic SRS: DCI trigger on PDCCH; or (Semi-)periodic SRS: rely on the configured periodicity/offset.
  - Make the UE actually transmit at precisely scheduled instants.
 
- **6. NRPPa MEASUREMENT REQUEST**
  - LMF → all involved TRPs (serving + neighbours).
  - Tell each TRP exactly what to capture and report.

- **7. UL SRS Measurements**
  - TRPs (receiver side).
  - Correlate with the known SRS sequence, estimate ToA at fractional-sample precision (often via interpolation on the correlation peak), optionally compensate TA/CFO, and keep the Rx time stamp in a common time base (or include local-clock bias).
 
- **8. NRPPa MEASUREMENT RESPONSE**
  - Each TRP → LMF.
  - <img width="648" height="219" alt="image" src="https://github.com/user-attachments/assets/ed2c69ab-be28-4917-a97f-ecfeec91e0a2" />

- <img width="552" height="416" alt="image" src="https://github.com/user-attachments/assets/acd5bb06-bd45-4742-b81f-97deed996206" />
- How a message actually travels
  - LMF asks a gNB to perform UL-TDoA measurements.
  - LMF creates an NRPPa MEASUREMENT REQUEST (NRPPa-PDU).
  - LMF sends it to AMF over NL1 as an HTTP/2 request (payload = NRPPa-PDU), protected by TLS, transported over TCP/IP.
  - AMF does not parse NRPPa; it encapsulates that PDU inside an NGAP message (transparent container).
  - AMF sends the NGAP message to the gNB over NG-C using SCTP/IP.
  - gNB extracts the NRPPa-PDU from NGAP and processes it in its NRPPa entity.
  - The response (e.g., NRPPa MEASUREMENT RESPONSE) returns the same path in reverse.

- TRP Information Request
- LMF starts UL-TDoA by asking all relevant gNBs/TRPs for their TRP information needed for positioning.
- A non-UE-associated NRPPa procedure. AMF forwards it to all connected gNBs.
- LMF → AMF (Namf Non-UE N2 Message Transfer) → gNB over NGAP (Downlink Non-UE Associated NRPPa Transport) → gNB-NRPPa → gNB-RRC → gNB-MAC (via F1AP: TRP Information Request).
  - LMF builds NRPPa PDU and sends to AMF (Namf Non-UE N2).
  - AMF uses NGAP Downlink Non-UE Associated NRPPa Transport to each gNB.
  - gNB receives NGAP, forwards NRPPa PDU to gNB-NRPPa.
  - gNB-NRPPa decodes and sends F1AP: TRP Information Request to gNB-RRC.
  - gNB-RRC forwards the F1AP request to gNB-MAC.
 - All addressed gNBs/TRPs are prompted to prepare their data; next message is TRP Information Response back to LMF.

- TRP Information Response
- Provides each TRP’s config & location to LMF for UL-TDoA positioning.
  - build FIAP: TRP Information Response, forward to gNB-RRC
  - gNB-RRC → pass FIAP to gNB-NRPPa
  - extract IEs, create NRPPa PDU, call NGAP Uplink Non-UE Associated NRPPa Transport via gNB-NGAP
  - gNB-NGAP → send to AMF
  - Namf Communication Non-UE N2 Info Notify → LMF
- Supplies geometry (IDs/CGI) + coordinates (+ uncertainty) so LMF can map TRPs in space and run TDoA fusion accurately.
- Consistent Transaction ID across 2/3/4/6/8; coordinate frames agreed (absolute vs. relative); security via TLS on AMF↔LMF.

- Positioning Information Request:
- After collecting all TRP Information Responses, the LMF requests and instructs the serving gNB to provision UL-SRS for the target UE, preparing radio resources for positioning measurements.
  - LMF builds the NRPPa PDU (Positioning Information Request) and sends it to AMF via Namf Communication N1N2 Message Transfer.
  - AMF forwards it to the serving gNB using NGAP Downlink UE Associated NRPPa Transport.
  - The gNB processes NGAP and forwards the NRPPa PDU to the Downlink UE Associated NRPPa Transport handler in the gNB-NRPPa thread.
  - gNB-NRPPa decodes the PDU and issues F1AP: Positioning Information Request to gNB-RRC.
  - gNB-RRC processes and forwards F1AP: Positioning Information Request to gNB-MAC.
- The serving gNB allocates UL-SRS resources and configures the UE’s UL-SRS resource sets, enabling upcoming measurements.
- Message handling implemented in LMF, AMF, gNB-NGAP, gNB-NRPPa, gNB-RRC, gNB-MAC.
- F1AP manages CU–DU interactions. OAI has added the basic framework for positioning messages (2–9 in Fig. 4) to support split mode, while full processing is currently available for monolithic mode; split-mode completion is planned.

- Positioning Information Response:
- The serving gNB responds to the LMF with the UL-SRS configuration of the target UE so that the LMF can correctly interpret subsequent measurements.
- OAI behavior/limitation: In OAI RAN the UL-SRS is pre-configured at the gNB (no per-UE dynamic change yet). Hence the gNB ignores the detailed request in Msg4 and returns the current SRS configuration applied to the UE.
- Termination: The message terminates at the LMF (forwarded by the AMF).
- Outcome: LMF obtains the effective UL-SRS parameters, enabling the upcoming Measurement Request/Response (Msg8/9) and ToA/TDoA processing.
  - (gNB-MAC): Creates F1AP: Positioning Information Response with SRS config → forwards to gNB-RRC.
  - (gNB-RRC): Forwards to gNB-NRPPa.
  - (gNB-NRPPa): Builds NRPPa PDU → hands to gNB-NGAP via Uplink UE-Associated NRPPa Transport.
  - (gNB-NGAP → AMF): Sends the NRPPa PDU to the AMF.
  - (AMF → LMF): Forwards to LMF with Namf Communication N2 Info Notify

- Positioning Activation Request
- Purpose: After the LMF learns the target UE’s SRS configuration, it activates UE SRS transmission by sending an NRPPa Positioning Activation Request to the serving gNB.
- Scope & Routing: This is a UE-associated NRPPa procedure. The AMF forwards the message only to the UE’s serving gNB.
- Termination / Processing in OAI: The message terminates at the gNB-MAC. Processing exists in LMF, AMF, gNB-NGAP, gNB-NRPPa, gNB-RRC, gNB-MAC.
  - 6.a LMF → AMF: Create NRPPa PDU for Positioning Activation Request and send via Namf Communication N1N2 Message Transfer.
  - 6.b AMF → gNB: Use NGAP: Downlink UE-associated NRPPa Transport, carrying the NRPPa PDU, to the serving gNB.
  - 6.c gNB (NGAP handler): Receive from AMF, then forward the NRPPa PDU to the gNB-NRPPa handler (downlink UE-associated NRPPa Transport).
  - 6.d gNB-NRPPa → gNB-RRC: Decode the NRPPa PDU and forward a Positioning Activation Request to RRC.
  - 6.e gNB-RRC → gNB-MAC: Process and forward the Positioning Activation Request to MAC to trigger/enable UE SRS.
- Outcome: UE SRS transmission is activated according to the specified SRS Resource Set ID and Trigger.

- Positioning Activation Response
- Context: In current OAI RAN, UE SRS is preactivated; real-time SRS modifications are not supported.
→ When the gNB receives the activation request, it generates a Positioning Activation Response to inform the LMF that the UE is transmitting SRS.
- Termination & Components: The message terminates at the LMF. Processing is implemented in LMF, AMF, gNB-NGAP, gNB-NRPPa, gNB-RRC, gNB-MAC.
Note: Even though SRS is preactivated (so a response could be generated entirely at gNB-NRPPa), OAI also wires the path through gNB-RRC and gNB-MAC for completeness and future on-the-fly SRS activation.
  - 7.a gNB-MAC handles the Positioning Activation Request, creates an F1AP: Positioning Activation Response, and forwards it to gNB-RRC.
  - 7.b gNB-RRC processes the F1AP Response from MAC and forwards it to gNB-NRPPa.
  - 7.c gNB-NRPPa parses the F1AP Response, builds the NRPPa PDU (Activation Response), and triggers NGAP: Uplink UE-associated NRPPa Transport to gNB-NGAP.
  - 7.d gNB-NGAP sends the NRPPa PDU to the AMF via NGAP: Uplink UE-associated NRPPa Transport.
  - 7.e AMF → LMF via Namf Communication N2 Information Notify, delivering the Activation Response.

- Measurement Request
- Trigger & Goal: After the Positioning Activation Response, the LMF requests ToA measurements from all participating gNBs/TRPs for the target UE.
- What’s sent: LMF sends a Measurement Request to each gNB, including the SRS configuration previously learned from the serving gNB (via the Positioning Information Response). gNBs use this to perform ToA and return results to the LMF.
- Procedure type & termination: This is a non-UE-associated NRPPa procedure. In OAI’s design, the AMF fans it out to all connected gNBs. The message terminates at the gNB-PHY (PHY handler).
  - 8.a LMF → AMF: Build NRPPa PDU (Measurement Request); send via Namf Communication Non-UE N2 Message Transfer.
  - 8.b AMF → gNBs: NGAP Downlink Non-UE Associated NRPPa Transport to all connected gNBs.
  - 8.c gNB NGAP handler → gNB-NRPPa: Forward NRPPa PDU to Downlink Non-UE Associated NRPPa Transport handler.
  - 8.d gNB-NRPPa → gNB-RRC: Decode PDU; send F1AP: Measurement Request.
  - 8.e gNB-RRC → gNB-MAC: Forward F1AP: Measurement Request.
  - 8.f gNB-MAC → gNB-PHY: Forward FAPI: Measurement Request (PHY executes ToA using the given SRS config).
- Outcome: Each gNB/TRP is instructed—down to PHY—to perform ToA using the specified SRS configuration; results are later reported back to the LMF.

- Measurement Response
- Each participating gNB/TRP, after executing the ToA measurement,
- High-precision ToA (with robust channel estimation + ToA estimation in gNB) is critical for positioning quality.
- Message terminates at LMF; processing wired in LMF, AMF, gNB-NGAP, gNB-NRPPa, gNB-RRC, gNB-MAC, gNB-PHY.
  - 9.a gNB-PHY → gNB-MAC: Build FAPI: Measurement Response (ToA + TRP info).
  - 9.b gNB-MAC → gNB-RRC: Build F1AP: Measurement Response (incl. TRP info).
  - 9.c gNB-RRC → gNB-NRPPa: Forward F1AP response.
  - 9.d gNB-NRPPa → gNB-NGAP: Parse, create NRPPa PDU (Measurement Response); start Uplink Non-UE Associated NRPPa Transport.
  - 9.e gNB-NGAP → AMF: NGAP Uplink Non-UE Associated NRPPa Transport with NRPPa PDU.
  - 9.f AMF → LMF: Namf Communication Non-UE N2 Info Notify to deliver results.
- Outcome: LMF receives per-TRP ToA (UL RTOA) and Rx–Tx time-difference metrics for multilateration and clock-bias handling.








