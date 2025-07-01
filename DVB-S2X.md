整理自[EN 302 307-2 - V1.4.1 - Digital Video Broadcasting (DVB)](https://www.etsi.org/deliver/etsi_en/302300_302399/30230702/01.04.01_60/en_30230702v010401p.pdf) and [EN 302 307 - V1.3.1 - Digital Video Broadcasting (DVB)](https://www.etsi.org/deliver/etsi_en/302300_302399/302307/01.03.01_20/en_302307v010301a.pdf)

# 📘 DVB-S2X Introduction(page 9)

## S2X 擴充說明
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
