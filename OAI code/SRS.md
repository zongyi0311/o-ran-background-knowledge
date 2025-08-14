# from https://openairinterface.org/wp-content/uploads/2025/05/OAI-Kista-Workshop-NEU-1.pdf
## Localization Architecture in 5G NR
- It illustrates control-plane positioning (CP-LCS) in 5G: the main entities, the interfaces between them, and which protocol is used by whom.
- UE — User Equipment.
- gNB — 5G base station (NG-RAN node).
- AMF — Access and Mobility Management Function in the 5G Core. It relays LPP messages to the LMF and provides UE context.
- LMF — Location Management Function. The brain of positioning: configures measurements/assistance, collects reports, and computes the position.

- Interfaces / protocols
  - NR-Uu (UE ↔ gNB): the radio interface. Carries the actual positioning reference signals and reports, e.g., DL-PRS/CSI-RS/SSB (downlink) and SRS/PRACH (uplink).
  - NG-C (N2) (gNB ↔ AMF): control-plane interface that transports RRC/NAS signaling.
  - LPP (UE ↔ LMF): LTE/NR Positioning Protocol (extended for NR in Rel-16).
    - Goes via NAS: the UE wraps LPP in NAS to the AMF, which forwards it to the LMF (the slide draws UE↔LMF, but AMF relays it).
    - Purpose: measurement configuration, assistance data, measurement reports, and results between UE and LMF.
  - NRPPa (gNB ↔ LMF): NR Positioning Protocol-A (Rel-15).
    - Purpose: LMF↔RAN coordination. The LMF requests/provides assistance (e.g., PRS scheduling) and measurement tasks; the gNB returns results (time/angle/power, etc.).
  - Nls / Nlmf_Location (AMF ↔ LMF): a 5GC service-based interface used for the AMF to forward LPP and exchange location-related context/events with the LMF.

- Two typical flows
  - Downlink positioning
    - LMF → gNB (NRPPa): send assistance/measurement requests (PRS config, timing, etc.).
    - gNB → UE (RRC over NR-Uu): deliver PRS/measurement config; the gNB transmits DL-PRS/CSI-RS as scheduled.
    - UE measures (ToA/TDoA, RSRP, AoD…) → UE → LMF (LPP over NAS via AMF): send measurement report.
    - LMF fuses/estimates the position → returns via LPP to the UE or exposes to applications.
  - Uplink positioning
    - LMF → gNB (NRPPa): ask one or more gNBs to measure the UE’s SRS/PRACH (arrival time/angle).
    - UE transmits SRS/PRACH (NR-Uu); gNB measures and reports via NRPPa to the LMF.
    - LMF combines multi-cell measurements to compute the position; if needed, informs the UE via LPP.


<img width="1401" height="664" alt="image" src="https://github.com/user-attachments/assets/a702d3b3-b790-40e3-b608-bec339a70b76" />

- The UE transmits uplink SRS (or PRACH). Multiple synchronized gNBs receive it and estimate the Time of Arrival (ToA).
- Each gNB sends its ToA to the LMF via NRPPa (relayed by the AMF).
- The LMF converts ToAs to TDoAs (pairwise differences), solves for the UE’s coordinates from the intersection of the resulting hyperbolas, and exposes the position through an LCS API.

- Signaling / interfaces (matching the figure)
  - NR-Uu: UE → gNB radio link carrying SRS.
  - NRPPa (NG-C / Nls): gNB reports ToA to LMF (via AMF).
  - LPP (not drawn): used in UE-based flows between UE and LMF; this slide focuses on the network-based case.
  - LCS: external location service interface (HTTP).

- In TDoA = ToAᵢ − ToAⱼ, the UE transmit clock bias cancels, so only inter-gNB synchronization must be tight (GNSS, IEEE-1588/TSN, etc.).
- Geometry: for gNBs 𝑖,𝑗
- <img width="650" height="129" alt="image" src="https://github.com/user-attachments/assets/10ec49c2-ce77-4c00-8f41-4c7f3ca6bc2c" />

- More bandwidth ⇒ finer ToA resolution: ideal timing resolution ≈ 1/BW; ~400 MHz can approach sub-meter (needs high SNR and calibration).
- Inter-gNB synchronization is critical: clock error maps directly to range-difference error.
- Multipath/NLOS: mitigate with peak selection, RANSAC/consistency checks, or add AoA aids.
- Geometry/observability: at least 3 gNBs (≥2 TDoAs) for robust intersections; poor geometry inflates error (GDOP).
- Uplink references: typically SRS (wideband, repeatable); PRACH is possible but with bandwidth/interpolation constraints.


