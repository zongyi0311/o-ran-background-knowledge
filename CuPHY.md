<img width="800" height="436" alt="image" src="https://github.com/user-attachments/assets/711d7425-2325-4ad1-a1bf-0e3416943197" />

**left side**
- gNB Architecture(CU + DU) + RU 

| Block                        | function                                                                            |
| ------------------------- | ----------------------------------------------------------------------------- |
| **Central Unit (CU)**     | Include `RRC` (Radio Resource Control) and `PDCP` (Packet Data Convergence Protocol) |
| **Distributed Unit (DU)** | Responsible for L2 and L1 upper layer functions (MAC / RLC / cuPHY-Upper)                                      |
| **Radio Unit (RU)**       | Responsible for RF and L1 Lower level（FFT、Precoder、RF DAC/ADC）                                        |

**Downlink**
- Downlink（PDSCH example）：

```
TB + CRC → CodeBlock Segmentation → LDPC Encode → Rate Matching → 
Scrambling → Modulation → Layer Mapping → Precoding → RE Mapping
```

**Uplink**
- Uplink (PUSCH example) :

```
RE De-Mapping → Channel Estimation → MIMO / Rx Beamforming → 
Demodulation → Descrambling → De-Rate Match → HARQ → LDPC Decode → CRC Check
```

**PHY-Lower**
- Processing FFT/IFFT, CP addition/removal, RF DAC/ADC

**Features of cuPHY**
- Target function : Processing of all 5G NR PHY channels (such as PUSCH, PDSCH, PUCCH, PBCH, SRS, etc.)
- Corresponding architecture : Corresponding to O-RAN 7.2x split , belongs to upper PHY (located in O-DU)
- How it works : Execution using NVIDIA GPU + CUDA acceleration


```
+------------------------------+
|      cuPHY - Upper PHY       |  ← (RE Demap, Channel Est, LDPC Decode etc.）
+------------------------------+
            ↑
     O-RAN 7.2x Split
            ↓
+------------------------------+
|      Lower PHY in O-RU       |  ← including FFT、Pre-coding、RF etc.
+------------------------------+
```

**Relationship with cuBB and cuMAC 
| name      | Function                                         | Location            |
| --------- | ------------------------------------------------ | ------------------- |
| **cuPHY** | L1 Upper PHY (demodulation, estimation, coding)  | **Part of cuBB**    |
| **cuMAC** | L2 MAC + Scheduler                               | **Part of cuBB**    |
| **cuBB**  | Complete O-DU side stack (cuPHY + cuMAC modules) | **Located in O-DU** |


- So cuBB(including cuPHY) and cuMAC are both in the O-DU and correspond to the High PHY + MAC processing part in the O-RAN Split 7.2x architecture.

# cuPHY Architecture Design Description
- cuPHY is NVIDIA’s CUDA-based implementation of the 5G NR physical layer. It offloads and parallelizes key PHY signal processing functions on the GPU to accelerate performance, scalability, and cell density. 

**GPU-Centric Parallelization**
- Each PHY layer function is implemented using CUDA kernels.
- Tasks are parallelized as much as possible across threads, slots, and cells.

**Channel-Based Pipeline Architecture**
- cuPHY defines one pipeline per PHY channel type.
- Examples:
  - PDSCH pipeline: for DL encoding
  - PUSCH pipeline: for UL decoding
  - SSB pipeline: for synchronization
 
**Pipeline Structure**
- Every step in the PHY process (e.g., LDPC decode, MIMO equalization) is executed using a separate CUDA kernel.
- Pipelines are optimized for high-throughput, low-latency execution.

**Slot-Based Pipeline Scheduling**
- cuPHY driver dynamically instantiates the required pipelines per slot.
- Based on the MAC-scheduled L1 configuration (e.g., which UEs are active, what channels are assigned), a set of pipelines is selected and launched.

