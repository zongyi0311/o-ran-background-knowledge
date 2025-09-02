<img width="1920" height="811" alt="image" src="https://github.com/user-attachments/assets/0b2e95b0-a6be-4ec1-aecb-f4a46bc39613" />

```
ESC[0mESC[34m229470.015337 [PHY] D {L1_tx_thread} PDSCH generation started (1) in frame 832.0
ESC[0mESC[34m229470.015337 [PHY] D {Tpool0_-1} aid 0, frame 831 slot 18, startSymbol 0, endSymbol 6
```
- In the log, will see two types of slots:
  - Scheduler slot: For example, frame 831 slot 18, This is where the gNB MAC layer runs the scheduler: allocating PRBs, MCS, and time resources.
  - Transmit slot: For example, frame 832 slot 0, This is where the gNB PHY actually generates the OFDM waveform.
 
- Why are the slot numbers different?
  - Because scheduling and sending involve processing delays:
  - The gNB decides what to transmit in slot N (scheduling) and then actually sends it in slot N+k (transmitting)
  - k = 2, 4, or 6, depending on min_rxtxtime, numerology, and configuration.
 
- **pdsch: BWPStart 27, BWPSize 48, rbStart 0, rbsize 16**
  - BWPStart 27 The Bandwidth Part (BWP) starts at PRB index 27 in the carrier grid.
  - BWPSize 48 The BWP has a total size of 48 PRBs.
  - rbStart 0, rbsize 16
    - rbStart 0: Resource allocation starts at PRB 0 inside this BWP.
    - rbsize 16: The allocated PDSCH size is 16 PRBs.
  - Mapping to the full carrier grid: The UE is using PRB indices 27 to 42 (27 + 0 → start, length 16).
  - this log shows the frequency-domain allocation of PDSCH, meaning the UE is given 16 PRBs at indices 27–42 to transmit/receive data.
- In other words: 48 is the upper limit of UE bandwidth, 27 is the starting point, and DCI only selects a part of these 48 PRBs for you to use each time.

- **nr segmentation B 824 Bprime 824 Kprime 824 Z 83**
- B is the original code block length (in bits), i.e., the Transport Block size including CRC bits.
- B′ is the padded length. If B is not aligned with LDPC requirements (multiple of 8 or Zc), padding (filler bits) is added.
- K′ is the length of one Code Block (CB). If the TB is too large (greater than Kcb, e.g., 8448 bits), it must be segmented into multiple CBs.
- z (Zc) is the LDPC lifting size, determined by the Base Graph (BG1/BG2) and TBS. It scales the LDPC base matrix.

- **final nr seg output Z 88 K 880 F 56**
- Original input (824 bits)
  - The 824 is the actual number of bits to encode (including CRC).
  - Before segmentation, this represents the payload length.
- Z = 83 (calculated requirement)
  - The LDPC base graph (BG1/BG2) is expanded using a lifting size Z.
  - From 824 bits, OAI computes that the minimum required Z is about 83.
  - But 83 is not a valid value according to 3GPP TS 38.212.
- Z = 88 (chosen from standard)
  - Since 83 is not in the list, the system chooses the next higher allowed Z = 88.
- K changes from 824 → 880
  - The code block length K = Z × (rows of base graph).
  - With Z=88, K=880.
  - Actual data = 824 → requires F = 56 filler bits.
  - These filler bits are set to zero and ignored during decoding.

- **C 1, K 880, Bprime_bytes 103, Bprime 824, F 56**
  - C 1
  → After segmentation, there is 1 code block.

  → Since the input (824 bits) is small, no need for multiple blocks.

  - K 880
  → Final code block length, derived from the chosen lifting size (Z=88).

  → Original = 824 bits, but must align to 880 → filler added.

  - Bprime_bytes 103
  → Equivalent of B′ in bytes (103 bytes ≈ 824 bits).

  → Matches the original payload + CRC length.

  - Bprime 824
  → Actual data length (in bits) before filler.

  → Directly from the payload size.

  - F 56
  → Number of filler bits = K – B′ = 880 – 824 = 56.

  → Filled with zeros during encoding, ignored during decoding.

- **nr_get_E : (G 2688, C 1, Qm 2, Nl 1, r 0), E 2688**
  - G 2688
    → The total number of bits that can be mapped to the allocated PRBs for this TB = 2688.
    
    → Derived from PRB × symbols × modulation order × layers.
    
  - C 1
    → Number of code blocks after segmentation = 1.
    
    → We saw earlier that only one block was generated.
    
  - Qm 2
    → Modulation order = 2 (QPSK).
    
    → Each symbol carries 2 bits.
    
  - Nl 1
    → Number of transmission layers = 1.
    
    → Single-layer MIMO transmission.
    
  - r 0
    → Redundancy version (RV) = 0.
    
    → First transmission (not a retransmission).
    
    → HARQ retransmissions may use RV=2,3, etc.
    
  - E 2688
    → The number of bits after rate matching for this code block.
    
    → Since C=1, E = G = 2688.
    
    → With multiple code blocks, G is divided, and each block has a smaller E.


