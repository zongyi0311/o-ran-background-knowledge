[rifsim simulator.c程式碼](#rifsim-simulator.c程式碼)
[OAI 專案資料夾結構說明](#OAI-專案資料夾結構說明)



## OAI 專案資料夾結構說明

| 資料夾路徑              | 說明 |
|-------------------------|------|
| `openair1/`             | Layer 1（PHY 層）實作，包含 LDPC encoder/decoder（例如 `nrLDPC_encoder.c`、`ldpc_decoder.c` 等） |
| `radio/rfsimulator/`    | RFSIM 模擬器實作，負責 gNB 與 UE 之間的模擬通道傳輸，主要檔案如 `simulator.c` |
| `cmake_targets/`        | 編譯與執行相關腳本與設定，執行 softmodem 時會進入這個目錄 |
| `executables/`          | gNB / UE 等程式的 main 函式與進入點 |
| `openair2/`             | Layer 2（MAC / RLC / PDCP / RRC）模組 |
| `openair3/`             | Layer 3（核心網路協定，例如 NGAP / GTP），與 LDPC 無直接關係 |
| `CMakeLists.txt`        | 專案頂層建置指令入口 |
| `doc/`                  | 說明文件與架構簡介（可找系統圖或流程） |
| `tools/`                | 程式碼格式化、分析等開發工具 |
| `docker/`               | Docker 建置相關檔案與設定 |
| `ci-scripts/`           | 自動測試與 CI 流程相關的腳本與設定檔 |

---


[rifsim simulator.c程式碼](#rifsim-simulator.c程式碼)

#rifsim simulator.c程式碼