- Implementation of 3GPP UL-TDoA Positioning in OAI
- RAN: Performs SRS→ToA estimation in the OAI gNB; implements NRPPa functionality and sends it to the AMF.
- Core: Integrates with the TU-Dresden LMF; communicates NRPPa PDUs between the gNB and AMF, and between the AMF and LMF; implements built-in/pluggable positioning algorithms in the LMF; and connects to the Fraunhofer IIS PaaS.
- API: Externally triggers positioning; accurately aligns antenna and UE positions using Cartesian coordinates.
---

# https://ieeexplore-ieee-org.ntust.idm.oclc.org/xpl/ebooks/bookPdfWithBanner.jsp?fileName=9963506.pdf&bkn=9962820&pdfType=chapter
## 1.1 Background
- Communications and radar have evolved separately but share algorithms, hardware, and partially architectures.Driven by spectrum/cost/energy sharing, there’s growing interest in coexistence → cooperation → joint design (JCAS/ISAC).
- Traditional coexistence: Two independent systems transmit **separate signals** (possibly overlapping in time/frequency) and rely on **interference management** (beamforming, cooperative/dynamic sharing, primary–secondary access).**Limits**: requires tight mobility constraints and information exchange; spectral-efficiency gains are limited and conditions are stringent.
- Passive sensing: Use arbitrary radio signals (TV, Wi-Fi, cellular) for sensing because the environment imprints on received signals.

  Pros: no dedicated radar waveform needed.
  
  Two major drawbacks:
  - TX/RX not synchronized → unknown, time-varying time/frequency/phase offsets ⇒ timing/ranging ambiguity and hard-to-fuse measurements.
  - Unknown signal structure → poor interference suppression and multi-user separation; waveforms are not optimized for sensing.

- From “radar” to broader “radio sensing”:
  - Radio sensing = extracting information from received radio signals (not from the carried comms data).
  - Two processing tracks: sensing parameter estimation (delay, AoA/AoD, Doppler, path magnitudes) and pattern recognition (device/object/activity “radio signatures”).
  - Works over existing infrastructures (IoT, Wi-Fi, 5G). Many studies show low-bandwidth comms signals can support sensing (e.g., Wi-Fi, RFID, ZigBee).
 
- JCAS/ISAC core idea: One jointly designed transmit signal serves both communications and sensing.
  - Hardware sharing: most TX modules and much of RX hardware are shared (baseband processing diverges).
  - Advantages over passive sensing: controllable synchronization, known signal structure, multi-user separation/interference suppression, and waveform/scheduling optimized for sensing.
  - Different from:
    - Separated waveforms (time/frequency/code partition),
    - Conventional coexistence (two independent systems avoiding interference),
    - Cognitive radio (secondary users opportunistically access spectrum).
JCAS = one system, one signal, jointly designed for both functions.

| Approach                  | Signal strategy             | Main mechanism                        | Pros                                                    | Cons                                                 |
| ------------------------- | --------------------------- | ------------------------------------- | ------------------------------------------------------- | ---------------------------------------------------- |
| Separated comms + sensing | Two waveforms               | Time/freq/code separation             | Simple, no mutual interference                          | Resource split → lower efficiency                    |
| Coexistence               | Two waveforms (may overlap) | Interference management/sharing       | Reuse spectrum; incremental deployment                  | Needs coordination/sync; limited net gain            |
| Passive sensing           | Any broadcast signals       | No TX changes                         | No dedicated radar TX                                   | Desync, unknown structure, poor interference control |
| Cognitive radio           | Comms-centric               | Sense spectrum → opportunistic access | Better spectrum utilization                             | Not designed for sensing tasks                       |
| **JCAS/ISAC**             | **Single, joint waveform**  | **Joint design + shared HW**          | **Sync/structure known; can optimize; high efficiency** | Design complexity; requires standard & HW support    |

## 1.2 Three Categories of JCAS Systems

| Aspect                  | Communication-centric                      | Radar-centric                                 | Joint design & optimization                           |
| ----------------------- | ------------------------------------------ | --------------------------------------------- | ----------------------------------------------------- |
| Design priority         | Communications first                       | Radar/sensing first                           | Balanced (tunable)                                    |
| Waveform basis          | Reuse comms (e.g., OFDM)                   | Reuse radar (pulse/chirp/FMCW)                | New or hybrid, co-designed (DFRC/ISAC)                |
| Comms performance       | **High**, standard-friendly                | Constrained throughput/SE                     | Potentially high (by design)                          |
| Sensing performance     | Scenario-dependent, harder to optimize     | **Near-optimal**                              | Potentially high (by design freedom)                  |
| Standards compatibility | Minor extensions                           | Radar-led, loosely tied to comms standards    | May need new specs / larger changes                   |
| HW changes              | Small–medium                               | Small (keep radar chain)                      | Medium–large (high sharing/co-design)                 |
| Typical strength        | Fast deployment, easy in existing networks | Excellent radar accuracy/range                | Best spectrum/energy efficiency; system-level optimum |
| Typical weakness        | Limited sensing flexibility/accuracy       | Limited data rate                             | Design complexity; ecosystem/standardization effort   |
| Good fit                | Add sensing inside 3GPP/Wi-Fi systems      | Radar-centric domains (automotive/industrial) | Greenfield/6G R\&D, private networks                  |


