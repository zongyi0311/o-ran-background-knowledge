![image](https://github.com/user-attachments/assets/e2a79c41-6732-4386-8606-826ecb63a40b)

# encode

| 步驟                                                  | 說明                                       | **為什麼需要？**                                                       |
| --------------------------------------------------- | ---------------------------------------- | ---------------------------------------------------------------- |
| **Code Block (含 CRC)**                              | 每個 Code Block 加上 24-bit CRC，用來獨立檢查錯誤     |  為了 **支援 HARQ 重傳與錯誤檢查**，每個 CB 必須能單獨驗證                          |
| **Encoding Parameters (BG, Kb, Z, i<sub>LS</sub>)** | 選擇 Base Graph、行列數、Lifting Size           |  為了**適應不同長度與碼率需求**，必須可配置 LDPC 矩陣                               |
| **Insert Filler bits**                              | 若實際輸入位元不足 (Kb × Z)，需補 0                  |  碼長固定時，確保 **矩陣維度正確且完整填滿**                                      |
| **Split to Codeword (Kb × Z)**                      | 將資料填入 info 矩陣，準備開始 LDPC 編碼               |  LDPC 是基於 **線性矩陣運算**，資料必須放入指定大小的編碼矩陣中                          |
| **Read Cycle Shift Coefficient of PCM**             | 從 Base Graph 的 Parity Check Matrix 取得移位值 |  使用循環移位矩陣可以**節省記憶體、提高硬體效率**（quasi-cyclic LDPC）                 |
| **Obtain Parity Code Block 1/2**                    | 使用矩陣乘法產生 parity bits (兩部分)               |  保證 LDPC 的**檢查條件 H·cᵗ = 0 成立**，達到糾錯能力                          |
| **Punching, Shortening, Concatenation**             | 依需求刪除部分 parity bits 或重複、再組合成連續比特串        |  **Rate Matching** 的關鍵步驟，讓 LDPC 可支援任意碼率與資源大小（e.g., 不固定的 PDSCH） |

**MAC 層與 PHY 層的 Segmentation & CRC 差異整理**
| 功能               | MAC 層                                                           | PHY 層                                                |
| ---------------- | --------------------------------------------------------------- | ---------------------------------------------------- |
| **Segmentation** | 將 **SDU**（RLC來的資料）分段成 **MAC SDU/PDU** 或多個 Transport Blocks (TB) | 將每個 TB 的內容拆成數個 **Code Blocks (CB)**，用於 LDPC          |
| **CRC 附加**       | 附加 **Transport Block CRC**（24bit），用來檢查整個 TB                     | 附加 **Code Block CRC**（24bit），每個 CB 各自加，用於 LDPC 解碼後檢查 |
```
[RLC SDU]
   ↓
[MAC 層]
   - Segmentation 成多個 TB
   - 每個 TB 加 Transport Block CRC（24bit）
   ↓
[PHY 層]
   - 若 TB 太大，再分成多個 Code Blocks (CB)
   - 每個 CB 加 CRC（也是24bit，但不同意義）
   - 執行 LDPC 編碼 → Rate Matching → 調變
```
| 檔案                  | 角色             | 模組層級                    | 實際內容                                          | 主要用途                           |
| ------------------- | -------------- | ----------------------- | --------------------------------------------- | ------------------------------ |
| `nr_dlsch_coding.c` | **LDPC 編碼主邏輯** | PHY Layer               | segmentation, CRC, LDPC encode, rate matching | 負責將 TB → bitstream             |
| `nr_dlsch.c`        | **下行調變/傳送邏輯**  | PHY Layer               | 調變、PDSCH 映射、DMRS/PTRS 資料插入                    | 把 encoded bits 變成 OFDM symbols |
| `dlschsim.c`        | **功能測試用例**     | Simulation Tool         | 建立簡化 gNB/UE 結構，模擬 `nr_dlsch_encoding()` 整體流程  | 驗證功能與除錯                        |
| `dlsim.c`           | **大型模擬框架**     | System-level Simulation | 整合 MAC/PHY/調變/通道環境，建立全鏈路模擬                    | 驗證整體系統通聯能力與參數效能                |


**PHY/CODING/nr_segmentation.c**
-  分段（Segmentation）
-  CRC 附加（CRC Attachment）呼叫crc_byte.c裡的函式
-  Z、K 等參數計算
-  F 補零數量（若有）
- nr_dlsch_coding.c/nr_dlsch_encoding() 裡面呼叫nr_segmentation() 

**nrLDPC_coding_interface_load.c**
-  載入 NR LDPC 編碼與解碼函式庫（動態連結）
- 在nr_dlsch_coding.c nr_dlsch_encoding()的290行  gNB->nrLDPC_coding_interface.nrLDPC_coding_encoder(&slot_parameters);
-  nrLDPC_coding_encoder實做在nrLDPC_coding_segment_encoder.c

**ldpc_encoder.c**
- 主要實作 5G NR LDPC 編碼器 Base Graph (BG1 或 BG2) 對資料進行編碼

**nrLDPC_coding_segment_encoder.c**
- 主要負責實作 整體 LDPC 編碼流程的頂層整合，屬於 5G NR 下行/上行傳輸的實體層編碼總控模組
- **nrLDPC_coding_encoder** 
- **nrLDPC_launch_TB_encoding** 一個完整傳輸區塊 (TB) 的 LDPC 編碼任務流程控制器，創建並派送對應的執行緒任務給 ldpc8blocks() 處理每 8 個 segments 的編碼
-  **ldpc8blocks()**  功能是在一個執行緒中處理 最多 8 個 LDPC segments 的編碼與處理流程
```
| 步驟  | 函式 / 操作                        | 說明                                          |
| --- | ------------------------------ | ------------------------------------------- |
| 1️⃣ | `LDPCencoder()`                | 執行單個 segment 的 **LDPC 編碼（BG + Zc）**         |
| 2️⃣ | `nr_rate_matching_ldpc()`      | 執行 **比特匹配與填補 (Rate Matching + Filler Bit)** |
| 3️⃣ | `nr_interleaving_ldpc()`       | 執行 **位元交錯（Bit Interleaving）**               |
| ✅   | `completed_task_ans()`         | 標記此並行任務已完成                                  |
```
-  **write_task_output()** 輸出組裝階段 它讓多個 LDPC 編碼後的 segment 輸出資料，能整齊對齊、打包、串接在一起，形成一條可傳輸的 bit stream

| 函式                            | 作用                 | 所在檔案                              |
| ----------------------------- | ------------------ | --------------------------------- |
| `nr_dlsch_encoding()`         | 編碼主流程              | `nr_dlsch_coding.c`               |
| `nrLDPC_coding_encoder()`     | 呼叫多 CB 編碼          | `nrLDPC_coding_segment_encoder.c` |
| `nrLDPC_launch_TB_encoding()` | 為每個 CB 建立編碼任務      | 同上                                |
| `ldpc8blocks()`               | 進行單個 8-block 的編碼處理 | 同上                                |
| `LDPCencoder()`               | LDPC 編碼核心邏輯        | `ldpc_encoder.c`                  |
| `nr_rate_matching_ldpc()`     | 碼率匹配               | `nr_rate_matching.c`              |
| `nr_interleaving_ldpc()`      | 比特交錯               | `nr_interleaving.c`               |

# LDPC編碼架構流程圖(downlink)
```
┌────────────────────────────┐
│ MAC 層傳輸區塊 (TB)          | Output: 傳輸區塊 TB (未編碼) 
└────────────┬───────────────┘
             │
             ▼
┌────────────────────────────┐Input : TB 
│ nr_segmentation()          │ PHY/NR_TRANSPORT/nr_dlsch_coding.c/nr_dlsch_encoding()
│ 進行 TB 分段與 CRC 附加      │C 個 segments + 每段附加 CRC 
└────────────┬───────────────┘
             │
             ▼
┌────────────────────────────┐Input : 多個 segments
│ nrLDPC_coding_encoder()    │ PHY/NR_TRANSPORT/nr_dlsch_coding.c/nr_dlsch_encoding()呼叫
│ 開始 LDPC 編碼流程           │ nrLDPC_coding_encoder()(實做於nrLDPC_coding_segment_encoder.c)
└────────────┬───────────────┘Output: 每段交給 nrLDPC_launch_TB_encoding 
             │
             ▼
┌────────────────────────────┐ Input : segment d_i  
│ nrLDPC_launch_TB_encoding()│ PHY/CODING/nrLDPC_coding/nrLDPC_coding_segment_encoder.c
│ 建立多個編碼任務              │Output: 任務參數傳給 ldpc8blocks()  
└────────────┬───────────────┘
             │
             ▼
┌────────────────────────────┐Input : segment d_i + encoder_implemparams 
│ ldpc8blocks() 任務          │PHY/CODING/nrLDPC_coding/nrLDPC_coding_segment_encoder.c
│ 處理 8 段並行編碼            │Output: 調用 LDPCencoder() 編碼  
└────────────┬───────────────┘
             │
             ▼
┌────────────────────────────┐ Input : 資料 segment d_i      
│ LDPCencoder()              │PHY/CODING/nrLDPC_encoder/ldpc_encoder.c
│ 執行 QC-LDPC 編碼           │由ldpc8blocks()呼叫
└────────────┬───────────────┘ Output: LDPC 碼字 c_i  
             │
             ▼
┌────────────────────────────┐Input : LDPC 碼字 c_i  
│ nr_rate_matching_ldpc()    │nr_rate_matching.c 由ldpc8blocks()呼叫
│ 處理碼率匹配 (puncture/rep)  │
└────────────┬───────────────┘Output: 碼率匹配後位元 e_i (長度 E bits) 
             │
             ▼
┌────────────────────────────┐ Input : e_i   
│ nr_interleaving_ldpc()     │nr_rate_matching.c 由ldpc8blocks()呼叫
│ 執行比特交錯                 │ Output: 交錯後位元 f_i   
└────────────┬───────────────┘
             │
             ▼
┌────────────────────────────┐ Input : 多段 f_i                           
│ write_task_output()        │nrLDPC_coding_segment_encoder.c裡面
│ 拼接所有 segment 結果        │Output: 拼接成完整 Codeword bits f_all   
└────────────┬───────────────┘
             │
             ▼
┌────────────────────────────┐
│ 已拼接的 Codeword bits      │
└────────────┬───────────────┘

```
# decode
<img width="388" height="803" alt="image" src="https://github.com/user-attachments/assets/9fa47ab7-b330-4b46-8d48-b4fb31d5da2d" />

```
| 流程步驟                      | 說明                               |                                         
| ------------------------- | -------------------------------- | 
| Start Decoder             | 啟動解碼器流程                          |
| Load Parameters / Buffers | 設定 C、K、Z、E、F、LLR 等資訊             | 
| BN → CN 初始處理              | Variable nodes 傳遞資訊給 Check nodes | 
| CN → BN 初始處理              | Check nodes 計算校驗並回傳給變數節點         |
| First Parity Check        | 嘗試解出正確 codeword                  | 
| Output Bits               | 若成功，輸出 b\[], 否則重傳                | 
| Iterative Loop            | 進行最多 `max_ldpc_iterations` 次反覆計算 | 
| CN Processing             | 更新所有 parity constraints          | 
| CN→BN 回傳                  | 傳遞回 variable node                | 
| BN Processing             | 更新變數節點訊息                         | 
```

- 主入口函式為: nrLDPC_decoder_core in nrLDPC_decoder.c
- Decoder Setup
   - 載入 LUT（lookup tables）：供後續 CN/BN 運算使用
   - 初始化緩衝區：分別為 CN、BN 與 LLRs 分配記憶體空間
   - 資料對齊與 SIMD 準備：便於後續使用 AVX 加速
**LUT**
- LUT 是一種預先建立好資料的「表格」，讓演算法可以用索引快速查出對應值，而不是每次都重複計算
- 為什麼使用 LUT？:
   - 速度快、節省運算資源、適合定義明確的映射關係
 
**SIMD**
- SIMD 是一種 CPU 向量運算技術，允許一條指令同時操作多筆資料
- 在 LDPC 解碼中如何用到 SIMD
```
| 階段          | SIMD 加速的用途                               |
| ----------- | ---------------------------------------- |
| LLR 輸入處理    | 加速整批 bit 或 symbol 的 LLR 計算與存取            |
| BN→CN 訊息傳遞  | 同時處理多個 variable node 的 message           |
| CN→BN 訊息傳遞  | 比對最小值（min-sum），適合用 SIMD 做 parallel min() |
| CRC 計算      | 多 bit 並行 XOR                             |
| LLR 解調（調變器） | 輸入 IQ 值轉換為 LLR 時用 SIMD 加速 inner product  |
```
- First Pass Processing
   - BN Pre-processing with nrLDPC_bnProcPc_* (rate & BG-specific)
   - CN Pre-processing via nrLDPC_bn2cnProcBuf_*
   - First CN processing: nrLDPC_cnProc_*
   - Back-propagate to BN via nrLDPC_cn2bnProcBuf_*
 
- Iterative Decoding Loop
**執行條件**
  - numIter < numMaxIter
  - Parity check (or CRC) failed
 
- Steps per iteration
   - CN 運算
   - 將 CN 傳回 BN 的訊息
   - BN 運算（更新 soft LLR）
   - 準備下一輪 CN 輸入
   - 進行 Parity Check 或 CRC 驗證
 
- Early Termination
   - If check_crc is enabled
     - 解碼後會使用 CRC 驗證硬判定資料
     - 若通過驗證，提前結束迴圈
   - 若未啟用 CRC：
     - 每次都執行 parity-check 決定是否提早終止

- Output Formatting
- Output mode: LLRINT8, BIT, or BITINT8


