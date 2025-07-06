# catalog
## PHY/
```
                    【 Downlink: gNB TX (模擬傳送端)】

Input: MAC PDU (bytes)
        ↓
┌──────────────────────────────┐
│ CRC 附加 (crc_byte.c)          │
│ Output: bits + CRC           │
└────────────┬─────────────────┘
             ↓
┌──────────────────────────────┐
│ Segmentation (nr_segmentation.c) │
│ Output: LDPC segments (bits) │
└────────────┬─────────────────┘
             ↓
┌──────────────────────────────┐
│ LDPC 編碼 (nr_dlsch_coding.c + CODING/) │
│ Output: encoded bits         │
└────────────┬─────────────────┘
             ↓
┌──────────────────────────────┐
│ Rate Matching + Interleaving │
│ Output: RM bits              │
└────────────┬─────────────────┘
             ↓
┌──────────────────────────────┐
│ 調變 (Modulation: QPSK, 16QAM) │
│ Output: complex symbols (IQ) │
└────────────┬─────────────────┘
             ↓
┌──────────────────────────────┐
│ OFDM IFFT + CP 加入 (ofdm_mod.c) │
│ Output: time-domain samples │
└────────────┬─────────────────┘
             ↓
┌──────────────────────────────┐
│ 模擬通道:  │
│ Input: TX samples            │
│ Output: RX samples           │
└────────────┬─────────────────┘
             ↓

                    【🔵 Uplink: gNB RX (模擬接收端)】

┌──────────────────────────────┐
│ OFDM FFT + CP 去除 (ofdm_mod.c) │
│ Input: RX samples            │
│ Output: frequency-domain symbols │
└────────────┬─────────────────┘
             ↓
┌──────────────────────────────┐
│ 通道估計 / 等化(nr_ulsch_demodulation.c)│
│ Output: equalized symbols   │
└────────────┬─────────────────┘
             ↓
┌──────────────────────────────┐
│ LLR 計算 (nr_ulsch_llr_computation.c) │
│ Output: soft bits (LLRs)    │
└────────────┬─────────────────┘
             ↓
┌──────────────────────────────┐
│ Rate Dematching              │
│ Input: LLRs                  │
│ Output: code blocks          │
└────────────┬─────────────────┘
             ↓
┌──────────────────────────────┐
│ LDPC 解碼 (nr_ulsch_decoding.c) │
│ Output: decoded bits         │
└────────────┬─────────────────┘
             ↓
┌──────────────────────────────┐
│ CRC 檢查 + Segment 合併       │
│ Output: MAC SDU (bytes)      │
└──────────────────────────────┘


```


| 子資料夾                           | 功能                                      | 重點說明                                              |
| ------------------------------ | --------------------------------------- | ------------------------------------------------- |
| `NR_TRANSPORT/`                | 5G NR Transport Channels（DLSCH、ULSCH 等） | LDPC 編碼、Rate Matching、HARQ                        |
| `NR_UE_TRANSPORT/`             | UE 對應的 Transport 功能                     | UE 上行的 LDPC、發射管理                                  |
| `NR_REFSIG/`                   | NR 的 reference signal 模組                | 包含 **DMRS, PTRS, PRACH, SSB** waveform 產生與插入      |
| `MODULATION/`                  | OFDM IFFT/FFT、調變、符號對映等                  |                   |
| `TOOLS/`                       | 通道估計、向量運算、FFT 工具、phase noise 等          |  |
| `INIT/`                        | Layer 1 變數初始化                           | 通常在 `phy_init_nr_ue()` 內呼叫                        |
| `CODING/`                      | LDPC、Polar 編碼與測試                        | 覆蓋 TS 38.212                                      |
| `defs.h`, `extern.h`, `vars.h` | 全域定義與變數引用                               | 模組間變數共用依賴這三個檔案架構                                  |

