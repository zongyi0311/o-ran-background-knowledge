📘 DVB-S2 標準介紹（ETSI EN 302 307）

---

🔷 **標準名稱：** Digital Video Broadcasting - Satellite - Second Generation  
🔷 **發布機構：** ETSI（歐洲電信標準協會）  
🔷 **主要用途：** 衛星通訊中的數位視訊廣播（如數位衛星電視、VSAT、IP over satellite）

🔷 **通訊結構：**
- **實體層架構：** 資料封裝 → 外部編碼 (BCH) → 內部編碼 (LDPC) → interleaving → 調變 → 發射
- **編碼流程：** LDPC (低密度奇偶檢查碼) + BCH (Bose–Chaudhuri–Hocquenghem) 為 concatenated code  
- **調變模式：** QPSK / 8PSK / 16APSK / 32APSK  
- **碼率支持：** 多種碼率（如 1/4、1/3、2/5、1/2、3/5、2/3、3/4、4/5、5/6、8/9、9/10）

🔷 **LDPC 特性：**
- **矩陣長度固定：** `N = 64800` 或 `N = 16200`（short/normal frame）  
- **根據碼率決定資訊長度 `K`，檢查長度 `M = N - K`**
- **結構化設計：** 使用 CPM（循環置換子矩陣）組合 → 適合硬體平行運算  
- **檢查矩陣大小：** `M × N`（稀疏矩陣）

🔷 **錯誤保護能力強：**
- 接近 Shannon 極限
- 支援高效能軟解碼（例如 belief propagation）

---

## `configureDVBS2Demo` 

---

```matlab
% === configureDVBS2Demo ===
% 建立與設定 DVB-S.2 模擬系統中使用的 System objects
% 參考自 MATLAB 範例 commDVBS2WithLDPC

%% 模擬參數設定
% 設定系統參數與常數

maxNumLDPCIterations = 200;  % LDPC 解碼器最大迭代次數
dvb = getParamsDVBS2Demo(subsystemType, EsNodB, maxNumLDPCIterations);

% 說明：
% - subsystemType：指定子系統組合（例如 QPSK 1/2）
% - EsNodB：符號能量與雜訊功率比 (SNR)
% - dvb：回傳參數結構，包含 Frame 長度、調變方式、編碼率等

%% 建立模擬元件
% 建立系統模擬所需的物件（如 BCH 編碼器、調變器等）

createSimObjDVBS2Demo;

% 說明：
% 這個函數會自動建立以下物件：
% - BCH 編碼器與解碼器
% - LDPC 編碼器與解碼器
% - 調變器（如 PSKModulator）
% - 通道（AWGNChannel）
% - 解調器（含 LLR 輸出支援）

%% 效能測量元件建立
% 建立封包與位元錯誤率量測器、星座圖觀察器、SNR 計算器

% === 錯誤率計算器 ===
PER     = comm.ErrorRate;  % Packet Error Rate
BERLDPC = comm.ErrorRate;  % LDPC 解碼後的 Bit Error Rate
BERMod  = comm.ErrorRate;  % 調變器輸出之 Bit Error Rate（未解碼）

%% 星座圖觀察器設定

% 若為 QPSK 或 8PSK，需暫時設定 BitInput = false 以取得參考星座
if dvb.ModulationOrder == 4 || dvb.ModulationOrder == 8
    cacheBitInput = pskModulator.BitInput;
    pskModulator.BitInput = false;
    constellation = pskModulator((0:pskModulator.ModulationOrder-1)');
    release(pskModulator);
    pskModulator.BitInput = cacheBitInput;
else
    constellation = dvb.Constellation;
end

% 建立星座圖物件
constDiag = comm.ConstellationDiagram(...
    'SamplesPerSymbol',       1, ...
    'ReferenceConstellation', constellation, ...
    'XLimits',                [-3.5 3.5], ...
    'YLimits',                [-3.5 3.5], ...
    'Position',               [50 50 560 420]);

% 說明：
% - constellation：參考星座點，用於與接收訊號比對
% - constDiag：星座圖繪圖器，顯示接收訊號分佈

%% SNR 測量器

meanCalc = dsp.MovingAverage;

% 說明：
% - 使用移動平均估計接收訊號功率
% - 搭配星座圖可分析雜訊對系統穩定性的影響

##  getParamsDVBS2Demo 設定參數筆記（BCH、LDPC、星座、交錯等）

```matlab
function dvb = getParamsDVBS2Demo(subsystemType, EsNodB, numLDPCDecIterations)
% 根據 modulation + code rate 選項，回傳 DVB-S.2 模擬參數結構 dvb