### 1.2.1 Major Differences Between Communications and Sensing
- Radar/radio-sensing signals are built to estimate physical parameters (range, Doppler, angle) accurately and simply.
- Communication signals are built to carry as much information as reliably as possible across many users and QoS profiles.
- Because the design goals differ, their preferred waveforms/structures conflict, so you can’t drop one into the other without tweaks.

| Dimension            | Radar / Sensing                                                                                                                                                                                                  | Communications                                                                                                                                       |
| -------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Primary goal**     | High-accuracy localization & tracking; simple parameter estimation                                                                                                                                               | Maximize information rate & reliability across users/QoS                                                                                             |
| **Waveform traits**  | Low **PAPR** → efficient PA & long range; **sharp ambiguity function** (narrow/steep mainlobe, low sidelobes) for high range/Doppler resolution; often deterministic (pulses, chirps, constant-envelope options) | Rich **modulation/coding** across time/frequency/space; **packetized**, scheduler-driven; may be **discontinuous/fragmented** over RBs, slots, beams |
| **Resource use**     | Typically contiguous time–frequency blocks, long coherent processing intervals                                                                                                                                   | Highly dynamic allocation (OFDMA/TDMA/beam hopping, CA), multiuser multiplexing                                                                      |
| **Receiver focus**   | Matched filtering, range–Doppler–angle estimation, CFAR                                                                                                                                                          | Sync, channel estimation, equalization, decoding, HARQ                                                                                               |
| **Typical waveform** | Pulsed/linear-FM (chirp), CW/FMCW, radar-OFDM with sensing-friendly design                                                                                                                                       | OFDM/OFDMA with high PAPR and data payloads + pilots/DMRS                                                                                            |

- Using comms for sensing suffers from high PAPR, fragmented resources, and an OFDM ambiguity function that isn’t ideal (range–Doppler coupling, sidelobes) → needs dedicated pilots/PRS, windowing, guard resources, or PAPR reduction.
- Using radar for comms preserves sensing accuracy, but limits data rate/spectral efficiency unless you relax radar optimality.
- Joint design can tune: pilot density, burst timing, bandwidth, beam scheduling, and waveform shaping to get both good sensing resolution and acceptable throughput/latency.


- Two conventional radar families:
  - Pulsed radar: transmits short, wideband pulses, then stays silent to receive echoes.
  - Continuous-wave radar (e.g., FMCW): transmits continuously (often chirps), sweeping over a wide frequency range.
  
  Both are typically non-modulated and used in SISO or MIMO (with orthogonal waveforms for MIMO).
  
- Design goals for radar waveforms:
  - Low PAPR → efficient power amplifiers and long-range operation.
  - Sharp ambiguity function (narrow/steep mainlobe, low sidelobes) → high range/Doppler resolution.
  - Simple receiver processing to estimate delay, Doppler, and angle of arrival.
  
- Sampling/processing differences:
  - Pulsed radar: sample roughly at 2× the pulse bandwidth, or at a lower rate set by the desired range (delay) resolution.
  - FMCW radar: sample only the beat frequency, at a rate much smaller than the sweep bandwidth, sized to the maximum delay to detect.
 
- What communication signals look like
  - They’re built to carry lots of data: heavily modulated, packetized, and often include non-modulated training (preambles/pilots) as in Fig. 1.2.
  - To serve many devices and QoS levels, they can be complex: discontinuous/fragmented in time–frequency, high PAPR, and use advanced modulations across time, frequency, and space (MIMO/beamforming).
 
- Can they be used for sensing?
  - Yes—potentially you can estimate delay, Doppler, angle, path power, etc. from comms signals.
  - But sensing needs more than the usual comms channel estimate.

- “Channel coefficients” vs “channel composition”
  - Comms receiver: estimates a compact set of channel coefficients (effective complex gains on pilots/subcarriers) good enough for equalization/decoding.
  - Sensing: needs the composition of the channel—i.e., the set of multipath components with their individual delays, Dopplers, angles, and amplitudes.
  That’s a finer, higher-resolution description and is hardware-limited (ADC rate, dynamic range/linearity, clocks/calibration) and algorithmically harder than standard channel estimation.

