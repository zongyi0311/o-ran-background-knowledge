| 檔案名稱                          | 功能                                                                |
| ----------------------------- | ----------------------------------------------------------------- |
| `nr_polar_encoder.c`          | Polar 編碼主程式：負責將 bit `u` 透過 Gₙ 轉換為碼字 `d`                           |
| `nr_polar_decoder.c`          | Polar 解碼主程式：包含 SC / SCL / CA-SCL 解碼器實作                            |
| `nr_polar_procedures.c`       | 封裝 encoder 使用的輸入資料處理，如加入 CRC、bit 選擇、rate matching 等               |
| `nr_polar_init.c`             | Polar encoder 初始化流程，包括 reliability sequence、bit mask、G matrix 等設定 |
| `nr_polar_decoding_tools.c`   | 解碼時使用的輔助工具，例如 LLR 操作與 decision support                            |
| `nr_polar_kernal_operation.c` | 具體實作 polar encoder/decoder 遞迴運算核心（如 butterfly network）            |
| 檔案名稱                    | 用途                                                    |
| ----------------------- | ----------------------------------------------------- |
| `nr_polar_defs.h`       | 通用 Polar encoder/decoder 參數結構定義（如 `t_nrPolar_params`） |
| `nr_polar_dci_defs.h`   | 定義 DCI 專用的 Polar encoding 配置參數                        |
| `nr_polar_pbch_defs.h`  | 定義 PBCH 專用 Polar 結構與輸出長度                              |
| `nr_polar_psbch_defs.h` | 定義 PSBCH（Positioning）相關 Polar 結構                      |
| `nr_polar_pucch_defs.h` | 定義 PUCCH 上行控制通道的 Polar 結構                             |
| `nr_polar_uci_defs.h`   | 定義通用上行控制資訊（UCI）的 Polar 配置                             |
| 檔案名稱                                  | 用途                                                              |
| ------------------------------------- | --------------------------------------------------------------- |
| `nr_polar_sequence_pattern.c`         | 初始化 Polar reliability sequence $Q_0^{N-1}$，即 bit reliability 順序 |
| `nr_polar_interleaving_pattern.c`     | 建立 Polar 特定的 interleaving 順序                                    |
| `nr_polar_matrix_and_array.c`         | 處理 G matrix、bit mapping、helper arrays 等                         |
| `nr_polar_kronecker_power_matrices.c` | 建立 Kronecker power Gₙ 矩陣（很少更新）                                  |
| 檔案名稱                      | 用途                                            |
| ------------------------- | --------------------------------------------- |
| `nr_bitwise_operations.c` | 提供 bit-level 操作，如 XOR、interleave、bit mask 操作等 |
| `nr_crc_byte.c`           | 提供 CRC24A/B/C 等演算法（Polar encoder 預加 CRC 用）    |
| 類別                    | 相關檔案                                                                                               |
| --------------------- | -------------------------------------------------------------------------------------------------- |
| Encoder / Decoder 主流程 | `nr_polar_encoder.c`, `nr_polar_decoder.c`, `nr_polar_procedures.c`, `nr_polar_kernal_operation.c` |
| 初始化 / 結構設定            | `nr_polar_init.c`, `nr_polar_matrix_and_array.c`, `nr_polar_sequence_pattern.c`                    |
| 解碼工具                  | `nr_polar_decoding_tools.c`                                                                        |
| 定義檔（Header）           | `*_defs.h` 全部                                                                                      |
| 工具與輔助功能               | `nr_bitwise_operations.c`, `nr_crc_byte.c`                                                         |

🔹 1. polar_encoder()
功能：標準 Polar 編碼器

用途：將輸入比特 (TB payload) 經過 CRC、interleaving、bit insertion、Polar transform 與 rate matching，產生編碼輸出。

主要流程：

取得對應的 Polar 參數：nr_polar_params()

將輸入位元從 uint32_t 轉換成 uint8_t：nr_bit2byte_uint32_8()

計算 CRC：nr_matrix_multiplication_uint8_1D_uint8_2D()

附加 CRC：填入 B 向量

Interleaving：nr_polar_interleaver()

Bit insertion：nr_polar_bit_insertion()

Polar encoding：G_N 矩陣乘法

🔹 2. polar_encoder_dci()
功能：針對 DCI 專用的 Polar 編碼器（加上 RNTI scrambling）

用途：符合 TS 38.212 §7.3.2 用於 DCI 的特殊編碼程序。

主要差異：

將輸入補上固定 1 值區塊：a' 長度為 K

附加 RNTI scrambling（只有 DCI 用）

其他步驟與 polar_encoder() 相似

🔹 3. nr_polar_rm_interleaving_cb()
功能：進行 rate-matching 後的 interleaving（CB interleaving）

用途：TS 38.212 §5.4.1.3，子區塊交錯

邏輯：

將輸入位元依三角矩陣轉置方式重排，強化編碼位元的均勻性

Rate matching：再次 interleaving

回傳：nr_byte2bit_uint8_32() 將 uint8_t 輸出轉成 uint32_t

🔹 4. polar_rate_matching()
功能：進行 Polar 編碼的 rate matching（重排序與截取）

用途：根據 rate_matching_pattern 抽取要傳送的位元

支援：

支援 group size = 2, 4, 8, 16

當 i_bil == 1 時執行 interleaving：呼叫 nr_polar_rm_interleaving_cb()

🔹 5. build_polar_tables()
功能：建立 Polar 編碼器需要的查表結構

主要工作：

建立 c' interleaving 對應的查表（cprime_tab0, cprime_tab1）

分析 rate matching 的最小 group size → 設定 groupsize

建立 rm_tab 查表，用於快速 rate matching

建構 Decoder Tree（呼叫 build_decoder_tree()）

🔹 6. polar_encoder_fast()

功能：快速版 Polar 編碼器（適用 DCI 與短訊框）

用途：使用查表與簡化記憶體結構來加速 Polar 編碼

步驟摘要：

optional：將 24 個 1 加在開頭（ones_flag 為 1 時）

flip endian、做 CRC → 使用硬體加速函式（如 crc24c()）

建立 B 向量：payload + CRC

查表組成 c′：使用 cprime_tab0, cprime_tab1

生成 u：呼叫 nr_polar_generate_u()

編碼成 D：nr_polar_uxG()

rate matching：polar_rate_matching()

