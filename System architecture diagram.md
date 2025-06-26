# 📘 LDPC 模擬對照筆記：MATLAB 自建 LDPC vs OAI 標準 LDPC 架構

---

## 🎯 模擬目的

- 比較 MATLAB 自建 LDPC + QPSK 模擬系統 與 OAI NR 系統中的標準 LDPC 架構
- 評估在相同 SNR、疊代數條件下的 BER 表現差異
- 確認兩者架構流程、模組分工與可比性

---

## 📐 系統模擬流程對照圖

| 模組階段 | MATLAB 自建模擬系統 | OAI 系統 (nr-softmodem + rfsimulator) | 對比觀察重點 |
|----------|----------------------|----------------------------------------|----------------|
| Bit 產生 | `randi([0 1], k, 1)` | MAC 層或 PHY 自產 | 資料來源是否一致（隨機 or 規則） |
| LDPC 編碼器 | 自建 G = [I P']，`mod(msg * G, 2)` | `nr_ldpc_encoder()` 使用 BaseGraph1/2 | 使用標準 vs 自建碼；補零與碼長一致性 |
| 調變器 | QPSK, π/4 rotated, Gray Mapping | `nr_modulation()` 控制調變類型與 Qm | 是否一致（如是否 Gray coding） |
| 通道模型 | `AWGN` 通道 | `rfsimulator` 中設 `modelname = "AWGN"` | 確保同樣的 noise variance 計算方式 |
| LLR 計算 | 根據實/虛部手算 LLR | `nr_rx_llr()` | 確認 noise_var 定義一致 |
| 解碼器 | Belief Propagation, Tanner Graph | `nr_ldpc_decoder()`，可設 max_iter | 是否一致的算法與疊代上限設定 |
| 解碼停止條件 | syndrome 是否為 0 | BP or Min-Sum，early termination | 是否使用 early stopping 與 log 條件一致 |
| 錯誤統計 | `sum(msg ~= est_msg)` | 須手動加 log 比對 bit | 相同 BER 評估準則與數據取樣規模 |

---

## 🧠 關鍵觀察與調整建議

| 項目 | 說明與建議 |
|------|------------|
| 碼率一致性 | MATLAB 使用 k/n = 64/128，OAI 須選擇 BaseGraph2 並調參數使得碼率 ~1/2 |
| 疊代次數 | 將 OAI 內設定設為 `nrUE->max_ldpc_iterations = 75` 與 MATLAB 對齊 |
| 比特來源 | 可考慮讓 OAI 使用隨機 data pattern 或記錄 UE 接收前後 bit 並儲存作 BER 計算 |
| 通道設定 | 確保 rfsimulator 的 model 設定為 `AWGN`，並對應 noise_variance 與 MATLAB 一致 |

---

## 🗂 OAI 中需觀察與修改的函式位置

| 模組 | 路徑 | 功能說明 |
|------|------|-----------|
| LDPC Encoder | `openair1/PHY/NR_TRANSPORT/nr_ldpc_encoder.c` | 編碼輸出位元流處理 |
| LDPC Decoder | `openair1/PHY/NR_TRANSPORT/nr_ldpc_decoder.c` | 可看 BP 與 Min-Sum 解碼流程、early stop 條件 |
| 調變器 | `openair1/PHY/NR_TRANSPORT/nr_modulation.c` | 調變器與符號產生 (modulation mapping) |
| LLR 解調 | `openair1/PHY/NR_UE_TRANSPORT/nr_rx_llr.c` | IQ → LLR 計算流程 |
| 通道 | `radio/rfsimulator/simulator.c` | 設定 `modelname = "AWGN"` 並調整雜訊參數 SNR |
| 疊代控制參數 | `openair1/PHY_INTERFACE/phy_interface_vars.h` | `nrUE->max_ldpc_iterations` 的設定位置 |

---

## 📌 結語與應用

本比較架構適用於：
- 確認你 MATLAB 模擬環境是否與 OAI 設定一致
- 評估自建 LDPC 架構與標準 LDPC 的性能差異
- 後續可用於做 AI LDPC 解碼器或調參測試的基準比較平台

> ✅ 建議搭配 MATLAB plot 與 OAI UE 解碼 log 實際結果進行 BER 圖疊圖比較