- Why new sensing algorithms are needed
  - Comms signals’ fragmentation, high PAPR, and rich structures make them unlike classic radar waveforms, so radar processors can’t be used as-is.
  - To do coherent sensing, you must know the exact signal structure (resource mapping in time/frequency/space and even the data symbols if you want data-aided processing).
Without that knowledge—as in most passive radar—you’re stuck with non-coherent detection, which yields fewer parameters and worse performance.

- Two real-world blockers for comm-integrated sensing:
  - Full-duplex (FD) with co-located TX/RX—as in monostatic radar—remains hard in communications (self-interference is tough).
  - Asynchrony between spatially separated TX/RX nodes—clocks aren’t aligned—hurts the fine time/phase info sensing needs.

- Why classic radar tricks don’t port cleanly
- Pulsed radar: uses transmit → long silence → receive (effectively TDD) to dodge FD. Great for sensing, but those long quiet windows conflict with high-throughput/low-latency comms.
- Continuous-wave (FMCW): uses the TX signal as the LO reference at the receiver and only processes the beat frequency (difference between TX and echo). This is low-complexity and efficient for sensing, but:
  - The availability and bandwidth of the beat signal are uncertain, so it’s unreliable for data carriage.
  - Overall, these designs constrain integration of communication functions and limit achievable data rates.

- Impact on ISAC/JCAS in communications
  - FD for comms is still immature (requires deep self-interference cancellation, high linearity, large ADC dynamic range).
  - Clock asynchrony across nodes degrades time-/phase-based sensing (precise ToA/TDoA, coherent processing).
  - Bottom line: simply dropping monostatic radar FD techniques into current comm systems hits reliability and throughput limits—you need new sensing solutions and system designs.


| Specification                | **Radar**                                                                                                                                                            | **Communications**                                                                                                                   | **JCAS system**                                                                                                                            |
| ---------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------ |
| **Signal waveform**          | Usually simple, **unmodulated** single-carrier; pulsed or CW/FMCW; orthogonal across time/freq/space/code for MIMO; **low PAPR**; emerging radar-OFDM / freq-hopping | Mix of **unmodulated** (pilots/training) + **modulated** data; complex structures and resource usage (OFDMA, MU-MIMO); **high PAPR** | Can adopt radar or comms waveforms and **modify/co-design** to support and jointly optimize both functions                                 |
| **Tx power / link distance** | Typically **high** (long-range); lower for short-range (e.g., automotive FMCW)                                                                                       | Typically **low**, link up to a few km                                                                                               | Comms integrated into radar → **very long** links; sensing integrated into comms → shorter per-link but **wide area** via network coverage |
| **Bandwidth**                | **Large**; range resolution ∝ bandwidth (FMCW beat bandwidth may be narrow)                                                                                          | **Smaller** than radar                                                                                                               | **mmWave** bandwidth is very promising; some sensing works even with **low BW** (e.g., Wi-Fi sensing)                                      |
| **Signal band**              | **X/S/C/Ku**                                                                                                                                                         | **Sub-6 GHz** and **mmWave**                                                                                                         | Choice of band impacts operating distance and resolution                                                                                   |
| **Duplex**                   | **Full-duplex** (CW) or **half-duplex** (pulsed)                                                                                                                     | Co-located TX/RX usually cannot operate same time & freq; systems are **TDD** or **FDD**                                             | **FD preferred** but **not essential**                                                                                                     |
| **Clock synchronization**    | TX and RX **clock-locked**                                                                                                                                           | Co-located nodes may share a clock; distributed nodes typically do **not**                                                           | **Clock-level sync** removes sensing ambiguities; helpful but not always required                                                          |

- Waveform: Radar favors low-PAPR, ambiguity-friendly waveforms; communications favors complex, packetized, high-PAPR signals; JCAS can use either and co-design to optimize both functions.
- Tx power & range: Radar is typically high-power (long range); communications is lower-power (up to a few km). JCAS embedded in radar can reach very long links; sensing embedded in comms has shorter per-link range but benefits from network coverage.
- Bandwidth: Radar uses large bandwidth for range resolution; comms is typically smaller. mmWave bandwidth is especially attractive for JCAS, though low-BW sensing (e.g., Wi-Fi) is also possible.
- Bands: Radar often in X/S/C/Ku; comms in sub-6 GHz and mmWave. Available band directly impacts JCAS distance and resolution.
- Duplexing: Radar can be FD (CW) or HD (pulsed). Comms co-located TX/RX rarely operate same-time/same-frequency (mostly TDD/FDD). JCAS: FD is nice to have, not strictly required.
- Clock sync: Radar TX/RX typically share a clock. In comms, only co-located nodes share a clock; distributed nodes don’t. JCAS: clock-level sync removes ambiguity for sensing, but isn’t mandatory for all apps.

