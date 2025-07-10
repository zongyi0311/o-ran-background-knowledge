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
若天線元素的場型在 LCS 中定義為 $A'(\theta', \phi')$，則其在 GCS 中的對應為：

$A(\theta, \phi) = A'(\theta', \phi')$  (7.1-1)

其中 $\theta', \phi'$ 是透過 GCS → LCS 的方向轉換公式

LCS 到 GCS 的轉換可透過三個角度完成：

1. **α（bearing angle）**：繞 z 軸旋轉，決定天線朝向
2. **β（downtilt angle）**：繞第一次旋轉後的 y 軸（記作 $\dot{y}$）旋轉，決定下傾角
3. **γ（slant angle）**：繞第二次旋轉後的 x 軸（記作 $\ddot{x}$）旋轉，決定極化傾斜角度

三次旋轉後的軸方向分別為 $\dddot{x}, \dddot{y}, \dddot{z}$，亦可記為 $x', y', z'$，代表 LCS。

## GCS → LCS 旋轉矩陣定義
為了將 GCS 中的點 $(x, y, z)$ 轉換至 LCS 中的 $(x', y', z')$，需建立三維旋轉矩陣 $R$。

這個旋轉矩陣是三個基本旋轉的乘積，旋轉順序為：

1. 沿 $z$ 軸旋轉角度 $\alpha$
2. 沿 $\dot{y}$ 軸旋轉角度 $\beta$
3. 沿 $\ddot{x}$ 軸旋轉角度 $\gamma$

旋轉矩陣為：

![image](https://github.com/user-attachments/assets/c3a8f19b-32b3-4570-a9bb-749cd03d68e1)

反向變換：旋轉矩陣的反矩陣:
![image](https://github.com/user-attachments/assets/1dd892ad-8a36-47d8-b9fd-f058b6205b6a)

正向旋轉矩陣展開:
![image](https://github.com/user-attachments/assets/529e91dd-4174-464f-8148-f023de7a541c)

反向旋轉矩陣展開:
![image](https://github.com/user-attachments/assets/18b3da47-6e7d-4eda-8db3-1ab42cc7c050)

θ:天頂角 φ:方位角

對應的 Cartesian 笛卡兒座標為：
![image](https://github.com/user-attachments/assets/8141d6ee-a6fb-463c-ab1b-2be098555dfc)

給定一個點在 GCS 中的方向 (θ, φ)，若已知 LCS 與 GCS 之間的旋轉角度 α、β、γ，則可推導該點在 LCS 中的角度 (θ', φ')：

θ'（local zenith angle） 是根據 GCS 下單位向量經過 R⁻¹ 旋轉後，與 z 軸夾角的反餘弦值計算

φ'（local azimuth angle） 是旋轉後單位向量在 x-y 平面的投影所對應的相位角

![image](https://github.com/user-attachments/assets/1ac62ce9-1e0b-472f-a689-8264a78c0f04)

當我們要將 GCS 的場（例如電場或磁場的分量）轉換為 LCS 的表示方式，我們使用一個 2x2 的變換矩陣，該矩陣由以下組成：
-  第一列：GCS 的 θ 方向單位向量投影到 LCS 的 θ' 與 φ' 方向。
-  第二列：GCS 的 φ 方向單位向量投影到 LCS 的 θ' 與 φ' 方向。
![image](https://github.com/user-attachments/assets/d5343d39-9451-4ca2-a2bb-2c80831d109a)

---
![image](https://github.com/user-attachments/assets/9a27399f-6e47-4112-b63e-fa8736844cb0)
-  這個旋轉矩陣（R） 將 LCS 中的單位向量轉換為 GCS 參考系中表示。

-  在計算天線極化方向變換、場分量轉換時，必須考慮這樣的旋轉關係，否則會導致方向與極化不一致。

---
為了描述 GCS 和 LCS 間的場方向變換，定義了一個旋轉角 ψ，代表 GCS 與 LCS 的球面單位向量間的角度差異。

**單位向量旋轉關係的精簡公式如下所示**
![image](https://github.com/user-attachments/assets/dd8a4f84-176e-4203-8710-f61cfd11f7be)
帶入(7.1-9)後：
![image](https://github.com/user-attachments/assets/71dc5b31-0a69-4f62-ac0b-9e3303db064e)

---

角度 ψ 可透過下列公式計算：
![image](https://github.com/user-attachments/assets/a73fbffb-53bf-433c-bda3-ef95cedac0f8)

這裡是 GCS 中 θ̂ 和 φ̂ 的表示式:
![image](https://github.com/user-attachments/assets/9d8653c3-5859-4663-91ee-e7cacb884cc2)

這些單位向量可以用來進行點積計算（dot product），用於 ψ 的推導或場方向的轉換

---
**小結**
-  ψ 角是 GCS 和 LCS 單位向量差異的關鍵角度。
-  可用它來旋轉場向量，從而將 LCS 中的場表示轉換到 GCS。
-  θ̂、φ̂ 的直角座標定義
-  旋轉矩陣 R 的作用
-  ψ 的定義與作用在場向量變換中的角色

![image](https://github.com/user-attachments/assets/b98bd9ad-b88e-4b58-93d2-e5029c72e366)

## Transformation from an LCS to a GCS for downtilt angle only
🔹 前提條件
-  此轉換假設：
  -  方位角 α = 0（無水平旋轉）
  -  傾仰角 β ≠ 0（有垂直 downtilt）
  -  偏轉角 γ = 0（無斜角）
-  換句話說：
  -  LCS 的 y'-軸 平行於 GCS 的 y-軸。
  -  LCS 的 x'-軸 與天線波束指向方向（sector pointing direction）一致。
  -  Downtilt 是繞 GCS 的 y 軸旋轉形成的。

![image](https://github.com/user-attachments/assets/f46f3cfc-f6ce-4d94-a827-d13556fafeaa)

**向量變換重點**
-  基底旋轉關係：
  -  當 LCS 繞 y 軸旋轉 β 後，原本 GCS 的基底向量 θ̂、φ̂ 與 LCS 中的基底 θ̂′、φ̂′ 之間產生旋轉
  -  這種旋轉會影響偏極化場型的對應方向，並由 ψ（斜角）表示

**ψ 的幾何意義**
-  在右側小圖中：
  -  ψ 為 GCS 與 LCS 球面基底向量之間的旋轉角差
  -  它定義了：在同一方向 n̂ 下，GCS 的球面基底 (θ̂, φ̂) 和 LCS 的球面基底 (θ̂′, φ̂′) 的夾角

# 7.2 Scenarios 

![image](https://github.com/user-attachments/assets/c455497e-d5bb-44d5-b606-3f926c9b20a3)

- 	UMi - 街道峽谷 (Street Canyon)
- 	UMa - 城市宏小區 (Urban Macro)

![image](https://github.com/user-attachments/assets/7a9f9331-aab2-4be5-98c7-cd53994e4be1)
**備註說明**：
-  LOS 機率差異 是 open office 與 mixed office 的唯一區別。
-  Open office 模型預設大部分情況是開放無遮蔽，因此 LOS 機率較高。
-  Mixed office 模型則包含牆壁與隔間，使 LOS 機率降低。

![image](https://github.com/user-attachments/assets/82638bee-9239-42d2-8d13-dbe584e6d5c6)

**場景簡述**
-  RMa 主要模擬 大範圍、連續性覆蓋 的鄉村場景。
-  適用於 高速移動車輛 的無線通訊。
-  通道受限於 熱噪聲與干擾（noise-limited/interference-limited）。
-  採用 macro TRPs 作為基地台部署形式。

![image](https://github.com/user-attachments/assets/22a48b36-366e-48db-a56d-b994ad00c8a8)

| 分類     | 縮寫                      | 說明            |
| ------ | ----------------------- | ------------- |
| InF-SL | Sparse clutter, Low BS  | 雜物少、基地台低於雜物高度 |
| InF-DL | Dense clutter, Low BS   | 雜物多、基地台低於雜物高度 |
| InF-SH | Sparse clutter, High BS | 雜物少、基地台高於雜物   |
| InF-DH | Dense clutter, High BS  | 雜物多、基地台高於雜物   |
| InF-HH | High Tx, High Rx        | 發送與接收端都高於雜物   |

| 參數                 | 值                     |
| ------------------ | --------------------- |
| **Room size**      | 長方形，面積 20–160,000 m²  |
| **Ceiling height** | 5–25 m（低 BS 為 5–15 m） |
| **有效雜物高度 $h_c$**   | 0–10 m，且小於天花板高度       |

| 參數                     | InF-SL / SH    | InF-DL / DH  | InF-HH |
| ---------------------- | -------------- | ------------ | ------ |
| **牆與天花板**              | 混凝土或金屬牆，金屬塗層窗戶 |              |        |
| **雜物類型**               | 大型規律金屬機械或倉儲    | 中小型、不規則機械與產線 | 任意     |
| **雜物大小 $d_{clutter}$** | 10 m           | 2 m          | 任意     |
| **雜物密度 $r$**           | 低密度（< 40%）     | 高密度（≥ 40%）   | 任意     |

| 參數                 | InF-SL / DL   | InF-SH / DH   | InF-HH     |
| ------------------ | ------------- | ------------- | ---------- |
| **BS 高度 $h_{BS}$** | 雜物中（低於雜物高度）   | 高於雜物          | 高於雜物       |
| **LOS/NLOS**       | LOS 與 NLOS 混合 | LOS 與 NLOS 混合 |  100% LOS |
| **UT 高度 $h_{UT}$** | 雜物中           | 高於雜物          | 高於雜物       |

# 7.3 Antenna modellin
**天線面板排列結構**
-  基站（BS）天線被建模為一個 矩形面板陣列，由下列五元組表示：(M_g, N_g, M, N, P)
| 參數    | 意義                                                     |
| ----- | ------------------------------------------------------ |
| `M_g` | 面板在**垂直方向**的數量                                         |
| `N_g` | 面板在**水平方向**的數量                                         |
| `M`   | 每列中具有相同極化的天線元素數量                                       |
| `N`   | 每列中的列數（即每個面板中的列數）                                      |
| `P`   | 極化類型：`1` 單極化（Single Polarized），`2` 雙極化（Dual Polarized） |

**面板間距**
- 水平方向間距：`dg,H`
- 垂直方向間距：`dg,V`

**面板內的天線元素配置**
- 在每個天線面板內，天線元素以矩陣排列，具有 M × N 元素。
- 水平間距：d_H
- 垂直間距：d_V

**坐標與編號**
- 天線面板的編號假設觀測方向是從天線正面看（x 軸朝向寬邊方向，y 軸向上表示欄位數遞增）
- 元素座標 (m, n)，其中：
  - m：第幾列（垂直方向）
  - n：第幾欄（水平方向）
 
 **這種模型廣泛用於：**
-  大規模 MIMO、系統波束賦形設計、通道建模與仿真
-  
<img width="786" height="270" alt="image" src="https://github.com/user-attachments/assets/0ea59782-aaa4-42af-b372-156583bd9110" />

---

<img width="786" height="412" alt="image" src="https://github.com/user-attachments/assets/a3c06229-f95f-4224-b33d-b7ecc44107e8" />

- 這些功率圖表示天線對不同方向的發射能力（以 dB 表示）
- 功率會隨著偏離主波束方向（90°）而下降，下降程度由(3dB beamwidth）決定
- min{} 是為了確保輻射功率不超過副瓣抑制限制（SLA）或最大衰減（A<sub>max</sub>）
- 3D 模式表示在整體空間內的輻射衰減，會綜合垂直與水平方向的特性
- 最大天線增益是指主波束方向的理論最強功率增益

**Antenna port mapping**
- 如何將傳統基站（Legacy BS）天線陣列的 垂直波束成形（vertical beamforming）使用 固定相位差（fixed phase shifts） 模擬為一組 複數權重（complex weights）

<img width="687" height="74" alt="image" src="https://github.com/user-attachments/assets/64dd7722-e91a-41b9-8d0a-44b05849035d" />

**參數定義**
| 符號     | 說明                                                                           |
| ------ | ---------------------------------------------------------------------------- |
| `m`    | 天線元素索引，m = 1, ..., M                                                         |
| `M`    | 垂直方向上的天線元素數量                                                                 |
| `θ` | 垂直方向上的電氣波束掃描角度（Electrical steering angle），定義在 \[0°, 180°]，其中 90° 表示垂直於天線陣列方向 |
| `λ`    | 波長（Wavelength）                                                               |
| `dᵥ`   | 垂直天線元素間距（Vertical element spacing）                                           |
| `j`    | 虛數單位（j = √−1）                                                                |

- 複數權重 wₘ 的相位是根據垂直方向的期望角度 θ 調整，使得所有天線發射訊號在該方向相位對齊、強化波束
- 係數 1/√M 是為了進行 能量歸一化

**Polarized antenna modelling**
- 天線的輻射功率圖（Power Pattern）與其輻射場（Radiation Field）之間的關係如下：

<img width="738" height="51" alt="image" src="https://github.com/user-attachments/assets/8e16c052-8e94-47c4-ba80-59784eeaf006" />
<img width="756" height="82" alt="image" src="https://github.com/user-attachments/assets/2665e17f-6509-4000-9a59-e839e6c34635" />

**Model-1：具有極化斜角的天線元素（Polarization Slant Angle）**
-  當天線具有極化特性時，假設：
  - ζ 是極化斜角（Slant angle）
    - ζ = 0° → 純垂直極化
    - ζ = ±45° → 交叉極化天線對
   
電場分量可透過下列矩陣轉換:
<img width="749" height="171" alt="image" src="https://github.com/user-attachments/assets/310c62ea-32da-4562-9485-50d9110cc506" />

<img width="749" height="171" alt="image" src="https://github.com/user-attachments/assets/242465d5-fb60-4f65-9829-5ce8612b0851" />

**角度無關的極化建模（Angle-Independent Polarization Model)**
- 在此模型中，假設極化方向在方位角（azimuth）與仰角（elevation）上均不隨角度變化，適用於局部坐標系（LCS）下的極化建模
- 對於一個線性極化天線（linearly polarized antenna），天線元素的電場分量如下：

<img width="728" height="122" alt="image" src="https://github.com/user-attachments/assets/e46d7a32-0c00-4171-be14-f3340fdb71bf" />

| 符號                     | 說明                                                        |
| ---------------------- | --------------------------------------------------------- |
| F'θ', F'φ'     | 對應垂直與水平極化方向的電場分量                                          |
| $A'(θ', φ')$           | 方向 $(θ', φ')$ 上的 3D 天線功率圖（由表 7.3-1 定義）                    |
| $ζ$                    | 極化斜角（0°：純垂直極化；±45°：交叉極化）                                  |
| $θ', φ'$               | 在局部坐標系（LCS）下的仰角與方位角                                       |
| $θ' = θ''$, $φ' = φ''$ | 表示 $A'(θ', φ')$ 與 $A''(θ'', φ'')$ 可視為相同方向下的值（此處假設兩套座標系一致） |

| 模型          | 特性          | 電場方向依賴角度？ | 是否需旋轉變換 $ψ$ |
| ----------- | ----------- | --------- | ----------- |
| **Model-1** | 考慮方向變化與極化旋轉 |  是       |  需要        |
| **Model-2** | 假設極化方向固定不變  |  否       |  不需要       |

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


