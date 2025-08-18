# Software-defined Acceleration in 5G CDU 

**buffer under-run**
This is a common error condition during data transmission or processing. It means that the data consumption end (consumer) reads data faster than the data production end (provider) writes data, resulting in the buffer being empty and unable to continue normal processing.


Relying solely on the CPU to perform FEC and L2 processing cannot maintain the real-time performance requirements of 5G base stations in high-user scenarios.

Hardware acceleration (such as GPUs or ASICs) is essential to overcome this bottleneck.

<img width="378" height="366" alt="image" src="https://github.com/user-attachments/assets/9d6687b2-916d-454d-9b09-8f27562b5fe3" />

| CPU-only Architecture    | Offloaded Architecture                    |
| -------------- | ------------------------------- |
| APP            | APP                             |
| Protocol Stack | Protocol Stack                  |
| PHY Processing | PHY Processing                  |
| **The bottleneck is the CPU**    | **PHY → ACC100**, **L2 → CUDA** |

In a CPU-only system, the ultimate bottleneck of the entire stack is the CPU. In an offloaded system, ACC100 and CUDA offload heavy workloads from the PHY and L2, respectively, freeing up the CPU.

# Architecture Overview

1.User Equipment Layer

2.RF Hardware Layer (USRP B210)

3.Host Machine Layer (OpenAirInterface (OAI) Stack)

- Data Path Execution
- PHY Layer

| Treatment         | function                          |
| ------------ | --------------------------- |
| **CPU**      | Standard processing mode                      |
| **ACC100**   | Hardware-accelerated LDPC encoding/decoding (FEC)    |
| **CUDA GPU** | Optional acceleration: LLR calculation, FFT operation |

- L2 Stack（PDCP / RLC / MAC）

| Treatment         | function                                                |
| ------------ | ------------------------------------------------- |
| **CPU**      | Standard processing mode PDCP、RLC、MAC                               |
| **CUDA GPU** | Accelerate PDCP header compression, RLC/PDCP ciphering, scheduling, etc. |

<img width="507" height="531" alt="image" src="https://github.com/user-attachments/assets/c6b7080f-b9f8-42f2-a745-ca1ea3b2337d" />

**Architecture Flow**
- UE → RF Hardware (USRP B210) → Host Machine (OAI Stack + GPU + ACC100)

## ACC100
### what is ACC100
- The Intel ACC100 (full name: Intel® vRAN Dedicated Accelerator ACC100) is a dedicated hardware accelerator card for 4G/5G radio access networks. It connects to the host using a PCIe interface and features a built-in fixed-function structured ASIC chip for accelerating forward error correction (FEC) operations.
- The ACC100 is designed to offload the most computationally intensive channel coding and decoding tasks (such as LDPC and Turbo encoding/decoding) in the physical layer of 4G LTE and 5G NR base stations, freeing up CPU resources and improving the channel capacity and efficiency of the entire base station system, particularly in virtualized RAN (vRAN).

### Core Features
The ACC100 is specifically designed for FEC codec acceleration. For 5G NR, it accelerates LDPC encoding/decoding, as well as related functions such as coded block CRC checking, rate matching/dematching, and HARQ buffer management.
For 4G LTE, the ACC100 accelerates Turbo code encoding/decoding, also performing coded block CRC generation/checking and rate matching.
a single ACC100 card can simultaneously support 5G and 4G FEC workloads, accelerating both uplink and downlink channel coding tasks.
In addition, the card features a hardware queue manager that manages resource scheduling and load balancing across multiple tasks, ensuring efficient scheduling of uplink and downlink codec requests.

### O-RAN Architecture: 
The ACC100's design is highly aligned with the open ecosystem advocated by the O-RAN Alliance. First, the ACC100 utilizes the open API (DPDK BBDev) adopted by O-RAN as its interface, facilitating software integration across vendors.