%% === 基本參數檢查 ===

% 檢查 subsystemType 合法性（如 QPSK 1/2、8PSK 3/5 等）
validatestring(subsystemType, { ... 
    'QPSK 1/4', 'QPSK 1/3', ..., '32APSK 9/10'}, ...
    'getParamsDVBS2Demo', 'TYPE', 1);

% 確保 SNR 為有限數值
validateattributes(EsNodB, {'numeric'}, {'finite', 'scalar'}, 'getParamsDVBS2Demo', 'ESNO', 2);

% 確保解碼次數為正整數
validateattributes(numLDPCDecIterations, {'numeric'}, {'positive', 'integer', 'scalar'}, 'getParamsDVBS2Demo', 'NUMITER', 3);

%% === 拆解 modulationType 與 codeRate ===

systemInfo = regexp(subsystemType, ' ', 'split');
modulationType = char(systemInfo(1));  % 調變方式
dvb.CodeRate = char(systemInfo(2));   % 代碼率
codeRate = str2num(dvb.CodeRate);     % 轉成數值
dvb.EsNodB = EsNodB;
dvb.ModulationType = modulationType;

%% === Packet 封包設定 ===

dvb.NumBytesPerPacket = 188;
dvb.NumBitsPerPacket = 188 * 8;

%% === BCH 編碼參數 ===

[dvb.BCHCodewordLength, dvb.BCHMessageLength, dvb.BCHGeneratorPoly] = getbchparameters(codeRate);
dvb.BCHPrimitivePoly = int2bit(65581, 17)';  % 固定使用的生成多項式

dvb.NumPacketsPerBBFrame = floor(dvb.BCHMessageLength / dvb.NumBitsPerPacket);
dvb.NumInfoBitsPerCodeword = dvb.NumPacketsPerBBFrame * dvb.NumBitsPerPacket;
dvb.BitPeriod = 1 / dvb.NumInfoBitsPerCodeword;

%% === LDPC 編碼參數 ===

dvb.LDPCCodewordLength = 64800;  % 固定長度（short frame 可自行擴充）
dvb.LDPCParityCheckMatrix = dvbs2ldpc(codeRate);
dvb.LDPCNumIterations = numLDPCDecIterations;

%% === Interleaving（交錯） ===

dvb.InterleaveOrder = (1:dvb.LDPCCodewordLength).';

switch modulationType
    case '8PSK'
        Ncol = 3;
        iTemp = reshape(dvb.InterleaveOrder, [], Ncol).';
        if codeRate == 3/5
            iTemp = flipud(iTemp);  % 特例（Figure 8）
        end
        dvb.InterleaveOrder = iTemp(:);
    case '16APSK'
        dvb.InterleaveOrder = reshape(dvb.InterleaveOrder, [], 4).';
        dvb.InterleaveOrder = dvb.InterleaveOrder(:);
    case '32APSK'
        dvb.InterleaveOrder = reshape(dvb.InterleaveOrder, [], 5).';
        dvb.InterleaveOrder = dvb.InterleaveOrder(:);
end

%% === 星座與映射 ===

