# fapi_nr_l1
## nr_schedule_response(NR_Sched_Rsp_t *Sched_INFO)
Implement the FAPI interface logic (L1 Adapter) between the OAI gNB physical layer (PHY-L1) and the MAC/DU layer, that is, process the DL_TTI, UL_TTI, TX_DATA and other requests in the FAPI specification, and store the results in the L1 layer temporary data structure for subsequent processing.

| Function Name                     | Description                                                                 |
|----------------------------------|-----------------------------------------------------------------------------|
| `handle_nr_nfapi_ssb_pdu()`      | Handles SSB-type DL_TTI PDU and stores it in `msgTx->ssb[]`.               |
| `handle_nfapi_nr_csirs_pdu()`    | Handles CSI-RS-type DL_TTI PDU and stores it in `msgTx->csirs_pdu[]`.      |
| `nr_schedule_dl_tti_req()`       | Processes downlink TTI scheduling information received from the MAC layer. |
| `nr_schedule_tx_req()`           | Processes downlink SDU payload data received from the MAC layer.           |
| `nr_schedule_ul_tti_req()`       | Processes uplink TTI scheduling information indicating what the UE will send and when. |
| `nr_fill_dlsch_dl_tti_req()`     | Fills PDSCH-type PDU into the corresponding slot's transmission buffer.    |
| `nr_fill_dlsch_tx_req()`         | Maps actual SDU payload data to the corresponding PDSCH PDU.               |

```
┌────────────────────────────────────────────┐
│          nr_schedule_response()           │
└────────────────────────────────────────────┘
                    │
                    ▼
     ┌────────────────────────────────────┐
     │ Extract module_id, frame, slot     │
     └────────────────────────────────────┘
                    │
                    ▼
     ┌────────────────────────────────────┐
     │ Ensure RC.gNB and RC.gNB[module_id]│
     │ are valid                          │
     └────────────────────────────────────┘
                    │
                    ▼
     ┌────────────────────────────────────┐
     │ Start timing measurement (profiling)│
     └────────────────────────────────────┘
                    │
                    ▼
     ┌────────────────────────────────────┐
     │ Determine slot type (DL/UL/Mixed)  │
     └────────────────────────────────────┘
                    │
                    ▼
     ┌────────────────────────────────────┐
     │ Clear beam_id info for this slot   │
     └────────────────────────────────────┘
                    │
                    ▼
     ┌────────────────────────────────────┐
     │ Ensure NFAPI_MODE == MONOLITHIC    │
     └────────────────────────────────────┘
                    │
                    ▼
     ┌────────────────────────────────────┐
     │ Save sched_response_id to TX buffer│
     └────────────────────────────────────┘
                    │
                    ▼
     ┌────────────────────────────────────┐
     │ Assert DL_req frame and slot match │
     └────────────────────────────────────┘
                    │
                    ▼
        ┌────────────┴─────────────┐
        ▼                          ▼
   is_dl == true              is_dl == false
        │                          │
        ▼                          ▼
┌─────────────────────┐   ┌────────────────────┐
│ Call nr_schedule_    │   │ Call nr_schedule_  │
│ dl_tti_req()         │   │ ul_tti_req()       │
└─────────────────────┘   └────────────────────┘
        │
        ▼
┌──────────────────────────────┐
│ Call nr_schedule_tx_req()    │
└──────────────────────────────┘
        │
        ▼
┌──────────────────────────────┐
│ Call nr_schedule_ul_dci_req()│
└──────────────────────────────┘
        │
        ▼
┌────────────────────────────────────────────────┐
│ Call inc_ref_sched_response() to increase ref  │
│ count (shared between threads)                 │
└────────────────────────────────────────────────┘
        │
        ▼
┌─────────────────────────────────────────────────────┐
│ If DL slot & MONOLITHIC mode:                       │
│ Call deref_sched_response() to release sched_info   │
└─────────────────────────────────────────────────────┘
        │
        ▼
┌────────────────────────────────────┐
│ Stop timing measurement (profiling)│
└────────────────────────────────────┘
```

