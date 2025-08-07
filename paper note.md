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
