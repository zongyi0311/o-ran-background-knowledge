```mermaid
flowchart TB
    A[Resource element demapping] --> B[Channel estimation<br/>Equalization<br/>IDFT]
    B --> C[Demodulation]
    C --> D[Descrambling]
    D --> E[Decoding]

    style A fill:#f9f,stroke:#333,stroke-width:1px
    style B fill:#bbf,stroke:#333,stroke-width:1px
    style C fill:#bfb,stroke:#333,stroke-width:1px
    style D fill:#ffb,stroke:#333,stroke-width:1px
    style E fill:#fbb,stroke:#333,stroke-width:1px
```

```mermaid
graph TD
P["CP removal & FFT\n(slot FEP)\nnr_slot_fep_ul()"] -.-> A

A["Resource element demapping\n資源元素解映射\nnr_ulsch_extract_rbs()"] --> 
B["Channel estimation / Equalization / IDFT\n通道估計/均衡/IDFT\nnr_pusch_dmrs_rx()\nnr_pusch_channel_estimation()\nnr_ulsch_channel_compensation()\nnr_freq_equalization()\nnr_idft()"] -->
C["Demodulation\n解調(LLR產生)\nnr_ulsch_compute_llr()"] -->
D["Descrambling\n解擾碼\nnr_ulsch_descrambling()"] -->
E["Decoding\nLDPC/HARQ\nnr_ulsch_decoding()"]



```
```mermaid
graph TD
A["Resource element demapping
fn: nr_ulsch_extract_rbs
in: rxdataF[aarx], PRB alloc, symbol
out: rxF_ext[aarx], len(nb_re)"] --> 
B["Equalization / optional IDFT
fn: nr_ulsch_channel_compensation, nr_freq_equalization, nr_idft
in: rxF_ext, H_est(provided), mod_order, layers
out: rxdataF_comp[layer], |h|^2 mags"] -->
C["Demodulation (LLR)
fn: nr_ulsch_compute_llr, nr_ulsch_compute_ML_llr, nr_ulsch_mmse_2layers
in: rxdataF_comp, |h|^2, rho(optional)
out: LLR[layer]"] -->
D["Descrambling (LLR)
fn: nr_codeword_unscrambling_init + apply in symbol loop
in: LLR, scramblingSequence(c_init from rnti, id)
out: LLR_descrambled"] -->
E["Decoding (LDPC / HARQ)
fn: nr_ulsch_decoding
in: LLR_descrambled, TB size A, BG/RM params
out: decoded bits(MAC PDU), CRC/HARQ"]
```

```mermaid
flowchart TD
  A[RX freq grid rxdataF]
  P[UL SCH PDU pusch_pdu]
  F[Frame Params frame_parms]

  NRTP[nr_rx_pusch_tp]
  CE[nr_pusch_channel_estimation]
  MEAS[nr_gnb_measurements]
  TDAVG[nr_chest_time_domain_avg]

  INIT[init resources G LLR scramble]
  CHLV[nr_channel_level]
  SCALE[nr_ulsch_scale_channel]

  SYM[nr_pusch_symbol_processing]
  IRX[inner_rx]

  EXTR[nr_ulsch_extract_rbs]
  COMP[nr_ulsch_channel_compensation]
  EQZ[equalization]
  IDFT[nr_idft]
  LLR1[nr_ulsch_compute_llr or ML]
  MMSE[nr_ulsch_mmse_2layers]
  DEMAP[layer de-mapping]
  UNSCR[unscrambling]
  OUTLLR[output LLR]
  SCOPE[scope or tracer]

  A --> NRTP
  P --> NRTP
  F --> NRTP

  NRTP --> CE
  CE --> MEAS
  CE --> TDAVG

  NRTP --> INIT
  INIT --> CHLV
  CHLV --> SCALE

  NRTP --> SYM
  SYM --> IRX
  IRX --> EXTR
  EXTR --> COMP
  COMP --> EQZ
  EQZ --> IDFT
  EQZ --> MMSE
  MMSE --> LLR1
  EQZ --> LLR1
  LLR1 --> DEMAP
  DEMAP --> UNSCR
  UNSCR --> OUTLLR

  IRX -.-> SCOPE
  CE -.-> SCOPE

```

