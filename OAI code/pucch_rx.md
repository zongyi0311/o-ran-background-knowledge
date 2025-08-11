# pucch_rx.c — Function-by-Function Reference（函式逐一說明）

## nr_fill_pucch()

**Signature**: `void nr_fill_pucch(PHY_VARS_gNB *gNB, int frame, int slot, nfapi_nr_pucch_pdu_t *pucch_pdu)`

**Purpose**: Reserve and populate a PUCCH entry for (frame, slot). Copy NFAPI PDU, select beam, mark active.  
**用途**：為 (frame, slot) 分配並填入 PUCCH 項目；複製 NFAPI PDU、選定波束並標記啟用。

**Data flow / 資料流向**：`NFAPI PDU + gNB config → gNB->pucch[i] (active)`

**Parameters / 參數**
- `*gNB`: `PHY_VARS_gNB`
- `frame`: `int`
- `slot`: `int`
- `*pucch_pdu`: `nfapi_nr_pucch_pdu_t`

**Returns / 回傳**
- `void`

---

## get_pucch0_cs_lut_index()

**Signature**: `int get_pucch0_cs_lut_index(PHY_VARS_gNB *gNB,nfapi_nr_pucch_pdu_t* pucch_pdu)`

**Purpose**: Return/create LUT index for PUCCH-0 cyclic-shift sequences using hopping_id & SCS.  
**用途**：依 hopping_id 與子載波間隔回傳/建立 PUCCH-0 循環位移的查表索引。

**Data flow / 資料流向**：`PUCCH-0 config → CS LUT index`

**Parameters / 參數**
- `*gNB`: `PHY_VARS_gNB`
- `pucch_pdu`: `nfapi_nr_pucch_pdu_t*`

**Returns / 回傳**
- `int`

---

## nr_decode_pucch0()

**Signature**: `void nr_decode_pucch0(PHY_VARS_gNB *gNB,
                      c16_t **rxdataF,
                      int frame,
                      int slot,
                      nfapi_nr_uci_pucch_pdu_format_0_1_t *uci_pdu,
                      nfapi_nr_pucch_pdu_t *pucch_pdu)`

**Purpose**: Decode PUCCH Format 0: correlate, estimate SNR/energy; produce UCI (SR/HARQ) and metrics.  
**用途**：解碼 PUCCH Format 0：相關與能量/SNR 判定；產生 UCI（SR/HARQ）與量測值。

**Data flow / 資料流向**：`rxdataF + PUCCH-0 PDU → UCI (SR/HARQ), ul_cqi, rssi`

**Parameters / 參數**
- `*gNB`: `PHY_VARS_gNB`
- `**rxdataF`: `c16_t`
- `frame`: `int`
- `slot`: `int`
- `*uci_pdu`: `nfapi_nr_uci_pucch_pdu_format_0_1_t`
- `*pucch_pdu`: `nfapi_nr_pucch_pdu_t`

**Returns / 回傳**
- `void`

**Mini flow / 簡易流程**
1. Extract REs & correlate
2. Compute energy/SNR
3. Thresholding (DTX/valid)
4. Build UCI: SR / HARQ-ACK
5. Set ul_cqi / rssi

---

## if()

**Signature**: `else if(pucch_pdu->bit_len_harq==1)`

**Purpose**: Internal helper for PUCCH receive/decoding path.  
**用途**：PUCCH 接收/解碼路徑的內部輔助函式。

**Parameters / 參數**
- `pucch_pdu->bit_len_harq==1`: `pucch_pdu->bit_len_harq==1`

**Returns / 回傳**
- `else`

---

## if()

**Signature**: `else if(pucch_pdu->bit_len_harq==1)`

**Purpose**: Internal helper for PUCCH receive/decoding path.  
**用途**：PUCCH 接收/解碼路徑的內部輔助函式。

**Parameters / 參數**
- `pucch_pdu->bit_len_harq==1`: `pucch_pdu->bit_len_harq==1`

**Returns / 回傳**
- `else`

---

## nr_decode_pucch1()