Second, within O-RAN's base station functional split, the ACC100 perfectly fills the role of O-DU physical layer acceleration and can be used with any radio unit (O-RU) that conforms to the O-RAN fronthaul interface. Because the ACC100 is a standard PCIe card, telecom system integrators can deploy it in common COTS servers to implement multi-vendor DU solutions without relying on closed, proprietary baseband hardware. In short, the ACC100 provides a plug-and-play FEC acceleration solution for 5G base stations within the O-RAN architecture and is widely adopted in applications such as 5G open vRAN and private networks.

### DPDK（Data Plane Development Kit）
- It is a high-performance network data processing framework, mainly used to accelerate packet processing in user space.
- Its characteristics
  - Bypassing the Linux kernel network stack (zero-copy)
  - Direct access to hardware using hugepage and poll mode driver (PMD)

<img width="887" height="461" alt="image" src="https://github.com/user-attachments/assets/6ca8feb8-013e-474d-b686-7deaa65b6bd8" />

**Role of DPDK**
Accelerates packet flow between:
- Central Unit (CU)
- Distributed Unit (DU)
- User Plane Function (UPF)
- gNB (base station)
- CU–DU interface (E1-F)
- RAN Intelligent Controller (RIC)
- Provides high-speed packet processing & scheduling
- Bypasses kernel bottlenecks for low-latency transmission (critical for both user-plane data & control messages)

**How DPDK Improves Performance**
- Offloads packet handling for CU/DU and UPF
- x86 CU/DU example:
  - Uses DPDK APIs to offload data-plane processing to:
    - FPGA-based SmartNIC
    - Intel ACC100
  - Reduces CPU load and improves latency

**Near Real-Time RIC Communication**
- gNB → E2 Agent → E2 Application Protocol (EAP) → RIC Service Models (SM/E2) → xApps

**Key Benefits**
- Handles both:
  - High-speed user-plane traffic
  - Timing-critical control-plane messages
- Maintains software-defined flexibility (network functions stay configurable)
- Maximizes CPU & hardware utilization
- Meets 5G performance demands for throughput & low latency

### BBDev(BaseBand Device API）
It is a set of APIs provided by DPDK specifically for accelerating baseband processing, mainly targeting 4G LTE / 5G NR PHY layer computing tasks, such as:
- LDPC（5G NR）
- Turbo（LTE）
- Rate Matching / De-Matching
- CRC

BBDev API = allows software to use a unified interface to call baseband acceleration hardware (ASIC / FPGA / GPU / CPU SIMD) from different manufacturers

**Purpose of the BBDev API**
- Abstracts hardware differences → Uses the same API regardless of backend: Intel QuickAssist (QAT), FPGA, or GPU
- Allows hardware acceleration to coexist with pure software implementation → Use software simulation during development and switch to hardware acceleration during deployment


### Bypassing the Linux kernel network stack
It's actually about a technique that "processes network data directly in user space, bypassing the traditional Linux TCP/IP stack." This technique is commonly used in high-performance networks (HPC, 5G, trading systems, etc.) to reduce CPU overhead and data replication latency.

Core Concepts:
- Avoids the Linux TCP/IP stack
- Directly transfers data from the NIC to user-space memory
- Applications are responsible for their own protocol handling (or use hardware offload)

## CPU-only Architecture

<img width="790" height="751" alt="image" src="https://github.com/user-attachments/assets/0fad3d97-8b01-4671-b33d-11becc3a10fd" />

**All baseband processing done on CPU**

- Advantages: Software-defined flexibility
- shortcoming :
  - High CPU load, especially under 100+ UEs
  - Latency bottlenecks at PHY decoding and L2 scheduling
  - No hardware acceleration
 
## CPU vs CUDA Mode

| mode   | features                           | dalay (L2)          | UE Scalability |
| ---- | ---------------------------- | ---------------- | ------- |
| CPU  | Sequential processing, serialized stacking                   | \~24ms           | Restricted      |
| CUDA | Parallel GPU core processing (PDCP+RLC+MAC simultaneously) | **0.38ms** (63×) | Highly scalable    |