```mermaid
flowchart LR
  R0[inputs: rxF ul_ch_estimates pusch_pdu frame_parms symbol]
  E1[nr_ulsch_extract_rbs]
  C1[nr_ulsch_channel_compensation]
  C2[nr_freq_equalization]
  TP[nr_idft]
  ZFMMSE[nr_ulsch_mmse_2layers]
  L1[nr_ulsch_compute_llr or ML]
  DM[layer de-mapping]
  US[unscrambling]
  O1[LLR out]
  P1[nr_pusch_ptrs_processing]

  R0 --> E1
  E1 --> C1
  C1 --> C2
  C2 --> TP
  TP --> L1
  C2 --> ZFMMSE
  ZFMMSE --> L1
  L1 --> DM
  DM --> US
  US --> O1
  E1 --> P1
```

```mermaid
flowchart TD
  IN[Inputs : 輸入接收符號rxFext 通道估測chFext 天線數 層數 buffer長度 調變階數 位移量]
  QAM[Select QAM constants : 根據調變階數選常數]
  L[Loop over layers : 逐層處理]
  A[Loop over antennas : 逐支接收天線處理]
  V[Vector loop : SIMD分段處理buffer]
  MRC[Compute conjH*y : 等化並MRC累加到rxComp]
  MAG[If mod>2 : 計算通道能量並更新maga magb magc]
  RHO[If multi-layer : 計算層間交叉相關rho]
  OUT[Outputs : rxComp 通道能量表 rho]

  IN --> QAM --> L --> A --> V --> MRC --> MAG --> RHO --> OUT

```
```mermaid
flowchart TD
  IN[輸入：rxdataF_comp 接收符號、ul_ch_mag 通道能量、ul_ch_magb 輔助能量、symbol 當前符號、Msc_RS 資源元素數、Qm 調變階數]
  CK[檢查參數範圍：symbol 與 Msc_RS 不可越界]
  LOOP[向量化迴圈：分段處理資源元素]
  AMP[讀取通道能量 amp 並限制最大值為 4095]
  INV[查表 nr_inv_ch 取得一除以通道幅度的係數]
  EQ[頻率等化：將接收符號乘上該係數並右移三位]
  CONST[依調變階數設定星座常數 QPSK 16QAM 64QAM]
  OUT[輸出：更新後的 rxdataF_comp 以及 ul_ch_mag 與 ul_ch_magb]

  IN --> CK --> LOOP --> AMP --> INV --> EQ --> CONST --> OUT


```

