# Contributions & Structure of the Paper
- What this survey contributes
  - Focus: JCAS specifically in cellular/mobile networks → “Perceptive Mobile Network (PMN)” viewpoint (infrastructure and signal-processing).
  - Depth on receiver processing: practical details for handling OFDMA, MU-MIMO/SDMA, heterogeneous RAN architecture, and complex propagation.
  - From comm-only → PMN: identifies system changes needed to evolve today’s networks to integrated communication + sensing.
  - Comprehensive tech review: performance bounds, joint waveform design, antenna/array, clutter suppression, sensing parameter estimation, ambiguity/resolution, pattern analysis, networked sensing, and sensing-assisted communications.
  - Clear roadmap: frameworks, evolution paths, challenges, key technologies, and open problems for PMN.
- Paper structure
  - Section II – JCAS & Motivation (what to integrate)
    - Radar-centric, Comm-centric, and Joint designs; future integration path.
  - Section III – PMN Framework (what it looks like)
    - Platform & infrastructure.
    - Three unified sensing modes in mobile networks.
    - Signals usable for sensing (e.g., SSB/PRS/SRS/CSI-RS).
    - Signal/channel models (Appendix).
  - Section IV – System Evolution (how to enable it)
    - Long-term full-duplex option.
    - Three near-term enablers without heavy network changes (esp. TDD):
      - Dedicated Tx for uplink sensing (IV-A).
      - Dedicated Rx for downlink sensing (IV-C).
      - Widely separated Tx & Rx antennas (IV-D).
  - Section V – Major Challenges (why it’s hard)
    - Sensing parameter extraction in cellular signals.
    - Joint design & optimization.
    - Networked sensing across many nodes.
  - Section VI– Key Technologies (how to solve)
  - Section VII – Conclusions & Open Problems
 
# II. Three Types of JCAS Systems
- Communication-centric design (PMN belongs here)
  - What: Add sensing to a primary mobile communication system (5G/6G).
  - How: Reuse cellular signals (SSB/PRS/SRS/CSI-RS, etc.) and infrastructure; apply slight system/protocol tweaks without degrading comm KPIs.
  - Pros/cons: Highest deployability (uses existing network); sensing must adapt to comm waveforms and scheduling.
  - Point-to-Point JCAS
    - Multi-carrier: OFDM/SC-FDMA links leveraged for sensing (standard pilots, PRS/SRS/CSI-RS).
    - Single-carrier: SC links (e.g., SC-FDE uplink) used for sensing; simpler RF chain, longer coherence.
  - Network JCAS (Wi-Fi, Cellular, etc.)
    - Monostatic / dynamic: Tx and Rx co-located at the same node (e.g., gNB) and possibly moving.
    - Bistatic / dynamic: Tx and Rx at different nodes (e.g., one BS illuminates, another senses); nodes can move.
    - Centralized / Distributed: Fusion/processing at a central unit (CU) vs. edge/distributed fusion across many BSs/UEs.
      
- Representative research by category
- A. Realizing Communication in Primary Radar Systems
  - Embed comms into MIMO radar/LFM-CPM waveforms; carrier-frequency hopping; design of beamforming for radar that carries data.
- B. Realizing Sensing in Primary Communication Systems
  - Use IEEE 802.11/802.11ad/OFDM signals for radar-like sensing; estimate delay/Doppler/AoA via ESPRIT, periodogram, pulse-Doppler; partition subcarriers between C & S.
- C. Joint Design without an Underlying System
  - Tailored mmWave JCAS waveforms and beamforming; SIC for comms + pulse-Doppler for sensing; matched filtering + clutter suppression for mobile networks; MISO access tailored for JCAS.
- D. Co-existence of C&S Systems
  - Spectrum sharing between MU-MIMO comms and co-located MIMO radar; optimization for shared/segmented antenna deployments; generic coexistence algorithms.


## A. Major Differences between C&S Signals
- Radar signal basics
  - Two families: pulsed radar and continuous-wave (e.g., FMCW chirp) radar; used in SISO and MIMO (with orthogonal waveforms for MIMO).
  - Typically unmodulated (aside from training) and designed for low PAPR → efficient power amplifier & long range.
  - Key design goal: good ambiguity function (sharp/narrow mainlobe, low sidelobes) to resolve delay, Doppler, and angle with simple receivers.
  - Receivers:
    - Pulsed radar: sample echoes at high rate (≈ twice pulse bandwidth) or at a rate matching desired range resolution.
    - mix echo with local chirp; the beat frequency (Tx–Rx frequency difference) ∝ target range, enabling fine ranging with modest sampling rates.
