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
 
- <img width="1267" height="450" alt="image" src="https://github.com/user-attachments/assets/a96ea293-80a7-4d3b-af60-0c10010ce500" />