**Benefits of cuPHY Pipeline Design**
| Feature                 | Benefit                                        |
| ----------------------- | ---------------------------------------------- |
| CUDA Kernel Execution   | Parallel and GPU-accelerated signal processing |
| Channel-Based Pipelines | Modular and reusable pipeline components       |
| Dynamic Scheduling      | Adapts per-slot, per-UE, per-channel           |
| Multi-Cell Ready        | Scalable to dense gNB deployments              |
| O-RAN 7.2x Compliant    | Compatible with industry RAN disaggregation    |


# cuPHY Pipeline API Interface

<img width="850" height="592" alt="image" src="https://github.com/user-attachments/assets/732e7153-69d4-42c0-b6d2-0c217e6c8fe6" />

**Left Side APIs (L2 → cuPHY)**

| API Name      | Description                                                                 |
| ------------- | --------------------------------------------------------------------------- |
| `Create()`    | Instantiates a new pipeline for a specific PHY channel (e.g., PUSCH decode) |
| `Destroy()`   | Destroys an existing pipeline when no longer needed                         |
| `Re-Config()` | Updates pipeline parameters (e.g., modulation scheme, UE-specific config)   |
| `Setup()`     | Initializes internal GPU buffers and context (but does not run yet)         |
| `Run()`       | Launches the CUDA-based signal processing kernels for the slot              |

**Right Side Hooks (cuPHY → L2 or System)**

| Signal/Event            | Description                                                    |
| ----------------------- | -------------------------------------------------------------- |
| `StateUpdate()`         | Sends status reports, results, or errors back to L2            |
| `Create/Modify/Destroy` | Internal triggers to modify the pipeline dynamically as needed |

# PDSCH Pipeline – cuPHY
- The PDSCH pipeline in cuPHY handles the complete downlink (DL) physical layer processing of transport blocks (TBs) and maps the output to IQ samples for physical resource elements (REs) corresponding to the PDSCH.

**High-Level Components**
- 1. CRC Calculation
- 2. LDPC Encoding
- 3. Fused Rate Matching and Modulation
  - Rate Matching
  - Scrambling
  - Layer Mapping
  - Precoding
  - Modulation
- 4. DMRS Generation
 
**CUDA Kernels Used**

| Kernel Name                            | Purpose                                              |
| -------------------------------------- | ---------------------------------------------------- |
| `prepare_crc_buffers`                  | Allocate/init memory for CRC                         |
| `crcDownlinkPdschTransportBlockKernel` | Compute TB-level CRC                                 |
| `crcDownlinkPdschCodeBlocksKernel`     | Compute CB-level CRC                                 |
| `ldpc_encode_in_bit_kernel`            | LDPC encode each code block                          |
| `fused_dl_rm_and_modulation`           | Fused rate matching, scrambling, mapping, modulation |
| `fused_dmrs`                           | Generate DMRS symbols for PDSCH                      |

**CSI-RS Related Kernels (Optional Execution)**
- These CUDA kernels are only triggered if CSI-RS (Channel State Information Reference Signal) parameters are present in the PDSCH configuration. CSI-RS is used for channel quality estimation, beamforming, and feedback mechanisms at the UE side.

| Kernel Name             | Description                                                              |
| ----------------------- | ------------------------------------------------------------------------ |
| `zero_memset_kernel`    | Initializes memory regions (e.g., clears RE grids before CSI-RS mapping) |
| `genCsirsReMap`         | Generates CSI-RS resource element mapping based on 3GPP configuration    |
| `postProcessCsirsReMap` | Applies post-processing steps (e.g., zero-masking, overlap handling)     |

- These kernels are not part of the default PDSCH pipeline. They are only executed when CSI-RS is configured, as defined in the RRC signaling and transmitted in the downlink.


**cuPHY PDSCH Processing Flow**

<img width="1806" height="707" alt="image" src="https://github.com/user-attachments/assets/5b126073-8350-4df7-b65a-5645f253e283" />

**PUSCH Channel Estimation & Equalization Pipeline**

<img width="1763" height="755" alt="image" src="https://github.com/user-attachments/assets/9c618134-cb6a-467d-8bff-18a0b967838a" />