- Communication signal basics
  - Built to maximize information rate → modulated, often with intermittent pilots/training inside packets/frames.
  - Can be fragmented across time & frequency (scheduling), often high-PAPR, and may use advanced modulations over time/frequency/space → complex signal structures.
- Using comm signals for sensing
  - Needs detailed channel composition (delays/angles/Dopplers of multipath), not just coarse channel coefficients as in standard channel estimation.
  - What limits/complicates it:
    - Hardware limits (front-end, sampling, dynamic range) and the complexity of comm waveforms → new sensing algorithms required (different from classic radar processing).
    - Practical constraints: lack of full-duplex, TX–RX asynchrony between nodes → must design new solutions.
    - Knowing the frame/signal structure (time/frequency resources, symbols) is critical for coherent detection.

| Specification             | Radar                                                                                                                                                                                                                          | Communications                                                                                                                                | JCAS / PMN (integrated C\&S)                                                                                                                                                                   |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Signal waveform**       | Usually **simple, unmodulated**, single-carrier; **pulsed** or **continuous-wave (e.g., FMCW/chirp)**. Orthogonality across time/freq/phase/code when MIMO is used. **Low PAPR** preferred. Emerging: OFDM, frequency hopping. | Mix of **pilots** (unmodulated) and **data** (modulated). **Complex** structures due to scheduling, multiuser MIMO/OFDM; often **high PAPR**. | Can leverage **both radar and comm** waveforms with tweaks so one signal serves **both**; goal is **joint performance optimization**.                                                          |
| **Transmit power**        | Often **high** for large-scale/long-range radar; **lower** for short-range (e.g., automotive FMCW).                                                                                                                            | Typically **low**, links up to a few km.                                                                                                      | If integrated into **radar**, can achieve **very long range**; if inside a **single comm device**, sensing is short-range, but **network-wide JCAS** covers large areas via cellular coverage. |
| **Bandwidth**             | Generally **large**; resolution ∝ bandwidth. (FMCW instantaneous BW may be narrow depending on propagation.)                                                                                                                   | Generally **smaller** than radar.                                                                                                             | **mmWave** bands are promising (large BW yet limited propagation). Not all sensing must rely on huge BW (cf. Wi-Fi sensing).                                                                   |
| **Signal band**           | X / S / C / Ku, etc.                                                                                                                                                                                                           | sub-6 GHz and **mmWave**.                                                                                                                     | Band choice affects **range** and **resolution** trade-offs.                                                                                                                                   |
| **Duplex capability**     | **Full-duplex** (continuous-wave) or **half-duplex** (pulse).                                                                                                                                                                  | Tx/Rx at same node usually **cannot** operate simultaneously; use **TDD** or **FDD**.                                                         | **Full-duplex is nice to have** for JCAS but **not mandatory**; various TDD/FDD schedules can work.                                                                                            |
| **Clock synchronization** | **Tx/Rx clock-locked**.                                                                                                                                                                                                        | Co-located Tx/Rx share one clock; across nodes, **not synchronized**.                                                                         | **Tight sync removes ambiguity** and helps coherent sensing, but **not essential** for some sensing modes (non-coherent, RTT-style, etc.).                                                     |

## B. Radar-Centric Design: Realizing Communication in Primary Radar Systems
- Baseline waveforms & info embedding
- Traditional bases: pulsed radar and continuous-wave (CW/FMCW/chirp) radar.
- A core challenge is how to embed data without hurting radar. Examples:
  - Random step-frequency (RSF): use the radar carrier/frequency steps to carry comm info.
  - LFM-CPM (quasi-orthogonal) from a MIMO radar: supports multi-user comm alongside sensing.
  - Newer radar waveforms closer to comms: MIMO-OFDM radar, frequency-agile (hopping) radar.
- Index modulation (IM) for radar-centric JCAS
  - Encode bits by which resource is active (indexes of subcarriers, antenna ports, time/frequency/code permutations).
  - Big advantage: IM doesn’t change the radar’s basic waveform/processing → negligible impact on sensing performance.
- What limits the comm side
- Throughput is modest, constrained by the radar waveform/design (range resolution, PRI, duty cycle, etc.).