## CUDA Implementation Details
- Shared memory sync
  - **Host-pinned ring buffer**: Host-side fixed-memory ring buffer for fast data exchange between CPU ↔ GPU
  - **Zero-copy access**: The CPU and GPU can directly access memory (no data copying required), reducing latency.
 
- CUDA Advantages
  - High concurrency: It is particularly effective for MAC layer logic and can handle scheduling and control tasks for a large number of UEs simultaneously.
  - Scalability: •Scales up to 5000 UEs using 5000 CUDA threads (1 per UE)
  - Latency improvement: ~3× faster than CPU loop


## Design Partitioning for Multi-Hardware Offload

- **ACC100 (PHY Layer)**: Handles LDPC encoding/decoding for UL/DL and Processes HARQ feedback loop in hardware.
- **CPU（MAC Layer & I/O**: Responsible for physical layer signal processing and data plane transmission
- **GPU（CUDA Offload – Control Plane)**: Responsible for control plane scheduling and encryption

<img width="703" height="633" alt="image" src="https://github.com/user-attachments/assets/7994fefa-9708-450c-b076-939a011ce310" />

**Shared Host-Pinned Ring Buffer**
- Semaphore Lock-free Ring: Reduce lock contention
- Supports Double-Buffering: allows processing while preparing the next batch of data

### Double-Buffering
- Double-Buffering = Using two buffers (Buffer A and Buffer B) to alternately read and write, allowing data transmission and calculation to proceed in parallel, thereby reducing waiting time.

- **No Double-Buffering**：First transfer data → perform computation → return results → then process the next batch (transfer and computation cannot overlap) → The GPU waits for data, while the CPU waits for the GPU, wasting time.
- **Double-Buffering**： While the GPU is computing data in Buffer A, the CPU is simultaneously preparing and transferring data from Buffer B. → Overlapping transfer and computation increases efficiency.

### cudaMemcpyAsync
- Asynchronous data transfer between the host (CPU) and the device (GPU)
- Asynchronous = Return immediately after the command call, and continue to perform other work without waiting for the transfer to complete

## Original RX Flow (Baseline) 
<img width="887" height="480" alt="image" src="https://github.com/user-attachments/assets/dedd50f4-ff19-4288-8984-b6661689ec05" />

- **rx_func()**: The wireless radio receiving function is responsible for coordinating each frame or slot to receive data from the radio front end.

- Initial Procedure:
  - PRACH procedure: Random access channel processing, used to process uplink access requests.
  - PUCCH decode: Decodes uplink control information (UCI).
 
- PUSCH procedure:

| Step                               | Function Description                                                                                                |
| ---------------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| **`slot_fep()`**                   | Time-frequency conversion (FFT) that transforms the time-domain signal of an entire slot into the frequency domain. |
| **Channel Estimation**             | Estimates the wireless channel response using reference signals.                                                    |
| **Channel Scaling / Compensation** | Applies the inverse of the channel response to the received signal to correct fading and distortion.                |
| **Equalization**                   | Performs equalization to restore the original symbols.                                                              |
| **`rx_llr_computation()`**         | Converts equalized symbols into soft bits (Log-Likelihood Ratios, LLR).                                             |
| **LDPC Decoding**                  | Uses the LDPC decoder to convert LLRs into the original bit sequence.                                               |
| **CRC Check**                      | Verifies the integrity of each decoded transport block.                                                             |
| **MAC SDU Extraction**             | Extracts the MAC subpacket from the physical layer frame.                                                           |
| **RLC Reassembly**                 | Reassembles segmented packets.                                                                                      |
| **PDCP Decipher & Reorder**        | If encryption is applied, performs decryption and reorders data into the correct sequence.                          |
| **IP/SDAP Delivery**               | Delivers the fully reassembled and decrypted data to the core network layer.                                        |

