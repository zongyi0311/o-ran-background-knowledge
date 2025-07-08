$$
A(\theta, \phi) = A'(\theta', \phi') \tag{7.1-1}
$$

# 7.1 Coordinate system
## Local and Global Coordinate Systems（局部與全球座標系統）

### GCS（Global Coordinate System）
- 為整體系統（含多個 BS 與 UT）所定義的全域參考座標系。
- 所有 BS/UT 的位置、角度、通道參數皆定義於 GCS。

---
### LCS（Local Coordinate System）
- 為單一 **天線陣列**（BS 或 UT）定義的**局部座標系**。
- 天線的**遠場輻射模式（Pattern）** 與 **極化（Polarization** 在 LCS 中定義。
- 預設天線場型資訊是**已知**且固定於 LCS。

---
### LCS 與 GCS 的對應與轉換
- 天線陣列的位置由 GCS → LCS 的平移（Translation）描述。
- 陣列方向的變化由 GCS → LCS 的一系列**旋轉（Rotation）**決定。

- 因 GCS 與 LCS 方向通常不同，需進行**向量場的座標轉換**。
- 此轉換只依賴天線陣列的方向與旋轉設定。

## Transformation from a LCS to a GCS

### 座標定義
- **GCS（Global Coordinate System）**
  - 座標向量：(x, y, z, θ ,φ )
  - 單位向量：( $\hat{θ}$ , $\hat{φ}$ )