## C. Communication-Centric Design: Realizing Sensing in Primary Communication Systems
- This is the JCAS category that PMN belongs to: keep a communication system primary, and add sensing on top.
- Topologies & analogies to radar
  - Point-to-point JCAS: often for vehicular links (e.g., Wi-Fi/802.11, V2V).
  - Network JCAS: large-scale (cellular, Wi-Fi networks).
  - Spatial layout vs. radar:
    - Monostatic: Tx and sensing Rx co-located.
    - Bistatic / Multistatic: Tx and Rx separated (two or many sites).
   
- Two fundamental integration problems
- Full-duplex in monostatic setups
- Clock asynchronization in bi/multi-static setups

- Prior art & example approaches
- Vehicular / 802.11 sensing:
  - Embed active radar functions in OFDM-based comms; estimate velocity, delay, Doppler via FFT, ESPRIT, pulse-Doppler, etc.
  - SC (single-carrier) PHY in 802.11p for automotive sensing; 802.11ad mmWave for V2V JCAS.
  - Brute-force designs that maximize mean-normalized channel energy for ranging.
 
- Mobile-network JCAS (cellular):
- Early OFDM-based sensing proofs.
- Sparse array / beam pattern optimization for MIMO JCAS.
- Power/subcarrier allocation via (weighted) mutual information to balance C&S.
- Waveform optimization to approximate a desired sensing beampattern/AF.
- Multiple-access performance bounds for multi-antenna JCAS.
- Interleaved multicarrier waveforms assigning different subcarriers (or subsets) to comm vs. sensing.

- D. Joint Design Without an Underlying System
- No legacy constraints. You can pick symbol timing, frame structure, pilots, duplexing, beams, and hardware to jointly optimize rate, latency, range/Doppler/angle resolution, and robustness.
- mmWave & sub-THz fit well.
- Pros: huge bandwidths → fine range resolution; short wavelengths → narrow beams & high angular resolution; potentially very high data rates.
- Cons: technology still maturing (RF front-end, calibration, blockage), limited deployment to date; sub-THz lacks mature standards.
- What the literature explores
  - Waveform co-design at mmWave: feasibility in indoor / vehicular networks; in-depth signal-processing reviews (esp. waveform design for JCAS).
  - Hybrid / adaptive beamforming: joint Tx/Rx beam designs to balance SNR for comms and sensing; adaptive structures that reconfigure by task.
  - Automotive JCAS: comparisons of phase-modulated continuous-wave vs OFDM-based JCAS by analyzing system models and design parameters.
  - Multibeam transmission: a single joint signal forms multiple beams to cover different sensing directions and carry data.
  - Terahertz massive-MIMO JCAS: beamforming methods targeting extreme resolution and high data rates.
 
- Multi-channel JCAS (key special case)
  - Concept: Use one or more channels at a time, but over a session occupy many channels
  - Benefits: Achieve large overall (synthetic) bandwidth for sensing without large instantaneous comm bandwidth → lower RF cost and better alignment with comm spectrum use.
  - Main challenge: Each channel’s receive chain introduces imperfections/distortions (gain/phase offsets, CFO, timing skew). You must calibrate/compensate and concatenate channels coherently.
  - JCAS twist: Design the waveform & schedule to make concatenation easier (shared pilots, inter-channel guards, known hop patterns) while keeping comm performance high.
 
- E. Advantages of JCAS Systems
  - Spectral Efficiency
  - Sharing the same spectrum for both communication and sensing can, in principle, double spectral efficiency versus running them separately.
  - Beamforming Efficiency
  - Sensing reveals real-time channel/scene structure (angles, paths, motion). Comms can exploit this for fast beam adaptation, direction optimization, and more reliable links.
 
# III FRAMEWORK FOR A PMN
- PMN can evolve from today’s mobile networks with added/modified hardware, system features, and algorithms—no full redesign required.
- PMN evolution is not tied to a specific cellular standard. The discussion is generalized around modern mobile ingredients: antenna arrays, broadband, MU-MIMO, and OFDMA. (5G NR is referenced only when helpful.)
- Deployment topologies for JCAS in PMN
  - Cloud-RAN (CRAN): centralized baseband with distributed radio units → natural place for cooperative sensing and data fusion.
  - Standalone BS: each site performs local comms + sensing → simpler deployment, less fronthaul dependence.

## A. System Platform and Infrastructure
### CRAN(Cloud-RAN):
- A central unit (CU) + many distributed remote radio units (RRUs) (a.k.a. radio remote units).
- RRUs connect to the CU over fiber; either quantized RF or baseband (I/Q) samples are transported.
- How PMN/JCAS runs on CRAN
  - In a PMN, densely deployed RRUs are coordinated by the CU to deliver comms and sensing.
  - The CU hosts the traditional BBU pool (communication processing) and a new sensing processor (for fusion/estimation).
  - This mirrors distributed radar architectures.
 
