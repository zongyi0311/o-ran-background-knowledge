<img width="659" height="395" alt="image" src="https://github.com/user-attachments/assets/d3416919-842c-472e-af12-eeefb531c2fe" />

- 在 O-RAN 架構下，gNB 基站內部由 O-CU、O-DU、O-RU 組成，並使用 Split Option 7-2x 來將高層與低層 PHY 功能分割

**DL**
- PHY-High:
  - Encoding:從 MAC 層接收傳輸區塊 (TB)，進行 CRC 附加與 LDPC 編碼。
  - Scrambling:對編碼bits進行亂數化處理，以減少bits streams中的規律性
  - Modulation:將bits轉換為複數符號（如 QPSK、16QAM、64QAM、256QAM）
  - Layer Mapping:根據天線數量將符號分配到不同的傳輸層（spatial layers）
  - Precoding:根據 MIMO 條件與通道狀態將層映射後的符號預編碼，提升傳輸效率**若移至 O-RU 做預編碼，O-DU 這裡會跳過**
  - Resource Element Mapping:將符號放到 OFDM 的頻域資源元素上（subcarrier × symbol）
- PHY-Low
  - Precoding（若未在 O-DU 執行）
  - Digital Beamforming:將多天線信號進行數位域合成，強化特定方向的訊號
  - IFFT:將頻域的資源元素轉為時域波形（生成 OFDM symbols）
  - Analog Conversion:將數位波形轉為類比訊號，準備送到 RF
  - Analog Beamforming:在類比域進行天線訊號加權與方向調整（可選）

**UL**
- PHY-Low
  - Analog Beamforming 接收端從多天線收集 RF 類比訊號
  - Analog to Digital Conversion:將接收到的類比訊號轉為數位
  - FFT:把時域 OFDM 信號轉換為頻域資源元素
  - Digital Beamforming:根據方向資訊進行數位波束合成
- PHY-High
  - Resource Element Demapping:將接收到的符號從頻域元素位置提取出來 **//nr_ulsch_extract_rbs()/nr_ulsch_demodulation.c**
  - Equalization:對通道影響進行等化處理（補償多路徑等衰減）**//nr_ulsch_channel_compensation/nr_ulsch_demodulation.c**
  - Demodulation:將符號轉換回 **bit/nr_ulsch_demodulation.c(控制)和nr_ulsch_llr_computation.c(計算)**
  - Descrambling:解亂數化，回復原始 bit 順序 **//nr_codeword_unscrambling()/nr_scrambling.c**
  - Decoding:執行 LDPC 解碼，提取原始傳輸區塊 TB
  
**Split Option 7-2x：O-DU 與 O-RU 之間傳送的是 IQ samples（頻域的 OFDM 資料）**

| 規範編號              | 標題                                      | 層級涵蓋           | 說明                             |
| ----------------- | --------------------------------------- | -------------- | ------------------------------ |
| **TS 38.211**     | *Physical channels and modulation*      | PHY-Low        | 定義通道格式、OFDM、SC-FDMA、CP         |
| **TS 38.212**     | *Multiplexing and channel coding*       | PHY-High       | 包含 segmentation、CRC、LDPC、Polar |
| **TS 38.213**     | *Physical layer procedures for control* | PHY-High + MAC | 時間安排、上行功控、控制信令                 |
| **TS 38.214**     | *Physical layer procedures for data*    | PHY-High       | 描述 MCS、調變選擇、傳輸模式               |
| **TS 38.215**     | *Physical layer measurements*           | PHY-High       | SNR、RSRP、CQI 等量測方法與回報          |
| **TS 38.104**     | *Base Station RF Requirements*          | RF (O-RU)      | 頻率範圍、發射功率、鄰頻干擾等 RF 規定          |
| **TS 38.141-1/2** | *Base Station Conformance Testing*      | RF (O-RU)      | 定義符合測試流程（頻譜遮蔽、EVM）             |

| 目的            | 建議規範                           |
| ------------- | ------------------------------ |
| 模擬通道 / fading | **TR 38.901**                  |
| 模擬切分方案        | **TR 38.801**, **TR 38.816**   |