- performance bottleneck:
- In this baseline design, each sub-stage from FFT to decoding to PDCP is implemented in software on a general-purpose CPU.
- When uplink throughput is high, the CPU can easily become a performance bottleneck.

## Modified RX Flow (Accelerated)

<img width="887" height="525" alt="image" src="https://github.com/user-attachments/assets/6c00d2a0-db76-4ad8-b14e-55201bc67af1" />

| Step                                                     | Baseline Processing                   | Accelerated Processing                         |
| -------------------------------------------------------- | ------------------------------------- | ---------------------------------------------- |
| **`slot_fep()`**<br>Time-Frequency Conversion (FFT)      | FFT executed on CPU (single-threaded) | CPU (SIMD or OpenMP) or GPU acceleration       |
| **UL Channel Estimation**                                | CPU (single-threaded)                 | CPU (OpenMP) or CUDA (optional)                |
| **UL Channel Compensation**                              | CPU (single-threaded)                 | Offloaded to CUDA (T4 GPU)                     |
| **`rx_llr_computation()`**<br>Soft Bit Calculation (LLR) | CPU (single-threaded)                 | Offloaded to CUDA (T4 GPU)                     |
| **LDPC Decoding**                                        | CPU (software decoding)               | Offloaded to Intel ACC100 hardware accelerator |
| **CRC Check**                                            | CPU (single-threaded)                 | CPU                                            |
| **MAC SDU Extraction**                                   | CPU                                   | CPU                                            |
| **RLC Reassembly**                                       | CPU                                   | CPU (multi-threaded buffering)                 |
| **PDCP Decipher & Reorder**                              | CPU                                   | Partially offloaded to CUDA                    |
| **IP/SDAP Delivery**                                     | CPU                                   | Software or `iperf`                            |

**Key Points**
- GPU (CUDA): Responsible for highly parallel computations, such as LLR calculations and some PDCP decryption.
- ASIC (Intel ACC100): Responsible for computationally intensive FEC tasks such as LDPC decoding.
- CPU (OpenMP/SIMD): Handles parallelizable but latency-sensitive tasks such as FFT, channel estimation, and channel compensation.
- Flexible Switching: If hardware accelerators are unavailable, the system automatically falls back to pure CPU processing, maintaining consistent functionality.

## Original TX Flow (Baseline)

1. Entry Point — tx_func() (MAC Layer)
- Starts the transmission for the current scheduling interval.
- MAC scheduler allocates time-frequency PRBs (Physical Resource Blocks) for downlink data.

2. PDCP Layer
- Ciphering (Encryption): Ensures confidentiality of transmitted packets.

3. RLC Layer
- Segmentation: Splits PDCP SDU into smaller RLC PDUs if needed.
- Sequence Numbering: Assigns sequence numbers for correct reassembly at the receiver.

4. MAC Layer — Transport Block Creation
- Logical Channel Multiplexing: Combines PDUs from multiple logical channels into one transport block.
- MAC PDU Assembly: Adds necessary MAC headers and finalizes the MAC PDU.

5. PHY Layer — Encoding & Modulation
- Code Block Segmentation: Splits large transport blocks into smaller code blocks (per LDPC size limit).
- CRC Addition: Appends CRC to each code block for error detection.
- LDPC Encoding: Applies forward error correction to improve reliability.
- Rate Matching: Adapts the coded bitstream to available PRBs (puncturing/repetition).
- Modulation Mapping: Converts bits into modulation symbols (QPSK, 16-QAM, 64-QAM).
- Resource Element Mapping: Places symbols into the OFDM grid, inserts reference signals (DMRS).

6. IFFT & OFDM Signal Generation
- IFFT (slot_fep_tx): Converts frequency-domain symbols to time-domain OFDM waveforms.
- Cyclic Prefix Insertion: Adds CP to each OFDM symbol to mitigate multipath effects.

7. RF Front-End Transmission
- Digital Baseband Output: Produces time-domain baseband signal.
- DAC + RF Front-End (USRP B210):