- **[PHY] D {ru_thread} RU proc: frame_rx = 831, tti_rx = 18**
- frame_rx = 831
→ The RU is currently processing frame number 831.

→ In NR, one frame = 10 ms, consisting of 10 subframes (1 ms each).

- tti_rx = 18
→ The RU is processing time transmission interval (TTI) = slot index.

→ Here, tti_rx = 18 means slot 18 of frame 831.

→ With numerology μ=1 (30 kHz SCS), each subframe has 2 slots, so each frame has 20 slots.

→ Thus, this log shows RU is processing slot 18 of frame 831.

```
searching for PRACH in 831.18 : prach_index 0 => 831.19
searching for PRACH in 831.18 : prach_index 1 => -1.-1
searching for PRACH in 831.18 : prach_index 2 => -1.-1
searching for PRACH in 831.18 : prach_index 3 => -1.-1
```
- At frame 831, slot 18, the gNB/RU checks for PRACH opportunities.
- prach_index 0 => 831.19: means the next valid PRACH opportunity is scheduled at frame 831, slot 19.
- prach_index 1/2/3 => -1.-1: means other possible PRACH candidates are invalid under the current PRACH configuration.
- This is controlled by the prach-ConfigIndex (RRC signaling).
  - gNB knows exactly which slots/subframes PRACH can appear.
  - UE will only transmit PRACH in those slots.
  - gNB only searches at those valid PRACH occasions.

- **nr_modulation: length 2688, mod_order 2**
  - length = 2688 → number of bits to be modulated
  - mod_order = 2 → QPSK modulation (2 bits per symbol)
  - These bits will be mapped into QPSK symbols

- **[RU_thread] read data: frame_rx = 831, tti_rx = 19**
  - RU thread is reading received data for frame 831, slot 19.
- **Reading 30720 samples for slot 19 (0x78de5c7da040)**
  - Reads 30720 IQ samples for slot 19 (address pointer shown).

---

- <img width="1219" height="162" alt="image" src="https://github.com/user-attachments/assets/1ba3f896-e0c5-4cf0-9397-c382bd8547ac" />
- The RU thread is currently in frame 947, slot 19, and has finished processing the fronthaul input. The pipeline is also preparing for the next TX (frame 948.5).
- The thread pool starts allocating DLSCH encoding for the UE (aid=0) in the two symbol intervals (0–6, 7–13) of slot 19.
- Coding details: 808-bit TB → 880-bit CB → rate-match to 2688 bits, using 16 RB, QPSK, HARQ rv=0.

- <img width="1219" height="162" alt="image" src="https://github.com/user-attachments/assets/626bd6dc-c1e9-4545-a008-ebf330c5ced3" />
- PDSCH allocation inside a 48-RB BWP starting at RB=27. This allocation uses 16 RBs, starting at offset 0.
- 2688 bits are modulated with QPSK (mod_order=2), producing 1344 QPSK symbols.
- Maps the 1344 symbols to 1 layer (Nl=1). Count matches (16 RB × 12 RE × 7 symbols = 1344).
- RU thread is processing frame 947, slot 19 reception.
- In frame 947, slot 19, RU scans PRACH occasions:
  - index 0 → 573.19: First occasion mapped to frame 573, slot 19 (due to PRACH periodicity).
  - index 1 → 947.19: Another occasion in the current slot.

- <img width="1348" height="215" alt="image" src="https://github.com/user-attachments/assets/5bb44c5d-6f66-4347-a571-f68f23932fe3" />
- Generate UE-specific DMRS in slot 0, symbol 2. nscid=0 is the sequence ID; x2 is the Gold sequence state.
- EN: RU is detecting PRACH in frame 947, slot 19:
  - format 5 maps to PRACH Format A2 in OAI.
  - numRA 0 = first RA occasion in this slot.
  - prachStartSymbol 0 = starts at symbol 0.
  - prachOccasion 0 = occasion index 0.
- TX (l1_tx_thread):
  - Generates UE DMRS for slot 0 (symbols 2, 6, and 9).
  - Also, splits the PDSCH into three segments of 1032 bits each, using QPSK modulation.
 
