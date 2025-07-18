# nr_rx_pusch_tp
```
┌─────────────────────────────────────────────────────────────┐
│ 【1】Initialization and Parameter Setup                     │1153 – 1167
│ ▸ Compute BWP start subcarrier (bwp_start_subcarrier)       │
│ ▸ Print debug logs                                           │
│                                                             │
│ 🔹Inputs:                                                    │
│   - gNB, ulsch_id, frame, slot, harq_pid, beam_nb           │
│ 🔹Outputs: None (initialization of local variables)          │
│ 🔹Called functions: None                                     │
└─────────────────────────────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────────────────────────────┐
│ 【2】Uplink Channel Estimation                              │
│ ▸ Loop over DMRS symbols                                    │1166 – 1247
│   - Call nr_pusch_channel_estimation()                      │
│   - Call nr_gnb_measurements() to compute SNR and power     │
│                                                             │
│ 🔹Outputs:                                                   │
│   - Updates gNB->pusch_vars.ul_ch_estimates                 │
│   - Updates noise power (nvar), max channel gain (max_ch)   │
│ 🔹Called functions:                                          │
│   - `nr_pusch_channel_estimation()`                         │
│   - `nr_gnb_measurements()`                                 │
│   - `allocCast2D()`                                         │
│   - `signal_energy_nodc()`                                  │
│   - `average_u32()`                                         │
└─────────────────────────────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────────────────────────────┐
│ 【3】Average Noise Energy                                    │
│ ▸ Average nvar across antennas and layers                   │1249
│                                                             │
│ 🔹Output: uint32_t nvar                                      │
│ 🔹Called functions: None                                     │
└─────────────────────────────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────────────────────────────┐
│ 【4】(Optional) Time Domain Channel Estimation Averaging     │
│ ▸ If enabled: gNB->chest_time == 1                          │
│                                                             │1251 – 1263 
│ 🔹Called function: `nr_chest_time_domain_avg()`             │
└─────────────────────────────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────────────────────────────┐
│ 【5】Scrambling Initialization & G-bit Calculation           │
│ ▸ Call nr_codeword_unscrambling_init()                      │
│ ▸ Call nr_get_G() to compute total bits per slot            │
│                                                             │
│ 🔹Outputs:                                                   │
│   - scramblingSequence[], G, unav_res                       │
│ 🔹Called functions:                                          │
│   - `set_ptrs_symb_idx()`                                   │
│   - `get_ptrs_symbols_in_slot()`                            │
│   - `nr_get_G()`                                            │
│   - `nr_codeword_unscrambling_init()`                       │
└─────────────────────────────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────────────────────────────┐
│ 【6】Channel Gain and log2_maxh Computation                 │
│ ▸ Extract RBs from rxdataF and ul_ch_estimates              │
│ ▸ Call:                                                     │
│   - nr_ulsch_extract_rbs()                                  │
│   - nr_ulsch_scale_channel()                                │
│   - nr_ulsch_channel_level()                                │
│                                                             │
│ 🔹Output:                                                    │
│   - pusch_vars->log2_maxh                                   │
│ 🔹Called functions:                                          │
│   - `get_valid_dmrs_idx_for_channel_est()`                  │
│   - `nr_ulsch_extract_rbs()`                                │
│   - `nr_ulsch_scale_channel()`                              │
│   - `nr_ulsch_channel_level()`                              │
└─────────────────────────────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────────────────────────────┐
│ 【7】LLR Demodulation (Multithreaded)                        │
│ ▸ Divide symbols into groups by numSymbols                  │
│ ▸ For each group call nr_pusch_symbol_processing()         │
│                                                             │
│ 🔹Output:                                                    │
│   - pusch_vars->llr[]                                       │
│ 🔹Called functions:                                          │
│   - `get_nb_re_pusch()`                                     │
│   - `nr_pusch_symbol_processing()`                          │
│   - `pushTpool()`, `init_task_ans()`, `join_task_ans()`     │
└─────────────────────────────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────────────────────────────┐
│ 【8】Copy Data to Scope Debug Buffers                        │
│ ▸ Copy demodulated IQ to scope: gNBscopeCopyUnsafe()        │
│ ▸ Copy LLRs: gNBscopeCopyWithMetadata()                     │
│                                                             │
│ 🔹Called functions:                                          │
│   - `gNBTryLockScopeData()`                                 │
│   - `gNBscopeCopyUnsafe()`                                  │
│   - `gNBscopeCopyWithMetadata()`                            │
│   - `gNBunlockScopeData()`                                  │
└─────────────────────────────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────────────────────────────┐
│ 【9】Function Ends                                           │
│ 🔹Return: 0 (Success)                                       │
└─────────────────────────────────────────────────────────────┘
```
```
nr_rx_pusch_tp(PHY_VARS_gNB *gNB,
                   uint8_t ulsch_id,//Upstream Stream ID
                   uint32_t frame,
                   uint8_t slot,
                   unsigned char harq_pid,//HARQ Process Number
                   int beam_nb)//Corresponding beam number
```
- **frame_parms**//gNB Frame structure setting
- **rel15_ul** //The UE's uplink PUSCH configuration (transmitted by FAPI)
- **pusch_vars**//Temporarily store PUSCH data in process, such as receive buffer and channel estimation results
- **bwp_start_subcarrier**//Calculate the starting subcarrier position of the UE's uplink frequency domain resources in the FFT buffer
- **end_symbol**//Calculate the OFDM symbol position where PUSCH transmission ends
- **if (dmrs_symbol_flag == 1**//When the OFDM symbol being processed is a DMRS (Demodulation Reference Signal) symbol, the DMRS-specific processing flow is executed, such as channel estimation.
- **for (int nl = 0; nl < rel15_ul->nrOfLayers; nl++)**//Perform Channel Estimation for each MIMO Layer of PUSCH and accumulate the estimated noise power (nvar) because Uplink PUSCH supports multi-layer transmission, so UE can use 1~4 layers to send data (multi-layer PUSCH), The gNB must estimate the channel individually for each layer

## nr_pusch_channel_estimation()//nr_ul_channel_estimaition.c
```
nr_pusch_channel_estimation(gNB,//Pointer to the gNB instance
                            slot,//Time-domain slot index
                            nl,//Layer index
                            get_dmrs_port(nl, rel15_ul->dmrs_ports),//DMRS antenna port corresponding to this layer
                            symbol,/The current DMRS OFDM symbol being processed
                            ulsch_id,//Identifier for the corresponding UE
                            beam_nb,//Beam index (used if beamforming is applied)
                            bwp_start_subcarrier,//Frequency-domain starting subcarrier in the FFT buffer
                            rel15_ul,//FAPI PUSCH configuration for this UE
                            &max_ch,//Output: estimated maximum channel power  
                            &nvar_tmp);//Output: estimated noise power

```
**Generate DMRS (Theoretical Reference Signal)**
```
if (pusch_pdu->transform_precoding == transformPrecoder_disabled)
Process description in CP-OFDM mode
Capture DMRS Gold sequence//const uint32_t *gold = nr_gold_pusch()
```
**CP-OFDM mode**
```
nr_gold_pusch()//Generate DMRS gold sequence for PUSCH (actually reuse nr_gold_pdsch())
uint64_t x2tmp0 = (((uint64_t)symbols_per_slot * slot + symbol + 1) * (((uint64_t)nid << 1) + 1)) << 17;
uint32_t x2 = (x2tmp0 + (nid << 1) + nscid) % (1U << 31);
this Calculation method in TS 38.211 7.4.1.1 
call gold_cache()//This function will use x2 as the seed and return a pseudo-random Gold sequence
```
**gold_cache()**//TS 138 211 5.2.1 pseudo-random-sequence generation
- Calculating the Gold Sequence **lte_gold_generic**

```
pusch_dmrs_type_t dmrs_type = pusch_pdu->dmrs_config_type == NFAPI_NR_DMRS_TYPE1 ? pusch_dmrs_type1 : pusch_dmrs_type2;
3GPP TS 38.211 §6.4.1.1.3//Is it to determine whether the DMRS "Configuration Type" is Type 1 or Type 2, and where it corresponds to?
Type 1 Lower density, inserting a small number of DMRS per RB
Type 2 Higher density, dense insertion in the frequency domain
```
```
float beta_dmrs_pusch = get_beta_dmrs_pusch(pusch_pdu->num_dmrs_cdm_grps_no_data, dmrs_type);//Calculate the normalization coefficient based on DMRS settings (Type1 or Type2, number of CDM groups)
int16_t dmrs_scaling = (1 / beta_dmrs_pusch) * (1 << 14);
get_beta_dmrs_pusch()//definition in nr_common.c
```

```
nr_pusch_dmrs_rx(gNB,
                 Ns,
                 gold,
                 pilot,//rxdataF corresponds to the DMRS RE captured by the symbol
                 (1000 + p),
                 0,
                 nb_rb_pusch,
                 (pusch_pdu->bwp_start + pusch_pdu->rb_start) * NR_NB_SC_PER_RB,
                 pusch_pdu->dmrs_config_type,//DMRS configuration type (Type 1 = 0, Type 2 = 1)
                 dmrs_scaling);//Power Scaling Factor
  } //do DMRS demodulation and channel estimation
```
```
typedef int array_of_w[2];
const array_of_w *wf = (dmrs_type == pusch_dmrs_type1) ? wf1 : wf2;
const array_of_w *wt = (dmrs_type == pusch_dmrs_type1) ? wt1 : wt2;
wf,wt //Pointer to the weight table, used to store the weight factor selected according to dmrs_type
wf1,wf2//Demodulation weighting coefficients corresponding to DMRS type 1 and type 2, used in time/frequency domain

const int dmrs_offset = re_offset / ((dmrs_type == pusch_dmrs_type1) ? 2 : 3);//DMRS insertion spacing: Type 1 → every 2 REs, Type 2 → every 3 REs

if ((p>=1000) && (p<((dmrs_type==pusch_dmrs_type1) ? 1008 : 1012)))//Port range validity check
Type 1: supports up to 8 ports (p=1000~1007)
Type 2: supports up to 12 ports (p=1000~1011)

for (int k = 0; k < nb_pusch_rb * nb_dmrs; k++) {
  int i = k + dmrs_offset;
  int w = (wf[p - 1000][i & 1]) * (wt[p - 1000][lp]);
  output[k] = get_modulated(nr_gold_pusch, i, w == 1);
  output[k] = c16mulRealShift(output[k], dmrs_scaling, 14);
}//Calculate the symbol on each DMRS resource point
get_modulate()//Gold → QPSK mapping, select conjugate or reverse according to the weight
c16mulRealShift()//Multiply by dmrs_scaling
```
**get_modulate**
```
example:
Gold bits = 0b01100011...
idx_gold = 2
bits = 10 → index = 1 → QPSK = (-1, +1)
若 inverse=false → 取反變成 (1, -1)
```

- Each output[k] is a complex QPSK symbol
- These symbols are the DMRS reference signals "generated by the receiver itself" and are used to:
  - Channel estimation (subtract and compare with the pilot signal actually received)
  - Demodulation compensation

**DFT-s-OFDM Mode**
```
const int index = get_index_for_dmrs_lowpapr_seq(nb_rb_pusch * (NR_NB_SC_PER_RB / 2));
NR_NB_SC_PER_RB = 12: 12 subcarriers per RB
Because Type 1 DMRS is placed every 2 REs → only 6 DMRS REs are used per RB
So the total DMRS length to be generated is nb_rb_pusch * 6
```
**get_index_for_dmrs_lowpapr_seq()**
```
The input parameter num_dmrs_res: represents the number of resource elements (REs) actually allocated to DMRS.
The returned value is an index used to find the corresponding item of the DMRS low-PAPR sequence.

if (index >= MAX_INDEX_DMRS_UL_ALLOCATED_REs)
  index = MAX_INDEX_DMRS_UL_ALLOCATED_REs - 1;
//If the calculated index exceeds the upper limit, it is set to the maximum available index (to avoid out-of-bounds)

for (; index >= 0; index--) {
  if (dmrs_ul_allocated_res[index] == num_dmrs_res)
    break;
}//dmrs_ul_allocated_res[] is a global table that lists the number of supported REs and their corresponding low-PAPR sequences.

const uint8_t u = pusch_pdu->dfts_ofdm.low_papr_group_number;//Controlling the Phase Rotation Sequence TS 38.211 §5.2.2.2 Total 30 groups (0~29)
const uint8_t v = pusch_pdu->dfts_ofdm.low_papr_sequence_number;
c16_t *dmrs_seq = gNB_dmrs_lowpaprtype1_sequence[u][v][index];

```
**464~513**
- The main function is to generate a demodulation reference signal (DMRS) pilot sequence at the receiving end for subsequent channel estimation.
Depending on whether transform precoding (i.e. SC-FDMA) is enabled, different sequence generation methods are used.
- Case 1: Transform precoding disabled →（CP-OFDM）
- case 2: Transform precoding enabled → DFT-s-OFDM(SC-FDMA)
```
                   +---------------------------+
                   | Check transform precoding |
                   +---------------------------+
                               |
        +----------------------+------------------+
        |                                         |
 [transformPrecoder_disabled]         [transformPrecoding enabled]
         |                                         |
 Generate Gold Sequence                   Get ZC sequence index
 (38.211 §5.2.1 + §6.4.1.1.1)               from dmrs_ul_allocated_res[]
         |                                         |
  BPSK/QPSK Mapping                        Load φ(n) rotated ZC seq(§5.2.2.2)
         |                                         |
 Apply CDM (Type1/Type2)                       Get from
         |                               gNB_dmrs_lowpaprtype1_sequence[u][v][index]
 Apply Beta Scaling                               |
         |                                  Use nr_pusch_lowpaprtype1_dmrs_rx()
nr_pusch_dmrs_rx()                              (Type 1 only)
         |
      pilot[] ← filled                        pilot[] ← filled
```

**nr_ul_channel_estimaition.c 524~607**
```
int nest_count = 0;//Presumably "noise estimation count", which is used to count the number of samples when calculating noise power
  uint64_t noise_amp2 = 0;//The accumulated squared noise power (64-bit), usually used to calculate the average noise intensity nvar
  delay_t *delay = &gNB->ulsch[ul_id].delay;
  memset(delay, 0, sizeof(*delay));

  int nb_antennas_rx = gNB->frame_parms.nb_antennas_rx;//Get the number of receive antennas the gNB currently has (for example, 4 for 4x4 MIMO).
  delay_t delay_arr[nb_antennas_rx];//Delay structure corresponding to each antenna
  uint64_t noise_amp2_arr[nb_antennas_rx];//The accumulated noise power of each antenna
  int max_ch_arr[nb_antennas_rx];//Channel maximum intensity
  int nest_count_arr[nb_antennas_rx];//Noise estimation times

  for (int i = 0; i < nb_antennas_rx; ++i) {
    max_ch_arr[i] = *max_ch;
    nest_count_arr[i] = nest_count;
    noise_amp2_arr[i] = noise_amp2;
    delay_arr[i] = *delay;
  }//Copy the initial value of the single variable above to the corresponding array elements of all antennas

int numAntennas = gNB->dmrs_num_antennas_per_thread;//Determine how many antennas to process per job
int num_jobs = CEILIDIV(gNB->frame_parms.nb_antennas_rx, numAntennas);//Calculate the number of jobs required
For example: If there are 4 antennas and each job processes 2 antennas, then num_jobs = 2

//Job task encapsulation and parameter initialization
puschAntennaProc_t *rdata = &rdatas[job_id];
task_t task = {.func = nr_pusch_antenna_processing, .args = rdata};

//Fill in the parameters required by the job. Each job processes data from a group of antennas independently.
rdata->Ns = Ns;
    rdata->nl = nl;
    rdata->p = p;
    rdata->symbol = symbol;
    rdata->aarx = job_id * numAntennas;
    rdata->numAntennas = numAntennas;
    rdata->bwp_start_subcarrier = bwp_start_subcarrier;
    rdata->pusch_pdu = pusch_pdu;
    rdata->max_ch = &max_ch_arr[rdata->aarx];
    rdata->pilot = pilot;
    rdata->nest_count = &nest_count_arr[rdata->aarx];
    rdata->noise_amp2 = &noise_amp2_arr[rdata->aarx];
    rdata->delay = &delay_arr[rdata->aarx];
    rdata->beam_nb = beam_nb;
    rdata->frame_parms = &gNB->frame_parms;
    rdata->pusch_vars = &gNB->pusch_vars[ul_id];
    rdata->chest_freq = gNB->chest_freq;
    rdata->rxdataF = gNB->common_vars.rxdataF;
    rdata->scope = gNB->scopeData;
    rdata->ans = &ans;

//Calling Execution Functions: Parallelism and Synchronization
if (job_id == num_jobs - 1) {
  nr_pusch_antenna_processing(rdata);  // 最後一個直接在主線程執行（避免排程延遲）
} else {
  pushTpool(&gNB->threadPool, task);  // 其餘交由 thread pool 處理
}//For the sake of efficiency, OAI does not assign the last job to the thread pool, but executes it synchronously on the main thread.

Compile statistics
for (int aarx = 0; aarx < gNB->frame_parms.nb_antennas_rx; aarx++) {
    *max_ch = max(*max_ch, max_ch_arr[aarx]);//Maximum channel value extraction
    noise_amp2 += noise_amp2_arr[aarx];//The total noise energy
    nest_count += nest_count_arr[aarx];//Estimated sample size
  }

Get the maximum delay
*delay = delay_arr[0];
for (int aarx = 1; aarx < nb_antennas_rx; aarx++) {
  if (delay_arr[aarx].est_delay >= delay->est_delay)
    *delay = delay_arr[aarx];
}
Why find the maximum delay?
Adjust Sync, Remove front and back noise, SNR / decoding reference

Calculate the average noise energy
if (nvar && nest_count > 0) {
    *nvar = (uint32_t)(noise_amp2 / nest_count);
  }
```
```
[Start: Begin Channel Estimation Procedure]

  ├─► Check if transform precoding is enabled
  │     ├─ If disabled (transformPrecoder_disabled):
  │     │     ├─ Retrieve Gold sequence (nr_gold_pusch)
  │     │     ├─ Determine DMRS type (Type 1 or Type 2)
  │     │     ├─ Compute power boosting factor (beta_dmrs_pusch)
  │     │     ├─ Compute DMRS scaling (dmrs_scaling)
  │     │     └─ Generate pilot symbols using nr_pusch_dmrs_rx()
  │     │
  │     └─ If enabled (transform precoding):
  │           ├─ Calculate index from number of PRBs (get_index_for_dmrs_lowpapr_seq)
  │           ├─ Retrieve low-PAPR group (u) and sequence (v)
  │           ├─ Fetch low-PAPR ZC sequence: gNB_dmrs_lowpaprtype1_sequence[u][v][index]
  │           └─ Generate pilot using nr_pusch_lowpaprtype1_dmrs_rx()
  │

  ├─► Initialize per-antenna variables
  │     ├─ max_ch_arr[]
  │     ├─ noise_amp2_arr[]
  │     ├─ nest_count_arr[]
  │     └─ delay_arr[]

  ├─► Launch per-antenna parallel channel estimation (multi-threaded)
  │     ├─ Divide receive antennas into jobs of `numAntennas` each
  │     ├─ Initialize each job’s parameter struct (puschAntennaProc_t)
  │     ├─ If it’s the last job → run inline
  │     └─ Else → dispatch to threadPool with nr_pusch_antenna_processing()

  ├─► Wait for all jobs to complete (join_task_ans)

  ├─► Aggregate results from all antennas
  │     ├─ max_ch = max of max_ch_arr[]
  │     ├─ noise_amp2 = sum of noise_amp2_arr[]
  │     ├─ nest_count = sum of nest_count_arr[]
  │     └─ delay = maximum from delay_arr[] → stored in gNB->ulsch[].delay

  ├─► If nvar != NULL and nest_count > 0:
  │     └─ Compute noise variance: *nvar = noise_amp2 / nest_count

  └─► return 0
```
**nr_pusch_antenna_processing()**
- The main processing unit in the gNB receiver performs PUSCH channel estimation for each receive antenna. It estimates the channel condition of each resource element by demodulating the DMRS (Demodulation Reference Signal) sequence.
```
[Start: nr_pusch_antenna_processing()]
        |
        v
[1. Initialization and Parameter Extraction]
  - Unpack rdata fields
  - Compute symbolSize, slot_offset, delta, etc.
        |
        v
[2. Access RX buffer and Channel Estimate Memory]
  - rxdataF: frequency-domain received signal
  - ul_ch: buffer to store channel estimate results
        |
        v
[3. Branch by DMRS Type and chest_freq]
        |
        +--------------------------+-----------------------------+------------------------------+-------------------------------+
        |                          |                             |                              |                               |
        v                          v                             v                              v
[Type1 + Freq Interp]     [Type2 + Freq Interp]         [Type1 No Interp]           [Type2 No Interp]
        |                          |                             |                              |
        v                          v                             v                              v
[3.1 LS Estimation]       [3.2 LS Estimation]           [3.3 Averaging 6 DMRS REs]  [3.4 Averaging 4 DMRS REs]
  - Pilot + rxdataF          - Pilot + rxdataF              - Use `c32x16cumul...`       - Use `c32x16mulShift`, etc.
  - Function: `c32x16maddShift`    - Function: `c16addShift`        - No interpolation            - Sum and average
        |
        v
[3.1.1 Delay Estimation] [3.2.1 Delay Estimation]       |                              |
  - Call `nr_est_delay()`       - Call `nr_est_delay()`       |                              |
        |                          |                             |                              |
        v                          v                             |                              |
[3.1.2 Lookup Delay Table] [3.2.2 Lookup Delay Table]    |                              |
  - Call `get_delay_idx()`      - Call `get_delay_idx()`      |                              |
  - Get `ul_delay_table[]`      - Get `ul_delay_table[]`      |                              |
        |                          |                             |                              |
        v                          v                             |                              |
[3.1.3 Freq-Domain Interp] [3.2.3 Freq-Domain Interp]    |                              |
  - Use `c16mulShift()`          - Use `c16mulShift()`         |                              |
  - Use `c16multaddVectRealComplex()` or `multadd_real_four...()` |
        |                          |                             |                              |
        v                          v                             |                              |
[3.1.4 Inverse Delay Comp] [3.2.4 Inverse Delay Comp]    |                              |
  - Call `get_delay_idx()`      - Call `get_delay_idx()`      |                              |
  - Apply `c16mulShift()`       - Apply `c16mulShift()`       |                              |
  - Update noise_amp2, nest_count |
        |                          |                             |                              |
        +--------------------------+-----------------------------+------------------------------+
                                      |
                                      v
[4. Final Update of Result Fields]
  - `*(rdata->noise_amp2) = noise_amp2`
  - `*(rdata->nest_count) = nest_count`
                                      |
                                      v
[5. Mark Task Completion]
  - Call `completed_task_ans(rdata->ans)`
                                      |
                                      v
                                   [End]

```
**nr_est_delay()**
- After IDFT conversion back to the time domain, find the maximum energy delay point (delay tap) of the channel response to estimate the transmission delay est_delay

**get_delay_idx**
This is the get_delay_idx() function, which is used to map the estimated delay value delay to a valid index value (delay_idx) so as to retrieve the corresponding phase compensation vector from the delay compensation table delay_table.

| Function Name                                       | Description                                                       |
| --------------------------------------------------- | ----------------------------------------------------------------- |
| `c32x16maddShift()`                                 | Complex multiply-add with shifting (used for LS estimation)       |
| `c16mulShift()`                                     | Scaled complex multiplication (delay compensation, interpolation) |
| `nr_est_delay()`                                    | Converts LS estimate to time domain and detects delay             |
| `get_delay_idx()`                                   | Converts estimated delay to lookup table index                    |
| `c16multaddVectRealComplex()`                       | Real-coefficient FIR vector multiply-add                          |
| `c32x16cumulVectVectWithSteps()`                    | Averages REs across PRB for channel estimate                      |
| `c16x32div()`                                       | Complex division (used for averaging)                             |
| `multadd_real_four_symbols_vector_complex_scalar()` | Type2 interpolation                                               |
| `completed_task_ans()`                              | Marks this thread task as finished                                |

**nr_gnb_measurements()**
```
void nr_gnb_measurements(
    PHY_VARS_gNB *gNB,
    NR_gNB_ULSCH_t *ulsch,//ULSCH structure, includes HARQ and measurement fields
    NR_gNB_PUSCH *pusch_vars,//
    unsigned char symbol,//	OFDM symbol index being processed
    uint8_t nrOfLayers//Number of MIMO layers
)
```

- Get receiving gain and RX Gain Offset
```
rx_gain = ru->rfdevice.openair0_cfg->rx_gain[0];
  rx_gain_offset = ru->rfdevice.openair0_cfg->rx_gain_offset[0];
```

- Calculate channel energy and power sum
```
for (int aarx = 0; aarx < fp->nb_antennas_rx; aarx++) {
  rx_power[aarx] = 0;
  for (int aatx = 0; aatx < nrOfLayers; aatx++) {
    rx_spatial_power[aatx][aarx] = signal_energy_nodc(...);
    rx_power[aarx] += rx_spatial_power[aatx][aarx];
  }
  rx_power_tot += rx_power[aarx];
}
For each receiving antenna aarx and each layer aatx:
  Take out the channel estimation value array corresponding to DMRS and calculate the energy through signal_energy_nodc().
  Add up the energy of all layers to get the total received power of this antenna.
Finally calculate the overall total power rx_power_tot.
```

- Convert to dB units and store CQI results
```
rx_power_tot_dB = dB_fixed(rx_power_tot);//dB conversion function for fixed decimal format
ulsch_measurements->wideband_cqi_tot = dB_fixed2(rx_power_tot, meas->n0_power_tot);//CQI is calculated based on the ratio of Rx Energy to Noise Power.
```

- Calculate RSSI (Received Power Index)
```
ulsch_measurements->rx_rssi_dBm =
    rx_power_avg_dB + 30 - SQ15_SQUARED_NORM_FACTOR_DB
    - (rx_gain - rx_gain_offset) - dB_fixed(fp->ofdm_symbol_size);

The formula for RSSI dBm can be understood as:
RSSI(dBm) ≈ rx_power_dBFS + 30 - Norm_offset - Gain - log(N_fft)

```

- output
```
struct ulsch->ulsch_measurements:
wideband_cqi_tot //Calculated overall CQI (expressed in dB),
rx_rssi_dBm//Calculated RSSI (received power per RE, in dBm)
```

**allocCast2D**
- Convert the 1D array gNB->measurements.n0_subband_power to a 2D array index n0_subband_power, so you can access the noise power of each receive antenna and each RB using n0_subband_power[receive antenna][RB number]

**1208-1245**
```
for (int aarx = 0; aarx < frame_parms->nb_antennas_rx; aarx++) {//Process each receiving antenna one by one (aarx: Antenna Array Rx index)

if (symbol == rel15_ul->start_symbol_index) {//Initialize Energy Slot
  pusch_vars->ulsch_power[aarx] = 0;
  pusch_vars->ulsch_noise_power[aarx] = 0;
}

//Calculate the start and end subcarrier index of RB
int start_sc = (rel15_ul->bwp_start + rel15_ul->rb_start) * NR_NB_SC_PER_RB;
int middle_sc = frame_parms->ofdm_symbol_size - frame_parms->first_carrier_offset;
int end_sc = (start_sc + rel15_ul->rb_size * NR_NB_SC_PER_RB - 1) % frame_parms->ofdm_symbol_size;

for (int s = rel15_ul->start_symbol_index; s < (rel15_ul->start_symbol_index + rel15_ul->nr_of_symbols); s++)//check all OFDM symbols occupied by the PUSCH

//Calculate the memory location of channel data
int offset0 = ((slot & 3) * frame_parms->symbols_per_slot + s) * frame_parms->ofdm_symbol_size;//The symbol starts in the entire RX buffer.
int offset = offset0 + (frame_parms->first_carrier_offset + start_sc) % frame_parms->ofdm_symbol_size;//Add carrier offset and PUSCH start subcarrier → Get the subcarrier start point to be analyzed
c16_t *ul_ch = &gNB->common_vars.rxdataF[beam_nb][aarx][offset];//Frequency domain data pointing to the antenna and the symbol

//Compensation when crossing FFT boundaries
if (end_sc < start_sc) {// Energy is divided into two parts: first half + second half
            int64_t symb_energy_aux = signal_energy_nodc(ul_ch, middle_sc - start_sc) * (middle_sc - start_sc);
            ul_ch = &gNB->common_vars.rxdataF[beam_nb][aarx][offset0];
            symb_energy_aux += (signal_energy_nodc(ul_ch, end_sc + 1) * (end_sc + 1));
            symb_energy += symb_energy_aux / (rel15_ul->rb_size * NR_NB_SC_PER_RB);
          } else {
            symb_energy += signal_energy_nodc(ul_ch, rel15_ul->rb_size * NR_NB_SC_PER_RB);
          }

```

**average_u32()**
- Compute the average of an array of uint32_t and speed it up using AVX2 SIMD instructions
---
- nvar /= (rel15_ul->nr_of_symbols * rel15_ul->nrOfLayers * frame_parms->nb_antennas_rx);//normalize

---
## Time Domain Channel Estimation Averaging 1251~1262
```
if (gNB->chest_time == 1)
    nr_chest_time_domain_avg(frame_parms,
                             pusch_vars->ul_ch_estimates,//Estimated PUSCH channel (complex frequency domain data)
                             rel15_ul->nr_of_symbols,
                             rel15_ul->start_symbol_index,
                             rel15_ul->ul_dmrs_symb_pos,
                             rel15_ul->rb_size);
```
**nr_chest_time_domain_avg()** //definition in dmrs_nr.c

Why do we need to do “time domain averaging?

When multiple OFDM symbols use the same channel (e.g. multiple symbols containing DMRS), the channel estimates of different symbols can be averaged on the time axis to:
- Suppress estimation noise
- Improve estimation stability

## Scrambling Initialization & G-bit Calculation   
### Scrambling initialization 1264~1285
- Calculate the number of available REs
```
int number_dmrs_symbols = 0;
for (int l = rel15_ul->start_symbol_index; l < end_symbol; l++)
  number_dmrs_symbols += ((rel15_ul->ul_dmrs_symb_pos)>>l) & 0x01;//Calculate how many symbols in a slot are configured as DMRS

if (rel15_ul->dmrs_config_type == pusch_dmrs_type1)
  nb_re_dmrs = 6 * rel15_ul->num_dmrs_cdm_grps_no_data;//type 1 → 6 REs per group
else
  nb_re_dmrs = 4 * rel15_ul->num_dmrs_cdm_grps_no_data;//type 2 → 4 REs per group
```
```
if (rel15_ul->pdu_bit_map & PUSCH_PDU_BITMAP_PUSCH_PTRS) {
    uint16_t ptrsSymbPos = 0;
    set_ptrs_symb_idx(&ptrsSymbPos,
                      rel15_ul->nr_of_symbols,
                      rel15_ul->start_symbol_index,
                      1 << rel15_ul->pusch_ptrs.ptrs_time_density,
                      rel15_ul->ul_dmrs_symb_pos);//Calculate which symbols contain PTRS
    int ptrsSymbPerSlot = get_ptrs_symbols_in_slot(ptrsSymbPos, rel15_ul->start_symbol_index, rel15_ul->nr_of_symbols);//Calculate the number of PTRS symbols in the slot
    int n_ptrs = (rel15_ul->rb_size + rel15_ul->pusch_ptrs.ptrs_freq_density - 1) / rel15_ul->pusch_ptrs.ptrs_freq_density;//Frequency domain resources occupied by PTRS in each symbol (depending on the frequency domain density)
    unav_res = n_ptrs * ptrsSymbPerSlot;//How much RE does PTRS occupy in total?
  }
```
**set_ptrs_symb_idx()** definition in ptrs_nr.c
**get_ptrs_symbols_in_slot** definition in ptrs_nr.c

### get how many bit in a slot 1287~1296
```
int G = nr_get_G(rel15_ul->rb_size,
                   rel15_ul->nr_of_symbols,
                   nb_re_dmrs,
                   number_dmrs_symbols, // number of dmrs symbols irrespective of single or double symbol dmrs
                   unav_res,
                   rel15_ul->qam_mod_order,
                   rel15_ul->nrOfLayers);//Calculate the total number of bits available for data transmission in a slot
```
**nr_get_G()** definition in nr_tbs_tools.c
- what is G?
- Determine based on the following conditions:
  - How many RBs and symbols are used (i.e., available time and frequency resources)
  - How many available Resource Elements (REs) are left after deducting DMRS and PTRS
  - The number of bits that each RE can carry (depends on the modulation order and the number of layers)

### initialize scrambling sequence
```
int16_t scramblingSequence[G + 96] __attribute__((aligned(32)));

  nr_codeword_unscrambling_init(scramblingSequence, G, 0, rel15_ul->data_scrambling_id, rel15_ul->rnti);
//scramblingSequence is a Output Array
//This code is used to generate the Gold sequence in advance for the subsequent descrambling step.
```
**nr_codeword_unscrambling_init()** definition in nr_scrambling.c
```
void nr_codeword_unscrambling_init(int16_t *s2, uint32_t size, uint8_t q, uint32_t Nid, uint32_t n_RNTI)
{
  const int roundedSz = (size + 31) / 32;//Calculate the required Gold sequence length (32-bit units)
  uint32_t *seq = gold_cache((n_RNTI << 15) + (q << 14) + Nid, roundedSz);//The returned value is a uint32_t array
  simde__m128i *s128=(simde__m128i *)s2;//Indicator type conversion
  for (int i = 0; i < roundedSz; i++) {
    uint8_t *s8 = (uint8_t *)(seq + i);
    *s128++ = byte2m128i[s8[0]];
    *s128++ = byte2m128i[s8[1]];
    *s128++ = byte2m128i[s8[2]];
    *s128++ = byte2m128i[s8[3]];
  }
}
```
```
//Convert each 32-bit word to the corresponding ±1 (represented as int16_t) format
for (int i = 0; i < roundedSz; i++) {
    uint8_t *s8 = (uint8_t *)(seq + i);
    *s128++ = byte2m128i[s8[0]];
    *s128++ = byte2m128i[s8[1]];
    *s128++ = byte2m128i[s8[2]];
//Each seq[i] is 32-bit (4 bytes), here it is split byte-wise
//Each s8[k] is 8-bit data, corresponding to 8 scrambling bits
//byte2m128i[x]: Look up the table to convert the 8-bit scrambling bits to ±1, and expand it to simde__m128i (128 bits = 8 × int16_t)
//Write 4 groups each time, a total of 4 × 8 = 32 bits (corresponding to 1 seq[i])
```

### first the computation of channel levels 1302~1312
```
int nb_re_pusch = 0, meas_symbol = -1;
  for(meas_symbol = rel15_ul->start_symbol_index; meas_symbol < end_symbol; meas_symbol++) 
    if ((nb_re_pusch = get_nb_re_pusch(frame_parms, rel15_ul, meas_symbol)) > 0)
      break;//Call get_nb_re_pusch(...) to calculate the number of valid REs in the symbol

  AssertFatal(nb_re_pusch > 0 && meas_symbol >= 0,
              "nb_re_pusch %d cannot be 0 or meas_symbol %d cannot be negative here\n",
              nb_re_pusch,
              meas_symbol);
//The function of this code is to select a symbol that can be used for channel estimation and confirm that it has enough PUSCH RE for energy calculation.
```
**get_nb_re_pusch()**
- Calculate the number of Resource Elements (REs) that can be used to transmit data in the symbol based on the PUSCH settings and symbol positions

---
```
int soffset = (slot % RU_RX_SLOT_DEPTH) * frame_parms->symbols_per_slot * frame_parms->ofdm_symbol_size;//Calculate the starting position of the slot in rxdataF (soffset)

  nb_re_pusch = ceil_mod(nb_re_pusch, 16);//Since subsequent processing (such as SIMD vectorization) requires alignment of 16
  int dmrs_symbol;
  if (gNB->chest_time == 0)
    dmrs_symbol = get_valid_dmrs_idx_for_channel_est(rel15_ul->ul_dmrs_symb_pos, meas_symbol);//Return a DMRS symbol index that can be used for channel estimation
  else // average of channel estimates stored in first symbol
    dmrs_symbol = get_next_dmrs_symbol_in_slot(rel15_ul->ul_dmrs_symb_pos, rel15_ul->start_symbol_index, end_symbol);//Find the first DMRS symbol position starting from start_symbol_index
```
**get_valid_dmrs_idx_for_channel_est()** definition in dmrs_nr.c
- It is used to obtain a reasonable DMRS symbol index as the basis for channel estimation.

**get_dmrs_symbols_in_slot()**
- Used to find the "next nearest DMRS symbol" for channel estimation or channel averaging

```
int size_est = nb_re_pusch * frame_parms->symbols_per_slot;
//Calculate the size of the channel estimate
//nb_re_pusch: Number of REs per OFDM symbol
//symbols_per_slot: Number of symbols in a slot (usually 14)

  __attribute__((aligned(32))) int ul_ch_estimates_ext[rel15_ul->nrOfLayers * frame_parms->nb_antennas_rx][size_est];//Declare an array of channel estimates for multiple layers and multiple receive antennas((layer × antenna) )

  memset(ul_ch_estimates_ext, 0, sizeof(ul_ch_estimates_ext));
  int buffer_length = rel15_ul->rb_size * NR_NB_SC_PER_RB;//Count the total number of REs used in a symbol

  c16_t temp_rxFext[frame_parms->nb_antennas_rx][buffer_length] __attribute__((aligned(32)));
```

**nr_ulsch_extract_rbs()**
- Extract the received symbols and channel estimation values of the corresponding RB from rxdataF and fill in rxFext and chFext. It is a pre-step for uplink demodulation

- Case 1: Non-DMRS Symbol
```
if (is_dmrs_symbol == 0)
Directly copy the entire PRB range (12 subcarriers per PRB) to rxFext and chFext
```

- Case 2: DMRS type 1 (6 REs per PRB are DMRS)
```
else if (pusch_pdu->dmrs_config_type == pusch_dmrs_type1)
DMRS occupies 6 out of every 12 REs, so only extract data symbols (interleaved)
Adjust the starting position according to the delta value (currently delta is 0)
```

- Case 3: DMRS type 2 (4 DMRS per PRB)
```
else if (pusch_pdu->dmrs_config_type == pusch_dmrs_type2)
Type 2 uses CDM (Code Division Multiplex), with different jump point locations
2*delta and 2*delta+1 in every 6 REs are DMRS
The rest of the REs are data, copied to the output
```
- output
- temp_rxFext: Received symbols after extraction
- ul_ch_estimates_ext: Extracted channel estimates

```
int avgs = 0;
  int avg[frame_parms->nb_antennas_rx*rel15_ul->nrOfLayers];//Channel energy average variable declaration
  uint8_t shift_ch_ext = rel15_ul->nrOfLayers > 1 ? log2_approx(max_ch >> 11) : 0;//Decide whether to scale the channel estimate based on the number of layers
```
  
