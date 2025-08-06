# nr_generate_pdsch() /PDSCH generation process
```
╔════════════════════════════════════════════════════════╗
║         nr_generate_pdsch(msgTx, frame, slot)         ║
╚════════════════════════════════════════════════════════╝
                         │
                         ▼
╔════════════════════════════════════════════════════════╗
║ Initialization: set frame parameters, timers, etc.     ║
╚════════════════════════════════════════════════════════╝
                         │
                         ▼
╔════════════════════════════════════════════════════════╗
║ Loop over each DLSCH (for each dlsch_id):              ║
║ ├─ Fetch HARQ structure and rel15 PDU                  ║
║ ├─ Parse Qm, RB allocation, and layer count            ║
║ └─ If PTRS is enabled: calculate unavailable resources ║
╚════════════════════════════════════════════════════════╝
                         │
                         ▼
╔════════════════════════════════════════════════════════╗
║ Compute total output buffer size (aligned to 64 bytes)║
╚════════════════════════════════════════════════════════╝
                         │
                         ▼
╔════════════════════════════════════════════════════════╗
║ Allocate and zero the output buffer (64-byte aligned) ║
╚════════════════════════════════════════════════════════╝
                         │
                         ▼
╔════════════════════════════════════════════════════════╗
║ Call nr_dlsch_encoding()                               ║
║ ├─ Append CRC                                          ║
║ ├─ Perform Code Block Segmentation                     ║
║ ├─ LDPC Encoding                                       ║
║ ├─ Rate Matching                                       ║
║ ├─ Bit Interleaving                                    ║
║ └─ Write encoded bits into output buffer               ║
╚════════════════════════════════════════════════════════╝
                         │
                         ▼
╔════════════════════════════════════════════════════════╗
║ Loop over each DLSCH                                  ║
║ └─ Call do_one_dlsch():                               ║
║    ├─ Modulate encoded bits (QPSK/QAM)                ║
║    ├─ Insert DMRS/PTRS                                 ║
║    └─ Map symbols to physical RE grid                  ║
╚════════════════════════════════════════════════════════╝

```


```
/* PTRS */
    uint16_t dlPtrsSymPos = 0;
    int n_ptrs = 0;
    uint32_t ptrsSymbPerSlot = 0;
    if (rel15->pduBitmap & 0x1) {
      set_ptrs_symb_idx(&dlPtrsSymPos,
                        rel15->NrOfSymbols,
                        rel15->StartSymbolIndex,
                        1 << rel15->PTRSTimeDensity,
                        rel15->dlDmrsSymbPos);
      n_ptrs = (rel15->rbSize + rel15->PTRSFreqDensity - 1) / rel15->PTRSFreqDensity;
      ptrsSymbPerSlot = get_ptrs_symbols_in_slot(dlPtrsSymPos, rel15->StartSymbolIndex, rel15->NrOfSymbols);
    }
    harq->unav_res = ptrsSymbPerSlot * n_ptrs;
```
## set_ptrs_symb_idx() definition in ptrs_nr.c
- A tool function that calculates the position of the OFDM symbol of the PTRS (Phase Tracking Reference Signal) in the slot. It determines which symbols can be inserted with the PTRS based on the start symbol, transmission length, PTRS time domain density, and DMRS position.
- 3GPP TS 138.211 6.4.1.2.2.1

## nr_dlsch_encoding()
```
╔══════════════════════════════════════════╗
║            Input: HARQ PDU (a[])         ║
║       - Transport Block (TB) data        ║
║       - TBSize / Qm / Layers             ║
╚══════════════════════════════════════════╝
                  │
                  ▼
╔══════════════════════════════════════════╗
║         CRC Attachment (16 or 24 bit)    ║
║ → Append CRC bits to a[] → get B bits   ║
╚══════════════════════════════════════════╝
                  │
                  ▼
╔══════════════════════════════════════════╗
║            Segmentation (nr_segmentation)║
║ → Split B bits into C segments c[0]~c[C] ║
║ → Get parameters: C, K, Z, F, Kb         ║
╚══════════════════════════════════════════╝
                  │
                  ▼
╔══════════════════════════════════════════╗
║     LDPC Encoding for each segment       ║
║ → One encoder per segment (Z, BG)        ║
║ → Output: Encoded bits per segment       ║
╚══════════════════════════════════════════╝
                  │
                  ▼
╔══════════════════════════════════════════╗
║     Rate Matching (per segment)          ║
║ → Puncturing / Shortening / Repeating    ║
║ → Based on E = nr_get_E(...)             ║
╚══════════════════════════════════════════╝
                  │
                  ▼
╔══════════════════════════════════════════╗
║        Bit Interleaving (per segment)    ║
║ → Reorder bits before mapping            ║
╚══════════════════════════════════════════╝
                  │
                  ▼
╔══════════════════════════════════════════╗
║         Output Buffer (output[])         ║
║ → Aligned on 64-byte boundary            ║
║ → Contains final codeword for modulation║
╚══════════════════════════════════════════╝
```