- RX (ru_thread):
  - Performs PRACH detection in frame 947, slot 19 (Format A2, startSymbol 0, occasion 0).
  - The log also displays the frequencyStart and startSymbol values ​​for this PRACH configuration.

- <img width="1341" height="519" alt="image" src="https://github.com/user-attachments/assets/c1110838-c373-4859-b4e7-fc7c7591508b" />
  - [l1_tx_thread] Rotating symbol N, slot 0, symbol_subframe_index N (±32767,0/−1)
  - This is the TX applying phase rotation to symbols 0..13 of slot 0.
  - (±32767,0/−1) are Q15 fixed-point values ≈ ±1+j·0.
  - Reason: For 15 kHz SCS with 7.5 kHz half-subcarrier shift, each OFDM symbol alternates by ±1.
- [ru_thread] Doing PRACH combining of 4 repetitions N_ZC 139
  - Format A2 (internally labeled as format 5).
  - With 4 repetitions.
  - Zadoff–Chu root = 139.
    RU combines the 4 repetitions to improve detection SNR.

- [ru_thread] frame 947, slot 19: doing rx_nr_prach_ru for format 5, numRA 0, prachStartSymbol 4, prachOccasion 1
- [ru_thread] PRACH (ru 0) in 947.19, format A2, msg1_frequencyStart 0, startSymbol 4
  - Detecting PRACH in frame 947, slot 19.
  - prachStartSymbol 4 = starts at symbol 4.
  - prachOccasion 1 = the 2nd occasion in this slot (the first was startSymbol 0).
  - msg1_frequencyStart 0 = frequency domain starts at RB 0.
 
- Transmitter (l1_tx_thread):
- In frame 948, slot 0 prepares for downlink PDSCH:TB segmentation → LDPC encoding → rate matching → QPSK modulation → layer mapping → DMRS insertion → RE segment modulation.

- Receiver (ru_thread):
- At the same time, uplink PRACH detection is performed in frame 947, slot 19: Scanning multiple occasions → Detecting the preamble (Format A2, N_ZC=139, repeated 4 times in the time domain).


- <img width="1773" height="834" alt="image" src="https://github.com/user-attachments/assets/db6db428-c695-43e8-9834-bf065c51f0be" />
TX (gNB):

Slot 0 symbols are rotated and compensated one by one.

PDSCH/PDCCH coding, modulation, and RE mapping.

The calling RU sends 30720 IQ samples.

RX (RU):

Simultaneously, a PRACH occasion (0, 4, 8) is detected in frame 947, slot 19.

Each preamble is repeated four times, and energy combining is performed.

IQ samples are read from the buffer and processed.

Thread pool:

Background data for slot 0 is generated, and beamforming for slot 1 is prepared.

-<img width="1788" height="576" alt="image" src="https://github.com/user-attachments/assets/32bd3dee-c939-4d9f-bbba-e32cc7025cab" />
- [l1_tx_thread] Rotating symbol 0..13, slot 1, symbol_subframe_index 14..27 (±32767,…)
  - TX side processes frame 948, slot 1 symbol rotation.
  - symbol_subframe_index 14..27 = continuous index across the frame (slot 0 = 0..13, slot 1 = 14..27).
  - (±32767,0/-1) = Q15 fixed-point ±1, implementing 7.5 kHz half-subcarrier shift compensation.

- [ru_thread] RU 0/0 TS 582481920, GPS 9.480500, SR 61440000.000000, frame 948, slot 1.6 / 20
  - RU reports status

- [l1_tx_thread] gNB: 948.1 : calling RU TX function
  - gNB calls RU to transmit frame 948, slot 1.

- [ru_thread] AFTER fh_south_in - SFN/SL:948.1 RU->proc[RX:948.1 TX:948.7] RC.gNB[0]:[RX:00 TX(SFN):0]
  - fh_south_in finished.
  - RU is receiving 948.1 and preparing for TX at 948.7.
  - OAI pipeline runs parallel RX/TX: RU receives current slot, prepares TX for future slots.

- [ru_thread] read data: frame_rx = 948, tti_rx = 2
- [ru_thread] Reading 30720 samples for slot 2
  - RU reads IQ samples for frame 948, slot 2.
  - Each slot = 30720 samples (61.44 Msps).
 
- [Tpool_-1] SFN/SF:RU:TX:948/7 aa 0 Generating slot 1 (first_symbol 0 num_symbols 7) slot_offset ...
  - Thread pool is generating slot 1 data.
  - first_symbol 0 num_symbols 7 → generates first half of slot 1 (symbols 0–6).
  - Next line = first_symbol 7 num_symbols 7 → generates second half (symbols 7–13).
 
