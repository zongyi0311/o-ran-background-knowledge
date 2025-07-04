# catalog
## PHY/

| 子資料夾                           | 功能                                      | 重點說明                                              |
| ------------------------------ | --------------------------------------- | ------------------------------------------------- |
| `NR_TRANSPORT/`                | 5G NR Transport Channels（DLSCH、ULSCH 等） | LDPC 編碼、Rate Matching、HARQ                        |
| `NR_UE_TRANSPORT/`             | UE 對應的 Transport 功能                     | UE 上行的 LDPC、發射管理                                  |
| `NR_REFSIG/`                   | NR 的 reference signal 模組                | 包含 **DMRS, PTRS, PRACH, SSB** waveform 產生與插入      |
| `MODULATION/`                  | OFDM IFFT/FFT、調變、符號對映等                  | 含 `nr_ofdm.c`, `nr_modulation.c`                  |
| `TOOLS/`                       | 通道估計、向量運算、FFT 工具、phase noise 等          | 包含 `nr_channel_estimation.c`, `get_phase_noise.c` |
| `INIT/`                        | Layer 1 變數初始化                           | 通常在 `phy_init_nr_ue()` 內呼叫                        |
| `CODING/`                      | LDPC、Polar 編碼與測試                        | 覆蓋 TS 38.212                                      |
| `defs.h`, `extern.h`, `vars.h` | 全域定義與變數引用                               | 模組間變數共用依賴這三個檔案架構                                  |

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