## Modified TX Flow

<img width="601" height="653" alt="image" src="https://github.com/user-attachments/assets/467b7d8d-d511-438d-9230-68c6f33e3de7" />

| Processing Stage             | Baseline (Original)                         | Enhanced (Optimized)                                                               |
| ---------------------------- | ------------------------------------------- | ---------------------------------------------------------------------------------- |
| **MAC Scheduling**           | Static scheduling by MAC scheduler.         | Dynamic BLER-aware scheduling; adjusts PRB allocation and MCS per user each frame. |
| **PDCP Ciphering**           | CPU-based ciphering.                        | GPU offload (NVIDIA T4) using CUDA for parallel encryption.                        |
| **PDCP Header Compression**  | CPU handles ROHC.                           | Remains on CPU.                                                                    |
| **RLC Segmentation**         | CPU sequential segmentation.                | Optimized with vectorized/memory-efficient code.                                   |
| **RLC Buffering**            | Single-threaded buffer.                     | Thread-safe multi-threaded buffer enabling parallel PDU handling.                  |
| **MAC PDU Assembly**         | Per-packet assembly.                        | Batched PDU packing by UE; improved cache/timing efficiency.                       |
| **LDPC Encoding**            | CPU-based software encoding.                | Offloaded to Intel ACC100 hardware encoder.                                        |
| **Rate Matching**            | CPU sequential bit selection/puncturing.    | CPU SIMD acceleration (AVX2/AVX-512) for bit selection/pruning.                    |
| **CB Segmentation**          | CPU handles segmentation.                   | Still on CPU but benefits from upstream accelerations.                             |
| **Modulation Mapping**       | CPU sequential mapping to QPSK/16QAM/64QAM. | CPU optimized, remains on CPU.                                                     |
| **Resource Element Mapping** | CPU maps symbols and inserts DMRS.          | CPU optimized, remains on CPU.                                                     |
| **IFFT (slot\_fep\_tx)**     | CPU sequential IFFT.                        | Parallelized via OpenMP or offloaded to GPU (CUDA).                                |
| **RF Transmission**          | USRP B210 DAC + RF upconversion.            | Same hardware but can sustain higher bandwidth due to faster pipeline.             |


## LDPC Decoding Offload with ACC100

<img width="480" height="741" alt="image" src="https://github.com/user-attachments/assets/86f9b678-b4f3-4361-921d-8f224e5bd010" />

**Uplink LDPC Decoding Flow with Intel ACC100 in OAI gNB**
- This flow shows how OAI’s gNB switches between:
  - Software LDPC decoding (CPU-based)
  - Hardware LDPC decoding (Intel ACC100 offload via DPDK BBDEV API)

- Common Entry
  - LDPC decoding stage: Function ldpc_decoder_opt() is reached after receiving an uplink transport block (UL TB).
  - Decision Point: Checks whether ACC100 offload is enabled.
 
## OAI Architecture and Processing Paths

<img width="640" height="730" alt="image" src="https://github.com/user-attachments/assets/88c3a8d9-146c-4db3-a3b3-c76ffc006007" />

### PDCCH (control signals)
- phy_procedure_gNB_TX0
- nr_common_signal_procedure
- PDCCH Processing
- nr_generate_dci_top_Generate_DCI() 

### PBCH (broadcast signal)
- nr_generate_pbch → Generates PBCH data
- nr_pbch_scrambling → Scrambles the PBCH
- Channel_Coding_Polar (Polar Code Encoding)
  - polar_encoder_fast / polar_generate_u → Polar Code Encoding
  - polar_rate_matching → Polar Code Rate Matching
- Modulation and Resource Mapping
  - Modulation_CUDA / nr_modulation_cuda → Modulation (GPU acceleration supported)
  - nr_pbch_codeword_scrambling → Codeword scrambling
  - nr_layer_mapping → Layer mapping
  - nr_dmrs_DMRS_Generation → Generate DMRS reference signals
  - nr_ptrs_PTRS_Generation → Generate PTRS reference signals
 