- **LCS（Local Coordinate System）**
  - 座標向量：\( (x', y', z', θ', φ') \)
  - 單位向量：\( $\hat{θ}$', $\hat{φ}$' \)

### 3D 旋轉描述
- 將 LCS 轉換為 GCS 的旋轉關係由三個角度控制：

  - **α（alpha）**：方位角（Bearing angle）
  - **β（beta）**：下傾角（Downtilt angle）
  - **γ（gamma）**：斜傾角（Slant angle）
  描述了陣列天線相對於 GCS 的完整方向設定

- 天線的向量場（如場型增益方向）**定義在 LCS**
- 為了與通道建模對接，需將其**轉換為 GCS 中的方向向量**
- LCS → GCS 的變換**只依賴旋轉角度 α, β, γ**，與位置無關
- 任意天線的方向設定都可由此三角度組合完成

![image](https://github.com/user-attachments/assets/bb13a3de-ce81-44db-a1d5-4f878607b324)

### 天線場型在 GCS 與 LCS 的轉換
若在 **LCS** 中定義的天線場型為：A'(θ', φ')，則轉換至 **GCS** 中的天線場型為：A(θ ,φ)


# 7.5 Fast fading model 

根據 3GPP TR 38.901 V17.0.0，無線通道實現是透過表格 7.5-1 所列的參數產生，並依據圖 7.5-1 所示的步驟流程建立。

- 幾何描述涵蓋了從**最後一次散射點**的入射角（arrival angles）以及**第一次散射點**的出射角（departure angles）。
- 傳播路徑中**首次與最後一次的交互之間的傳播**並未定義，因此這種模型能夠模擬**多重散射互動**。
- 這也代表，例如：**多徑延遲**無法透過幾何方式單獨決定。
- 假設為下行（downlink）模擬，若為上行（uplink）需將到達與離開角度的參數對調。

---
![image](https://github.com/user-attachments/assets/9e86cfd9-c5dc-4169-98c9-e49b3fc4750a)

**圖 7.5-1：通道係數生成流程（Channel Coefficient Generation Procedure）**

---
**一、General Parameters（大尺度參數）**

1. **Set scenario, network layout and antenna parameters**  
   設定模擬場景、網路拓撲與天線參數。

2. **Assign propagation condition (NLOS / LOS)**  
   指定傳播條件：視距（LOS）或非視距（NLOS）。

3. **Calculate pathloss**  
   計算路徑損耗（Pathloss）。

4. **Generate correlated large scale parameters**  
   生成大尺度參數，並建立其相關性：  
   - DS（Delay Spread）  
   - AS（Angular Spread）  
   - SF（Shadow Fading）  
   - K（Ricean K-factor）

---

**二、Small Scale Parameters（小尺度參數）**

5. **Generate delays**  
   產生群組（Cluster）的延遲（Delays）。

6. **Generate cluster powers**  
   為每個 Cluster 分配功率。

7. **Generate arrival & departure angles**  
   計算到達角（AOA）與離開角（AOD）。

8. **Perform random coupling of rays**  
   執行多徑射線（Rays）的隨機耦合。

9. **Generate XPRs**  
   為每個路徑分配交叉極化比（Cross-Polarization Ratio, XPR）。

---

**三、Coefficient Generation（通道係數產生）**

10. **Draw random initial phases**  
    為每個多徑分量生成初始隨機相位。

11. **Generate channel coefficient**  
    基於前述參數計算每個時間點的通道係數。

12. **Apply pathloss and shadowing**  
    將路徑損耗與遮蔽效應應用於通道係數。

---
**本流程為產生時間不變（quasi-static）或瞬時通道係數的標準程序。可應用於低複雜度、驗證性模擬，或作為高階通道模型的基礎。**

![image](https://github.com/user-attachments/assets/df43ad23-4610-4c95-bb0a-6168563c9878)

# 圖 7.5-2：全球座標系統與球面角度定義（Global Coordinate System）

此圖定義了 3GPP 快速衰落模型中使用的全球三維座標系統，並說明了球面座標中的兩個主要角度：

---
**座標系定義**
- 採用右手座標系（Right-handed Cartesian coordinate system）：
  - `x` 軸與 `y` 軸在水平平面內
  - `z` 軸垂直向上

---

**球面角度定義（Spherical Angles）**
- **θ（Theta）**：天頂角（Zenith angle）
  - 取值範圍：`0° ≤ θ ≤ 180°`
  - θ = 0° → 指向天頂（垂直上方，沿 z 軸正方向）
  - θ = 90° → 指向地平線（水平方向）

- **φ（Phi）**：方位角（Azimuth angle）
  - 取值範圍：`0° ≤ φ < 360°`
  - 定義於 `x-y` 平面中
  - φ = 0° 通常與 `x` 軸對齊，逆時針方向增加

---
**傳播方向與球面基底向量**

- **n̂（傳播方向單位向量）**：表示波的傳播方向
- 以 n̂ 為基準，定義球面基底向量：
  - **θ̂**：沿天頂角 θ 的遞增方向
  - **φ̂**：沿方位角 φ 的遞增方向（垂直於 θ̂）

---

在 3GPP TR 38.901 中，這套角度定義：
- 用於表示到達角（AOA）與離開角（AOD）
- 適用於天線場型、通道建模與極化計算等模組
---

**全球座標系統（GCS）中的符號定義**

本表整理了在 3GPP 通道建模中，全球座標系統下常見的角度與天線場型符號對應。

| **參數說明** | **符號（Notation）** |
| ------------ | -------------------- |
| LOS AOD（Line-of-Sight Departure Azimuth） | φ<sub>LOS,AOD</sub> |
| LOS AOA（Line-of-Sight Arrival Azimuth） | φ<sub>LOS,AOA</sub> |
| LOS ZOD（Line-of-Sight Departure Zenith） | θ<sub>LOS,ZOD</sub> |
| LOS ZOA（Line-of-Sight Arrival Zenith） | θ<sub>LOS,ZOA</sub> |
| AOA for cluster *n* | φ<sub>n,AOA</sub> |
| AOD for cluster *n* | φ<sub>n,AOD</sub> |
| AOA for ray *m* in cluster *n* | φ<sub>n,m,AOA</sub> |
| AOD for ray *m* in cluster *n* | φ<sub>n,m,AOD</sub> |
| ZOA for cluster *n* | θ<sub>n,ZOA</sub> |
| ZOD for cluster *n* | θ<sub>n,ZOD</sub> |
| ZOA for ray *m* in cluster *n* | θ<sub>n,m,ZOA</sub> |
| ZOD for ray *m* in cluster *n* | θ<sub>n,m,ZOD</sub> |

---

**天線場型向量定義（Antenna Field Patterns）**

| **描述** | **符號** |
| -------- | -------- |
| 接收天線元素 *u* 在方向 $\hat{θ}$ 上的場型 | F<sub>Rx,u,θ</sub> |
| 接收天線元素 *u* 在方向 $\hat{φ}$ 上的場型 | F<sub>Rx,u,φ</sub> |
| 發送天線元素 *s* 在方向 $\hat{θ}$ 上的場型 | F<sub>Tx,s,θ</sub> |
| 發送天線元素 *s* 在方向 $\hat{φ}$ 上的場型 | F<sub>Tx,s,φ</sub> |

---
- φ 表示 **方位角 Azimuth**，θ 表示 **天頂角 Zenith**
- AOD/ZOA 為 **發射端參數**，AOA/ZOD 為 **接收端參數**
- 上述符號應用於通道係數計算、天線增益、極化矩陣生成等模組中

## Step 1：設定環境、網路佈局與天線陣列參數（Set environment, network layout, and antenna array parameters）

---

### a) 選擇場景並定義座標系統

- 選擇一個模擬場景（Scenario）：
  - UMa（Urban Macro）
  - UMi（Urban Micro – Street Canyon）
  - RMa（Rural Macro）
  - InH-Office / InF（Indoor Office / Factory）

- 定義全球座標系統中的角度與向量：
  - 天頂角（Zenith angle）：θ
  - 方位角（Azimuth angle）：φ
  - 球面基底向量（Spherical basis vectors）： $\hat{θ}$ $\hat{φ}$

注意：RMa 適用頻率最高為 7GHz，其餘場景支援至 100GHz。

---

###  b) 指定基地台（BS）與用戶端（UT）數量

---

###  c) 指定 3D 位置與 LOS 角度

- 提供 BS 與 UT 的三維座標（x, y, z）
- 計算並定義以下 LOS 角度（基於 GCS）：
  - LOS AOD： $\phi_{LOS,AOD}$
  - LOS ZOD： $\theta_{LOS,ZOD}$
  - LOS AOA： $\phi_{LOS,AOA}$
  - LOS ZOA： $\theta_{LOS,ZOA}$

---

### d) 提供天線場型（Antenna Field Pattern）

- 在 GCS 中指定天線的方向性增益函數：
  - BS 與 UT 的發射/接收場型：
    -  $F_{rx}$、 $F_{tx}$

---

### e) 陣列朝向定義（Array Orientation）

- BS 陣列由三個角度定義：
  - $\Omega_{BS,\alpha}$：BS 方位角（Bearing angle）
  - $\Omega_{BS,\beta}$：BS 下傾角（Downtilt angle）
  - $\Omega_{BS,\gamma}$：BS 傾斜角（Slant angle）

- UT 陣列由三個角度定義：
  - $\Omega_{UT,\alpha}$：UT 方位角
  - $\Omega_{UT,\beta}$：UT 下傾角
  - $\Omega_{UT,\gamma}$：UT 傾斜角

---

### f) 定義使用者移動速度與方向

- 指定 UT 的移動速度與方向（基於 GCS）

---

### g) 指定系統中心頻率與頻寬

- 中心頻率： $f_c$
- 頻寬： $B$

---

## Step 2：指派傳播條件（Assign propagation condition）

- 根據 **Table 7.4.2-1(page 30)** 指定每個 BS-UT link 的 **LOS 或 NLOS** 狀態
- 每條 BS-UT link 的 LOS 狀態互不相關（Uncorrelated）
- 為每個 UT 指定 **室內或室外狀態**（Indoor/Outdoor）
  > 同一個 UT 所有 link 的室內/室外狀態相同

---

## Step 3：計算路徑損耗（Pathloss）

- 使用 **Table 7.4.1-1(page27~29)** 的公式，根據場景與 link 類型計算 pathloss

---

## Step 4：生成大尺度參數（Large Scale Parameters, LSPs）

- 需產生以下參數：
  - **DS**：Delay Spread
  - **ASD** / **ASA**：Azimuth Spread of Departure/Arrival
  - **ZSD** / **ZSA**：Zenith Spread of Departure/Arrival
  - **K**：Ricean K-factor（僅 LOS）
  - **SF**：Shadow Fading

- 產生方式：
  - 參照 **Table 7.5-6** 中的均值、標準差與交叉關聯（Cross-Correlation）矩陣
  - 使用 **Cholesky 分解** 取得方根矩陣 $\sqrt{C_{M×M}}(0)$
  - 對應的參數向量排序為：

    ```
    s_M = [SSF, SK, SDS, SASD, SASA, SZSD, SZSA]^T
    ```

> 不同 BS-UT link 的 LSP 是 **不相關的**（Uncorrelated）  
> 但來自同一扇區（co-sited sectors）到同一 UT 的 LSP 是相同的  
> 不同樓層之間的 UT link LSP 也視為不相關

---
## Step 5：產生 Cluster Delays（群組延遲 τₙ）
### 延遲生成步驟
延遲值根據指數分布隨機產生，計算方式如下：
![image](https://github.com/user-attachments/assets/ce8249ef-ddd6-478c-b2c5-d062d1fe94f8)

### 延遲正規化與排序
延遲正規化與排序步驟如下：
![image](https://github.com/user-attachments/assets/7d26ee58-0e32-4ef6-a8cb-44c13f422412)

### LOS 情況下延遲縮放
若 link 為 **LOS**，需補償 LOS peak 對延遲展布的影響，使用經驗公式縮放：
![image](https://github.com/user-attachments/assets/cd656923-db34-404f-9145-9327069fe056)
where K [dB] is the Ricean K-factor

### 將延遲縮放
![image](https://github.com/user-attachments/assets/84e43009-782a-4fb7-8f64-6e6b4a0a1652)

**LOS 縮放後的延遲不可用於後續的 cluster power 計算。**


