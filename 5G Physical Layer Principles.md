# Chapter 2
## 2.1
![image](https://github.com/user-attachments/assets/c80f5e6b-7886-431a-adf2-82d7e0de1d80)
| 協定層級     | 全名                               | 主要功能                                              |
| -------- | -------------------------------- | ------------------------------------------------- |
| **SDAP** | Service Data Adaptation Protocol | - QoS flow ↔ Radio bearer 映射<br>- 根據 QoS 分類 IP 封包 |
| **PDCP** | Packet Data Convergence Protocol | - IP header 壓縮／解壓<br>- 加密與完整性保護<br>- 重排序、重複資料檢測   |
| **RLC**  | Radio Link Control               | - ARQ 錯誤修正<br>- 資料分段與重組<br>- 按序資料傳遞               |
| **MAC**  | Medium Access Control            | - HARQ 錯誤修正<br>- 排程與資源分配<br>- 載波聚合下的多工            |
| **PHY**  | Physical Layer                   | - 編碼／調變處理<br>- 多天線處理（MIMO）<br>- 對應時間頻率資源          |

-  User Plane 主要針對 資料傳輸與效能最佳化
-  Control Plane 涉及 連線管理、安全性與控制命令處理

## 2.2
### MODULATION
| 類別             | 調變方式                                 | 備註                    |
| -------------- | ------------------------------------ | --------------------- |
|  下行（Downlink） | QPSK, 16QAM, 64QAM, 256QAM           | 與 LTE 相同              |
|  上行（Uplink）   | QPSK, 16QAM, 64QAM, 256QAM, π/2-BPSK | π/2-BPSK 提升效率，適合 mMTC |

### WAVEFORM
| 技術      | 下行波形    | 上行波形                        | 備註               |
| ------- | ------- | --------------------------- | ---------------- |
| **LTE** | CP-OFDM | DFTS-OFDM                   | 上下行使用不同波形        |
| **NR**  | CP-OFDM | CP-OFDM（主）<br>DFTS-OFDM（可選） | 上下行統一主波形，可根據場景選擇 |
-  在上、下行使用相同波形有助於簡化整體系統設計
-  DFTS-OFDM 適用情境:覆蓋範圍受限、單流上行傳輸、無空間多工
### MULTIPLE ANTENNAS
-  more fundamental role in the system design
-  For low frequencies
  
| 技術               | 說明                  |
| ---------------- | ------------------- |
| LTE 發展延伸         | 建立在既有多天線功能上         |
| 頻譜效率需求高          | 面對頻譜擁擠，需提升容量與速率     |
| **massive MIMO** | 控制大量天線，提升空間解析度與效率   |
| **MU-MIMO 支援增強** | 同時服務多使用者，提升空間重複使用效率 |
| **CSI 架構更新**     | 靈活參考訊號傳輸、更高解析度、易擴展  |
-  For high frequencies

| 項目                 | 說明                            |
| ------------------ | ----------------------------- |
| **挑戰重點**           | 涵蓋不足（非效率）                     |
| **波束成形是關鍵**        | 特別適用於 LoS 場景                  |
| **類比 beamforming** | 適合現有硬體限制                      |
| **支援初始接入與廣播**      | NR 首次支援 beamforming 應用於非資料面傳輸 |

### CHANNEL CODING
-  使用的通道編碼類型

| 應用類型        | 使用的通道編碼            | 備註                  |
| ----------- | ------------------ | ------------------- |
| 數據傳輸（MBB）   | LDPC（低密度同位檢查碼）   | 支援高資料率與 HARQ        |
| 控制訊號（如 RRC） | Polar Code（極化碼）  | 適合短封包，支援 SC-List 解碼 |
| 最小控制負載      |  Reed–Muller Code | 僅用於極短封包控制場景         |

## 2.3 PHYSICAL TIME-FREQUENCY RESOURCES

| 單位                                | 定義                       | 備註                  |
| --------------------------------- | ------------------------ | ------------------- |
| **Resource Element (RE)**         | 1 子載波 × 1 OFDM 符號        | 最小資源單位              |
| **PRB (Physical Resource Block)** | 12 子載波 × N OFDM symbols  | NR 中最基本的調度單位        |
| **Slot**                          | 14 個 OFDM symbols        | 可根據 numerology 調整長度 |
| **Mini-slot**                     | 2 / 4 / 7 個 OFDM symbols | 適用於突發或低延遲場景         |
| **Subframe**                      | 1 ms                     | 包含一個或多個 slots       |
| **Frame**                         | 10 ms                    | 包含 10 個 subframes   |

## 2.4 PHYSICAL CHANNELS
-  **下行**

| 通道                                           | 功能說明                                                   |
| -------------------------------------------- | ------------------------------------------------------ |
| **PDSCH**（Physical Downlink Shared Channel）  | 用於下行資料傳輸                                               |
| **PDCCH**（Physical Downlink Control Channel） | 傳送下行控制資訊，例如：<br>‣ PDSCH 的排程決策<br>‣ 授權 UE 傳送上行資料（PUSCH） |
| **PBCH**（Physical Broadcast Channel）         | 廣播系統資訊，協助 UE 連線至網路                                     |

-  1.UE 監控 PDCCH：通常每 slot 至少一次（也可設定為多次以支援超低延遲）。
-  2.若偵測到有效 PDCCH，UE 接收對應 PDSCH 上的資料（稱為 transport block）。
-  3.UE 回傳 HARQ 回應：表示資料是否正確解碼，若錯誤則 gNB 安排重傳。


-  **上行**

| 通道                                         | 功能說明                           |
| ------------------------------------------ | ------------------------------ |
| **PUSCH**（Physical Uplink Shared Channel）  | UE 上傳數據的通道                     |
| **PUCCH**（Physical Uplink Control Channel） | UE 傳送控制訊息（如 ACK/NACK、排程請求、CSI） |
| **PRACH**（Physical Random Access Channel）  | UE 發送隨機接入請求，用於建立連線或初始接入        |

-  1.UE 傳送排程請求（SR） 至 gNB，透過 PUCCH 發送。
-  2.gNB 回應 PDCCH 排程授權（scheduling grant），授權 UE 使用某些 PUSCH 資源。
-  3.UE 透過 PUSCH 傳送資料，並由 gNB 接收與解碼。
-  4.gNB 傳送 HARQ ACK/NACK：指示資料是否解碼成功。

## 2.5 PHYSICAL SIGNALS

| 信號      | 全名                                       | 傳輸方向 | 功能用途                |
| ------- | ---------------------------------------- | ---- | ------------------- |
| DM-RS   | Demodulation Reference Signal            | 上／下行 | 通道估計                |
| PT-RS   | Phase Tracking Reference Signal          | 上／下行 | 相位雜訊補償              |
| CSI-RS  | Channel State Information RS             | 下行   | CSI 獲取、同步、beam mgmt |
| PSS/SSS | Primary/Secondary Synchronization Signal | 下行   | 初始接入與同步             |
| SRS     | Sounding Reference Signal                | 上行   | 測量與 beamforming 支援  |

```
🔸 DM-RS（解調參考信號）
-  用於估計解調所需的通道資訊（channel estimation）
-  為 UE 專屬（UE-specific）
-  可搭配 beamforming 使用，僅在必要時傳送（如低延遲應用）
-  支援 slot 前置（front-loaded）配置
-  在低速場景：使用低密度
   在高速場景：密度提高（增加 OFDM 符號內的 RE 數量）

🔸 PT-RS（相位追蹤參考信號）
-  補償載波震盪器引起的 相位雜訊（phase noise）
-  特別適用於高頻傳輸（如毫米波）
-  疏頻配置於頻域、高密度配置於時域
-  PT-RS 的分布依據：震盪器品質、頻率、子載波間距、調變方式等

🔸 CSI-RS（通道狀態資訊參考信號）
-  用途：
   CSI 獲取（計算 CQI、PMI、RI 等）
   beam management / RSRP 量測 
   時頻追蹤 / 頻偏補償

🔸 SRS（Sounding Reference Signal）
-  上行傳輸，用於：
   CSI 測量、上行 link adaptation
   UE 回報用於 precoding 的通道資訊（Reciprocity-based beamforming）
```

### 2.6 DUPLEXING SCHEME

| 類型      | 全名                        | 備註           |
| ------- | ------------------------- | ------------ |
| **TDD** | Time Division Duplex      | 高頻段常用，支援動態配置 |
| **FDD** | Frequency Division Duplex | 低頻段常用，頻譜成對配置 |

| 項目     | LTE       | NR                     |
| ------ | --------- | ---------------------- |
| 雙工類型   | TDD / FDD | TDD / FDD（相同）          |
| 動態 TDD | 不支援       |  支援                   |
| 週期配置   | 固定        |  支援 semi-static 與動態週期 |
| 排程決策   | eNB       |  gNB，支援鄰小區協調          |

# chapter 3
## 3.1 PROPAGATION FUNDAMENTALS
-  建模過程中，**天線效應與傳播效應不應混為一談**

| 散射/吸收類型     | 描述                                     | 常見場景                        |
|------------------|----------------------------------------|-------------------------------|
| 鏡面反射/折射     | 平坦大表面，符合光學反射定律             | 建築物玻璃、大型牆面           |
| 繞射              | 小尺寸物體或陰影區域                    | 建築邊角、牆後區域             |
| 漫射              | 粗糙或非均勻表面                         | 石牆、植被、地面               |
| 吸收              | 材質或介質吸收波能量                    | 森林傳輸、室內外穿牆、大氣遠距離 |