### PDSCH
- nr_generate_pdsch → **Generate PDSCH data**
- nr_pdsch_codeword_scrambling → **Scramble PDSCH codewords**
- nr_segmentation → **Segmentation processing**
- **LDPC encoding (hardware acceleration)**
  - LDPC_Encoding_ACC100 → **LDPC encoding using the Intel ACC100**
  - ldpcBlocks_acc100 → **Process LDPC blocks**
  - nr_rate_matching_ldpc → **LDPC rate matching**
  - nr_interleaving_ldpc → **LDPC interleaving**
- **Modulation and resource mapping**
  - Scrambling → **Scrambling**
  - Modulation_CUDA → **Modulation (GPU acceleration supported)**
  - nr_layer_mapping → **Layer mapping**
  - nr_resource_element_mapping → **Resource element mapping**

### Waveform generation and transmission
- OFDM_Symbol_Generation_CUDA → **OFDM symbol generation (GPU-accelerated)**
- nr_ofdm_modulation_cuda → **OFDM modulation**
- Cyclic_Prefix_Insertion / nr_cp_insertion → **Cyclic prefix insertion**
- Transmission_via_Antenna_Array / nr_transmit_antenna_array → **Antenna transmission**

<img width="640" height="730" alt="image" src="https://github.com/user-attachments/assets/b4146611-15ae-408e-a739-10d7cf813e22" />

### PRACH Path (Random Access)
- UL Processing → PRACH Reception (rx_func) Receives the PRACH signal and performs preliminary processing.
- L1_nr_prach procedures PRACH physical layer processing procedures.
- apply_nr_rotation_R: Performs PRACH frequency rotation compensation.
- rx_nr_prac: Completes PRACH reception and detection, obtaining the UE's random access information.

### PUSCH
- UL Processing → phy_procedures_gNB_uespec_rx Handles UE-specific uplink receive procedures.
- nr_decode_pusch / nr_decode_pusch_t Main entry point for PUSCH reception, responsible for demodulation, symbol processing, etc.
- nr_srs_channel_estimation Performs channel estimation based on the Sounding Reference Signal (SRS).
- nr_pusch_channel_estimation Performs PUSCH dedicated channel estimation based on the DMRS.
- nr_ulsch_extract_rbs Extracts the REs (Resource Elements) corresponding to the PUSCH from the received frequency domain resources.
- nr_pusch_symbol_processing Symbol-level processing, including channel compensation, equalization, and descrambling.
- Inner RX (inner_rx) Inner layer receive processing (including LLR calculation, layer demapping, etc.).
- ULSCH Procedures Uplink Shared Channel (UL-SCH) processing, including decoding and error checking.
- Post Decode (nr_postDecode) Post-decoding processing (e.g., CRC verification, retransmission control).

**Importance and Load Characteristics of Low-Density Parity-Check Code (LDPC) Encoding and Decoding**
- High Computational IntensityLDPC encoding and decoding is one of the most computationally intensive tasks in the 5G PHY layer.
- Pure CPU Mode: Completely implemented in software, this consumes significant CPU cycles, especially at high modulation and coding order (MCS) levels and full PRB configurations.
- Coding Load Position: Primarily affects the gNB downlink path.
- Decoding Load Position: Affects both the gNB uplink path and the UE downlink path.

**Advantages of Intel ACC100 Hardware Acceleration**
- Dedicated Hardware Path
  - Once configured, the OAI software stack sends LDPC tasks to the ACC100 via the BBDev interface.
  - The ACC100 performs the encoding or decoding and transmits the results back with extremely low latency.
- Benefits
  - Frees up CPU resources: Reduces the computational burden on the PHY layer, preventing CPU saturation.
  - Increased decoding throughput: Enables processing of larger code blocks or higher traffic rates without missing deadlines.
  - Improves HARQ timing: Ensures feedback messages are sent within their deadlines, reducing retransmission delays and maintaining link stability.
 