- Typical comms scenario
  - Multiple RRUs serve UEs cooperatively using MU-MIMO on the same RBs (same time/frequency).
  - Even without explicit cooperation, one can still support sensing by raising DL power so other RRUs can receive the signal; this doesn’t harm MU-MIMO (no downlink inter-RRU interference).
- RRUs are typically GPS-clock synchronized, giving an excellent timing reference for distributed sensing algorithms.

### Standalone BS:
- You don’t need CRAN to do PMN sensing. A single (standalone) base station can sense using the signals it transmits itself (DL/monostatic) or signals from UEs (UL/bistatic).
- Pros: simple, low latency (edge), no centralized sync required, good for targeted coverage.
- Cons: less multi-site cooperation/fusion than CRAN; coverage and diversity depend on one site unless multiple standalone BSs are coordinated.

## B. Three Types of Sensing Operations
### Downlink Active Sensing:
- An RRU/BS senses using its own downlink (DL) communication signal by listening to the echoes (reflections/diffractions). Tx and Rx are co-located → essentially mono-static radar.
- Full-duplex (FD) (transmit and receive at the same time) or an equivalent FD solution (e.g., strong analog - digital self-interference (SI) cancellation).

### Downlink Passive Sensing
- An RRU/BS uses the downlink (DL) signals transmitted by other RRUs to sense. Tx and Rx are spatially separated → a bi-static / multi-static radar setup (their clocks may be synchronized). It mainly senses the environment between RRUs.
- At a given RRU, you receive (i) active echoes from its own DL signal and (ii) passive echoes from other RRUs’ DL signals. Passive echoes usually arrive slightly later (longer path). With SDMA (multi-UE beams), time/frequency alone may not separate them → algorithms must handle mixtures.
- Angles & Doppler
  - <img width="611" height="94" alt="image" src="https://github.com/user-attachments/assets/16eb6381-33d6-4d25-a21c-cddaa55c6708" />

### Uplink Sensing
- The BS/ gNB estimates the environment using uplink communication signals transmitted by UEs.
- Architecturally similar to passive sensing: transmitter (UE) and receiver (BS) are spatially separated and not clock-synchronized.
- Key difference from passive sensing
  - In UL sensing the BS fully knows the protocol and signal structure of the received UL waveforms (pilots, frame timing, coding, etc.), which simplifies processing compared with generic passive radar.
- Can be deployed directly on today’s networks: no hardware change, no network topology change, and no full-duplex requirement at the BS.
- Because UE and BS clocks are not locked, the BS estimates relative (not absolute) time delay and Doppler,so Ambiguities from unsynchronized clocks must be handled with special techniques.

### Downlink vs. Uplink Sensing — comparison
- DL sensing can often achieve higher accuracy than UL sensing because:
  - RRUs/BSs have more antennas and higher transmit power.
  - The entire transmitted signal is centrally known and controlled.
- Deployability:
  - UL sensing is easier to deploy (no FD and no extra hardware).
  - DL sensing may require full-duplex or equivalent self-interference suppression at the sensing node.
 
## Signals Usable from 5G NR for Radio Sensing
### Reference Signals Used for Channel Estimation:
- DMRS (UL/PUSCH & DL/PDSCH), SRS (UL), CSI-RS (DL). Most are comb-type pilots placed across OFDM symbols and subcarriers, and orthogonal across users.
- DMRS with data payload: DMRS that accompanies data on PDSCH/PUSCH is user-specific and its time–frequency pattern is irregular (because it depends on scheduling, layers/ports, mapping with data). This demands sensing algorithms that can handle irregular pilots.
- SRS & CSI-RS: Can be periodic or aperiodic, thus easier to use for structured sensing (e.g., subspace/spectrum methods like ESPRIT for angle/delay).
- Positions known & tunable: BS knows where the DMRS OFDM symbols are; their number/positions can be adjusted across slots and subcarriers. Jointly optimizing comms & sensing means designing pilot density/placement.
- Grid anatomy: One resource element (RE) = 1 subcarrier × 1 OFDM symbol. One resource block (RB) = 12 consecutive subcarriers in frequency; slot usually has 14 OFDM symbols. A single NR carrier supports up to ~3300 active subcarriers
- Why it matters: The number and pattern of subcarriers occupied by DMRS directly affect sensing resolution & ambiguity (e.g., finer comb → better delay/angle resolution; poor geometry → higher ambiguity).
- Sequences: DMRS generated from Gold sequences (per TS 38.211) for both PDSCH and PUSCH.
- DL vs UL mapping:
  - PDSCH (downlink) DMRS is typically interleaved across subcarriers → good for comms, but can introduce sensing ambiguity.
  - PUSCH (uplink) DMRS often uses grouped / non-interleaved subcarriers → cleaner structure for sensing.