switch modulationType
    case 'QPSK'
        dvb.Constellation = ([+1 +1; -1 +1; -1 -1; +1 -1] * [1; 1i]) / sqrt(2);
        dvb.SymbolMapping = [0 2 3 1];
        dvb.PhaseOffset = pi/4;
    case '8PSK'
        A = sqrt(1/2);
        Ry = [+A +1 -1 -A  0 +A -A  0].';
        Iy = [+A  0  0 -A  1 -A +A -1].';
        dvb.Constellation = Ry + 1i*Iy;
        dvb.SymbolMapping = [1 0 4 6 2 3 7 5];
        dvb.PhaseOffset = 0;
    case '16APSK'
        dvb.Constellation = dvbsapskmod((0:15)', 16, 's2', dvb.CodeRate, 'UnitAveragePower', true);
        dvb.SymbolMapping = [12 14 15 13 4 0 8 10 2 6 7 3 11 9 1 5];
        dvb.PhaseOffset = [pi/4 pi/12];
    case '32APSK'
        dvb.Constellation = dvbsapskmod((0:31)', 32, 's2', dvb.CodeRate, 'UnitAveragePower', true);
        dvb.SymbolMapping = [...長列表省略...];
        dvb.PhaseOffset = [pi/4 pi/12 pi/16];
end

numModLevels = length(dvb.Constellation);
dvb.BitsPerSymbol = log2(numModLevels);
dvb.ModulationOrder = 2^dvb.BitsPerSymbol;

%% === 通道參數 ===

dvb.SequenceIndex = 2;  % 雜訊亂碼種子
dvb.NumSymsPerCodeword = dvb.LDPCCodewordLength / dvb.BitsPerSymbol;
dvb.NoiseVar = 1 / (10^(dvb.EsNodB/10));
dvb.NoiseVarEst = dvb.NoiseVar / (2 * sin(pi/numModLevels));

%% === 解碼延遲參數 ===

dvb.RecDelayPreBCH = dvb.BCHMessageLength;

end

%% AWGN 通道
% 建立 AWGNChannel，雜訊定義來自 dvb.NoiseVar
% NoiseMethod 設為 'Variance'，確保正確加入指定 SNR 的雜訊

chan = comm.AWGNChannel( ...
    'NoiseMethod', 'Variance', ...
    'Variance', dvb.NoiseVar);

##  dvbs2ldpc 筆記：根據 DVB-S.2 標準產生 LDPC 檢查矩陣

```matlab
function H = dvbs2ldpc(R, varargin)
% 根據 DVB-S.2 標準，取得特定碼率 R 的 LDPC parity-check 矩陣 H
% 輸出格式可為 sparse 邏輯矩陣 或 row-column indices

% 範例：
% H = dvbs2ldpc(3/5);
% spy(H);  % 可視化非零元素
% encoderCfg = ldpcEncoderConfig(H);
% decoderCfg = ldpcDecoderConfig(H);

%% === 基本設定 ===

narginchk(1,2);                 % 必須提供 1 或 2 個參數
lenCodeWord = 64800;           % 固定 codeword 長度（long frame）
NB = 360;                      % Node group size，依據 DVB-S.2 標準

numInfoBits   = lenCodeWord * R;
numParityBits = lenCodeWord - numInfoBits;

%% === 取得檢查節點的排列表（兩組） ===

[ct1, ct2] = getchecknodetable(R);   % 每種碼率對應不同 check node 表

% 將 check table 轉成實際位置（向量形式）
ck1 = nodeindices(ct1, numParityBits, NB);
ck2 = nodeindices(ct2, numParityBits, NB);

% d：每組分群後的行/列數與擴充
d = [size(ck1,2) size(ck1,1) size(ck2,2) size(ck2,1) numParityBits-1 2 1 1];

% r：對應的 row index（check node index）
r = [ck1(:); ck2(:); 0; reshape(ones(2,1)*(1:numParityBits-1),[],1)];

% S：之後要組成的列位置（column index）
S = zeros(length(r),1);
numGroup = length(d)/2;
n = 0;
ncol = 1;

% 按照每組分群方式產生 column 索引
for i = 1:numGroup
    p = d(2*i-1)*d(2*i);  % 該組有幾個元素
    S(n+1:n+p) = reshape(ones(d(2*i),1)*(ncol:ncol+d(2*i-1)-1),p,1);
    ncol = ncol + d(2*i-1);
    n = n + p;
end

%% === 組合 LDPC 矩陣 H ===

outputFormat = 'sparse';  % 預設輸出為 sparse 邏輯矩陣
if nargin == 2
    if ~strcmp(varargin{1}, 'sparse') && ~strcmp(varargin{1}, 'indices')
        error('outputFormat must be either ''sparse'' or ''indices''');
    end
    outputFormat = varargin{1};
end

if strcmp(outputFormat, 'sparse')
    H = logical(sparse(double(r+1), S, 1));  % MATLAB 用 1-based index
else
    H = [double(r+1), double(S)];
end


%% === 子函式：nodeindices ===
function ck = nodeindices(ct, M, NB)
% 將檢查表 ct 映射成位置（行與列的關聯表）

[N, D] = size(ct);
q = (M / NB);
b = (1:NB);  % 分 NB 組
bq = (b-1).' * q;  % 位移向量
ck = zeros(D, NB*N);  % 預先配置大小

for r = 1:N
    ck(:, NB*(r-1)+1:NB*r) = mod(addcr(bq, ct(r,:)), M)';
end


%% === 子函式：addcr ===
function A = addcr(c, r)
% 將每一列向量 r 加上常數 c 形成展開表
M = length(c);
N = length(r);
A = zeros(M, N);
for m = 1:M
    A(m, :) = r + c(m);
end


%% === 子函式：getchecknodetable ===
function [ct1, ct2] = getchecknodetable(R)
% 根據碼率 R，回傳兩組 check node base 表（ct1, ct2）
% 每個碼率在標準中都有固定對應的 base 表
% 詳細數據可參考 ETSI DVB-S.2 原始標準


# 📦 `dvb` 結構欄位總整理（DVB-S.2 模擬參數）

此結構由 `getParamsDVBS2Demo()` 函式產生，是整個 DVB-S.2 系統模擬的核心配置，供後續模組（編碼、調變、通道等）使用。

---

## 🧾 基本參數

| 欄位名稱 | 說明 |
|----------|------|
| `ModulationType` | 調變方式，如 `'QPSK'`, `'8PSK'`, `'16APSK'`, `'32APSK'` |
| `CodeRate` | 編碼率（字串），如 `'3/5'`、`'2/3'` |
| `EsNodB` | 每符號能量與雜訊功率比（SNR），以 dB 表示 |

---

## 🧮 資料長度與封包設定

| 欄位名稱 | 說明 |
|----------|------|
| `NumBytesPerPacket` | 每個 MPEG-TS 封包的大小（固定為 188 bytes） |
| `NumBitsPerPacket`  | 每個封包的位元數（通常為 188 × 8 = 1504） |
| `NumPacketsPerBBFrame` | 每個 BBFrame 中封包數量，與 BCH 長度對齊 |
| `NumInfoBitsPerCodeword` | 每個 LDPC 編碼輸入（總訊息位元數） |
| `BitPeriod` | 一個位元的週期（1 / `NumInfoBitsPerCodeword`） |

---

## 🔐 BCH 編碼設定

| 欄位名稱 | 說明 |
|----------|------|
| `BCHMessageLength` | BCH 訊息長度（`kBCH`） |
| `BCHCodewordLength` | BCH 碼字長度（`nBCH`） |
| `BCHGeneratorPoly` | BCH 生成多項式（向量形式） |
| `BCHPrimitivePoly` | BCH 原始多項式（GF(2^m) 上的） |

---

## 🧩 LDPC 編碼設定

| 欄位名稱 | 說明 |
|----------|------|
| `LDPCCodewordLength` | LDPC 碼字長度（固定為 64800） |
| `LDPCParityCheckMatrix` | LDPC 檢查矩陣（由 `dvbs2ldpc` 產生） |
| `LDPCNumIterations` | 解碼最大迭代次數 |

---

## 🔄 交錯與符號映射

| 欄位名稱 | 說明 |
|----------|------|
| `InterleaveOrder` | 交錯順序（不同 modulation 有不同列轉換規則） |
| `Constellation` | 星座圖（複數數字組成的調變符號點） |
| `SymbolMapping` | 自定義調變符號對應表（符合 DVB 標準） |
| `PhaseOffset` | 星座圖相位偏移量（例如 QPSK 使用 π/4） |
| `BitsPerSymbol` | 每個調變符號對應的位元數（如 QPSK 是 2） |
| `ModulationOrder` | 調變階數（等於 2^BitsPerSymbol） |

---

## 📡 通道與估計參數

| 欄位名稱 | 說明 |
|----------|------|
| `SequenceIndex` | Scrambler 種子序號（僅模擬用途） |
| `NumSymsPerCodeword` | 每個 LDPC 碼字對應的 modulation 符號數 |
| `NoiseVar` | 根據 SNR 推算的通道雜訊變異數 |
| `NoiseVarEst` | 提供 demodulator 使用的估計雜訊變異數（視 modulation 調整） |

---

## 🕒 延遲參數

| 欄位名稱 | 說明 |
|----------|------|
| `RecDelayPreBCH` | 接收端 BCH 前的預延遲補償設定（通常等於 BCHMessageLength） |