- [l1_tx_thread] [TXPATH] RU 0 tx_rf, writing to TS 582481920, 948.1, unwrapped_frame 0, slot 1, flags 1, siglen+sf_extension 30720, returned 30720, E -inf
- gNB writes frame 948, slot 1 IQ samples (30720) into RU TX RF.
- E -inf = energy estimation (−∞ means not calculated in this log level).

---
- <img width="1764" height="709" alt="image" src="https://github.com/user-attachments/assets/9d756a1f-e374-4306-9253-3ef44d633150" />
- **PRACH (ru 0) in 747.19, format A2, msg1_frequencyStart 0, startSymbol 0**  RU 0 PRACH detected: frame 747 slot 19, format A2, freqStart=0, startSymbol=0.
- **frame 747, slot 19: doing rx_nr_prach_ru for format 5, numRA 0, prachStartSymbol 4, prachOccasion 1**  RU receiving PRACH format 5, RA index=0, startSymbol=4, occasion=1.
- prachStartSymbol 0, prachOccasion 0, prachStartSymbol 4, prachOccasion 1, prachStartSymbol 8, prachOccasion 2
- In 5G NR, the PRACH (Physical Random Access Channel) transmission positions are determined by the time-domain configuration.
- A single slot can contain multiple PRACH occasions.
  - Each PRACH occasion is identified by its start symbol (prachStartSymbol).
  - For example, in a Format A2 configuration, the typical PRACH occasions within a slot are:
  - Occasion 0 → prachStartSymbol = 0
  - Occasion 1 → prachStartSymbol = 4
  - Occasion 2 → prachStartSymbol = 8
 
- Why this design exists:
  - It allows multiple UEs to attempt random access within the same slot, but separated in different OFDM symbols, reducing collision probability.
  - The gNB must monitor all PRACH occasions in the slot, regardless of which one the UE chooses.

- <img width="1795" height="239" alt="image" src="https://github.com/user-attachments/assets/8e1623d3-bcfd-40ae-8ac8-06ad4836b0df" />
- <img width="1864" height="100" alt="image" src="https://github.com/user-attachments/assets/993c4e29-7951-4813-8219-d5e096e719c1" />
- PUCCH Configuration
  - nr_symbols 1 → Uses 1 OFDM symbol.
  - start_symbol 13 → Located at symbol 13 (last symbol in slot).
  - prb_start 8 → Allocated at PRB index 8.
  - second_hop_prb 0 → No frequency hopping.
  - group_hop_flag 0, sequence_hop_flag 0 → No group/sequence hopping.
  - O_ACK 0 → No HARQ-ACK bits.
  - O_SR 1 → 1-bit Scheduling Request (SR) is being sent.
  - mcs 0 → Not relevant here.
  - initial_cyclic_shift 0 → Cyclic shift index 0.
  - This confirms it’s PUCCH Format 0 carrying a Scheduling Request.

- PUCCH Sequence Parameters
  - u = 10, v = 0 → Root sequence index u=10, cyclic shift v=0.
  - Used for Zadoff-Chu (ZC) sequence generation.
 
- PUCCH Demodulation
  - PUCCH IDFT[0/9] = (0,0) => -inf → The IDFT-based detection returned -infinity, meaning no valid energy detected.
  - Indicates either no UE transmission or extremely low SNR.
 
- Energy Statistics
  - n00[8] = 0, n01[8] = 0 → Energy bins show zero values.
  - This confirms that PUCCH wasn’t successfully received.
 
```
PRACH RX preamble_index 0, preamble_offset 0
PRACH RX new dft preamble_offset-first_nonzero_root_idx 0
PRACH RX preamble_index 1, preamble_offset 0
PRACH RX preamble_index 2, preamble_offset 0
PRACH RX preamble_index 3, preamble_offset 0
```
**PRACH RX preamble_index N**

This shows that the gNB, during PRACH detection (RX), is checking preamble index N (UE chooses one index during random access).

Here, it cycles from index 0 to index 3.

**preamble_offset**

This indicates the timing/frequency offset of the detected preamble.

All are 0 here, which means either the preamble is perfectly aligned or no timing advance is detected.

**new dft preamble_offset-first_nonzero_root_idx**

PRACH sequences are Zadoff-Chu sequences. The gNB applies DFT detection to check correlation with configured root sequences.

**The log shows first_nonzero_root_idx = 0, meaning the first ZC root sequence being tested produced a valid detection.**

This log indicates the PRACH decoding process at the gNB:

It scans through preamble indices 0–3,

Runs correlation with the Zadoff-Chu roots,

Finds signals aligned with zero offset.