- Both UL & DL DMRS are feasible for sensing (simulations show excellent potential),

### Non-Channel Estimation Signals
- Deterministic, non-channel-estimation signals such as SS and PBCH/SSB can also be used for sensing.
- They appear with regular periodicity (interval of several to tens of milliseconds) and occupy only a limited number of subcarriers, which limits multipath-delay identification.
- Transmission intuition: The SSB bundle (PSS, SSS, PBCH + DMRS) is mapped to fixed time–frequency REs per NR pattern and can be beam-swept over multiple directions; this makes it predictable (good for synchronization/initial acquisition) but sparse for fine delay profiling.

### Data Payload Signals
- Data payload on PDSCH (DL) and PUSCH (UL) can be exploited as additional sensing illumination.
- DL: the network knows the bits and modulation → can treat symbols as known for coherent processing.
- UL: symbols are unknown → need decision-directed reconstruction; demodulation errors degrade sensing.
- Multi-layer MIMO streams are generally non-orthogonal, which adds interference for sensing.
- Pros: greatly increases usable REs → better SNR, resolution, and statistics; precoders can be jointly optimized for C&S.
- Cons: higher implementation/compute complexity; UL is sensitive to sync and detection errors.

# IV. EVOLUTION: SYSTEM MODIFICATIONS TO ENABLE SENSING
- What C&S can share (in a MIMO-OFDM transceiver)
  - Shared signal chain: the whole transmitter and many receiver modules can be common to both Communication & Sensing.
  - Waveform co-design: the transmitted waveform can be jointly optimized to satisfy requirements of C&S simultaneously.
  - Where estimation happens: sensing parameter estimation can be done in time or frequency domain.
  - What the sensing app may output: either numerical parameters (delay, Doppler, AoA/AoD, etc.) and/or pattern-recognition results (e.g., detection/classification).
 
- Why today’s mobile network still needs modifications
  - No full-duplex today → a node cannot transmit & receive at the same frequency at the same time ⇒ mono-static radar sensing is infeasible without HW changes.
  - No clock synch between separated Tx/Rx (e.g., two different nodes) → only relative timing/Doppler; ranging ambiguity and cross-packet processing becomes hard.
  - Therefore, classic bi-static radar methods cannot be directly used on current comms infrastructure.
 
## A. Dedicated Transmitter for Uplink Sensing
- Baseline (conventional UL sensing): Works much like passive sensing; BS/RRU receivers—already in the network—process UL comm signals for both comms & sensing.
- Main issue: UE–BS clocks are not synchronized → only relative delay/Doppler can be measured → ranging ambiguity.
  - If this ambiguity is acceptable, no HW/system changes are needed.
  - In special cases it can be mitigated by signal processing (see Sec. VI-F)
- Ambiguity-free option: deploy a dedicated (static) UE that is clock-synchronized to the BSs and transmits UL sensing signals.
  - Enables SIMO sensing (one Tx, many spatially separated BS/RRUs as Rx) with joint/collaborative processing across sites.

## B. Using Full-Duplex Radios for Downlink Sensing
- What problem it solves
  - In DL sensing, a base station must hear very weak echoes while it is transmitting. Its own transmit signal leaks into the receiver and can drown out those echoes.
- How full-duplex (FD) helps
  - FD lets a node transmit and receive on the same channel at the same time, then cancel the self-interference (the leaked TX signal) using a mix of:
  - antenna isolation/separation,
  - RF analog suppression,
  - digital/baseband cancellation.
- For JCAS, FD is a bit easier than for pure communications: we mostly need to suppress our own leaked TX while keeping the environmental echoes; we don’t have to manage simultaneous signals from two comm nodes.
- Still hard in practice, especially for MIMO: many TX-RX antenna pairs create many leakage paths that must be canceled simultaneously.
- Mobility/dynamics make cancellation tougher.

