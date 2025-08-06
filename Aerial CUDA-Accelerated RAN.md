# AERIAL CUBB
```
Overall PHY capabilities include:
• Error detection on the transport channel and indication to higher layers
• FEC encoding/decoding of the transport channel
• Hybrid ARQ soft-combining
• Rate matching of the coded transport channel to physical channels
• Mapping of the coded transport channel onto physical channels
• Power weighting of physical channels
• Modulation and demodulation of physical channels including:
– Frequency and time synchronization
– Radio characteristics measurements and indication to higher layers
– Multiple Input Multiple Output (MIMO) antenna processing
– Transmit Diversity (TX diversity)
– Digital and Analog Beamforming
– RF processing
```
# Aerial CUDA-Accelerated RAN PHY Numerologies

<img width="832" height="351" alt="image" src="https://github.com/user-attachments/assets/521eb2d7-6124-45a5-9d6f-af86cf4281a1" />

**oai document PHY/INIT/nr_parm.c**

**set_Lmax()**:
  Set fp->Lmax to reflect the maximum number of SSB beams that the 5G NR system can use for synchronization and initial reception under different frequency conditions.

**nr_get_ssb_start_symbol()**:
- Determine the starting position of each SSB beam in the slot
- Help UE to correctly decode synchronization signals
- Perform beam scheduling and resource alignment during gNB transmission

**set_scs_parameters()**
- From the set_scs_parameters() function, we can clearly see that OAI (OpenAirInterface) already supports numerology index μ = 0 to μ = 4
- Aerial CUDA-Accelerated RAN（NVIDIA cuRAN）Currently only supports μ = 1（SCS = 30 kHz）

## Comparison: Aerial cuPHY vs OAI – PHY Physical Resources Support

| **Feature**                | **Aerial cuPHY** | **OAI Supported** | **OAI Implementation and File References**                                                                                             |
| -------------------------- | ---------------- | ----------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| **Antenna Ports**          |  Yes            |  Yes             | `NR_DL_FRAME_PARMS.nb_antenna_ports_gNB` in `nr_phy_init.c`<br>Supports up to 8 ports (used in DMRS, beamforming)                      |
| **Resource Grid**          |  Yes            |  Yes             | Grid structure stored in `rxdataF[Rx][symbol][subcarrier]`<br>See `nr_slot_fep_ul()` and `nr_common.h`                                 |
| **Resource Elements (RE)** |  Yes            |  Yes             | Each `symbol × subcarrier` unit; handled in `nr_extract_rbs()` and `dmrs_symbol_map[]`                                                 |
| **Resource Block (RB)**    |  Yes            |  Yes             | PRB = 12 subcarriers; used for `N_RB_DL`, `N_RB_UL` channel mapping                                                                    |

## Aerial CUDA-Accelerated RAN Overall Carrier Aggregation

| **Feature**             | **Aerial cuPHY** | **OAI Support**       | **OAI Implementation Notes**                                         |
| ----------------------- | ---------------- | --------------------- | -------------------------------------------------------------------- |
| **Carrier Aggregation** |  Yes            |  (simulation only)   | Supports basic 2CC aggregation (DU mode); MAC layer needs extension  |
| **Narrowband CA**       |  Yes            |  Not fully supported | NB-IoT CA not supported; only single-band simulation available       |
| **4CC Aggregation**     |  Yes            | ⚠ Experimental       | Can be simulated via multi-gNB setup, but not full CA implementation |

## Aerial CUDA-Accelerated RAN PHY Modulation Mapper

| **Modulation Scheme** | **Aerial cuPHY** | **OAI Support** | **OAI Implementation Notes**                                                                  |
| --------------------- | ---------------- | --------------- | --------------------------------------------------------------------------------------------- |
| **π/2 BPSK**          |  Yes            |  Yes           | Used in PUSCH with transform precoding<br>Handled in `nr_modulation.c`  |
| **BPSK**              |  Yes            |  Yes           | Used in PBCH and special channels like PRACH                                                  |
| **QPSK**              |  Yes            |  Yes           | Standard modulation for control and low-rate data channels                                    |
| **16QAM**             |  Yes            |  Yes           | Used for moderate-rate PDSCH/PUSCH                                                            |
| **64QAM**             |  Yes            |  Yes           | Supported in `nr_modulation()` for higher throughput                                          |
| **256QAM**            |  Yes            |  Yes           | Supported in OAI with `Qm=8`, enabled via DCI configuration                                   |

