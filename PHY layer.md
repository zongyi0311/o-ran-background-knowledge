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
  - Resource Element Demapping:將接收到的符號從頻域元素位置提取出來
  - Equalization + IDFT:對通道影響進行等化處理（補償多路徑等衰減）
  - Demodulation:將符號轉換回 bit
  - Descrambling:解亂數化，回復原始 bit 順序
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
