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
  - 