## 1.2.2 Communications-Centric Design
- CC-JCAS
  - CC-JCAS adds radio sensing to an existing communication system as a secondary function.
  - The main signals/protocols stay largely unchanged; you may need some infrastructure tweaks or feature extensions.

- Two fundamental hurdles
  - Monostatic full-duplex (FD): TX and sensing RX are co-located and must transmit and receive simultaneously on the same band.
  - Clock asynchrony in bi/multi-static setups: spatially separated TX/RX nodes usually don’t share clocks, so timing/phase errors corrupt sensing estimates.
 
- Why classic radar tricks don’t port to comms
  - Pulsed radar avoids FD by separating TX/RX in time (TDD), which creates near-field blind zones and clashes with continuous high-throughput comms.
  - FMCW radar uses the TX signal as the LO reference and only processes the beat frequency, which carries little of the transmitted data; modern comms use continuous waveforms with sinusoidal LOs, so this approach isn’t practical unless you bolt on FMCW-like sensing hardware.

- FD cancellation for communications is a promising long-term solution for monostatic sensing, but not yet mature for broad deployment.
- Clock sync in bi/multi-static radar is often done via cables or GPS disciplining—workable in some comm setups, but not a general solution.


- CC-JCAS by network topology
- Two subcategories
  - Point-to-point (P2P) JCAS – sensing added to a single comms link (common in vehicular networks).
  - Network JCAS – sensing realized in a multi-node network (e.g., cellular/mobile).

- Geometry analogy for sensing
  - Monostatic: TX and sensing RX co-located.
  - Bistatic: TX and sensing RX at different nodes.
  - Multistatic: multiple spatially separated TX/RX nodes.

| Aspect                             | Wi-Fi networks (vs Mobile)                                                   | Implications for **Wi-Fi-JCAS** (vs PMN)                                                                              |
| ---------------------------------- | ---------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| **Signal format & transmission**   | Simpler, flexible packets; timing/channel structure less rigid than cellular | More freedom to optimize waveforms; sensing opportunities are **more random in time** (less deterministic scheduling) |
| **Multiuser access**               | Simpler MAC; cellular has complex resource allocation & mixed MU access      | Parameter estimation can be **simpler**; more algorithm choices (but fewer tightly scheduled pilots)                  |
| **Deployment environment**         | Mostly **indoor**, low speed                                                 | Richer **multipath** but **stable clutter** → sensing is often **easier** than outdoor cellular                       |
| **Network infrastructure & scale** | Smaller networks; less powerful infra (e.g., smaller arrays)                 | Limited **networked sensing** potential; typically **lower sensing resolution** than large cellular arrays            |


## 1.2.3 Radar-Centric (RC) Design
- What RC-JCAS is
  - Start from a radar (often long-range, military/aviation) and add communications on top of the radar waveform/hardware.
  - Often called Dual-Function Radar–Communications (DFRC); in this book DFRC ≈ RC-JCAS.

- Very long range links (hundreds of km possible) and lower latency than satellite, since you reuse high-power radar transmitters and narrow, directive beams.

- Main limitation
  - Data rate is constrained by the radar waveform (pulsed or CW/FMCW). You can convey data, but not at cellular-like throughputs unless you change the waveform/receiver.
 
- Core technical challenge: information embedding
  - Embed data into pulsed or continuous-wave radar signals while barely disturbing radar performance.
  - Techniques surveyed in the literature include various information embedding schemes; goal = minimal radar impact + as high a data rate as feasible.


- Index Modulation (IM) for RC-JCAS
  - Carry bits via the indices/combinations of resources in space, time, frequency, code (e.g., which subcarriers or TX antennas are active).
  - Key advantage: IM does not alter the basic radar waveform/structure, so sensing performance is essentially preserved while enabling data transfer.

1.2.4 Joint Design without an Underlying System
- Design JCAS from scratch—not constrained by legacy comms or radar—so waveform/beamforming/scheduling can be jointly optimized to trade off comms throughput and sensing accuracy.
- Design freedom: choose waveforms, frames, pilots, beams, and MAC specifically for joint goals.
- Better trade-offs: tune metrics (rate, latency, range/angle/Doppler resolution) explicitly.

- mmWave & sub-THz JCAS
  - Pros: huge bandwidth + short wavelength ⇒ high data rates and fine sensing resolution.
  - Status: standards still emerging; ideal for new designs.

- Multichannel / Frequency-Hopping JCAS
- Use one (or a few) channels at a time, but hop across many over a period (e.g., Bluetooth-like FH).
- Net effect: large aggregate sensing bandwidth without large instantaneous comm bandwidth → lower RF/ADC cost and fits spectrum usage.
- Key challenge: stitching channels—remove per-channel distortions/offsets (gain/phase/timing/LO drift) and concatenate coherently.
- JCAS twist: co-design signals/frames to ease concatenation while not hurting comm rate/latency.