- Step-by-Step Breakdown
  - Initial Estimation
    - LS Channel Estimation:
    Performs Least-Squares estimation using DMRS.
    - MMSE Channel Estimation:
    Enhances LS results using noise/interference statistics (if available).
  
  - Covariance & Whitening
    - Noise and Interference Covariance Estimation:
    Estimates the correlation structure of noise + interference.
    - Shrinkage and Whitening:
    Applies shrinkage (to regularize) and whitening to remove noise correlation.
  
  - Equalization Coefficients
    - Channel Equalization Coefficients:
    Computed from MMSE output and whitening results.
  
  - Data Equalization
    - Equalize Data:
    Applies coefficients, then performs Soft Demapping (e.g., LLR extraction).

- Impairment Estimation
- These occur in parallel or as feedback paths:
  - CFO Estimation & Averaging:
    Estimates Carrier Frequency Offset.
  - Timing Offset Estimation:
    Tracks timing drift.
  - Both results may feed into the shrinkage/whitening and equalization stages.
 
- Noise & Power Estimation
  - Estimate Noise Variance:
    - Generates:
      - preEqualizationNoisePower
      - postEqualizationNoisePower
  - RSSI Estimation and Averaging:
  From IQ data buffer → yields RSSI estimate.
  - RSRP Estimation and Averaging:
  Measures Reference Signal Received Power from DMRS.
  - SNR Estimation:
  Based on noise and RSRP estimates → yields SNR estimate.

**cuPHY UCI Decoding Pipeline (PUSCH)**

<img width="1727" height="777" alt="image" src="https://github.com/user-attachments/assets/ee10540f-2915-4586-99c9-33f2e6e54ff1" />

**1. Start**
- Continue from PUSCH Front End
- Input: Soft bits from PUSCH front-end demodulation and equalization.

**2. De-ratematching + UCI Descrambling**
- First step: undo rate matching and descrambling.
- After this step, the payload is routed based on its bit length and content (e.g., CSI Part 2, HARQ, etc.)

**3. Decoder Selection Based on Payload Type/Size**
- If No CSI Part 2:
  ➡ LDPC Backend
  → Performs LDPC decoding of transport blocks (TBs)
  → Output: TB CRC result

-  If UCI payload:Depending on the number of bits:

| Payload Size    | Decoder Used                 |
| --------------- | ---------------------------- |
| ≤ 2 bits        | ➡ `Simplex Decoder`          |
| ≤ 11 bits       | ➡ `RM (Reed-Muller) Decoder` |
| 12 to 1706 bits | ➡ `Polar Decoder`            |

**4. Output and Next Step**
- Output: Decoded UCI Payload (HARQ ACK/NACK and CSI Part 1)
- Then: Proceed to CSI Part 2 control kernel (if needed)

**CSI Part 2 Control Kernel and Decoding Pipeline (cuPHY)**

<img width="1747" height="608" alt="image" src="https://github.com/user-attachments/assets/bff35b05-e0ac-4e10-8467-5950247731a2" />

- This flow picks up after decoding UCI Part 1 (HARQ + CSI Part 1) and handles CSI Part 2 and SCH (Shared Channel) decoding.

**Input**
- Decoded UCI Payload (HARQ and CSI Part 1)
  - This output from the previous UCI decoder stage triggers the CSI Part 2 processing.
 
- Control Kernel: Decoder Backend Setup
- Based on CSI Part 2 and SCH payload size/type, the control kernel performs backend setup:
  - LDPC Decoder Backend Setup
  - Simplex Decoder Backend Setup
  - RM Decoder Backend Setup
  - Polar Decoder Backend Setup
 
- De-ratematching + Descrambling

| Decoder Type        | Used For                                  |
| ------------------- | ----------------------------------------- |
| **LDPC Decoder**    | For large SCH data blocks                 |
| **Simplex Decoder** | For ultra-short payloads (e.g., 1–2 bits) |
| **RM Decoder**      | For small UCI or CSI payloads (≤ 11 bits) |
| **Polar Decoder**   | For CSI/UCI payloads (12–1706 bits)       |

