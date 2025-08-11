# nr_prach.c — Function-by-Function Reference（函式逐一說明）

> PRACH initialization, list management (RU/gNB), and detection path.


## init_prach_list()

**Signature**: `void init_prach_list(PHY_VARS_gNB *gNB)`

**Purpose**: Initialize the gNB PRACH entry list (set empty markers).  
**用途**：初始化 gNB PRACH 清單（設為空）。

**Parameters / 參數**
- `*gNB`: `PHY_VARS_gNB`

**Returns / 回傳**
- `void`

---

## free_nr_prach_entry()

**Signature**: `void free_nr_prach_entry(PHY_VARS_gNB *gNB, int prach_id)`

**Purpose**: Free and reset a gNB PRACH list entry by ID.  
**用途**：釋放並重置指定 ID 的 gNB PRACH 項目。

**Parameters / 參數**
- `*gNB`: `PHY_VARS_gNB`
- `prach_id`: `int`

**Returns / 回傳**
- `void`

---

## find_nr_prach()

**Signature**: `int16_t find_nr_prach(PHY_VARS_gNB *gNB,int frame, int slot, find_type_t type)`

**Purpose**: Find an existing or free PRACH entry for (frame, slot).  
**用途**：依 (frame, slot) 尋找現有或可用的 PRACH 項目。

**Parameters / 參數**
- `*gNB`: `PHY_VARS_gNB`
- `frame`: `int`
- `slot`: `int`
- `type`: `find_type_t`

**Returns / 回傳**
- `int16_t`

---

## nr_fill_prach()

**Signature**: `int nr_fill_prach(PHY_VARS_gNB *gNB, int SFN, int Slot, nfapi_nr_prach_pdu_t *prach_pdu)`

**Purpose**: Populate a gNB PRACH entry from an NFAPI PRACH PDU (preamble, timing, format).  
**用途**：依 NFAPI PRACH PDU 填入 gNB PRACH 項目（前導、定時、格式）。

**Data flow / 資料流向**：`NFAPI PRACH PDU → gNB->prach[i]`

**Parameters / 參數**
- `*gNB`: `PHY_VARS_gNB`
- `SFN`: `int`
- `Slot`: `int`
- `*prach_pdu`: `nfapi_nr_prach_pdu_t`

**Returns / 回傳**
- `int`

---

## init_prach_ru_list()

**Signature**: `void init_prach_ru_list(RU_t *ru)`

**Purpose**: Initialize the RU-side PRACH entry list and mutex.  
**用途**：初始化 RU 端 PRACH 清單與互斥鎖。

**Parameters / 參數**
- `*ru`: `RU_t`

**Returns / 回傳**
- `void`

---

## find_nr_prach_ru()

**Signature**: `int16_t find_nr_prach_ru(RU_t *ru,int frame,int slot, find_type_t type)`

**Purpose**: Find an existing or free RU PRACH entry for (frame, slot).  
**用途**：依 (frame, slot) 尋找現有或可用的 RU PRACH 項目。

**Parameters / 參數**
- `*ru`: `RU_t`
- `frame`: `int`
- `slot`: `int`
- `type`: `find_type_t`

**Returns / 回傳**
- `int16_t`

---

## nr_fill_prach_ru()

**Signature**: `void nr_fill_prach_ru(RU_t *ru, int SFN, int Slot, nfapi_nr_prach_pdu_t *prach_pdu, int *beam_id)`

**Purpose**: Populate an RU PRACH entry (frame/slot/occasion metadata).  
**用途**：填入 RU PRACH 項目的框架/時槽/Occasion 等資訊。

**Data flow / 資料流向**：`RU occasion cfg → RU prach list entry`

**Parameters / 參數**
- `*ru`: `RU_t`
- `SFN`: `int`
- `Slot`: `int`
- `*prach_pdu`: `nfapi_nr_prach_pdu_t`
- `*beam_id`: `int`

**Returns / 回傳**
- `void`

---

## free_nr_ru_prach_entry()

**Signature**: `void free_nr_ru_prach_entry(RU_t *ru, int prach_id)`

**Purpose**: Free and reset an RU PRACH list entry by ID.  
**用途**：釋放並重置指定 ID 的 RU PRACH 項目。

**Parameters / 參數**
- `*ru`: `RU_t`
- `prach_id`: `int`

**Returns / 回傳**
- `void`

---

## rx_nr_prach_ru()

**Signature**: `void rx_nr_prach_ru(RU_t *ru,
                    int prachFormat,
                    int numRA,
                    int beam,
                    int prachStartSymbol,
                    int prachStartSlot,
                    int prachOccasion,
                    int frame,
                    int slot)`

**Purpose**: RU-side PRACH capture & processing (DFT, repetition combining, detection).  
**用途**：RU 端 PRACH 擷取與處理（DFT、重複合併、偵測）。

**Data flow / 資料流向**：`ADC/recall buffer → detection metrics`

**Parameters / 參數**
- `*ru`: `RU_t`
- `prachFormat`: `int`
- `numRA`: `int`
- `beam`: `int`
- `prachStartSymbol`: `int`
- `prachStartSlot`: `int`
- `prachOccasion`: `int`
- `frame`: `int`
- `slot`: `int`

**Returns / 回傳**
- `void`

**Mini flow / 簡易流程**
1. Gather RU PRACH samples (per repetition)
2. DFT and combine repetitions
3. Detect preamble & estimate TA/energy
4. Forward/aggregate results for gNB

---

## rx_nr_prach()

**Signature**: `void rx_nr_prach(PHY_VARS_gNB *gNB,
                 nfapi_nr_prach_pdu_t *prach_pdu,
                 int prachOccasion,
                 int frame,
                 int slot,
                 uint16_t *max_preamble,
                 uint16_t *max_preamble_energy,
                 uint16_t *max_preamble_delay)`

**Purpose**: gNB-side PRACH receive & detection; report preamble index, energy, TA.  
**用途**：gNB 端 PRACH 接收與偵測；回報前導索引、能量與 TA。

**Data flow / 資料流向**：`RU PRACH buffer → PRACH_indication (preamble/energy/TA)`

**Parameters / 參數**
- `*gNB`: `PHY_VARS_gNB`
- `*prach_pdu`: `nfapi_nr_prach_pdu_t`
- `prachOccasion`: `int`
- `frame`: `int`
- `slot`: `int`
- `*max_preamble`: `uint16_t`
- `*max_preamble_energy`: `uint16_t`
- `*max_preamble_delay`: `uint16_t`

**Returns / 回傳**
- `void`

**Mini flow / 簡易流程**
1. Fetch PRACH buffer for selected occasion
2. Correlation / DFT-based detection
3. Measure peak energy and delay
4. Build PRACH indication (preamble, energy, TA)

---