- Design levers
  - Waveform: OFDM/PMCW/SC, guard intervals, PAPR control, pilot density/placement.
  - Beams: multibeam patterns, update cadence, hybrid precoder splitting power/time across tasks.
  - Scheduling: mini-frames or sensing bursts that preserve HARQ/latency budgets.
  - Resources: subcarrier/PRB partition (static, dynamic, interleaved) for comms vs sensing.
  - Calibration & sync: per-beam/ per-channel phase & delay calibration to enable coherent sensing.







- TS 38.211 — Physical channels and modulation (UL reference signals: SRS) → §6.4.1.4 (sequence generation & resource mapping), comb, cyclic shift, group/sequence hopping, antenna ports.
- TS 38.213 — Physical layer procedures (UL timing/power/control; SRS configuration & triggering) → §4 (Transmission timing adjustments / TA), §7 (UL power control), §9 (SRS: periodic/semi‑persistent/aperiodic, bandwidth configuration, hopping, resource sets, triggering via DCI), plus multiplexing constraints.
- TS 38.214 — Physical layer procedures for data (use of measurements for link adaptation, subband/wideband metrics that gNB may derive from SRS; CSI framework linkage).
- TS 38.331 — RRC (I‑E definitions for SRS‑Config, SRS‑Resource, SRS‑ResourceSet, RRCReconfiguration delivery).

# openair2/RRC/NR/MESSAGES/ASN.1/nr-rrc-*.asn1// TS 38.331 6.2.1
- → locate SRS‑Config / SRS‑Resource / SRS‑ResourceSet 

# UE apply: openair2/LAYER2/NR_MAC_UE/config_ue.c
## configure_dedicated_BWP_ul()
- config_ue.c is OAI’s UE-side RRC→MAC/PHY configuration bridge. It parses the gNB’s RRC messages (MIB, SIB1/SIB19, CellGroupConfig, etc.) and translates their ASN.1 IEs into OAI UE internal state and PHY/FAPI config. It sets up carrier/BWP/CORESET & search spaces, PDSCH/PUSCH/PUCCH/SRS, CSI-RS & CSI reporting, timers (BSR/SR/TA), TDD pattern, PRACH, builds the valid SSB list, applies reconfiguration-with-sync, handles NTN timing advance (Rel-17), and cleans up on reset.
- **Guard & fetch BWP slot**
  - If ul_dedicated is non-NULL, it calls get_ul_bwp_structure(mac, bwp_id, true) to get (or create) the internal NR_UE_UL_BWP_t for bwp_id.
  - Sets bwp->bwp_id = bwp_id.
- **UCCH configuration (uplink control channel)**
  - Checks ul_dedicated->pucch_Config.
  - Release case: if present == ...PR_release, frees the existing bwp->pucch_Config via asn1cFreeStruc(asn_DEF_NR_PUCCH_Config, ...).
  - Setup case: if present == ...PR_setup, ensures bwp->pucch_Config exists (allocates with calloc if needed), then calls
    setup_pucchconfig(ul_dedicated->pucch_Config->choice.setup, bwp->pucch_Config)
    to translate the ASN.1 RRC IE into the UE’s internal structure.
- **PUSCH configuration (uplink shared channel)**
  - Same pattern:
    - Release: asn1cFreeStruc(asn_DEF_NR_PUSCH_Config, bwp->pusch_Config).
    - Setup: allocate if NULL, then
      setup_puschconfig(mac, ul_dedicated->pusch_Config->choice.setup, bwp->pusch_Config).

- **SRS configuration (sounding reference signal)**
  - Same pattern:
    - Release: asn1cFreeStruc(asn_DEF_NR_SRS_Config, bwp->srs_Config).
    - Setup: allocate if NULL, then
      setup_srsconfig(bwp, ul_dedicated->srs_Config->choice.setup, bwp->srs_Config).

- Typically invoked when the UE processes an RRC(Re)configuration that modifies uplink BWP-dedicated settings. After this call, the UE MAC (and then PHY) will use the updated per-BWP configs for uplink scheduling, control reporting (PUCCH), data (PUSCH), and channel sounding (SRS).