**nr_schedule_dl_tti_req()**
```
MAC (or DU) tells PHY: "In the nth slot, which signals to send, to whom, and how to send them."
After receiving these "instructions", PHY (physical layer) temporarily stores them in msgDataTx.
When it reaches the slot, OAI's L1 processing flow will perform the actual transmission according to msgDataTx (such as SSB RF transmission, PDCCH encoding, PDSCH beamforming, etc.)
Start
  │
  ├──▶ Check if gNB and DL_req are NULL (DevAssert)
  │
  ├──▶ Get config pointer: cfg = &gNB->gNB_config
  │
  ├──▶ Read frame and slot from DL_req
  │
  ├──▶ Determine slot_type = nr_slot_select(cfg, frame, slot)
  │
  ├──▶ Assert slot_type is DL or Mixed (DevAssert)
  │
  ├──▶ Initialize msgTx = gNB->msgDataTx
  │       ├── Set msgTx->slot = slot
  │       └── Set msgTx->frame = frame
  │
  ├──▶ Get number_dl_pdu = DL_req->dl_tti_request_body.nPDUs
  │
  └──▶ For each dl_tti_pdu in DL_req:
           │
           ├── Check dl_tti_pdu->PDUType:
           │
           ├── case SSB_PDU:
           │       └── Call handle_nr_nfapi_ssb_pdu()
           │
           ├── case PDCCH_PDU:
           │       ├── Store into msgTx->pdcch_pdu[]
           │       └── Increment msgTx->num_dl_pdcch
           │
           ├── case CSI_RS_PDU:
           │       └── Call handle_nfapi_nr_csirs_pdu()
           │
           └── case PDSCH_PDU:
                   └── Call nr_fill_dlsch_dl_tti_req()

End
```
**nr_schedule_ul_tti_req**
```
This is the main uplink scheduling entry from MAC layer to PHY layer:
MAC tells PHY: "What data and signals will the UE send in a certain slot"
PHY temporarily stores this information and is ready to use channel estimation, FFT, demodulation, etc. to receive the corresponding signal in this slot
Start
  │
  ├──▶ Check if gNB and UL_tti_req are NULL
  │
  ├──▶ Extract frame and slot from UL_tti_req
  │
  └──▶ For each PDU in UL_tti_req->pdus_list:
           │
           ├── If PDUType == PUSCH:
           │       └── Call nr_fill_ulsch()
           │
           ├── If PDUType == PUCCH:
           │       └── Call nr_fill_pucch()
           │
           ├── If PDUType == PRACH:
           │       ├── Call nr_fill_prach()
           │       └── If RU enabled: call nr_fill_prach_ru()
           │
           └── If PDUType == SRS:
                   └── Call nr_fill_srs()

End
```
# phy_frame_config_nr.c
## do_tdd_config_sim(Automatically generate a legal TDD timeslot configuration)
- Allow the PHY layer to initialize TDD information and perform transmission simulation normally
```
┌──────────────────────────────────────────────────────┐
│ do_tdd_config_sim(PHY_VARS_gNB *gNB, int mu)         │
└──────────────────────────────────────────────────────┘
                     │
                     ▼
      ┌────────────────────────────────────────────┐
      │ Initialize frame_structure_t fs             │
      │ fs.frame_type = TDD                         │
      │ fs.numb_slots_frame = (1 << mu) * 10        │
      │ fs.numb_period_frame = get_nb_periods(...)  │
      │ fs.numb_slots_period = slots_frame / period │
      └────────────────────────────────────────────┘
                     │
                     ▼
     ┌────────────────────────────────────────────┐
     │ Set DL/UL slot counts based on numerology: │
     │   mu = 0 → DL=3, UL=1                       │
     │   mu = 1 → DL=7, UL=2                       │
     │   mu = 3 → DL=27, UL=12                     │
     └────────────────────────────────────────────┘
                     │
                     ▼
     ┌────────────────────────────────────────────┐
     │ For first num_dl_slots slots:              │
     │   slot_type = TDD_NR_DOWNLINK_SLOT         │
     └────────────────────────────────────────────┘
                     │
                     ▼
     ┌────────────────────────────────────────────────────┐
     │ For intermediate slots (after DL, before UL):       │
     │   slot_type = TDD_NR_MIXED_SLOT                    │
     │   num_dl_symbols = 6                                │
     │   num_ul_symbols = 4                                │
     └────────────────────────────────────────────────────┘
                     │
                     ▼
     ┌────────────────────────────────────────────┐
     │ For last num_ul_slots slots:               │
     │   slot_type = TDD_NR_UPLINK_SLOT           │
     └────────────────────────────────────────────┘
                     │
                     ▼
     ┌────────────────────────────────────────────┐
     │ Call set_tdd_config_nr(&gNB->gNB_config, &fs) │
     └────────────────────────────────────────────┘
```
## set_tdd_config_nr
In 5G TDD systems:
- Slots can only be roughly distinguished as DL, UL or Mixed
- But in fact, the PHY layer needs to know the transmission direction of each OFDM symbol accurately
- This function organizes and aligns this information into the format specified by FAPI
- In this way, the underlying hardware or simulator can perform transmission and reception control and scheduling according to the correct symbol configuration.
```
┌────────────────────────────────────────────┐
│ Function Entry: set_tdd_config_nr(cfg, fs) │
└────────────────────────────────────────────┘
                    │
                    ▼
        Check if fs is NULL (sanity check)
                    │
                    ▼
      Calculate total number of slots (nb_slots_to_set)
                    │
                    ▼
  Allocate memory for cfg->tdd_table.max_tdd_periodicity_list
                    │
                    ▼
  For each slot and each symbol:
    → Allocate slot_config struct
    → Set TLV tag for NFAPI usage
                    │
                    ▼
  ┌───────────────────────────────────────────────┐
  │ Enter while loop to process all slots         │
  └───────────────────────────────────────────────┘
                    │
                    ▼
    Based on slot_type, handle each slot:
     ┌────────────────────────────────────────────┐
     │ If slot_type == DOWNLINK                   │
     │ → Set all symbols in slot to 0 (DL)        │
     └────────────────────────────────────────────┘
     ┌────────────────────────────────────────────┐
     │ If slot_type == MIXED                      │
     │ → First part: set to 0 (DL)                │
     │ → Middle part: set to 2 (FLEXIBLE)         │
     │ → Last part: set to 1 (UL)                 │
     └────────────────────────────────────────────┘
     ┌────────────────────────────────────────────┐
     │ If slot_type == UPLINK                     │
     │ → Set all symbols in slot to 1 (UL)        │
     └────────────────────────────────────────────┘
                    │
                    ▼
     After configuring all slots, loop over
     1 TDD period and print configuration:
     ┌────────────────────────────────────────────┐
     │ If slot is FLEXIBLE → print D/U/F string   │
     │ If slot is DL/UL     → print type          │
     └────────────────────────────────────────────┘
                    │
                    ▼
        ┌────────────────────────────┐
        │ End: all TDD config done   │
        └────────────────────────────┘
```
| Value | Symbol | Meaning             |
| ----- | ------ | ------------------- |
| 0     | D      | Downlink            |
| 1     | U      | Uplink              |
| 2     | F      | Flexible (F symbol) |


# phy_procedures_nr_gNB.c
## beam_index_allocation()
- Ensure that the beam corresponding to each slot symbol does not conflict, and assign an analog beam to the transmission data of the slot+symbol if conditions are met

## nr_common_signal_procedures
- Calculate the starting symbol and starting frequency of SSB
- Allocate the corresponding analog beam (through beam_index_allocation())
- Call the function to generate SSB component signals such as PSS, SSS, PBCH, PBCH DMRS, etc.
- Write the result to the transmit buffer txdataF

## clear_slot_beamid
- Clear the beam allocation information of all symbols in the specified slot to prepare for new beam allocation (such as SSB or PDSCH, etc.)
- When a slot has not been configured or needs to be reconfigured, this function needs to be called first to clear the record to avoid beam resource conflicts.
## phy_procedures_gNB_TX