## C. Dedicated Receiver for Downlink (and Uplink) Sensing
- To avoid needing full-duplex, a near-term path is to deploy a receive-only BS/node. It can be set to (a) downlink-only sensing, or (b) both communications and downlink sensing.
- Implementation notes
  - Why changes are needed: Today’s BS receivers are built mainly to receive uplink signals; downlink sensing needs them to receive downlink signals.
  - TDD is easier: In TDD, the transceiver already uses a TX/RX switch. Downlink sensing mainly requires scheduling the switch so the antennas connect to RX during the needed period → minimal hardware changes.
  - FDD is harder/costlier: The BS RX path may not cover downlink bands, so hardware mods are likely. Hence DL sensing is typically more cost-effective in TDD than FDD.
- Deploy a dedicated receiving-only node that can do both DL and UL sensing (and even comms). This fits TDD well because DL and UL can be time-separated at the receiver. To remove delay/Doppler ambiguity you need clock synchronization between transmit and receive nodes.


## D. BS with Spatially Widely Separated Transmitting and Receiving Antennas
- Use a well-separated transmit (Tx) and receive (Rx) antenna for downlink sensing to greatly reduce direct-leakage from the transmitter. Baseband self-interference cancellation can still be applied using a feedback path from Tx baseband to Rx baseband.
- TDD example (Fig. 8).
- <img width="609" height="480" alt="image" src="https://github.com/user-attachments/assets/c1e77cf0-049a-4c15-8047-a9ee54bc285f" />
- (a) General concept
  - The “Normal Tx/Rx” block does regular TDD (TX in DL slot, RX in UL slot).
  - A separate sensing-only receiver is fed by its own antenna/port so it can keep receiving even when the comm side is transmitting.
- (b) A practical wiring on existing hardware
  - SPDT1–SPDT4 are RF switches that steer an antenna feed to either TX ports (Tx1–Tx4) during DL or to comm RX ports (Rx1–Rx4 (C)) during UL.
  - Rx5 (S) is a dedicated sensing RX; the purple line shows a permanent path to the sensing chain.
 
## E. Summary and Insights
- Full-duplex radios would be the ideal long-term solution for seamless downlink sensing + communications, but are impractical today.
- The other three options are near-term, sub-optimal choices that require only minor hardware/system tweaks to current networks. Among them, the single spatially separated receive antenna dedicated to sensing in a conventional MIMO base station appears to be the most cost-effective approach for downlink sensing.
- Beyond hardware changes, hardware calibration is also important. Experiments show that receiver and array imperfections can harm high-resolution sensing-parameter estimation in channel sounding, while antenna-array calibration can effectively mitigate these impacts.
- <img width="1297" height="344" alt="image" src="https://github.com/user-attachments/assets/adf3fa15-f2db-4e7f-a651-f2bb212dfc37" />


# V. MAJOR RESEARCH CHALLENGES FOR PMN
## A. Sensing Parameter Extraction from Sophisticated Mobile Signals
- Sensing needs continuous parameters (delay, Doppler, AoA/AoD), while comms channel estimation often returns grid-quantized composite coefficients or only LOS info.
- Why it’s hard
  - Irregular, user-dependent allocations → incomplete / nonuniform measurements.
  - Multi-user, non-orthogonal spatial layers → interference and mixing.
  - Comms estimators target link reliability, not physical parameter identifiability.
- Parametric channel models + gridless sparse / super-resolution estimation (e.g., atomic norm/iterative ML/peak tracking) to recover continuous delay/Doppler/angles.
- Leverage DMRS / CSI-RS / SRS for joint time-freq-space estimation; add cross-slot phase alignment.

## B. Joint Design and Optimization
- Why it’s hard: C & S want different things.
  - In multiuser MIMO communications, the transmit signal is a mix of users’ random, modulated symbols; beamforming maximizes gain/directivity.
  - In ideal MIMO radar, sensing signals are preferably unmodulated and orthogonal; arrays try to form virtual sub-arrays to enlarge aperture and angular resolution.
- Open directions: Beyond standalone waveform tuning, study joint optimization at system/network levels (e.g., coordinated beam/scheduling, resource sharing), and quantify mutual benefits (so far mostly studied for propagation-path optimization and secure communications).

## C. Networked Sensing
- Embedding sensing into cellular topology promises big gains—high frequency reuse, wide coverage, distributed nodes, and hence larger “sensing capacity.”
- Algorithms that exploit cellular structure: inter-cell interference handling, BS cooperation, multi/static sensing, and sensing handover across BSs.
- Scheduling & resource co-design between C & S; synchronization and data fusion across many BSs; defining metrics for networked sensing capacity.
- Goal: Develop both theory (bounds/metrics) and practical algorithms that make distributed, multi-BS sensing reliable and scalable.