## setup_srsconfig()
- setup_srsconfig(...) merges the source RRC SRS-Config (source) into the UE's current BWP SRS-Config (target). It follows the RRC AddMod/Release rules: if there is an Add/Mod, the corresponding SRS-Resource / SRS-ResourceSet is added or updated; if there is a Release, the corresponding target item is released based on the id; simple fields (such as tpc_Accumulation) are also updated.
- ket steps:
  - UPDATE_IE: If the source provides tpc_Accumulation, copy it to the target.
  - SRS-Resource:
    - AddModList → Ensure the target list exists, then add/update it using srs_ResourceId as the key.
    - ReleaseList → Remove from the target list based on srs_ResourceId.
  - SRS-ResourceSet:
    - AddModList → Compare srs_ResourceSetIds one by one. If they don't exist, calloc + append a new set (ASN_SEQUENCE_ADD); then use setup_srsresourceset(...) to write the source set's fields to the target set.
    - ReleaseList → Remove from the target list based on srs_ResourceSetId.
  - bwp passes setup_srsresourceset(...) to facilitate the helper's calculation/validation of derived fields based on the BWP state.
 
## setup_srsresourceset()
- What setup_srsresourceset(...) does (purpose & flow)
- This routine merges one SRS-ResourceSet IE (source) into the UE’s current UL-BWP SRS-ResourceSet (target) and updates any derived power-control cache in the BWP when key fields change. It handles the resourceType CHOICE (aperiodic/periodic/semi-persistent), membership lists, usage, and SRS power-control knobs (alpha, p0, pathlossReferenceRS, adjustment states).

# TS 138 214  6.2 UE reference signal (RS) procedure 
## 6.2.1 UE sounding procedure 
- SRS configuration scope
  - UE can be configured with one or more SRS-ResourceSets (and, separately, SRS-PosResourceSets for positioning).
  - In each SRS-ResourceSet, the UE may have K ≥ 1 SRS-Resources:
    - K (normal SRS) is bounded by UE capability (see UE-cap in 38.306).
    - K (PosSRS): max K = 16.
  - Whether a resource/set applies depends on the usage field of the SRS-ResourceSet.
- Simultaneous transmission rule
  - If usage = beamManagement: at any instant, only one SRS resource per set may be transmitted.
  - However, resources from different sets with the same time-domain behavior (periodicity/offset etc.) within the same BWP may be transmitted simultaneously.
- TCI linkage (beam state the UE should use)
  - If the UE is configured with DLorJoint-TCIState or UL-TCIState, then (with two exceptions noted below) SRS resources in any set can be associated with those TCI states or updated per the referenced clause.
  - Reference RS for TCI:
    - DLorJoint-TCIState: reference can be a CSI-RS in an NZP-CSI-RS-ResourceSet configured with repetition or with trs-Info (TRS).
    - UL-TCIState: reference can be a CSI-RS (as above), an SRS resource with usage=beamManagement, or an SS/PBCH block (SSB), possibly with a different PCI than the serving cell.
  - Exceptions: the generic assumption above does not apply to
    - positioning SRS resources, and
    - SRS resource sets configured with followUnifiedTCIState-r17 (those follow the unified TCI behavior explicitly).
- Follow-Unified-TCI (r17) for SRS-ResourceSet
   - If an SRS-ResourceSet (not for positioning) is configured with followUnifiedTCIstate-r17, the UE shall transmit the target SRS resource(s) using the spatial relation (beam / spatial filter) of the reference RS tied to the indicated TCI state:
     - The reference RS is either:
     - For DLorJoint-TCIState: a CSI-RS in an NZP-CSI-RS-ResourceSet with repetition or with trs-Info (TRS).
     - For UL-TCIState: a CSI-RS as above, or an SRS with usage=beamManagement, or an SS/PBCH block (SSB) — possibly with PCI ≠ serving cell PCI.
  - Effect: the SRS beamforming follows the downlink (or unified) TCI reference, ensuring DL/UL spatial consistency.
- Aperiodic SRS selection (by DCI)
  - For aperiodic SRS, at least one DCI state selects at least one of the configured SRS-ResourceSet(s).
  - Practical: the gNB uses a DCI field (trigger state) to point to which set (and therefore which resources) the UE should transmit in a given slot.

### Semi-static (higher-layer) SRS parameters
- These are the RRC-configured knobs that define how an SRS looks and when/where it is sent. They change rarely (until a new RRC reconfiguration) and drive both scheduler choices and PHY mapping
- These parameters define when (slot/symbol), where (RBs/subcarriers), how wide (bandwidth/hopping), with which beam/ports (spatial relation, ports), and with what orthogonality (comb/shift/sequence) the UE transmits SRS.
- The scheduler uses them to pick resources/sets (and offsets for aperiodic), while PHY uses them to generate the exact SRS waveform and set the TX spatial filter.

---
- Resource mapping (how many symbols, where in the slot)