## do_one_dlsch()
```
╔════════════════════════════════════════════════╗
║         Input: Encoded Codeword (input_ptr)    ║
║     - Output from nr_dlsch_encoding()          ║
╚════════════════════════════════════════════════╝
                      │
                      ▼
╔════════════════════════════════════════════════╗
║        Codeword Scrambling (per codeword)      ║
║ → Function: nr_pdsch_codeword_scrambling()     ║
║ → Inputs: dataScramblingId, RNTI               ║
║ → Output: scrambled_output[]                   ║
╚════════════════════════════════════════════════╝
                      │
                      ▼
╔════════════════════════════════════════════════╗
║         Modulation (e.g., QPSK, 16QAM...)       ║
║ → Function: nr_modulation()                    ║
║ → Input: scrambled_output[], Qm                ║
║ → Output: mod_symbs[][]                        ║
╚════════════════════════════════════════════════╝
                      │
                      ▼
╔════════════════════════════════════════════════╗
║              Layer Mapping                     ║
║ → Function: nr_layer_mapping()                 ║
║ → Input: mod_symbs, numLayers                  ║
║ → Output: tx_layers[numLayers][symbols]        ║
╚════════════════════════════════════════════════╝
                      │
                      ▼
╔════════════════════════════════════════════════╗
║        DMRS Generation (per symbol)            ║
║ → Function: nr_gold_pdsch() + nr_modulation()  ║
║ → Inputs: dmrsScramblingId, SCID, slot, l      ║
║ → Output: mod_dmrs[]                           ║
╚════════════════════════════════════════════════╝
                      │
                      ▼
╔════════════════════════════════════════════════╗
║          Resource Mapping (per symbol)         ║
║ → Function: do_onelayer()                      ║
║ → Input: tx_layers + mod_dmrs + PTRS info      ║
║ → Output: txdataF_precoding[layer][symbolSize] ║
╚════════════════════════════════════════════════╝
                      │
                      ▼
╔════════════════════════════════════════════════╗
║      Precoding + Antenna Port Mapping          ║
║ → Function: do_txdataF()                       ║
║ → Inputs: txdataF_precoding, beamform info     ║
║ → Output: txdataF[ant][samples]                ║
╚════════════════════════════════════════════════╝
```
## do_onelayer()
- Key function that maps modulation symbols of a single layer to Resource Elements (REs):
Insert data, DMRS, and PTRS symbols into the corresponding OFDM symbol and subcarrier positions according to TS 38.211 standard and output them to the precoding buffer
```
╔════════════════════════════════════════════════════════════╗
║           Input: txl_start, output, DMRS/PTRS config       ║
║   - OFDM symbol index (l_symbol)                           ║
║   - Resource block size (rel15->rbSize)                    ║
║   - Subcarrier start index (start_sc)                      ║
║   - DMRS/PTRS presence bitmaps and parameters              ║
╚════════════════════════════════════════════════════════════╝
                          │
                          ▼
╔════════════════════════════════════════════════════════════╗
║  Step 1: Determine Resource Element Mapping Bounds         ║
║  - Compute `upper_limit` and `remaining_re`                ║
║  - Based on RB size and start_sc vs OFDM symbol size       ║
╚════════════════════════════════════════════════════════════╝
                          │
                          ▼
╔════════════════════════════════════════════════════════════╗
║  Step 2: Check if Current Symbol Contains PTRS             ║
║  - If yes:                                                 ║
║    → Generate PTRS QPSK symbols using Gold sequence        ║
║    → Call `do_ptrs_symbol()` to map them                   ║
╚════════════════════════════════════════════════════════════╝
         │                                                │
     Yes PTRS                                         No PTRS
         ▼                                                ▼
[Call do_ptrs_symbol()]                   Check if DMRS symbol via (dlDmrsSymbPos bit)
                                                       │
                                               Is it a DMRS symbol?
                                                       │
                                                       ▼
                      ╔════════════════════════════════════════════════════╗
                      ║  Step 3: DMRS Mapping                              ║
                      ║  - Determine DMRS port based on layer & config     ║
                      ║  - Use `interleave_with_0_*`, `interleave_signals`║
                      ║    or `dmrs_case00()` to map DMRS + data           ║
                      ╚════════════════════════════════════════════════════╝
                                                       │
                                               Not a DMRS symbol
                                                       ▼
                      ╔════════════════════════════════════════════════════╗
                      ║  Step 4: Regular PDSCH Data Mapping                ║
                      ║  - Map normal data QAM symbols to resource elements║
                      ║  - Call `no_ptrs_dmrs_case()`                      ║
                      ╚════════════════════════════════════════════════════╝
                                                       │
                                                       ▼
                      ╔════════════════════════════════════════════════════════════╗
                      ║         Output: txl = txl_start + number of REs mapped     ║
                      ║         - Return value: number of resource elements written║
                      ╚════════════════════════════════════════════════════════════╝
```
## dump_pdsch_stats()
- Outputs DLSCH transmission statistics for each active UE of the gNB to a file or the screen only once per frame, aiding debugging or performance analysis.

