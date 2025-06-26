# 📊 LDPC 模擬系統比較：MATLAB vs OAI ulschsim vs rfsim

```text
+----------------------------------------------------------+
| Step 1: Generate random data                             |
| → data = randi([0 1], K, 1)                              |
+----------------------------------------------------------+
                            |
                            v
+----------------------------------------------------------+
| Step 2: LDPC encode                                       |
| → codeword = ldpcEncode(data, cfgEnc)                    |
+----------------------------------------------------------+
                            |
                            v
+----------------------------------------------------------+
| Step 3: π/4-QPSK modulation                               |
| → modsignal = pskmod(codeword, M=4, pi/4)                |
+----------------------------------------------------------+
                            |
                            v
+----------------------------------------------------------+
| Step 4: Add AWGN noise                                    |
| → rxsig = awgn(modsignal, SNR_dB)                        |
+----------------------------------------------------------+
                            |
                            v
+----------------------------------------------------------+
| Step 5: Demodulate to LLR                                 |
| → demod = pskdemod(rxsig, M=4, pi/4, 'llr')              |
+----------------------------------------------------------+
                            |
                            v
+----------------------------------------------------------+
| Step 6: LDPC decode                                       |
| → rxbits = ldpcDecode(demod, cfgDec, max_iter)           |
+----------------------------------------------------------+
                            |
                            v
+----------------------------------------------------------+
| Step 7: Compare result and count errors                   |
| → errors = sum(rxbits ~= data)                           |
+----------------------------------------------------------+
                            |
                            v
+----------------------------------------------------------+
| Step 8: Accumulate BER statistics                         |
| → BER(idx) = total_errors / total_bits                   |
+----------------------------------------------------------+
```




## 🧩 系統架構圖對照

```mermaid
flowchart TB

subgraph MATLAB 模擬
  A1[資料產生<br>randi([0 1])]
  A2[LDPC 編碼<br>dvbs2ldpc + ldpcEncode()]
  A3[\u03c0/4-QPSK 調變<br>pskmod()]
  A4[AWGN 通道<br>awgn()]
  A5[\u03c0/4-QPSK 解調<br>pskdemod → LLR]
  A6[LDPC 解碼<br>ldpcDecode()]
  A7[錯誤統計<br>BER]
end

subgraph OAI ulschsim
  B1[資料產生]
  B2[LDPC 編碼<br>nr_ldpc_encoder()]
  B3[QPSK 調變<br>nr_modulation()]
  B4[AWGN 通道<br>內建]
  B5[LLR 計算<br>nr_rx_llr()]
  B6[LDPC 解碼<br>nr_ldpc_decoder()]
  B7[錯誤統計<br>BER / FER]
end

subgraph OAI rfsim 模式
  C1[UE 資料產生<br>MAC → PHY]
  C2[LDPC 編碼<br>nr_ldpc_encoder()]
  C3[QPSK 調變<br>nr_modulation()]
  C4[傳送 IQ 樣本<br>socket 傳至 rfsimulator]
  C5[AWGN/fading 通道<br>rfsimulator.c]
  C6[接收 IQ 樣本<br>gNB PHY RX]
  C7[LLR 計算<br>nr_rx_llr()]
  C8[LDPC 解碼<br>nr_ldpc_decoder()]
  C9[錯誤統計<br>需自加 BER 紀錄]
end
```

---

## ✅ 三者對照比較表

| 項目             | MATLAB 模擬                     | OAI `ulschsim`                   | OAI `rfsim`                        |
|------------------|----------------------------------|----------------------------------|------------------------------------|
| 使用 LDPC 標準   | DVB-S2                           | 3GPP NR LDPC (BG1/BG2)          | 同左                             |
| 解碼演算法       | BP 解碼器（內建）                | Min-sum BP（nr_ldpc_decoder）   | 同左                             |
| 調變方式         | π/4-QPSK                         | QPSK、16QAM 等可選              | 可由 MCS 控制                     |
| 通道模型         | AWGN（使用 awgn() 函式）         | 內建 AWGN                       | `rfsimulator` (AWGN / fading)     |
| 資料來源         | 自訂 bit stream                  | 自動產生                         | MAC 產生                           |
| 控制程度         | 完全自訂                         | 高（透過 CLI 控制參數）         | 中（需改參數或 config）           |
| 支援雙向傳輸     | ❌ 單向模擬                       | ❌ 單向模擬                      | ✅ 雙向 socket 傳輸                |
| 測試模擬目標     | BER 模擬與理論比對               | PHY層測試、驗證解碼             | 真實模擬完整 5G 通訊鏈路         |
| 執行速度         | 快                               | 中快                             | 慢（含 UE + gNB 啟動）            |
| 是否能和硬體整合 | ❌ 不行                           | ✅ 可嵌入硬體前測                | ✅ 實驗室可用（軟體模擬射頻）     |

---

## 🎯 實驗設計比較建議

| 實驗目標                        | 最適合模擬器 |
|---------------------------------|---------------|
| 驗證 LDPC + QPSK + AWGN BER     | ✅ MATLAB + ulschsim |
| 比較 DVB-S2 與 3GPP LDPC 效能   | ✅ MATLAB vs ulschsim |
| 驗證完整 UE ↔ gNB 解碼流程     | ✅ rfsim |
| 評估通道模型對錯誤率影響       | ✅ rfsim（AWGN vs fading） |
| 平均疊代數比較                  | ✅ MATLAB vs ulschsim |

---

## 📌 總結：你可以比較的重點

| 比較項目            | MATLAB vs ulschsim | MATLAB vs rfsim | ulschsim vs rfsim |
|---------------------|--------------------|------------------|-------------------|
| BER vs SNR          | ✅ 精準比較         | ✅ 可對齊參數     | ✅ 測試一致性     |
| LDPC 結構與性能     | ✅ 可直接比較       | ✅ 間接比較       | ✅ 同解碼器但流程不同 |
| 解碼疊代收斂速度    | ✅ 可統計           | ❌ 不易取出       | ❌ 需加 debug code |
| 實務通訊行為        | ❌ 不支援           | ✅ 完整連線模擬   | ✅ 符合實體流程    |
