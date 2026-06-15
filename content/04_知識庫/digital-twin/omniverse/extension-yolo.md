---
title: Omniverse YOLO Socket Extension 範例
author: metaarchetech
created: 2026-05-24
updated: 2026-05-24
source: visustwin-vault
source_path: 04 Knowledge/Lived/Omniverse/Extension Yolo.md
tags: [type/howto, status/done, area/digital-twin, area/omniverse, area/computer-vision]
---

# Omniverse YOLO Socket Extension 範例

## 1. 專案總覽

本專案將**多攝影機的即時 YOLO 物件偵測**結果，透過 TCP Socket 傳送到 NVIDIA Omniverse Kit/Composer 的自訂 Extension，實現：

- 圖像即時顯示於 Omniverse UI
- 偵測結果同步更新（類別、信心分數、座標）
- 可跨多台電腦運行，易於擴充及維護

---

## 2. 架構圖

```
[攝影機 RTSP串流]
       │
       ▼
[tapo.py (YOLO Client)]
   └─ YOLOv8 偵測/追蹤
   └─ OpenCV 抓取畫面
   └─ 封包送出:
       - 圖片 (JPEG)
       - 偵測結果 (JSON)
       │
       ▼
[Socket Server (Omni Extension)]
   └─ 接收圖片並快取本機 temp 資料夾
   └─ 解析 detection JSON
   └─ 於 Omniverse UI 顯示圖片/標註資訊
       │
       ▼
[Omniverse Composer/Kit App]
```

---

## 3. 專案檔案結構

```
ov_yolo_socket/
├─ tapo.py                # Client: 負責攝影機偵測/推送資料
├─ ext.yolosocket_ui_extension/
│    ├─ extension.py      # Omniverse extension 主程式 (UI 與 server)
│    └─ socket_server.py  # socket server 實作
└─ requirements.txt       # Python 依賴 (client)
```

---

## 4. 功能與流程

### tapo.py

- 支援多路 RTSP 攝影機 (CameraStream thread)
- 用 YOLOv8 (Ultralytics) 實時偵測物件與追蹤
- 物件資訊封裝成 JSON
- 抓圖並壓縮成 JPEG
- 以 TCP Socket 協定傳送：
    - 影像（`MSG_TYPE_IMAGE`）
    - 偵測結果（`MSG_TYPE_JSON`）

### socket_server.py

- 等待 client 連線
- 收到圖片存為本地暫存檔案
- 收到 detection JSON 直接存於 memory
- 支援回呼（callback）通知 Extension UI

### extension.py

- Omniverse UI 視窗
    - 影像即時刷新（支援手動刷新測試）
    - 偵測結果即時列出 (Label)
- UI 綁定 server 啟動/關閉
- 自動更新 temp 圖片，並處理路徑斜線與 cache 問題
- 支援多 client 並行運作

---

## 5. 安裝與運行

### 依賴安裝（client 端）

```bash
pip install torch ultralytics opencv-python requests
```

### Extension 部署（server/Omni 端）

1. 將 extension 資料夾複製到 Omniverse extensions 目錄
2. 以 Omniverse Kit/Composer 啟動，啟用此 Extension
3. 在 UI 按下「Start Server」
4. 客戶端執行 `python tapo.py`

---

## 6. 注意事項與 troubleshooting

- 圖片 temp 資料夾須可寫入
- socket port (預設 12345) 若有衝突或被防火牆擋，請調整
- 若圖像無法顯示，多半是 temp 路徑權限或 cache 機制失效，可用「手動刷新」測試
- 若 Omniverse 無法啟動 Extension，請檢查 py 路徑、build 結果與版本相容性

---

## 7. 待優化 & 未來工作

- 檢查並優化多 client 支援
- 增加錯誤提示與自動 reconnect
- 圖像延遲/失真優化（壓縮率調整）
- 支援更多 detection 資料結構（如 tracking id、其他 meta）
- 部署到 docker 或 cloud container 易於多場域佈署

---

## 8. 常見問答（FAQ）

- **Q: 其他電腦可以跑嗎？**
    - 只要安裝依賴、設好路徑/權限，完全可行。環境可攜性高。
- **Q: Extension 不顯示圖片怎麼辦？**
    - 確認 temp 路徑存取權、檔案寫入與 UI 路徑斜線相容性。

---

## 相關

- [[extensions]] — Extension 基本概念
- [[xr-integration]] — XR 相關 extension