# **Comparison with NVIDIA Aerial RAN**

**OAI** 

Single-User MIMO (SU-MIMO) multi-antenna transmission is supported, but only at a lower number of layers. Currently, OAI supports up to 4 layers of MIMO transmission in the downlink and up to 2 layers in the uplink.

OAI gNB can send data to a single UE using up to 4 parallel Spatial Streams (typically with a 4x4 antenna configuration).

However, it does not yet support spatial reuse of co-frequency resources for Multi-User MIMO (MU-MIMO).

In the current OAI scheduling, each time-frequency resource block is still allocated to only one UE, and it is not possible to spatially multiplex multiple UEs through beamforming.

**Aerial CUDA-Accelerated RAN** 

PUSCH **SU-MIMO layers: up to 4** **MU-MIMO layers: up to 8**
PDSCH **SU-MIMO layers: up to 4** **MU-MIMO layers: up to 16** **Precoding (4 layers)**
MIMO Features Support 64 Transmit and Receive antenna ports
## nr_layer_mapping() definition in nr_modulation.c
- Layer Mapping = assigning modulation symbols to multiple independent "transmission channels" (Layers)

- case 2

<img width="719" height="53" alt="image" src="https://github.com/user-attachments/assets/fcc8914a-80aa-41cc-8f52-d9b96d6076d9" />

```
for (; i < n_symbs; i += 2) {
        *tx0++ = mod[i];// Layer 0
        *tx1++ = mod[i + 1];//Layer 1
      }
    }
```

- case 3

```
for (; i < n_symbs; i += 3) {
        *tx0++ = mod[i];// Layer 0
        *tx1++ = mod[i + 1];// Layer 1
        *tx2++ = mod[i + 2];// Layer 2
      }
```

- case 4

```
for (; i < n_symbs; i += 4) {
        *tx0++ = mod[i];
        *tx1++ = mod[i + 1];
        *tx2++ = mod[i + 2];
        *tx3++ = mod[i + 3];
}
```
- case 5 ~ case 8 are not yet implemented


## do_txdataF() //precoding
- Currently supported

<img width="805" height="176" alt="image" src="https://github.com/user-attachments/assets/44b6137b-b5e9-40c6-b3c5-8dc3ef64f3c8" />

<img width="805" height="513" alt="image" src="https://github.com/user-attachments/assets/4e20e51a-ce6b-4d8e-b991-83d1e55f0399" />

<img width="772" height="190" alt="image" src="https://github.com/user-attachments/assets/e47b30ca-3694-4738-8da2-905de5017872" />

<img width="772" height="714" alt="image" src="https://github.com/user-attachments/assets/78c39f20-11d2-46fb-a77d-5b4135121f67" />

<img width="782" height="373" alt="image" src="https://github.com/user-attachments/assets/02313df9-6ca0-4113-baae-ed8e6d8face7" />

<img width="827" height="335" alt="image" src="https://github.com/user-attachments/assets/b2f5eca9-9ff8-4c95-8fb8-4f183fbc6c1b" />

- output **txdataF_precoded**
  - txdataF_precoded is the frequency domain signal data of a single antenna after precoding. Each element corresponds to a subcarrier and is used to feed the IFFT → DAC → transmitter.
 

