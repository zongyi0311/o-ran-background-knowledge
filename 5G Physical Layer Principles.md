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

## 3.2 PROPAGATION CHANNEL CHARACTERIZATION
| 項目                       | 說明                                      |
| ------------------------ | --------------------------------------- |
| **主要問題**              | 無線傳輸中最重要的是了解通道特性，特別是時變性與散射效應。           |
| **自由空間損耗來源**          | 主要受降雨影響，導致訊號衰減和雨衰機率上升。                  |
| **蜂巢式系統通道特性**         | 受環境散射、用戶移動性、非靜態障礙物影響，導致訊號隨時間與空間變化。      |
| **多重路徑（Multipath）影響** | 訊號來自不同方向與延遲的路徑，彼此疊加形成接收訊號。              |
| **建模方式**              | 將通道建模為多個離散的平面波，每一個波代表一條傳播路徑。            |
| **公式描述**              | 使用平面波展開來計算 $H_{nm}$，包含方向向量、相位、頻率與多普勒效應。 |
| **所需路徑數**             | 真實場景中可能需要數萬條路徑來準確建模（尤其是在移動通信場景中）。       |

![image](https://github.com/user-attachments/assets/5590304d-371b-4b85-b92d-1c675270b9e2)
| 參數或符號                             | 意義                                   |
| --------------------------------- | ------------------------------------ |
| $\mathbf{A}_l$                    | 複數極化振幅矩陣，描述第 $l$ 條平面波的振幅與極化          |
| $\mathbf{g}_m^r(-\mathbf{k}_l^r)$ | 接收天線在波向量方向 $-\mathbf{k}_l^r$ 上的天線方向圖 |
| $\mathbf{g}_n^t(\mathbf{k}_l^t)$  | 發射天線在波向量方向 $\mathbf{k}_l^t$ 上的天線方向圖  |
| $\mathbf{k}_l^r, \mathbf{k}_l^t$  | 分別為接收與發射端的波向量                        |
| $\mathbf{r}_n^t, \mathbf{r}_m^r$  | 發射與接收天線元件相對於各自參考點的位置向量               |
| $\omega$                          | 訊號的角頻率，$\omega = 4\pi f$，表示時間變化項     |
| $\tau_l$                          | 傳播延遲，表示訊號從 Tx 到 Rx 所需的時間差            |
| $\omega_{D_l}$                    | 多普勒頻移造成的角頻率偏移，用於描述移動造成的頻率偏移效應        |

![image](https://github.com/user-attachments/assets/91cc901e-9bf2-4935-a7d4-cd1cb8ab5d0e)
![image](https://github.com/user-attachments/assets/999d2b22-1ad0-499e-aa26-772aa405446f)
![image](https://github.com/user-attachments/assets/5c94abad-dda5-494b-a566-97feb25ce5af)

| 主題                   | 重點說明                                                                                       |
| -------------------- | ------------------------------------------------------------------------------------------ |
|  極化座標系定義           | $\mathbf{e}_1 = \hat{\phi},方位角方向單位向量 \mathbf{e}_2 = \hat{\theta},仰角方向單位向量 \mathbf{e}_3 = \hat{r}$，徑向方向單位向量，建立極座標基底系統 |
|  極化矩陣 $\mathbf{A}$ | 描述 Tx 到 Rx 的極化耦合，並含有方向與距離資訊                                                                |
|  增益公式 (對齊)         | 極化方向一致時，最大接收功率為 $\lambda / 4\pi d$                                                         |
|  增益公式 (正交)          | 若 Tx/Rx 極化方向正交，則完全無法接收訊號                                                                   |
|  本地座標轉換            | Rx 座標系相對於 Tx 座標系有 180 度旋轉（造成矩陣中的負號）                                                        |

| 編號  | 向量形式                                                  | 種類                  | 特性                             |
| --- | ----------------------------------------------------- | ------------------- | ------------------------------ |
|  1 | $E_1 = 1, E_2 = 0$                                    | 水平線性極化              | 只有在水平方向上有電場                    |
| 2️ | $E_1 = -\frac{1}{\sqrt{2}}, E_2 = \frac{i}{\sqrt{2}}$ | 圓極化                 | 幅度相同，相位差 $\frac{\pi}{2}$，右手圓極化 |
| 3️ | 橢圓極化                                                  | $E_1, E_2$ 含複數項與相位差 | 更一般化的極化型式                      |

![image](https://github.com/user-attachments/assets/e292f8a2-eb3a-41a7-995c-d45aa4ad17f4)

| 主題                       | 說明                                      |
| ------------------------ | --------------------------------------- |
|  **頻率域通道響應 $H(f)$**    | 描述不同頻率下的通道增益與相位                         |
|  **延遲域通道響應 $h(\tau)$** | 描述通道對單一脈衝的時間延遲響應（類似脈衝響應）                |
|  **轉換關係**              | 兩者是**傅立葉轉換對**，可以互換表示                    |
|  **應用場景**              | 延遲域常用於時域模擬（如多路徑模擬），頻率域則常見於 OFDM 等頻域傳輸技術 |

![image](https://github.com/user-attachments/assets/749efdb5-7b42-454d-8cce-3d648cab217d)

**基頻（Baseband）等效轉換**
![image](https://github.com/user-attachments/assets/a199cac2-62c3-412f-88c6-a9a1385515ac)

![image](https://github.com/user-attachments/assets/9e0c08c6-8d5b-4ab1-9add-37434f7126af)

-  上方圖（時間域）
  -  藍線：Passband 輸出（有高頻震盪）
  -  橘線：Baseband 輸出（經過下變頻處理）

-  下方圖（頻率域）
  -  顯示 baseband 與 passband 的頻譜能量分布
  -  藍色 passband：對稱分布於 ±2000 MHz，頻寬 200 MHz
  -  橘色 baseband：集中於 0 Hz 附近，頻寬仍為 200 MHz

**Hann 濾波器**
![image](https://github.com/user-attachments/assets/e593b1f0-df0b-41c2-a8cf-e81b8e7683e9)
![image](https://github.com/user-attachments/assets/4d55830b-a2df-422b-b824-17bed487a31e)
| 圖片  | 描述             | 說明           |
| --- | -------------- | ------------ |
| (A) | Uniform 濾波器的頻譜 | 平坦帶通（理想矩形）   |
| (B) | Hann 濾波器的頻譜    | 餘弦形頻譜，邊緣逐漸衰減 |
| (C) | Uniform 的時間響應  | 出現明顯副瓣（振鈴）現象 |
| (D) | Hann 的時間響應     | 主脈衝清晰，副瓣明顯減弱 |

**副瓣（Side-lobes）／振鈴（Ringing）**
-  當一個訊號經過**理想頻率濾波器（如矩形頻譜）**處理後，其時域響應會變成 sinc 函數
-  sinc 函數的主峰清楚，但左右會有連續震盪波——這就是副瓣（side-lobes）
-  因為這些副瓣不會馬上衰減為零，導致訊號在時間上擴展，這種時間上的震盪稱為振鈴現象（ringing）
**為什麼副瓣會造成問題？**
| 問題類型          | 說明                          |
| ------------- | --------------------------- |
|  **時間擴展**   | 導致訊號能量擴散，與其他符號重疊（符號間干擾 ISI） |
|  **能量混疊**   | 多徑環境中不同路徑難以辨識（副瓣互相干擾）       |
|  **通道估計不準** | 會誤判主徑與反射徑的延遲與功率，影響估測精度      |

**Exponential Decaying Channel**
![image](https://github.com/user-attachments/assets/a312eb30-5150-41d2-b4b6-bbcea2d81f3a)

![image](https://github.com/user-attachments/assets/c5fe8ab1-190e-49b3-8764-346cb79e7212)
![image](https://github.com/user-attachments/assets/81f39bd7-7f4a-4ace-8180-4d6fb8fa719e)


![image](https://github.com/user-attachments/assets/3af4cada-c3c3-4375-b992-e56c6335c736)

**DOPPLER-TIME DOMAIN**
![image](https://github.com/user-attachments/assets/6fdf560e-616d-4bc3-bc8c-c2ed4fc26c54)
| 子圖  | 條件                                              | 左：時域延遲響應 | 右：頻域響應         |
| --- | ----------------------------------------------- | -------- | -------------- |
| (A) | $\sigma_\tau = 100\text{ns}, B = 100\text{MHz}$ | 長延遲、可分多徑 | 頻域劇烈起伏，頻率選擇性強  |
| (B) | $\sigma_\tau = 100\text{ns}, B = 10\text{MHz}$  | 長延遲、解析度差 | 頻域變化緩和（解析不出多徑） |
| (C) | $\sigma_\tau = 20\text{ns}, B = 100\text{MHz}$  | 短延遲、多徑較少 | 有些許頻率選擇性       |
| (D) | $\sigma_\tau = 20\text{ns}, B = 10\text{MHz}$   | 短延遲、頻寬小  | 頻域幾乎平坦，頻率非選擇性  |

-  延遲擴散大、頻寬高 ⇒ 頻率選擇性強（圖 A）
-  延遲擴散小、頻寬低 ⇒ 頻率選擇性弱（圖 D）
-  頻率選擇性造成頻域響應有明顯「起伏」或「深谷（notches）」

**時頻轉換與多普勒建模基礎**
![image](https://github.com/user-attachments/assets/81d8a0d7-d1d8-424c-bebd-36dc7b06f425)

在**時域中建模通道**時，通常將多條具有不同 **多普勒頻率 $f_D$** 的波疊加起來。產生這些不同多普勒頻率的主要原因包括：

-  **發射端或接收端移動**：
  - 向波源靠近：產生 **正多普勒偏移（頻率上升）**
  - 遠離波源：產生 **負多普勒偏移（頻率下降）**

-  **環境中的散射體也可能在移動**：
  - 例如：車輛、樹木、行人等物體，導致反射波具有時間變化的特性

**經典多普勒分佈（Classical Doppler Distribution）**

公式 (3.19) – 多普勒通道模型

一個具有經典多普勒分佈的通道可表示為：

$$
h(t) = \sum_{l=1}^{N} a_l \cdot \exp(i 2\pi f_{D_l} t)
$$

其中每個分量的多普勒頻率為：

$$
f_{D_l} = \frac{v}{\lambda} \cdot \cos\left( \frac{2\pi l}{N} \right)
$$

振幅為：

$$
a_l = \exp(i \phi_l)
$$

- \( v \)：移動終端的速度  
- \( \lambda \)：波長  
- \( \phi_l \)：第 \( l \) 條路徑的隨機相位  

---

類比延遲擴散的頻率分析：多普勒擴散

**多普勒平均值與 RMS 多普勒擴散** 定義如下：

```math
\mu_{f_D} = \frac{\sum_{l=1}^{N} f_{D_l} |a_l|^2}{\sum_{l=1}^{N} |a_l|^2}
```
![image](https://github.com/user-attachments/assets/b63eafc5-77b7-4b5e-8ceb-05bc490f57ec)

**DIRECTIONAL DOMAIN**
方向域與多普勒域有直接關係，其轉換公式為：

$$
f_{D_l} = \frac{\mathbf{v} \cdot \mathbf{k}_l}{2\pi} = \frac{v}{\lambda} \cos(\theta_l)
$$

- $\theta_l$ 為移動方向 $\mathbf{v}$ 與波向量 $\mathbf{k}_l$ 之間的夾角
- $\lambda$ 為波長

---

**根據多普勒頻率推得波向量 $\mathbf{k}$ 和方向向量 $\mathbf{u}$**：

$$
k_{x,y,z} = 2\pi \cdot \frac{f_{D_{x,y,z}}}{v_{x,y,z}}
$$

$$
u_{x,y,z} = \lambda \cdot \frac{f_{D_{x,y,z}}}{v_{x,y,z}}
$$

- $\mathbf{u}$ 為方向單位向量（Direction Unit Vector）
- 透過三維空間的多普勒頻率，可以反推通道的方向分佈

---
實務上，可利用三維空間的通道樣本（spatial samples）進行傅立葉轉換，取得**方向頻譜（Directional Spectrum）**，對應內容可參考 Section 3.3.2.1。

---

**角度擴散（Angular Spread）**

方向擴散可透過**方位角（Azimuth）**與**仰角（Elevation）**來量化。但角度變數存在以下特性與挑戰：

| 問題 | 說明 |
|------|------|
|  週期性（cyclic） | 角度會在 \( [0^\circ, 360^\circ] \) 或 \( [-\pi, \pi] \) 中循環，處理上需小心邊界截斷位置 |
|  non-Euclidian | 角度不是線性變數，無法直接使用一般統計運算（例如均值或方差） |
|  not invariant | 若通道以角度分佈定義，則在座標系旋轉下通道分佈不具不變性（非旋轉對稱） |

---
- 多普勒頻率可反推出波向量及方向資訊
- 方位角與仰角可用來描述空間通道擴散程度
- 方向域建模與處理需考慮角度的週期性與幾何性質

![image](https://github.com/user-attachments/assets/f2eb3187-0cc1-4a91-8c96-1ff234080572)

---

 3.8：多普勒域與時域的通道響應

| 編號 | 說明 | 多普勒擴散 $\sigma_{f_D}$ | 觀察時間 $T$ | 特殊情況 |
|------|------|-----------------------------|---------------|-----------|
| (A)  | 經典多普勒模型                     | $70\,\mathrm{Hz}$ | $250\,\mathrm{ms}$ | 無 |
| (B)  | 經典多普勒模型                     | $70\,\mathrm{Hz}$ | $25\,\mathrm{ms}$  | 無 |
| (C)  | 經典多普勒 + 靜止通道（額外路徑）   | $20\,\mathrm{Hz}$ | $250\,\mathrm{ms}$ | 含 0 Hz 分量 |
| (D)  | 經典多普勒 + 靜止通道（額外路徑）   | $20\,\mathrm{Hz}$ | $25\,\mathrm{ms}$  | 含 0 Hz 分量 |

- 加入靜止通道分量（如圖 C、D）會造成時域中的通道功率變動趨近常數，對應**無限大的相干時間**。

---

(3.23)：三維方向擴散（3D Directional Spread）

為克服傳統角度週期與非歐幾里得問題，改用基於三維多普勒資訊的**方向擴散指標** $\sigma_{\text{dir}}$：

$$
\mu_{u_n} = \frac{\sum_{l=1}^N u_{n,l} |a_l|^2}{\sum_{l=1}^N |a_l|^2}
$$

$$
\sigma_{\text{dir},n} = \frac{180}{\pi} \sqrt{ \frac{ \sum_{l=1}^N (\mu_{u_n} - u_{n,l})^2 |a_l|^2 }{ \sum_{l=1}^N |a_l|^2 } }
$$

$$
\sigma_{\text{dir}} = \sqrt{ \sigma_{\text{dir},x}^2 + \sigma_{\text{dir},y}^2 + \sigma_{\text{dir},z}^2 }
$$

- $n \in \{x, y, z\}$：對應三個空間分量方向
- $\sigma_{\text{dir}}$ 是總方向擴散度，可描述**通道角度分佈在空間中的發散程度**

---

- 此定義具備**座標旋轉不變性**，可有效描述多路徑能量在空間中的分佈情形。
- 與多普勒頻率模型類似，但更適用於**方向性天線**與**三維 MIMO 通道建模**。