## Aerial CUDA-Accelerated RAN PHY Sequence Generation

| **Feature**                  | **Aerial cuPHY** | **OAI** |
| ---------------------------- | ---------------- | ------- |
| **Gold Sequence Generation** |  Yes            |  Yes   | 
| **Low-PAPR Type 1**          |  Yes            |  Yes   | 
| **Low-PAPR Type 2**          |  Yes            |  Yes   | 

## TS 38.211 Channels
**Aerial CUDA-Accelerated RAN Overall Channel - PDSCH(PHY DL Shared Channel)**

<img width="817" height="490" alt="image" src="https://github.com/user-attachments/assets/af9382c1-ddef-414d-b710-e91240451df5" />

<img width="800" height="305" alt="image" src="https://github.com/user-attachments/assets/7a6323a5-e088-4822-a77e-97f554740285" />

**OAI PDSCH** 

| **Feature**                     | **Configuration**               | **OAI support** | 
| ------------------------------- | ------------------------------- | ---------- | 
| **Scrambling**                  | -                               | Y          | 
| **Modulation schemes**          | QPSK / 16QAM / 64QAM / 256QAM   | Y          | 
| **Transmission Mode**           | 4T4R SU-MIMO（最多 4 layers）       | Y          | 
|                                 | 64T64R MU-MIMO（最多 16 layers）    | N     | 
| **Number of codewords**         | 1                               | Y          | 
|                                 | 2                               | Y          | 
| **Number of antenna ports**     | 1000–1011                       | Y          | 
| **Number of physical antennas** | 4                               | Y          | 
|                                 | 64                              | N     | 
| **Beamforming weights**         | Manual matrix (BF m2)           | Y      | 
| **Precoding**                   | Non-codebook                    | Y          | 
|                                 | Pre-coding weight (manual)      | Y          | 
|                                 | Type I Single-Panel Codebook    | Y）      | 
|                                 | Type I Multi-Panel Codebook     | N          | 
|                                 | Type II Codebook                | N          | 
|                                 | Type II Port Selection Codebook | N          | 
| **PDSCH Mapping Type**            | Type A/B                 | Y          | 
| **Resource Allocation Type**      | Type 0 (bitmap-based)  | Y          | 
|                                   | Type 1                 | Y          | 
| **VRB to PRB Mapping**            | Non-interleaved        | Y          | 
|                                   | Interleaved            | Y          | 
| **PDSCH DMRS CDM Groups**         | 1                      | Y          | 
|                                   | 2                      | Y          | 
|                                   | 3                      | (N/A)    | 
| **Number of PDSCH users per TTI** | 16                     | Y          | 
| **Power Control**                 | PDSCH                  | Y          |
|                                   | DMRS - PDSCH           | Y          | 


| **Feature**                       | **Configuration**                  | **Aerial 支援** | **OAI 支援** | 
| --------------------------------- | ---------------------------------- | ------------- | ---------- |
| **Scrambling**                    | Up to 2 codewords                  | N             | only1      |
| **CORESET**                       | Normal / RMSI                      | Y             | Y          | 
| **SSB-RMSI CORESET multiplexing** | Pattern 1                          | Y             | Y          | 
| **Aggregation Level**             | 1, 2, 4, 8, 16                     | Y             | Y          | 
| **Modulation**                    | QPSK                               | Y             | Y          | 
| **Layer Mapping**                 | Supported                          | Y             | Y          | 
| **Antenna Port Mapping**          | Supported                          | Y             | Y          | 
| **Mapping to VRBs**               | Supported                          | Y             | Y          | 
| **VRB to PRB Mapping**            | Non-interleaved                    | Y             | Y          | 
| **Polar Code**                    | Block length up to 128 bits        | Y             | Y          | 
| **DMRS**                          | m-sequence                         | Y             | Y          | 
| **CCE to REG Mapping**            | Non-interleaved / Interleaved      | Y             | Y          | 
| **OFDM Symbols per CORESET**      | 1 / 2 / 3                          | Y             | Y          | 
| **Power Control**                 | PDCCH / DMRS-PDCCH                 | Y             | Y          | 
| **DCI Format**                    | 0\_0 / 0\_1 / 1\_0 / 1\_1 / 2\_x   | NA        | Y      | 
| **Precoding**                     | Precoding Matrix Index-based in DU | Y             | Y          | 