| Config                       | Allowed Ns (adjacent symbols) | Placement               |
| ---------------------------- | ----------------------------- | ----------------------- |
| Baseline (`resourceMapping`) | 1, 2, 4                       | **last 6** symbols only |
| r16 (`SRS-Resource`)         | 1, 2, 4                       | **anywhere**            |
| r16 (`SRS-PosResource`)      | 1, 2, 4, 8, 12                | **anywhere**            |
| r17 (`SRS-Resource`)         | 1, 2, 4, 8, 10, 12, 14        | **anywhere**            |

- Overlap with PUSCH/PUCCH (priority rules)
  - PUSCH (priority index 0) + SRS (same slot, serving cell): UE may only send SRS after finishing PUSCH and its DM-RS in that slot.
  - PUSCH (priority 1) or PUCCH (priority 1) overlapping with SRS: UE does not transmit SRS in the overlapping symbol(s).       

- Periodic SRS — spatial relation (beam to use)
  - When resourceType = periodic and spatialRelationInfo (or …PDC, …Pos) is configured, the UE transmits SRS with the same spatial-domain TX filter as the reference RS indicated:
  - Reference = SSB (ssb-Index / …Serving / …Ncell): use SSB-based spatial filter.
  - Reference = CSI-RS (csi-RS-Index / …Serving): use the filter used to receive that periodic or semi-persistent CSI-RS.
  - Reference = SRS (srs / srs-spatialRelation): use the filter used to transmit the reference periodic SRS.
  - Positioning SRS (SRS-PosResource + dl-PRS): use the filter used to receive the DL-PRS.
  - spatialRelationInfo-PDC with dl-PRS-PDC: use the filter used for DL-PRS reception for RTT-based delay compensation (per spec Clause 9).
The transmit beam of the periodic SRS is not determined autonomously, but follows the indicated reference RS/SSB/PRS to ensure DL/UL spatial consistency or the consistency required by the positioning algorithm.

- UE has one or more SRS resources and the higher-layer resourceType is semi-persistent (either SRS-Resource or SRS-PosResource).
- When the activation actually takes effect
  - The gNB sends a MAC activation command (38.321 §6.1.3.17 / §6.1.3.36) on a PDSCH.
  - Suppose the HARQ-ACK for that PDSCH is sent by the UE on PUCCH in slot n.Then the SRS start assumptions (and the MAC actions) become valid from the first slot strictly after:

<img width="661" height="148" alt="image" src="https://github.com/user-attachments/assets/e60b804d-500e-4e32-b0db-6a5e3b5f01d6" />

- The activation command also tells you which beam to use
- It includes a list of reference-signal IDs, one per element of the activated SRS-ResourceSet.
- Each ID defines the spatial relation (i.e., which RX/TX spatial filter/beam to copy) for the corresponding SRS.
  - If the set is SRS-ResourceSet (non-positioning):
    - Each ID may refer to one of:
      - an SS/PBCH block (SSB),
      - an NZP-CSI-RS on the serving cell indicated by Resource Serving Cell ID in the command (or the set’s cell if absent),
      - an SRS resource on the serving cell and UL BWP indicated by Resource Serving Cell ID and Resource BWP ID (or the set’s cell/BWP if absent).
  - If the set is SRS-PosResourceSet (positioning):
    - Each ID may refer to one of:
      - an SSB on a serving or non-serving cell (given by the PCI field),
      - an NZP-CSI-RS on the serving cell (as above),
      - an SRS resource on the serving cell + UL BWP (as above),
      - a DL-PRS on a serving or non-serving cell (by DL-PRS ID).
     
- In practice: when you activate a semi-persistent SRS set, the command binds each SRS in the set to a specific reference RS/SSB/PRS, and the UE must transmit that SRS using the same spatial-domain filter/beam as that reference.

- Override rule (activation wins)
  - If a resource in the activated set already had an RRC-configured spatialRelationInfo/…Pos, the reference ID(s) carried in the activation command override those.
  → During the active period, use the activation’s reference to choose the SRS TX beam, not the one from RRC.
- Deactivation timing
  - When the UE gets a deactivation command and would send the HARQ-ACK for its PDSCH on PUCCH in slot n, the stop takes effect from the first slot after:
  - <img width="571" height="102" alt="image" src="https://github.com/user-attachments/assets/7330b3c9-0b76-4a18-8589-8d0e35c39d34" />
- How to choose the SRS transmit beam (spatial relation)
  - If spatialRelationInfo/…Pos identifies one of the following, transmit the target SRS with the same spatial-domain filter used for that reference:
  - ssb-Index / …Serving / …Ncell (SSB): use the SSB receive beam.
  - csi-RS-Index / …IndexServing (NZP-CSI-RS): use the receive filter for that periodic/semi-persistent CSI-RS.
  - srs / srs-SpatialRelation (SRS): use the transmit filter used for that periodic/semi-persistent SRS.
  - dl-PRS (only for SRS-PosResourceSet): use the DL-PRS receive filter.
