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