```mermaid
flowchart LR
  RX["rxdataF\n頻域接收IQ"]
  CHEST["ul_ch_estimates\n通道估測(來自DMRS)"]
  PDU["pusch_pdu / cfg\n(PRB, Qm, Layers, DMRS位置...)"]

  EXTRACT["RB/RE 擷取\nnr_ulsch_extract_rbs"]

  RX --> EXTRACT
  CHEST --> EXTRACT
  PDU --> EXTRACT

  EXTRACT --> RXFEXT["rxFext\n擷取後IQ"]
  EXTRACT --> CHFEXT["chFext\n擷取後通道"]

```
```mermaid
flowchart LR
  CHFEXT["chFext\n擷取後通道"]
  RXFEXT["rxFext\n擷取後IQ"]

  SCALE["通道縮放\nnr_ulsch_scale_channel"]
  COMP["通道補償 + MRC\nnr_ulsch_channel_compensation"]

  CHFEXT --> SCALE --> COMP
  RXFEXT ----------> COMP

  COMP --> RXCOMP["rxdataF_comp\n等化/合併後IQ(逐層)"]
  COMP --> MAGA["ul_ch_mag{a,b,c}\n|h|^2 對應QAM常數"]
  COMP --> RHO["rho (層間相關)\n(多層用)"]
```
```mermaid
flowchart LR
  RXCOMP["rxdataF_comp[0]\n第0層IQ"]
  MAGA["ul_ch_maga\n|h|^2項"]
  MAGB["ul_ch_magb"]
  CFG["pusch_pdu\n(transform_precoding==enabled 且 Qm≤64)"]

  FEQ["頻域等化\nnr_freq_equalization"]
  IDFT["IDFT (SC-FDMA)\nnr_idft"]

  RXCOMP --> FEQ --> IDFT
  MAGA --> FEQ
  MAGB --> FEQ
  CFG  --> FEQ

  IDFT --> TD["時域符元(單層)"]
```
```mermaid
flowchart LR
  IQ["(校正後)樣點\n每RE逐層"]
  MAG["ul_ch_mag{a,b,c}"]
  RHO["rho\n(若兩層)"]
  NVAR["noise_var\n(僅MMSE用)"]
  CFG["pusch_pdu\n(Qm, Layers)"]
  SCRAM["scramblingSequence\n解擾序列"]

  DECIDE{"層數/調變條件?"}

  ML["兩層ML LLR\nnr_ulsch_compute_ML_llr"]
  MMSE["兩層MMSE/ZF核\nnr_ulsch_mmse_2layers"]
  LLR1["單層/其他 LLR\nnr_ulsch_compute_llr"]

  LDMAP["Layer De-mapping\n層去交織(內部)"]
  UNSCR["Unscrambling\n解擾"]

  IQ --> DECIDE
  MAG --> ML
  MAG --> MMSE
  MAG --> LLR1
  RHO --> ML
  NVAR --> MMSE
  CFG --> DECIDE

  DECIDE -- "2層 & Qm≤64" --> ML --> LDMAP
  DECIDE -- "2層 & Qm>64" --> MMSE --> LDMAP
  DECIDE -- "其他情形" ----> LLR1 --> LDMAP

  LDMAP --> UNSCR --> LLRBUF[/"LLR Buffer\n軟位元輸出(送LDPC)"/]
  SCRAM --> UNSCR
```
```mermaid
flowchart TD
  IN[輸入：gNB 參數 PUSCH配置 當前符號]
  LOOPA[迴圈：逐根接收天線]
  INIT[初始化相位緩衝DMRS符號設 幅度1 相位0]
  PTRSCHECK{是否為PTRS符號?}
  EST[進行相位估計nr_ptrs_cpe_estimation]
  LASTCHECK{是否為時槽最後一個符號?}
  INTERP[時間域內插相位nr_ptrs_process_slot]
  ROTATE[相位補償資料符號rotate_cpx_vector]
  OUT[輸出：已補償的rxdataF_comp每符號相位估測]

  IN --> LOOPA --> INIT --> PTRSCHECK
  PTRSCHECK -->|是| EST --> LASTCHECK
  PTRSCHECK -->|否| LASTCHECK
  LASTCHECK -->|是| INTERP --> ROTATE --> OUT
  LASTCHECK -->|否| OUT

```

