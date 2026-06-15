---
title: SIBR Remote Gaussian Viewer 操作說明
author: metaarchetech
created: 2026-05-24
updated: 2026-05-24
source: visustwin-vault
source_path: 04 Knowledge/Lived/Gaussian-Splatting/feature-3dgs/SIBR Remote Gaussian Viewer.md
tags: [type/howto, status/done, area/3d-reconstruction, area/gaussian-splatting, area/tools]
---

# SIBR Remote Gaussian Viewer

SIBR Remote Gaussian Viewer 是 3DGS / Feature-3DGS 訓練與互動視覺化的標準 viewer，本檔說明各 UI 與參數。

---

## Point view（左側主視窗）

### Viewer Setting 主選單

- **Run / Pause** — 啟動或暫停訓練／渲染（與主程式狀態同步）
- **Render Mode** — 渲染模式切換：
  - **RGB**：標準彩色影像
  - **Depth**：深度圖（像素到相機距離）
  - **Edge**：邊緣偵測視圖
  - **Normal**：法線視覺化
  - **Curvature**：曲率視圖（表面彎曲程度）
  - **Feature Map**：CNN feature map 可視化
  - **Show Input Points**：顯示輸入點雲

### Visualization & Processing Controls

- **Keep model alive (after training)** — 訓練後是否保留模型於記憶體（建議勾選）
- **Scaling Modifier** — 點雲縮放倍率（一般 1.0）

### Live Performance Metrics

- 顯示即時指標（如目前的點數）

---

## Camera Point View 底部控制區

- **FPS** — 顯示主視窗的 frame rate
- **Mode / Load camera / Save camera (bin)** — 攝影機模式、載入／儲存攝影機設定（bin 格式）
- **Snap to closest / Snap to** — 相機跳到最近的點或特定位置
- **Fov / Far** — 視角（Field of View）與最大可視距離
- **Key cameras / Add key / Save key cameras** — 設定動畫相機路徑（用於動畫錄製）
- **Play (No Interp) / Record / Stop / Speed**
  - Play (No Interp)：沿 key camera 路徑播放（無內插）
  - Record：沿當前路徑錄製相機動畫
  - Stop：停止播放／錄製
  - Speed：動畫播放速度
- **Load path / Save path** — 載入／儲存相機動畫路徑
- **Save video (from playing) / Save frames (from playing)** — 動畫存影片／逐張圖片
- **Acceleration / Rot. speed** — 相機移動加速度／旋轉速度

---

## Top view（右上 + 右下）

### Top view（右上）

- 主視窗顯示目前點雲的俯視圖（xy 平面）
- 用於直觀觀察空間結構與相對佈局

### Top view settings（右下）

- **Save copy (TM)** — 備份當前視圖設定（"TM" 用途待查官方文件）
- **Camera scale / Draw labels / Draw Input Images**
  - Camera scale：攝影機圖示大小
  - Draw labels：是否顯示標籤
  - Draw Input Images：在俯視圖顯示輸入影像縮略圖
- **Trackball / Mode / Load camera / Save camera (bin)**
  - Trackball：開啟軌跡球相機操作
  - Mode：切換顯示模式
  - Load / Save camera：儲存／載入攝影機
- **Snap to closest / Show trackball / Far**
  - Snap to closest：相機對齊最近物件
  - Show trackball：顯示相機操作輔助
  - Far：最遠可視距離
- **Key cameras / Add key / Save key cameras** — 設定相機動畫（同左側）
- **Play (No Interp) / Record / Stop / Speed** — 控制播放／錄影
- **Load path / Save path** — 儲存／載入動畫路徑
- **Save video (from playing) / Save frames (from playing)** — 錄影或存 frame
- **Acceleration / Rot. speed** — 相機動畫加速度／旋轉速率
- **Meshes list / Cameras** — 下拉選單，選擇顯示的 mesh 與相機

---

## 待補充

- **Save copy (TM)**：「TM」的意義待查官方文件
- 進階 Trackball Mode 細項
- 其他 Performance Metrics 指標

## 相關筆記

- [[development-direction]] — Feature-3DGS 開發方向
- [[sam-limitations]] — SAM 在 viewer 中的實際效果