**TS 38.300**
- TS 38.300 定義整個 5G 無線接取網路（NG-RAN） 的架構與模組分工，是設計 CU/DU、O-DU/O-RU、gNB、UE 等的總體依據

# 7/11　
## nr_ulsch.c
-  5G NR PHY-High 層 的上行接收控制邏輯（PUSCH 任務管理），負責接收 FAPI 封包 → 分配資源 → 初始化 ULSCH 資料結構 → 執行 Beamforming slot 管理，讓後續 FFT、解調、解碼等模組能正確處理該 UE 的上行資料

### static NR_gNB_ULSCH_t *find_nr_ulsch(PHY_VARS_gNB *gNB, uint16_t rnti, int pid()
- 根據 RNTI（UE 識別碼） 與 HARQ Process ID
  - 找到對應的 NR_gNB_ULSCH_t（用來儲存 UE 的上行資料）
  - 如果找不到，就回傳一筆「尚未啟用」的 ULSCH 作為新的資源
- 常被 nr_fill_ulsch() 呼叫，用於初始化任務
- 每個 slot 處理時都必須先經過它來找出要用哪個 buffer

**struct NR_gNB_ULSCH_t /defs_gNB.h**
這個 typedef struct 在 OAI 中代表 gNB 接收 UE 上行資料（UL-SCH）的狀態管理結構

**struct PHY_VARS_gNB /defs_gNB.h**
這是 gNB（基地台）在實體層（PHY layer）中整體控制與資源儲存的主要結構體，所有 PHY 處理用到的變數都從這裡取出

### nr_fill_ulsch()
- gNB 上行接收主控流程的初始化函式，負責從 MAC/FAPI 層下來的 nfapi_nr_pusch_pdu_t 中提取參數、配置接收任務，對應 PHY-High 的任務排程管理

| 步驟                 | 功能說明                                                                             | 
| ------------------ | -------------------------------------------------------------------------------- | 
| 1️ 尋找或建立任務        | 利用 `find_nr_ulsch()` 查找對應的 UE + HARQ PID 的接收任務，如果沒有就分配新的空位                       | 
| 2️ 設定基本參數         | 寫入 RNTI、HARQ PID、frame、slot、任務 active 狀態等欄位                                      | 
| 3️ 設定 Beamforming | 如果啟用 analog beamforming，根據 FAPI beam index 呼叫 `beam_index_allocation()` 決定所使用的波束 | 
| 4️ HARQ 控制        | 若是新資料（NDI=1）則清除前次狀態，否則累積重傳次數                                                     | 
| 5️ 記錄 PUSCH PDU   | 將整份 `ulsch_pdu` 複製進 `ulsch->harq_process` 結構中，供後續解碼與統計使用                         | 
| 6️ 印出 log         | 記錄初始化過程與 HARQ 狀態變化                                                               | 

**struct nfapi_nr_pusch_pdu_t /nfapi/open-nFAPI/nfapi/public_inc/nfapi_nr_interface_scf.h**
- 這個結構定義了 gNB 接收 UE 上行 PUSCH（Physical Uplink Shared Channel）資料所需的控制參數，根據 3GPP TS 38.212 / 38.214 / 38.331 規範而設計

**SL_to_bitmap**
- 這是一個將連續的 OFDM symbol 範圍轉換為 位元圖（bitmap） 的小工具函式，用於表示：slot 中哪些 symbol 被佔用來傳輸 PUSCH（或其他物理通道）

**beam_index_allocation**
- 是在 OpenAirInterface (OAI) 的 gNB 端，針對 上行（UL）傳輸時的 beamforming 資源分配 所使用的邏輯。它會根據 scheduler 下發的 beam index，幫你找一個可以「佔用」的 slot 位置，並寫入 common_vars->beam_id[][]

### nr_ulsch_layer_demapping
- 將 Layer LLR 結構重新組合為單一 codeword LLR 序列 的重要步驟，屬於 PHY-High 層中 LLR 處理的前處理邏輯

### dump_pusch_stats()
- 用來列印 gNB (基地台) 端上行鏈路 PUSCH 的統計資訊，屬於除錯 / 日誌輸出用工具函式