```mermaid
flowchart TD
  IN[Inputs rxdataF chF rxoffset choffset pusch_pdu frame_parms is_dmrs_symbol]
  SR[Compute start_re and nb_re_pusch]
  DCHK{Is DMRS symbol}
  D0[Copy contiguous PUSCH REs handle wrap around]
  T1{DMRS type1}
  T1SEL[Select data REs by odd even step2 handle wrap around]
  T2SEL[Select data REs by skipping mod6 equals 2*delta or 2*delta+1 handle wrap around]
  OUT[Outputs rxFext chFext]

  IN --> SR --> DCHK
  DCHK -->|No| D0 --> OUT
  DCHK -->|Yes| T1
  T1 -->|Yes| T1SEL --> OUT
  T1 -->|No  type2| T2SEL --> OUT

```
```mermaid
flowchart TD
  A[Demodulation得到 LLR] --> B[Decoding Slot建立參數]
  B --> C[Segmentation產生 C K Z F]
  C --> D[設定 segments計算 E R]
  D --> E[呼叫 LDPC 接口]
  E --> F[逐段呼叫 LDPCdecoder]
  F --> G[LDPCdecoder coreCN BN 迭代]
  G --> H[回傳狀態]
  H --> I[Post decode\n合併到 TB]


```
```mermaid
sequenceDiagram
  participant SLOT
  participant IFACE
  participant DEC
  participant CORE

  SLOT->>SLOT: segmentation to C K Z F
  loop r blocks
    SLOT->>SLOT: compute E R and set segment r
  end
  SLOT->>IFACE: start decoder

  loop each segment
    IFACE->>DEC: call LDPCdecoder
    DEC->>CORE: run core
    CORE-->>DEC: iters or crc ok
    DEC-->>IFACE: return iters maybe abort
  end

  IFACE-->>SLOT: results
  alt ok
    SLOT->>SLOT: merge to TB
  else nok
    SLOT->>SLOT: mark error
  end

```
```mermaid
flowchart TD
  IN["輸入：ULSCH LLR (pusch_vars[ULSCH_id].llr)"] --> SEG["分段 nr_segmentation計算 C, K, Z, F"]
  
  SEG --> LOOP{"對每個分段 r = 0..C-1"}
  LOOP --> PARAMS["設定分段參數:E, R, RV, BG, Qm, Layers"]
  PARAMS --> MAP["對應指標:llr[r_offset], d[r], c[r], d_to_be_cleared[r]"]

  MAP --> DEC["呼叫 LDPC 解碼器 (nrLDPC_coding_decoder)"]
  DEC -->|解碼成功| MERGE["合併段結果到 TB buffer b去掉填充 F、子 CRC"]
  DEC -->|解碼失敗| ERR["標記 HARQ 錯誤 保留軟值 d[r]"]

  MERGE --> OUT["輸出：完整解碼的 TB"]
  ERR --> OUT

```
```mermaid
flowchart TD
  A[Input HARQ TB<br/>and PDSCH config] --> B[nr_generate_pdsch]
  B --> C[nr_dlsch_encoding<br/>CRC LDPC rate match interleave segmentation]
  C --> D[Encoded bits]
  D --> E[do_one_dlsch]
  E --> F[Scrambling<br/>nr_pdsch_codeword_scrambling]
  F --> G[QAM modulation<br/>nr_modulation]
  G --> H[Layer mapping<br/>nr_layer_mapping]
  H --> I[Per symbol loop]
  I --> J[DMRS sequence<br/>nr_gold_pdsch nr_modulation]
  I --> K[PTRS positions<br/>set_ptrs_symb_idx]
  I --> L[Per layer mapping<br/>do_onelayer]
  J --> L
  K --> L
  L --> M[Precoding and antenna<br/>do_txdataF]
  M --> N{PMI check}
  N --> N0[PMI 0<br/>unitary copy or mute]
  N --> N1[PMI non zero<br/>codebook precoding]
  N0 --> O[txdataF frequency grid]
  N1 --> O
  O --> P[OFDM IFFT and CP]

```
```mermaid
flowchart LR
  subgraph DL[下行 PDSCH 發送鏈]
    A0[輸入 HARQ 與 PDSCH 設定<br/>rel15 harq pdu] --> A1[nr_dlsch_encoding<br/>CRC LDPC 速率匹配 交錯 分段]
    A1 --> A2[do_one_dlsch<br/>擾碼 調變 層映射]
    A2 --> A3[逐符號處理<br/>DMRS 產生 PTRS 位置 資源對映]
    A3 --> A4[預編碼 與 天線對映<br/>do_txdataF]
    A4 --> A5[txdataF 頻域格點]
  end

  A5 --> AIR[無線介面]

  subgraph UL[上行 PUSCH 接收鏈]
    B0[接收頻域格點<br/>rxdataF] --> B1[通道估測 DMRS<br/>nr_pusch_channel_estimation]
    B1 --> B2[量測與統計<br/>nr_gnb_measurements]
    B2 --> B3[初始化與參數<br/>G 計算 擾碼序列 LLR 緩衝]
    B3 --> B4[逐符號處理 可能多執行緒<br/>nr_pusch_symbol_processing]
    B4 --> B5[LLR 匯總 與 指標輸出]
  end
```

