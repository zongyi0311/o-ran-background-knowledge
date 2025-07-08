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

**多普勒域與時域的通道響應**

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

## 3.3 EXPERIMENTAL CHANNEL CHARACTERISTICS

為了完整地描述通道特性，需要結合以下幾個面向：
1. **頻率–延遲域（Frequency-Delay Domain）**
2. **多普勒–時間域（Doppler-Time Domain）**
3. **空間/方向域（Spatial/Directional Domain）**

**連續波（CW）法** 是一種常用於量測「傳輸損耗（Transmission Loss）」的技術。
- 利用固定頻率的**正弦波訊號**進行發射
- 接收端透過**窄頻濾波器**過濾訊號
- 搭配**高功率發射**與**低雜訊放大器（LNA）**，可達到高靈敏度接收
---
**優點**
- 硬體架構簡單、成本低
- 適合進行**大範圍、高速、連續掃描**
- 常用於量測不同地區的**路徑損耗地圖（path loss map）**
---
**缺點**
- **無法解析多重路徑（Multipath）效應**
- 多路徑未被分離，導致**空間衰落（Spatial Fading）**
---
![image](https://github.com/user-attachments/assets/337c2ba1-7408-423e-bd0d-3dc7a0fbea96)

- 實驗條件：街道微型基地台、NLoS（非視距）環境
- 頻率：5.1 GHz CW 訊號
- 為減少快衰落效應，採用 **1.7 公尺滑動平均**

| 線條說明 | 說明 |
| -------- | ---- |
| 🔵 `No averaging` | 原始訊號，出現劇烈衰落波動 |
| 🔴 `Averaging`     | 加上滑動平均後平滑化結果，利於觀察總體衰減趨勢 |

**Vector Network Analyzer**

與 CW 量測方法不同，向量網路分析儀（Vector Network Analyzer, VNA）允許最大的測量頻寬。
原理與方法
- 基本原理是進行頻率掃掠，對通道在預定頻寬內進行取樣。
- 若要得到延遲域的通道響應，可透過離散傅立葉轉換（DFT）。
---
**優點**
- VNA 透過 RF 線纜同時連接發射與接收天線，測量兩者之間的**完全相干比值**。
- 接收信號與發射信號同步，可實現：
  - 絕對延遲量測
  - 長時間平均以抑制雜訊
- 適合高精度多徑通道分析。
---
**缺點**
- **行動性受限**：由於 RF 線纜連接，裝置難以移動。
- **測試時間長**：掃頻時間受限於接收端 SNR，可能需數秒。
- **量測距離有限**：RF 線纜訊號會衰減，每公尺可能衰減數 dB。
---
![image](https://github.com/user-attachments/assets/0efb4cbb-0052-464a-9aa7-2980fc55dd09)

- 圖 3.10 展示 58.7 GHz、2 GHz 頻寬下的量測結果，應用於 NLoS 街道場景，使用光纖延伸超過 100 公尺。
- 高頻寬（2 GHz）使得通道中的大量多徑組件得以清楚分離
- 頻域響應顯示在整個頻帶內的起伏，反映出豐富的頻率選擇性行為


**Correlation-Based Channel Sounding（相關性通道探測）**

**優點**
- **可移動**與**寬頻**的通道量測技術。
- 傳送**專用的探測訊號（sounding signal）**，在接收端依據延遲與其做相關（correlation）。
- 一般使用 **OFDM 或偽隨機序列（PN sequence）** 配合 **滑動相關器（sliding correlator）**。
- 適用於 **行動裝置量測** 及 **頻寬較大的通道特性分析**。

**取捨與限制**
- **通道採樣頻率與抗雜訊能力之間的權衡**。
- 訊號同時具有**相位與振幅調變**，因此 **受限於放大器非線性（amplifier non-linearities）**。

---
**Directional Characteristics（方向性特性）**

- 通道的方向性特性在進入**毫米波頻段**（mmWave）後顯得特別重要。
- 使用**全向天線**在高頻下會導致訊號損失增加，**天線口徑變小、覆蓋變差**。
- 為此，必須採用 **波束成形技術（beam-forming）** 來聚焦訊號收發，**減少損耗並提升性能**。

**Two Methods for Directional Characteristics**

方法一：實體指向性天線（Physical Directive Antennas）

- 使用實體高增益天線，如：
  - **角喇叭天線（horn antenna）**
  - **拋物面反射天線（parabolic reflector antenna）**
- 通過在仰角（elevation）與方位角（azimuth）方向旋轉，來**掃描空間角度**。
- 特點：
  - 不易受時變通道影響。
  - 適用於 **CW** 和 **Correlation-based** 通道探測。

方法二：虛擬天線法（Virtual Antenna Method）

- 使用**單一實體天線**搭配**機械平台（如機器手臂）**，在多個空間點進行量測。
- 實現空間取樣（spatial sampling），再離線使用 **陣列天線技術（array processing）** 計算方向性資訊。

**優點**：
- 可實現 **高解析度**。
- 有效抑制 **副瓣（sidelobes）**。

**缺點**：
- **量測時間長**（大型陣列可能需數小時以上）。
- 需**同步鎖相的發射與接收裝置**。
- 僅適用於 **靜態通道環境**。
- 多用於 **VNA（Vector Network Analyzer）** 測量。

**Spectral Analysis**

此方法基於對量測信號的頻譜進行直接分析，透過傅立葉轉換（Fourier Transform）獲得頻率–延遲與都卜勒–時間分佈。
- **應用技術**：需對空間中三個方向（x, y, z）取樣後進行三維 DFT 分析，得到波向量 **k** 的功率譜。
- **k 值計算**：

$$
|k| = \sqrt{k_x^2 + k_y^2 + k_z^2}
$$

**實例說明**：
  - 使用立方體虛擬天線陣列，共 $25^3 = 625$ 個採樣點（見 Figure 3.11）
  - 使用 Hann 窗抑制側瓣干擾（side-lobe suppression）
  - 結果在 k-domain 空間中，設定一固定半徑進行濾波，形成 **directional spectrum**

  **優勢**：
  - 側瓣抑制超過 50 dB
  - 遠優於實體天線，後者僅可抑制至約 30 dB
  - 最終可將光譜量測結果視為多徑組合，對應於每個多徑的相位與幅度

![image](https://github.com/user-attachments/assets/59b8462c-eebb-464a-ae15-4e48394a9ce0)
**上圖：Measured**  
  實際量測的室內直視通道（LoS）在 60 GHz 下的空間功率分佈（azimuth × elevation）。

**下圖：Synthesized using 400 MPCs**  
  使用 400 條多徑成分（MPCs）合成出的通道響應，與實測結果相當吻合。

**說明**：  
  - 顯示以虛擬立方體天線進行三維空間掃描所得到的結果。
  - 證明透過 **spectral analysis + multipath component synthesis** 能夠高度準確地重建實際通道特性。

![image](https://github.com/user-attachments/assets/73d1f942-edcd-4d9c-812a-90130362548b)
-  下方圖（延遲功率輪廓圖）
顯示了在 5 GHz 下的非視距 (NLoS) 都市巨微蜂窩場景中的功率延遲輪廓（Power Delay Profile, PDP）。
- 藍色線條：實際量測的 PDP（Measured）
- 深色線條：根據超解析（super-resolution）估計的模型 PDP（Modeled）
- X 軸：Delay（μs）
- Y 軸：Relative Power（dB）
---

-  上方圖（平面波方向估計）

此圖展示從基地台（BS）觀察方向上的平面波估計結果：
- 每個白色圓圈代表一個估計的平面波方向，其大小表示相對功率大小（越大圓越強）。
- 圓的位置對應到波的方位角與仰角。
- 三角形標記表示行動台（MS）的方向。

**Measurement Comparability **

為了使全球不同的傳播量測資料具可比較性，研究社群需遵守一套統一的準則，以確保在不同實驗條件下獲得的結果可以公平比較。
-  Requirements for comparability
- **Equal measurement bandwidth**  
  ➤ 提供相同的延遲解析度。
- **Comparable antenna pattern**（either physical or synthesized）  
  ➤ 確保天線方向圖一致，無論是實體天線還是虛擬天線。
- **Equal dynamic power range**（in domain of analysis, e.g., delay or angle）  
  ➤ 確保功率測量的一致性。
- **Same environment and antenna locations**  
  ➤ 不同頻段或量測方法需在相同環境下執行，以便比較。

---
**為何需要等化頻寬？**
- 在高頻（如毫米波）中，**可使用的量測頻寬通常較大**。
- 頻寬大 → 延遲解析度高 → **偵測到更多多徑**。
- 若不同量測使用不同頻寬，會造成誤導性的延遲擴散變化 → **延遲擴散看似減少**，但實際只是解析度提升。
- 所以：**為了公平比較，應使用相同頻寬** 或將頻寬等化處理。

---
![image](https://github.com/user-attachments/assets/484e6253-f8d4-41aa-a858-0a94dd988a1d)
在 NLoS 微蜂窩場景中，分析了兩種頻寬的 power delay profile：
- **2 GHz 頻寬（左圖）**  
  - 解析度高  
  - delay spread ≈ **7 ns**
- **80 MHz 頻寬（右圖）**  
  - 解析度低  
  - delay spread ≈ **28 ns**
重點：當使用固定的 20 dB 動態功率範圍下進行 RMS delay spread 計算時，**頻寬不同會導致明顯的結果差異**。
---
- 頻寬越寬，能解析的多徑越細緻。
- 使用固定動態功率範圍做統計時，頻寬的選擇對結果影響重大。
- 在比較不同頻段測量結果時，**務必注意頻寬、天線模式、功率範圍與量測位置的一致性**。

**TRANSMISSION LOSS MEASUREMENTS**
**測量目的**
傳輸損耗（Transmission Loss）反映了由於電波傳播造成的接收訊號強度衰減，是無線通道最基本也最關鍵的特性之一

**實驗設計與方法**
- **頻率範圍**：1–100 GHz。
- **天線類型**：
  - 垂直極化 omni dipole 天線（常見，用於所有頻段）
  - 垂直 patch 天線或 open waveguide（用於戶外傳送時的戶外到室內量測）
- **短距離 LoS 標準校正**：使用 **0.1–1.0 m 的直視（LoS）距離**，確保資料準確、排除天線頻率響應影響。

**氧氣吸收補償（60 GHz）**
- 在 60 GHz 附近，會受到 **氧氣吸收（約 1.5 dB / 100 m）** 影響。
- 為了確保模型可跨頻平滑外推/內插，**在建模時須加入氧氣吸收補償項**。

![image](https://github.com/user-attachments/assets/68e8782b-c517-48f8-86e6-d80760f81906)

**Delay Domain Measurements**

- 延遲領域有助於建構通道的 **頻率選擇性（frequency selectivity）** 特性  
  （參考 Section 3.2.1）
- 對於 **傳輸波形最佳化**（例如 OFDM）至關重要，尤其是延遲擴展（delay spread）對循環前綴（Cyclic Prefix, CP）長度的影響
- 3GPP 選擇 OFDM 作為 NR 調變方式，因此需根據延遲特性設定 CP 長度


**延遲領域中的頻率趨勢（General Frequency Trend in Delay Domain）**

實驗觀察總結
- 除了「戶外到室內」場景外，**實驗資料未顯示明確的頻率趨勢**
- 根據 3GPP 早期研究：延遲擴展（delay spread）**隨頻率上升而下降**

**3GPP 模型的問題點**
- 3GPP 在制定模型時，**頻段間可比性要求未完全符合**
- 導致這些結果在學術上可能不完全可靠

**mmMAGIC 實驗專案（EU-funded）**
- 採用 15 個量測計畫、6 個組織合作，符合頻段可比性要求
- 模型包含五種場景的參數，透過統計整合而來

**模型比較結果**
- 圖 3.25 與模型 (3.29) 顯示：
  - **3GPP 模型**：延遲擴展明顯隨頻率下降
  - **mmMAGIC 數據**：**未顯示相同趨勢**
  - 僅「都市峽谷視距（street canyon LoS）」與「室內視距（indoor office LoS）」略呈現下降趨勢，且落於 95% 信賴區間

結論：3GPP 模型在高頻段的 delay spread 下降趨勢 **不完全被實驗支持**

![image](https://github.com/user-attachments/assets/367aa4e8-a023-4f67-97d9-60a8bf1269f1)

**延遲領域中的頻率趨勢（General Frequency Trend in Delay Domain）**

-  實驗與模型比較分析

- **mmMAGIC 實測資料**：
  - 大多情境下延遲擴展（Delay Spread）**並未隨頻率明顯下降**
  - 只有：
    - Street Canyon LoS
    - Indoor Office LoS  
    顯示出輕微下降趨勢（在 95% 信賴區間內）

- **3GPP TR 38.901 模型**：
  - 預測延遲擴展會**明顯隨頻率上升而下降**
  - 與 mmMAGIC 測量資料之間存在 **明顯差異（discrepancy）**

---
![image](https://github.com/user-attachments/assets/2e132b30-c642-4bc4-81fd-5fc7bc17ca91)

**Figure 3.25：模型與實測結果比較圖**

🔹 上圖：延遲頻率係數 α 的比較

- `α` 為 delay spread 對頻率的擬合斜率
- **3GPP α 明顯為負值**
- **mmMAGIC α 接近 0 或微負，區間更廣**
🔹 中圖：Indoor Office 場景

- **3GPP 模型**顯示 LOS 與 NLOS 延遲擴展隨頻率下降
- **mmMAGIC**：
  - NLOS 幾乎**不變**
  - LOS 輕微下降但整體較平坦

🔹 下圖：Street Canyon 場景

- **3GPP 模型**持續呈現下降趨勢
- **mmMAGIC 資料**則大致維持水平或僅微降

---
| 特性              | 3GPP 模型                  | mmMAGIC 實測資料          |
|-------------------|-----------------------------|-----------------------------|
| 頻率影響趨勢      | 延遲擴展明顯下降             | 多數情境中趨勢不明顯        |
| 實測驗證一致性    | 模型斜率與實測不符           | 僅少數場景稍具一致性        |
| 建模依據          | 未完整符合頻段比較要求       | 精心設計，滿足可比性需求     |

結論：3GPP 模型雖簡化設計，但可能**高估高頻延遲擴展下降趨勢**。mmMAGIC 數據提供更符合實際情況的依據。

**方向領域量測（Directional Domain Measurements）**
- 在高頻（尤其是毫米波）通訊中，必須透過精確控制**發射與接收天線方向**來克服高傳輸損耗  
- 使用全向天線時，天線孔徑與波長平方成正比，導致在高頻時有效接收面積變小 → **只能支援短距離連結**
---
- 探討無線通道的 **方向性特性（Directional Properties）**
- 特別關注於**高頻段**（如 24–100 GHz）中傳播特性對指向性的影響

---
目的在於支援毫米波與高頻移動通訊的波束對準與通道建模