**Signature**: `void nr_decode_pucch1(c16_t **rxdataF,
                      pucch_GroupHopping_t pucch_GroupHopping,
                      uint32_t n_id, // hoppingID higher layer parameter
                      uint64_t *payload,
                      NR_DL_FRAME_PARMS *frame_parms,
                      int16_t amp,
                      int nr_tti_tx,
                      uint8_t m0,
                      uint8_t nrofSymbols,
                      uint8_t startingSymbolIndex,
                      uint16_t startingPRB,
                      uint16_t startingPRB_intraSlotHopping,
                      uint8_t timeDomainOCC,
                      uint8_t nr_bit)`

**Purpose**: Demodulate PUCCH Format 1 to a 1–2 bit payload using channel estimate and combining.  
**用途**：以通道估測與合併解調 PUCCH Format 1，得到 1–2 位元負載。

**Data flow / 資料流向**：`rxdataF + PUCCH-1 cfg → payload bits`

**Parameters / 參數**
- `**rxdataF`: `c16_t`
- `pucch_GroupHopping`: `pucch_GroupHopping_t`
- `n_id`: `uint32_t`
- `*payload`: `// hoppingID higher layer parameter
                      uint64_t`
- `*frame_parms`: `NR_DL_FRAME_PARMS`
- `amp`: `int16_t`
- `nr_tti_tx`: `int`
- `m0`: `uint8_t`
- `nrofSymbols`: `uint8_t`
- `startingSymbolIndex`: `uint8_t`
- `startingPRB`: `uint16_t`
- `startingPRB_intraSlotHopping`: `uint16_t`
- `timeDomainOCC`: `uint8_t`
- `nr_bit`: `uint8_t`

**Returns / 回傳**
- `void`

**Mini flow / 簡易流程**
1. Channel estimate (DMRS)
2. MRC combine
3. QPSK/BPSK demod
4. Decide 1–2 bits
5. Write payload

---

## init_pucch2_luts()

**Signature**: `void init_pucch2_luts()`

**Purpose**: Precompute lookup tables for PUCCH-2/3/4 decoding (polar/short-block patterns, LLR helpers).  
**用途**：預先建立 PUCCH-2/3/4 解碼所需查表（極化/短區塊圖樣與 LLR 輔助表）。

**Data flow / 資料流向**：`— (initialization only)`

**Parameters / 參數**
- (none)

**Returns / 回傳**
- `void`

---

## nr_decode_pucch2()

**Signature**: `void nr_decode_pucch2(PHY_VARS_gNB *gNB,
                      c16_t **rxdataF,
                      int frame,
                      int slot,
                      nfapi_nr_uci_pucch_pdu_format_2_3_4_t* uci_pdu,
                      nfapi_nr_pucch_pdu_t* pucch_pdu)`

**Purpose**: Decode PUCCH Formats 2/3/4: extract REs, correlate DMRS, build LLRs, run polar decoder; fill UCI (HARQ/CSI/SR).  
**用途**：解碼 PUCCH Format 2/3/4：RE 擷取、DMRS 相關、LLR 建立與極化解碼；填入 UCI（HARQ/CSI/SR）。

**Data flow / 資料流向**：`rxdataF + PUCCH-2/3/4 PDU → UCI (HARQ bits+CRC, CSI part1/part2, SR), ul_cqi, rssi`

**Parameters / 參數**
- `*gNB`: `PHY_VARS_gNB`
- `**rxdataF`: `c16_t`
- `frame`: `int`
- `slot`: `int`
- `uci_pdu`: `nfapi_nr_uci_pucch_pdu_format_2_3_4_t*`
- `pucch_pdu`: `nfapi_nr_pucch_pdu_t*`

**Returns / 回傳**
- `void`

**Mini flow / 簡易流程**
1. RE extraction
2. DMRS correlation
3. LLR build (SIMD)
4. Polar decode
5. Fill UCI (HARQ/CSI/SR)

---

## nr_dump_uci_stats()

**Signature**: `void nr_dump_uci_stats(FILE *fd,PHY_VARS_gNB *gNB,int frame)`

**Purpose**: Dump per-UE UCI decode statistics to a file for diagnostics.  
**用途**：將每 UE 的 UCI 解碼統計輸出到檔案以利診斷。

**Data flow / 資料流向**：`UCI decode stats → file`

**Parameters / 參數**
- `*fd`: `FILE`
- `*gNB`: `PHY_VARS_gNB`
- `frame`: `int`

**Returns / 回傳**
- `void`

---
