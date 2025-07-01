整理自[EN 302 307-2 - V1.4.1 - Digital Video Broadcasting (DVB)](https://www.etsi.org/deliver/etsi_en/302300_302399/30230702/01.04.01_60/en_30230702v010401p.pdf) and [EN 302 307 - V1.3.1 - Digital Video Broadcasting (DVB)](https://www.etsi.org/deliver/etsi_en/302300_302399/302307/01.03.01_20/en_302307v010301a.pdf)

# DVB-S2X Introduction(page 9)(EN 302 307-2 - V1.4.1 - Digital Video Broadcasting (DVB))

## DVB-S2X 擴充說明
- DVB-S2X 是 **DVB-S2 的非相容擴充版本**，於 2014 年通過。
- 雖然 **與原 ETSI EN 302 307 標準不相容**，但根據 ETSI EN 302 307-1，它是新的接收器實作的 **可選擴充項目**。
- 然而，在 S2X 的新標準中，這些擴充是 **強制實作的規範要求**。

## 主要應用場景
- 傳統 S2 應用：  
  - 數位電視廣播（DVB）
  - 互動服務前向鏈路（ACM）
  - 數位衛星新聞採集（DSNG）
  - 專業點對點傳輸與網路幹線

- 新興應用領域：  
  - **VL-SNR（極低信噪比）操作條件**  
  - 包含：
    - 航空寬頻（如商務噴射機）
    - 海事與民航網路
    - 高頻或熱帶區的 VSAT 終端
    - 可攜式記者終端（如直播）

## 以 DTH 為例的使用案例
- 可用於 **UHDTV-1（如 4K）電視服務**，搭配 **HEVC 編碼**。
- 可將多個 DTH 載波的小頻寬碎片 **聚合成邏輯串流**（transponder bonding）：
  - 避免空間浪費
  - 提升統計式多工效率

## S2X 技術優勢
- 支援極低信噪比 **（最低 -10 dB）** 運作條件
- 適用高噪環境與高效率傳輸市場
- 提供更高的頻譜效率與吞吐量，特別對於高品質專業鏈路
- 加入以下技術強化：
  - 更細緻的 **MODCOD 階段設計**
  - 更尖銳的 **roll-off 濾波器**
  - 多轉發器頻寬 **聚合（Bonding）技術**
  - **Periodic Super-Frame** 結構
  - 延伸的 **PLHEADER 訊號格式**
  - 支援 **GSE-Lite** 訊號

![image](https://github.com/user-attachments/assets/6f70fbaa-f2d6-4a72-9121-cdc957a76154)

# 4.2 系統架構（System Architecture）(page 14)（DVB-S2)

## 系統組成概述
根據 Figure 1，**DVB-S2 系統架構**由一系列功能模組構成，主要包含以下幾個階段：

---

## Mode Adaptation（模式調整）

### 功能定義
- 根據應用類型決定調整方式
- 主要職責：
  - 輸入串流介面（Input Interface）
  - 資料流同步（Input Stream Synchronization）
  - Null 封包刪除（僅限 TS 格式 + ACM 模式）
  - CRC-8 編碼（僅限封包化的輸入格式）
  - 多路輸入串流合併（MIS 模式下）
  - 切割成 **DATA FIELDs**

###  運作模式
- **CCM + 單一 TS 輸入**：
  - 透明傳輸（DVB-ASI / DVB-parallel → logical bits）+ CRC-8 編碼
- **ACM 模式**：
  - 需符合 Annex D 的模式調整流程

---

##  Base-Band Header 插入
- 每個 Data Field 前都會加上一個 **BBHEADER**
- 用於：
  - 標示輸入串流格式（TS / GS）
  - 指定 Mode Adaptation 類型
- ⚠ MPEG TS 封包與 Base-Band Frame 的對應可能是非同步的

---

## Stream Adaptation（串流調整）

### 功能包含：
- **Padding**：補足不足的 BBFRAME 長度
- **Scrambling**：對 Base-Band Frame 進行隨機化處理

---

## Forward Error Correction (FEC) 

###  編碼架構
- 採用 **串接式錯誤更正機制**：
  - **BCH 外碼**（Outer Code）
  - **LDPC 內碼**（Inner Code）

###  支援的 LDPC 編碼率（Code Rates）
- 1/4、1/3、2/5、1/2、3/5、2/3、3/4、4/5、5/6、8/9、9/10

###  編碼區塊大小（FECFRAME）
| 應用區塊類型      | 區塊大小（n<sub>LDPC</sub>） |
|-------------------|-------------------------------|
| Normal FEC Frame  | 64,800 bits                   |
| Short FEC Frame   | 16,200 bits                   |

---

### 可變與自適應模式支援
- 在使用 **VCM（Variable Coding and Modulation）** 或 **ACM（Adaptive Coding and Modulation）** 模式時：
  - **每個 frame 可切換不同 FEC 編碼與調變方式**
  - 但一個 frame 內的 FEC 和 modulation 必須固定不變

###  Bit Interleaving（位元交錯）
- 對下列調變格式的 FEC 輸出位元 **進行交錯處理** 以改善錯誤擴散特性：
  - **8PSK**
  - **16APSK**
  - **32APSK**

## 調變與實體層框架（Modulation & Physical Layer Framing）

### 調變方式（Constellation Mapping）

- 支援以下調變格式，依應用領域選擇：
  - **QPSK**
  - **8PSK**
  - **16APSK**
  - **32APSK**
- QPSK 與 8PSK 採用 **Gray Mapping**，以減少位元錯誤擴散。

---

### 實體層框架（Physical Layer Framing）

####  功能
- 同步於 FECFRAME，提供：
  - **Dummy PLFRAME 插入**（當無可用資料傳送時）
  - **PL Signalling**（傳輸模式、MODCOD、Frame 開始位元等）
  - **Pilot Symbol 插入（可選）**
  - **PL Scrambling**（分散功率密度，避免頻率集中）

#### PLFRAME 結構（以 SLOT 為單位）
- 系統以 **SLOT = 90 調變符號** 為單位組成 frame。
- 每個 SLOT 中包含：
  - 資料符號
  - 或 PILOT 符號（每 16 個 SLOT 插入 36 個 PILOT 符號）
  - 或 PL Signalling 欄位（包含 Start-of-Frame 等）

---

### 功能支援
- 支援 **VCM / ACM** 調變切換：
  - Frame-by-frame 調整 MODCOD，可提升頻譜效率
- 支援 **接收端同步**：
  - 使用 SLOT 結構 + PILOT 配置，增強 **Frame 對齊與載波恢復（Carrier Recovery）**

---

##  基帶濾波與正交調變（Base-Band Filtering & Quadrature Modulation）

###  功能目的
- 將數位調變後的訊號轉換為適合射頻（RF）傳輸的模擬訊號
- 同時控制頻譜形狀，降低旁帶干擾與頻寬占用

---

###  基帶濾波（Base-Band Filtering）
- 採用 **根升餘弦濾波器（Square Root Raised Cosine, SRRC）**
- 用於限制訊號頻譜展寬，減少符號間干擾（ISI）

####  支援的 Roll-off 系數（α）
| α 值 | 頻譜利用效率 | 特性說明                     |
|------|----------------|------------------------------|
| 0.35 | 低（最寬頻譜） | 傳統 DVB-S2 支援              |
| 0.25 | 中              | 折衷效率與實作難度           |
| 0.20 | 高（最窄頻譜） | 頻譜壓縮更好，但設計更複雜   |

---

###  正交調變（Quadrature Modulation）
- 使用 I/Q 組合將訊號調變至載波頻率，產生 RF 輸出
- 支援的調變方式（依 MODCOD 選擇）：
  - **QPSK**
  - **8PSK**
  - **16APSK**
  - **32APSK**

---

###  最終輸出
- 經過濾波與調變後，輸出為適合傳送到 **RF 衛星通道** 的訊號

## 📋 系統組態與應用領域對照表（Table 1）(page 16)(DVB-S2)

| **System Configurations**         | **Broadcast** | **Interactive** | **DSNG** | **Professional** |
|----------------------------------|---------------|------------------|----------|------------------|
| **QPSK**                         | 1/4, 1/3, 2/5  | O                | N        | N                |
|                                  | 1/2, 3/5, 2/3, 3/4, 4/5, 5/6, 8/9, 9/10 | N | N | N |
| **8PSK**                         | 3/5, 2/3, 3/4, 5/6, 8/9, 9/10 | N | N | N |
| **16APSK**                       | 2/3, 3/4, 4/5, 5/6, 8/9, 9/10 | O | N | N |
| **32APSK**                       | 3/4, 4/5, 5/6, 8/9, 9/10     | O | N | N |
| **CCM**                          | N             | N (see note 1)   | N        | N                |
| **VCM**                          | O             | O                | O        | O                |
| **ACM**                          | NA            | N (see note 2)   | O        | O                |
| **FECFRAME (normal)** (64,800)   | N             | N                | N        | N                |
| **FECFRAME (short)** (16,200)    | NA            | N                | O        | N                |
| **Single Transport Stream**      | N             | N (see note 1)   | N        | N                |
| **Multiple Transport Streams**   | O             | O (see note 2)   | O        | O                |
| **Single Generic Stream**        | NA            | O (see note 2)   | NA       | O                |
| **Multiple Generic Streams**     | NA            | O (see note 2)   | NA       | O                |
| **Roll-off: 0.35, 0.25, 0.20**   | N             | N                | N        | N                |
| **Input Stream Synchronizer**    | NA except (note 3) | O (note 3) | O (note 3) | O (note 3)      |
| **Null Packet Deletion**         | NA except (note 3) | O (note 3) | O (note 3) | O (note 3)      |
| **Dummy Frame Insertion**        | NA except (note 3) | N         | N        | N                |
| **Wide-band Mode** (Annex M)     | O             | O                | O        | O                |

---

- **Note 1**: 互動式接收器（Interactive receivers）需支援 **CCM** 和 **Single Transport Stream**
- **Note 2**: 互動式接收器需支援 **ACM**，且需實作以下任一輸入：
  - Multiple Transport Streams
  - Generic Stream（單/多重輸入皆可）
- **Note 3**: 當 TS 輸入結合 VCM/ACM 或 multiple TS 輸入結合 CCM 時，此功能為 **Normative（強制）**

---

###  標記說明

- `N` = **Normative（強制實作）**
- `O` = **Optional（可選實作）**
- `NA` = **Not Applicable（不適用）**



##  系統架構擴充（DVB-S2X System Architecture Extension）(page 12)

> 延伸自 **ETSI EN 302 307-1 , clause 4.2** 的 DVB-S2 架構，並針對 DVB-S2X 做出擴充與改進。

---

###  架構沿用與基礎
- DVB-S2X **延續 DVB-S2 的系統架構設計**
  - 基於 **ETSI EN 302 307-1 Figure 1**
- 保留整體流程（如 Mode Adaptation → FEC → Modulation → RF）
- 核心升級項目包括：

---

###  架構改進項目（DVB-S2X 擴充）

####  更細的調變/編碼階段（MODCOD）
- 增加 MODCOD 數量與解析度
- 便於對應多樣化的信道狀況與服務需求

####  更小的 roll-off 系數
- 支援更尖銳的濾波（最低至 α = 0.05，在其他段落中定義）
- 提升頻譜利用效率

####  寬頻訊號 Time-Slicing 支援
- 可實現 **降低接收端處理速率**
- 適用於寬頻衛星 transponder 使用情境

####  Transponder Bonding
- 多個轉發器（transponder）**頻寬聚合**
- 實現更高通量與靈活頻寬管理

---

###  額外訊號與控制機制

#### Super-Frame 結構（Periodic Superframe）
- 提供以下優勢：
  - 規則的同步點（特別有利於 VL-SNR 條件）
  - 對 **Pilot Symbol 與 PL Scrambling** 做定期插入
  - 傳送 **Superframe 格式識別資訊**

#### 🔹 延伸型 PLHEADER（Extended PLHEADER）
- 新增欄位以支援：
  - 更完整的 **MODCOD 識別**
  - **Mobile Frame 模式**（針對 VL-SNR）
  - **GSE-HEM 高效率基帶框架模式**

#### 🔹 支援 **GSE / GSE-Lite** 傳輸
- 相容於 T2 / C2 系統的封包格式
- 提升靈活性與小型終端支援度

---

###  Beam Hopping 支援（Annex E）

- 定義了 **可選的波形與訊號格式**，支援兩種 Beam Hopping 策略：
  1. **預先排程式（periodic, pre-scheduled）**
  2. **隨機/流量驅動式（random, traffic-driven）**
- 適用於極低信噪比環境（最低 -10 dB）

#  DVB-S2X 系統配置與應用區域對照(page 13~15)(DVB-S2X)

##  FECFRAME = 64,800 bits（Normal）

| 調變方式         | MODCODs 範例                                         | 廣播 | 互動 | DSNG | 專業服務 | VL-SNR |
|------------------|------------------------------------------------------|------|------|------|----------|--------|
| **QPSK**         | 1/4, 1/3, 2/5, ..., 11/20                             |  N   |  N   |  N   |    N     |   N    |
| **8PSK**         | 3/5, ..., 9/10, 23/36, 25/36, 13/18                  |  N   |  N   |  N   |    N     |   N    |
| **8APSK-L**      | 5/9, 26/45                                           |  N   |  N   |  N   |    N     |   N    |
| **16APSK**       | 2/3, ..., 9/10, 26/45, 28/45, ..., 7/9, 77/90        |  N   |  N   |  N   |    N     |   N    |
| **16APSK-L**     | 5/9, 8/15, 1/2, 3/5, 2/3                              |  N   |  N   |  N   |    N     |   N    |
| **32APSK**       | 3/4, ..., 9/10, 32/45, 11/15, 7/9                     |  N   |  N   |  N   |    N     |   N    |
| **32APSK-L**     | 2/3                                                  |  N   |  N   |  N   |    N     |   N    |
| **64APSK**       | 11/15, 7/9, 4/5, 5/6                                 |  NA  |  O   |  O   |    O     |   O    |
| **64APSK-L**     | 32/45                                               |  NA  |  O   |  O   |    O     |   NA   |
| **128APSK**      | 3/4, 7/9                                             |  NA  |  O   |  O   |    O     |   NA   |
| **256APSK**      | 32/45, 3/4                                           |  NA  |  O   |  O   |    O     |   NA   |
| **256APSK-L**    | 29/45, 2/3, 31/45, 11/15                             |  NA  |  O   |  O   |    O     |   NA   |

---

##  FECFRAME = 16,200 bits（Short）

| 調變方式         | MODCODs 範例                                         | 廣播 | 互動 | DSNG | 專業服務 | VL-SNR |
|------------------|------------------------------------------------------|------|------|------|----------|--------|
| **QPSK**         | 1/4, 1/3, 2/5, ..., 32/45                            |  NA  |  N   |  O   |    N     |   N    |
| **8PSK**         | 3/5, ..., 8/9, 7/15, ..., 32/45                     |  NA  |  N   |  O   |    N     |   N    |
| **16APSK**       | 2/3, ..., 8/9, 7/15, ..., 32/45                     |  NA  |  N   |  O   |    N     |   N    |

---

##  擴充配置支援狀態

| 配置項目                             | 廣播服務 | 互動服務 | DSNG | 專業服務 | VL-SNR |
|--------------------------------------|-----------|-----------|------|------------|--------|
| **32APSK**（S2-MODCODs）             | NA        | N         | O    | O          | N      |
| **VL-SNR Header**（Note 1）          | O         | N         | N    | N          | N      |
| QPSK（2/9）                          | NA        | O         | O    | NA         | N      |
| **BPSK**（1/5, 4/15, 1/3）           | NA        | O         | O    | NA         | N      |
| **BPSK-S Spreading Factor 2**        | NA        | O         | O    | NA         | N      |
| **Fixed Size Super-frame**（Note 8,11） | NA     | O         | O    | O          | O/NA   |
| **Part 2 PLHEADER**（8 bits）        | N         | N         | N    | N          | N      |
| **Extended PLHEADER**（8+8 bits）    | O         | O         | NA   | O          | N      |
| **GSE-High Efficiency Mode**         | N         | N         | N    | N          | N      |
| **Roll-off: 0.15 / 0.10 / 0.05**     | N         | N         | N    | N          | N      |
| **Channel bonding**（Note 2）        | NA        | NA        | NA   | O          | NA     |
| **VCM**（Note 4）                    | N         | N         | N    | N          | N      |
| **ACM**                              | NA        | N         | O    | O          | N      |
| **Beam Hopping Periodic BH, VLSNR**  | O         | O         | O    | O          | O      |
| **Traffic Driven BH, VLSNR**         | O         | O         | O    | O          | O      |

---

- **Note 1**：所有接收器需能識別 MODCOD，即使未實作 XFECFRAME
- **Note 2**：需配合 Input Stream Synchronizer、Null Packet Deletion、Dummy Frame Insertion
- **Note 3**：互動式接收器需支援多重調諧器與多路輸入流
- **Note 4**：VCM 為所有模式都需支援
- **Note 5**：PLHEADER 與 Extended PLHEADER 不可與 VL-SNR Header 共存
- **Note 6**：GSE 為可選；GSE-Lite 為 GSE-HEM 所需功能，需全服務支援
- **Note 7**：xxx-L 模式為 quasi-linear channel 優化版本
- **Note 8**：Annex E 中的 Superframe 格式皆為可選
- **Note 9**：非所有 Annex E 格式皆支援 VL-SNR（僅部分格式支援）
- **Note 10**：Format 5 可用於連續傳輸
- **Note 11**：Fixed Superframe 指 Annex E 的 Format 0~4

---


![image](https://github.com/user-attachments/assets/6f70fbaa-f2d6-4a72-9121-cdc957a76154)

##📦 模組詳解
1.  Mode Adaptation
支援：
Single or Multiple Input Streams
ACM/VCM 命令
**ACM / VCM 傳輸技術筆記**

## 定義與目的

| 名稱 | 全稱 | 功能說明 |
|------|------|----------|
| **VCM** | Variable Coding and Modulation | 根據不同服務需求調整編碼與調變參數 |
| **ACM** | Adaptive Coding and Modulation | 根據通道條件動態調整編碼與調變，提升頻譜效率 |

---

##  模式比較

| 模式 | 調變與碼率固定？ | 使用場景 | 優點 |
|------|------------------|----------|------|
| **CCM** | ✅ 是 | 單一廣播節目 | 簡單、相容性好 |
| **VCM** | ❌ 否 | 不同內容服務（例如 HDTV vs SDTV）| 提高整體頻譜利用率 |
| **ACM** | ❌ 否 | 單播/多播 + 回饋通道支援 | 可動態最佳化每位使用者連線品質與效率 |

---

##  系統運作流程

### ACM 工作流程：
1. UE（使用者）透過回饋通道送出 **Channel Quality Indicator (CQI)**。
2. 發送端根據 CQI 選擇對應的 **MODCOD（調變+碼率）**。
3. 更新 **PLHEADER**（傳輸訊頭），通知接收端。
4. UE 使用對應參數解碼。

### VCM 工作流程：
1. 系統根據內容需求設定不同的 **BBFRAME + MODCOD**。
2. 所有用戶接收整個波束，但只解碼符合自己服務的部分。

---

##  DVB-S2 / DVB-S2X 支援

| 標準 | MODCOD 數量 | ACM 支援 | 備註 |
|------|--------------|-----------|------|
| **DVB-S2** | 28 組 |  是 | 初始支援 VCM/ACM |
| **DVB-S2X** | 多達 64 組 |  加強 | 支援 VL-SNR、beam hopping、HEM 模式 |

---

## 🧪 MATLAB / OAI 模擬實作建議

### MATLAB：
- 手動設定不同碼率與調變，模擬 VCM 效果。
- 若實作 ACM，需設計回饋機制並根據通道條件自動切換參數。

```matlab
% 範例：根據 Eb/N0 調整 MODCOD
if EbN0 < 2
    mod = 'QPSK'; rate = 1/4;
elseif EbN0 < 5
    mod = '8PSK'; rate = 2/3;
else
    mod = '16APSK'; rate = 3/4;
end

---

功能：
Input Interface
Input Stream Synchronizer（可選）
Null-Packet Deletion（ACM/TS 模式）
CRC-8 編碼（封包錯誤偵測）
Buffer 暫存
Merger & Slicer：合併多路輸入流
🔹 註：虛線區塊為非廣播應用所需功能（e.g., Multiple TS）