```mermaid
flowchart TD
  A[輸入 HARQ 與 PDSCH 配置] --> B[nr_generate_pdsch]
  B --> C[編碼階段<br/>CRC LDPC 速率匹配 交錯 分段]
  C --> D[do_one_dlsch]
  D --> E[擾碼<br/>nr_pdsch_codeword_scrambling]
  E --> F[調變<br/>nr_modulation]
  F --> G[層映射<br/>nr_layer_mapping]
  G --> H[逐 OFDM 符號迴圈]
  H --> I[DMRS 序列<br/>nr_gold_pdsch 加上調變]
  H --> J[PTRS 符號位置<br/>set_ptrs_symb_idx]
  H --> K[逐層資源對映<br/>do_onelayer]
  I --> K
  J --> K
  K --> L[預編碼 與 天線對映<br/>do_txdataF PMI 檢查]
  L --> M[txdataF 頻域格點<br/>後續 IFFT CP 於他檔處理]
```

```mermaid
flowchart TD
  R0[入口 nr_rx_pusch_tp] --> R1[通道估測迴圈<br/>掃描含 DMRS 符號]
  R1 --> R1a[nr_pusch_channel_estimation<br/>每層每天線估測]
  R1a --> R1b[量測與雜訊估計<br/>nr_gnb_measurements 平均能量]
  R1b --> R2[時間域平均 可選<br/>nr_chest_time_domain_avg]
  R2 --> R3[計算 G 與 未可用 RE 數<br/>get_ptrs_symbols_in_slot 等]
  R3 --> R4[初始化擾碼序列 與 LLR 緩衝<br/>nr_codeword_unscrambling_init]
  R4 --> R5[選定量測符號 與 通道縮放<br/>nr_ulsch_scale_channel]
  R5 --> R6[計算通道等級 log2_maxh<br/>nr_channel_level]
  R6 --> R7[逐符號處理 可能多執行緒<br/>nr_pusch_symbol_processing]

  subgraph S[單一符號 inner_rx]
    S0[抽取 PUSCH RE 與 通道係數<br/>nr_ulsch_extract_rbs] --> S1[通道補償 與 合併 MRC<br/>nr_ulsch_channel_compensation]
    S1 --> S1a[SC FDMA 可選<br/>nr_freq_equalization 及 nr_idft]
    S1 --> S1b[PTRS 處理 可選<br/>nr_pusch_ptrs_processing]
    S1a --> S2
    S1b --> S2
    S1 --> S2{層數 與 調變階數}

    S2 -->|兩層 且 QPSK 到 64QAM| S3[最大似然 LLR<br/>nr_ulsch_compute_ML_llr]
    S2 -->|兩層 且 256QAM 或更高| S4[MMSE 等化 兩層<br/>nr_ulsch_mmse_2layers<br/>內含 HhH 組建與行列式計算]
    S2 -->|單層 或 非上述| S5[一般 LLR 計算<br/>nr_ulsch_compute_llr]

    S3 --> S6[層去映射 與 去擾碼<br/>乘以擾碼序列]
    S4 --> S6
    S5 --> S6
  end

  R7 --> R8[彙總 LLR 並輸出到上層解碼]
```

```mermaid
sequenceDiagram
  participant 調度器
  participant PDSCH編碼 as PDSCH編碼<br/>nr_dlsch_encoding
  participant 擾碼調變 as 擾碼 調變 層映射<br/>do_one_dlsch
  participant 對映 as 資源對映 DMRS PTRS<br/>do_onelayer do_ptrs_symbol dmrs_case00
  participant 預編碼 as 預編碼 天線對映<br/>do_txdataF
  participant 無線介面

  調度器->>PDSCH編碼: 呼叫 nr_generate_pdsch()
  PDSCH編碼->>PDSCH編碼: CRC LDPC 速率匹配 交錯 分段
  PDSCH編碼->>擾碼調變: 交付編碼後位元
  擾碼調變->>擾碼調變: 擾碼 nr_pdsch_codeword_scrambling<br/>調變 nr_modulation<br/>層映射 nr_layer_mapping
  擾碼調變->>對映: 逐符號處理
  對映->>預編碼: 交付各層頻域格點
  預編碼->>預編碼: PMI 檢查<br/>unitary 或 codebook<br/>nr_layer_precoder_simd / cm
  預編碼->>無線介面: 輸出 txdataF

```