**openair1/PHY/NR_TRANSPORT/** //NR gNB transport channel procedures are here
| 檔案名稱             | 功能                                 |
| ---------------- | ---------------------------------- |
| `nr_dci.c`       | 處理 DCI 格式（如 format 1\_0/1\_1）與資訊打包 |
| `nr_dci.h`       | 定義 DCI 結構與控制符號參數                   |
| `nr_dci_tools.c` | 提供 DCI 解包、RNTI 比對、CRC 檢查等工具函式      | 
| `nr_dlsch.c`        | 下行資料調製、DMRS 插入與符號映射 | 
| `nr_dlsch.h`        | DLSCH 結構定義與資源管理     | 
| `nr_dlsch_coding.c` | CRC 加附、LDPC 編碼與率匹配  | 
| `nr_dlsch_tools.c`  | SIMD 資料對齊與符號操作輔助工具  | 
| `nr_ulsch.c`                 | UE 發送 ULSCH 資料封包（映射到 PUSCH）   | 
| `nr_ulsch.h`                 | 定義 ULSCH 結構與緩衝控制              | 
| `nr_ulsch_decoding.c`        | gNB 解碼 PUSCH（LDPC 解碼與 CRC 驗證） | 
| `nr_ulsch_demodulation.c`    | 頻域通道估計、MMSE 等化與符號提取           | 
| `nr_ulsch_llr_computation.c` | 根據調製方式計算軟判決 LLR               | 
| `nr_prach.c`        | 產生 PRACH 前導碼與接收匹配過濾器 | 
| `nr_prach.h`        | PRACH 參數與函式原型定義      | 
| `nr_prach_common.c` | PRACH 共用邏輯與前導分析工具    | 
| `nr_pbch.c` | MIB 封裝與 PBCH 導頻插入    | 
| `nr_pss.c`  | 產生主同步信號 PSS（用於時間同步）  | 
| `nr_sss.c`  | 產生次同步信號 SSS（用於小區 ID） | 
| `nr_sch_dmrs.c` | 產生 PDSCH/PUSCH 用的 DMRS | 
| `nr_sch_dmrs.h` | 定義 DMRS 參數與 API 原型     | 
| `nr_scrambling.c`             | 擾碼與解擾碼，符合 TS 38.211                        | 
| `nr_tbs_tools.c`              | TBS 計算、分段與組合邏輯                             | 
| `nr_transport_proto.h`        | 各類 NR 傳輸通道函式介面定義                           | 
| `nr_transport_common_proto.h` | 通用傳輸功能定義（如 PRACH 設定）                       | 
| `nr_uci_tools_common.c`       | 處理 UCI（如 SR, HARQ-ACK, CSI）                | 
| `pucch_rx.c`                  | gNB 接收 UE 傳送的 PUCCH（上行控制通道）                | 
| `srs_rx.c`                    | gNB 接收 UE 的 SRS（Sounding Reference Signal） |
| `nr_prs.c`                    | 位置參考訊號（Positioning Reference Signal）處理     | 


## SCHED_NR/、SCHED_NR_UE/

| 資料夾                  | 功能                                | 備註                   |
| -------------------- | --------------------------------- | -------------------- |
| `SCHED_NR/`          | gNB 端物理層資源分配（prach, pucch, pdsch） | 包含 Msg1\~4 控制        |
| `SCHED_NR_UE/`       | UE 端物理層資源處理                       | 決定何時發送 PRACH、PUSCH 等 |
| `prach_procedures.c` | LTE 的 PRACH 實作                    | NR 的在 `NR_REFSIG` 處理 |

## SIMULATION/

| 子資料夾                  | 說明                                                |
| --------------------- | ------------------------------------------------- |
| `SIMULATION/TOOLS/`   | **Channel model 實作主體**（AWGN、EPA、EVA、phase noise） |
| `SIMULATION/NR_PHY/`  | 5G NR 模擬主程式，如 `nr_dlschsim.c`, `nr_pbchsim.c`    |
| `SIMULATION/RF/`      | RF 模擬收發器邏輯，可模擬 USRP 環境下的傳輸延遲                      |
| `SIMULATION/LTE_PHY/` | 舊版 LTE PHY 模擬器（4G 測試用）                            |

# [channel_simulation.md](https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/openair1/SIMULATION/TOOLS/DOC/channel_simulation.md#channel-modeling)

##  Channel Modeling（通道建模）

### 定義
- 通道模型是模擬無線訊號在傳輸媒介中傳播行為的數學模型。
- 模型會考慮：
  - 衰減（Attenuation）
  - 干擾（Interference）
  - 衰落（Fading）

### 應用目的
- 用於預測不同環境下無線系統的效能。
- 幫助設計與測試更可靠的通信系統。

##  OAI Channel Simulation 功能

### 功能說明
- RFSimulator 提供 **通道模擬功能**。
- 能夠 **修改時域樣本**，模擬真實通道行為。
- 使用 **預定模型**（如 3GPP TR 36.873、TR 38.901）。

### 實作架構
- 通道模擬程式碼存在於：
  - UE（User Equipment）
  - gNB（5G 基站）
  - eNB（4G 基站）
- 可與：
  - **RFSimulator**
  - **L1 Simulator**
  - **PHY Simulators** 搭配使用。

### 配置方式
- RFSimulator 提供 **最完整的配置與即時參數調整能力**。
- PHY 模擬器只能透過 CLI 指令進行部分設定。

## new_channel_desc_scm()
這個函式是用來建立一個新的通道模型描述子（channel_desc_t 結構），並根據提供的參數初始化相關設定
```
channel_desc_t *new_channel_desc_scm(
  uint8_t nb_tx,
  uint8_t nb_rx,
  SCM_t channel_model,
  double sampling_rate,
  uint64_t center_freq,
  double channel_bandwidth,
  double DS_TDL,
  double maxDoppler,
  const corr_level_t corr_level,
  double forgetting_factor,
  int32_t channel_offset,
  double path_loss_dB,
  float noise_power_dB
);
```

| 參數名稱                | 說明                                     |
| ------------------- | -------------------------------------- |
| `nb_tx`             | 傳輸天線數（number of TX antennas）           |
| `nb_rx`             | 接收天線數（number of RX antennas）           |
| `channel_model`     | 使用的通道模型（例如 AWGN、TDL、CDL）               |
| `sampling_rate`     | 取樣率（Hz），通常與 numerology 與子載波間距有關        |
| `center_freq`       | 中心頻率（Hz），模擬通道所運作的載波頻率                  |
| `channel_bandwidth` | 頻寬（Hz），代表整個頻道覆蓋的頻率範圍                   |
| `DS_TDL`            | 延遲擴展參數（Delay Spread），TDL 模型中用來調整多徑分佈程度 |
| `maxDoppler`        | 最大 Doppler 位移（Hz），模擬移動造成的頻率偏移（如用戶移動）   |
| `corr_level`        | 天線間的相關性等級（Low, Medium, High）           |
| `forgetting_factor` | 衰減記憶係數，用於模型內部時間相關性運算                   |
| `channel_offset`    | 通道的初始偏移量，用於測試偏移效應                      |
| `path_loss_dB`      | 路徑損耗（dB）                               |
| `noise_power_dB`    | 加入的雜訊功率（dB）                            |

## Channel Model configuration file
e.g. a simple scenario to with an AWGN channel:
```
channelmod = {
  max_chan = 10; -- 最大支援 10 條通道
  modellist = "modellist_rfsimu_1";
  modellist_rfsimu_1 = (
    {
      model_name     = "rfsimu_channel_enB0",  -- 下行通道（gNB -> UE）
      type           = "AWGN";                 -- 通道模型類型：AWGN
      ploss_dB       = 20;                     -- 路徑損耗：20 dB
      noise_power_dB = -4;                     -- 雜訊功率：-4 dB
      forgetfact     = 0;                      -- 忘記因子（時變相關性），0 表示靜態
      offset         = 0;                      -- 初始偏移
      ds_tdl         = 0;                      -- TDL 延遲擴散參數（此處為 0 表示不用）
    },
    {
      model_name     = "rfsimu_channel_ue0",   -- 上行通道（UE -> gNB）
      type           = "AWGN";
      ploss_dB       = 20;
      noise_power_dB = -2;                     -- 注意這裡的雜訊功率不同：-2 dB
      forgetfact     = 0;
      offset         = 0;
      ds_tdl         = 0;
    }
  );
};

```
- `rfsimu_channel_ue0`：UE 端通道（Server），模擬 **Uplink**。
- `rfsimu_channel_enB0`：gNB 端通道（Client），模擬 **Downlink**。
- `ue0`、`ue1`、`ue2`...：代表不同 client 的通道模型。

## Set channel simulation parameters via CL:

### Global parameters

| CL 參數名稱            | 類型          | 預設值                  | 說明                                                                               |
| ------------------ | ----------- | -------------------- | -------------------------------------------------------------------------------- |
| `modellist`        | char string | `DefaultChannelList` | 指定要載入的通道模型清單名稱，來自設定檔                                                             |
| `max_chan`         | integer     | `10`                 | 系統允許的最大通道模型數量，必須大於實際使用的模型數量                                                      |
| `noise_power_dBFS` | integer     | `0`                  | 全域雜訊功率（以 dBFS 表示）。若設定此值，每個模型中的 `noise_power_dB` 將無效。為了產生正的 SNR，請使用低於 -36 dBFS 的值 |

### Model List

| 參數名稱             | 類型           | 預設值      | 說明                                              |
| ---------------- | ------------ | -------- | ----------------------------------------------- |
| `model_name`     | 字串 (string)  | **必要**   | 模型的名稱（在代碼中用來識別模型）                               |
| `type`           | 字串 (string)  | `"AWGN"` | 所使用的通道模型類型，例如：`AWGN`、`TDL`、`CDL`。定義於 `sim.h`    |
| `ploss_dB`       | 實數 (float)   | `0`      | 通道總路徑損耗（含 shadow fading），單位為 dB                 |
| `noise_power_dB` | 實數 (double)  | `-50`    | 雜訊功率，用來計算信噪比（SNR），值越大表示雜訊越強                     |
| `forgetfact`     | 實數 (double)  | `0`      | 忘記因子，用於模擬時間變化。0 代表每個樣本都是新通道；1 則通道保持不變           |
| `offset`         | 整數 (integer) | `0`      | 對輸入訊號的取樣位移（單位為樣本數）                              |
| `ds_tdl`         | 實數 (double)  | `0`      | 延遲擴展（Delay Spread），用於 TDL 類型的模型（如 TDL-A, TDL-C） |

# random_channel.c
-  #include "PHY/TOOLS/tools_defs.h"
-  #include "sim.h"
-  #include "scm_corrmat.h"
-  #include "common/config/config_userapi.h"
-  #include "common/utils/telnetsrv/telnetsrv.h"
-  #include "common/utils/load_module_shlib.h"

| 標頭檔案                  | 功能簡述                      |
| --------------------- | ------------------------- |
| `tools_defs.h`        | 一些底層定義與通用工具               |
| `sim.h`               | 模擬器參數與模型定義（例如 AWGN、TDL 等） |
| `scm_corrmat.h`       | SCM correlation matrix 計算 |
| `config_userapi.h`    | 用來操作 YAML/JSON 設定檔的介面     |
| `telnetsrv.h`         | 用來啟動 telnet server，支持命令操作 |
| `load_module_shlib.h` | 支援模組化動態載入（shared library） |

# `random_channel.c` 函式整理
---

## 通道模型初始化與記憶體管理

| 函式名稱                  | 功能說明                                           |
|---------------------------|----------------------------------------------------|
| `new_channel_desc_scm`    | 建立並回傳一個新的通道模型描述子                  |
| `fill_channel_desc`       | 將模型參數填入 descriptor 結構                    |
| `init_channelmod`         | 初始化通道模擬模組與 telnet 控制命令註冊          |
| `free_channel_desc_scm`   | 釋放通道模型所佔的記憶體資源                      |

---

## 通道模擬核心邏輯

| 函式名稱                  | 功能說明                                           |
|---------------------------|----------------------------------------------------|
| `random_channel`          | 根據設定參數產生衰落通道樣本（AWGN、TDL...）     |
| `tdlModel`                | 模擬 TDL 類通道（例如 TDL-A, TDL-C 等）           |
| `get_cexp_doppler`        | 產生 Doppler 頻移效應所需的複數指數樣本          |
| `get_normalization_ch_factor` | 計算通道能量正規化係數                           |
| `get_noise_power_dBFS`    | 將 dBFS 雜訊功率轉換為通道模擬中的真實值         |

---

## 通道模型屬性設定與查詢

| 函式名稱                  | 功能說明                                           |
|---------------------------|----------------------------------------------------|
| `set_channeldesc_name`    | 設定通道模型描述子的名稱                          |
| `set_channeldesc_direction` | 設定為上行/下行通道                               |
| `set_channeldesc_owner`   | 指定該通道屬於哪個模擬端（如 gNB 或 UE）          |
| `find_channel_desc_fromname` | 根據名稱尋找通道模型結構                         |
| `get_channel_params`      | 回傳目前通道參數（for telnet 顯示）              |

---

## Telnet 與 Web 操控支援函式

| 函式名稱                  | 功能說明                                           |
|---------------------------|----------------------------------------------------|
| `get_modchannel_index`    | 取得指定模型的通道索引                            |
| `get_currentchannels_type`| 查詢目前通道模型的型態（AWGN、TDL 等）           |

---

## 模型載入與轉換工具

| 函式名稱                  | 功能說明                                           |
|---------------------------|----------------------------------------------------|
| `load_channellist`        | 從設定檔載入 modellist，並建立對應通道模型        |
| `modelid_fromstrtype`     | 字串模型名稱轉換為內部類型（例如 AWGN → 0）      |

---

## 通道參數與頻率對應工具

| 函式名稱                  | 功能說明                                           |
|---------------------------|----------------------------------------------------|
| `N_RB2sampling_rate`      | 根據 N_RB 計算對應的取樣率（Hz）                  |
| `N_RB2channel_bandwidth`  | 根據 N_RB 推算通道頻寬（Hz）                      |

---

### **random_channel()**
```
int random_channel(channel_desc_t *desc, uint8_t abstraction_flag) {
  double s;
  int i,k,l,aarx,aatx;
  struct complexd anew[desc->nb_tx*desc->nb_rx];
  struct complexd acorr[desc->nb_tx*desc->nb_rx];
  struct complexd phase, alpha, beta;
  start_meas(&desc->random_channel);
```
**strust channel_desc_t**:通道描述結構，包含所有通道參數與內部狀態
```
uint8_t nb_tx;           // 傳送天線數量
uint8_t nb_rx;           // 接收天線數量
uint8_t nb_taps;         // 多徑 taps 數量
double *amps;            // 每個 tap 的振幅（大小）
double *delays;          // 每個 tap 的延遲（單位 microseconds）
uint8_t channel_length;  // 脈衝響應長度 (11 + 2*bw*Tmax)
struct complexd **a;     // 時變 channel 狀態向量，每個 tap 對應每對 TX-RX
struct complexd **ch;    // 經過內插的通道脈衝響應（sample-spaced）
struct complexd **chF;   // 頻域響應（90 kHz resolution）
double Td;               // 最大 path delay（µs）
uint64_t center_freq;    // 載波中心頻率（Hz）
double channel_bandwidth;// 通道頻寬（MHz）
double sampling_rate;    // 系統取樣率（Msps）
double ricean_factor;    // Ricean K 因子轉換 (0=AWGN, 1=Rayleigh)
corr_level_t corr_level; // 通道矩陣的相關性等級
typedef enum {//列舉型別
  CORR_LEVEL_LOW,     // 低相關性（例如理想 MIMO 條件）
  CORR_LEVEL_MEDIUM,  // 中度相關性
  CORR_LEVEL_HIGH     // 高相關性（例如密集天線排布）
} corr_level_t;
double aoa;              // 到達角度（radians）
int8_t random_aoa;       // AoA 是否隨機產生
double max_Doppler;      // 最大 Doppler 頻率（尚未實作）
double path_loss_dB;     // 路徑損耗（dB）
float noise_power_dB;    // 雜訊功率（dB）
uint64_t channel_offset; // 額外通道延遲（samples）
double forgetting_factor;// 通道變化因子（0=fast fading, 1=static）
uint8_t first_run;       // 是否為第一次呼叫
double ip;               // 頻偏模擬的初始相位
uint16_t nb_paths;       // 有效路徑數量
int modelid;                         // 模型 ID（AWGN / TDL-A / CDL 等）
channelmod_moduleid_t module_id;    // 來源模組（如 rfsimulator, phy-sim）
typedef enum {
  UNSPECIFIED_MODID = 0,//預設不指定
  RFSIMU_MODULEID   = 1//表示通道模型要應用在 rfsimulator 裡
} channelmod_moduleid_t;
char *model_name;                   // 模型名稱（例如：rfsimu_channel_ue0）
unsigned int chan_idx;              // 描述子在全域陣列中的索引
unsigned int free_flags;            // 是否釋放記憶體
float sat_height;                   // LEO 衛星高度（公里）
bool enable_dynamic_delay;          // 是否啟用變動延遲模擬
bool enable_dynamic_Doppler;        // 是否啟用變動 Doppler 模擬
float Doppler_phase_inc;            // Doppler 相位遞增量
float *Doppler_phase_cur;           // 當前 Doppler 相位（每個 RX 天線）
bool is_uplink;                     // 是否為上行通道
time_stats_t random_channel;        // 整體通道模擬時間
time_stats_t interp_time;           // 時域內插時間
time_stats_t interp_freq;           // 頻域轉換時間
time_stats_t convolution;           // 卷積處理時間
```

**struct complexd anew**
- 新生成的 channel tap 值
- 每一對 TX-RX 對應一個複數（例如 2x2 MIMO 有 4 個）

**struct complexd acorr**
- 上一個時間點的 channel tap（用於時間相關性）

**struct complexd phase, alpha, beta**
- `phase`: 複數相位旋轉（如 Doppler shift）
- `alpha`: 記憶因子（0=新通道, 1=完全保留前一個）
- `beta`: 新 tap 的權重 

```
if (desc->modelid == AWGN || desc->modelid == SAT_LEO_TRANS || desc->modelid == SAT_LEO_REGEN) {//如果模型是 AWGN 或 LEO 衛星通道，就進入此區塊

    for (aarx=0; aarx<desc->nb_rx; aarx++) {//建立 AWGN 模型下的通道矩陣
      for (aatx = 0; aatx < desc->nb_tx; aatx++) {//掃描每一對 TX → RX 天線組合。
        desc->ch[aarx+(aatx*desc->nb_rx)][0].r = aarx%desc->nb_tx == aatx ? 1.0 : 0.0;
        desc->ch[aarx+(aatx*desc->nb_rx)][0].i = 0.0;
        acorr[aarx+(aatx*desc->nb_rx)].r = desc->ch[aarx+(aatx*desc->nb_rx)][0].r;
        acorr[aarx+(aatx*desc->nb_rx)].i = desc->ch[aarx+(aatx*desc->nb_rx)][0].i;
      }
    }
    memcpy(desc->a[0], acorr, desc->nb_tx * desc->nb_rx * sizeof(*acorr));//把通道資料從 acorr 複製到 desc->a[0]
    stop_meas(&desc->random_channel);
    desc->first_run = 0;
    return 0;
  }
```
- 範例：AWGN 模型通道矩陣初始化 (nb_tx = 2, nb_rx = 2)
ch[] 實體索引（展平記憶體）：
- index = aarx + aatx * nb_rx

運算結果：
| aarx | aatx | index | 設定結果 (ch[index].r) |
|------|------|-------|--------------------------|
| 0    | 0    | 0     | 1.0                  |
| 1    | 0    | 1     | 0.0                  |
| 0    | 1    | 2     | 0.0                   |
| 1    | 1    | 3     | 1.0                   |


```
memcpy(desc->a[0], acorr, desc->nb_tx * desc->nb_rx * sizeof(*acorr));//把通道資料從 acorr 複製到 desc->a[0]
    stop_meas(&desc->random_channel);//結束測量通道初始化
    desc->first_run = 0;//第一次初始化完後，設定為 0，避免再次重建通道矩陣
```
bzero(acorr,desc->nb_tx*desc->nb_rx*sizeof(struct complexd));//將複數陣列 acorr[] 清零

**bzero()** 是 老式 UNIX/C 函式，作用是將記憶體區塊的所有位元設為 0 它的功能等價於：
memset(acorr, 0, desc->nb_tx * desc->nb_rx * sizeof(struct complexd));
```
desc->nb_tx = 2;
desc->nb_rx = 2;
bzero(acorr, 2×2×sizeof(struct complexd));
等同於清除 acorr[0] ~ acorr[3] 的所有 r 與 i 欄位 → 都設成 0.0
```
---

```
for (i=0; i<(int)desc->nb_taps; i++) {//處理多徑 tap 與 MIMO 通道 為每個 tap × 每對天線產生通道樣本模擬 fading、Doppler、時變通道等效應
    for (aarx=0; aarx<desc->nb_rx; aarx++) {
      for (aatx=0; aatx<desc->nb_tx; aatx++) {
```

```
struct complexd *anewp = &anew[aarx + (aatx * desc->nb_rx)];
        anewp->r = sqrt(desc->ricean_factor * desc->amps[i] / 2) * gaussZiggurat(0.0, 1.0) * desc->normalization_ch_factor;
        anewp->i = sqrt(desc->ricean_factor * desc->amps[i] / 2) * gaussZiggurat(0.0, 1.0) * desc->normalization_ch_factor;
```
模擬 Ricean 通道模型時產生複數通道係數（fading tap）

| 變數名稱                            | 解釋                                      |
| ------------------------------- | --------------------------------------- |
| `gaussZiggurat(0.0, 1.0)`       | 產生一個均值為 0、標準差為 1 的高斯亂數                  |
| `desc->amps[i]`                 | 第 i 個 tap 的功率（來自 power delay profile）   |
| `desc->ricean_factor`           | Ricean K 因子的轉換參數（0 = AWGN，1 = Rayleigh） |
| `desc->normalization_ch_factor` | 通道正規化係數，確保總能量一致                         |

![image](https://github.com/user-attachments/assets/70af680b-c5de-4eb4-8a3f-cfb3283b732a)