**Pure software mode: Highly resilient, but prone to latency and excessive CPU stress under high load.**

**Hardware acceleration mode (ACC100): Reduces CPU stress, improves throughput, and ensures stable timing. It is a key optimization method for high-performance 5G base stations.**

| Mode            | Hardware     | Acceleration Layer | Accelerated Tasks                                       | Advantages                                              | Limitations                                                     |
| --------------- | ------------ | ------------------ | ------------------------------------------------------- | ------------------------------------------------------- | --------------------------------------------------------------- |
| Mode 1 CPU-only | None         | L1–L2 all on CPU   | All software execution                                  | Easy to deploy                                          | CPU bottleneck under high load, severe latency and jitter       |
| Mode 2 ACC100   | Intel ACC100 | L1                 | LDPC encoding/decoding, HARQ                            | Reduces PHY load, stabilizes subframe timing            | L2 remains CPU-bound, queuing issues under large UE/bursty load |
| Mode 3 GPU      | NVIDIA T4    | L2                 | RLC segmentation, PDCP reordering, MAC SDU multiplexing | Reduces L2 CPU load by 20%, improves multi-UE stability | No L1 acceleration, LDPC decoding still on CPU                  |

## Framing the Problem
### Background
- Prior results:
  - CPU-only execution and ACC100 PHY offload improved specific areas.
  - Layer 2 bottlenecks remained, even after L1 acceleration.
 
- Main issues in L2:
  - PDCP SDU reordering under multi-UE burst traffic.
  - RLC block segmentation for high-throughput, multi-user cases.
  - These tasks consumed significant CPU cycles, causing:
    - Scheduling jitter.
    - Missed scheduling opportunities.
   
- Core Idea
- Question: Can a general-purpose GPU, originally built for graphics, handle telecom-grade Layer 2 packet manipulation?
- Rationale:
  - CUDA programming model supports:
    - Parallel streams — multiple tasks processed concurrently.
    - Kernel isolation — separation of workloads for predictable performance.
  - This could match the parallelizable nature of PDCP and RLC operations.
 
## Integration Phase — Wiring CUDA into OAI
- Embed CUDA execution paths into the OAI Layer 2 stack
- Key Integration Tasks
  - Enable Host–Device Data Transfers
  - CUDA Stream Insertion
  - Pinned Memory Allocation
  - CPU–GPU Synchronization

- Impact
  - L2 operations that were previously sequential now run concurrently on the GPU.
  - Increased processing throughput and reduced L2-related queuing delays.
 
## Functional Validation
- Validation Strategy
  - HARQ Combining Parity
    - Compared GPU-accelerated HARQ soft combining against CPU routines.
    - Tested even under deep retransmission chains.
    - Result: Identical retransmission feedback in all cases.
  - PDCP Reordering Checks
    - Verified packets maintained strict sequence integrity.
    - Tested under both real UE traffic and simulated UE reordering stress.
  - Buffer Boundary Safety
    - Used CUDA debug flags to monitor memory access patterns.
    - Confirmed no:
      - Memory leaks
      - Overwrites
      - Misalignments
     
## System Architecture

<img width="1092" height="674" alt="image" src="https://github.com/user-attachments/assets/76400991-a8d4-4e72-9893-f964a0276e2b" />

- Component Roles
  - USRP B210 — RF Front End
- Intel ACC100 — PHY Layer Offload
  - Positioned in the uplink & downlink PHY chain.
  - Offloads most CPU-intensive PHY tasks:
    - LDPC encoding.
    - LDPC decoding.
    - HARQ retransmissions.
  - Removes L1 bottlenecks in CPU-only setups.
 
- NVIDIA T4 GPU — Layer 2 Offload
- Handles L2 processing:PDCP、RLC
- Uses CUDA streams for parallel execution.
- Reduces L2 processing latency and CPU load.

