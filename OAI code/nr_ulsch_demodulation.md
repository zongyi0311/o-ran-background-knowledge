# nr_rx_pusch_tp
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

## nr_pusch_channel_estimation
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

nr_gold_pusch()//Generate DMRS gold sequence for PUSCH (actually reuse nr_gold_pdsch())
uint64_t x2tmp0 = (((uint64_t)symbols_per_slot * slot + symbol + 1) * (((uint64_t)nid << 1) + 1)) << 17;
uint32_t x2 = (x2tmp0 + (nid << 1) + nscid) % (1U << 31);
this Calculation method in TS 38.211 7.4.1.1 
call gold_cache()//This function will use x2 as the seed and return a pseudo-random Gold sequence

pusch_dmrs_type_t dmrs_type = pusch_pdu->dmrs_config_type == NFAPI_NR_DMRS_TYPE1 ? pusch_dmrs_type1 : pusch_dmrs_type2;
3GPP TS 38.211 §6.4.1.1.3//Is it to determine whether the DMRS "Configuration Type" is Type 1 or Type 2, and where it corresponds to?
Type 1 Lower density, inserting a small number of DMRS per RB
Type 2 Higher density, dense insertion in the frequency domain

float beta_dmrs_pusch = get_beta_dmrs_pusch(pusch_pdu->num_dmrs_cdm_grps_no_data, dmrs_type);//Calculate the normalization coefficient based on DMRS settings (Type1 or Type2, number of CDM groups)
int16_t dmrs_scaling = (1 / beta_dmrs_pusch) * (1 << 14);
get_beta_dmrs_pusch()

