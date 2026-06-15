---
title: Feature-3DGS 語意分割模型落地規劃
author: metaarchetech
created: 2026-05-24
updated: 2026-05-24
source: visustwin-vault
source_path: 04 Knowledge/Lived/Gaussian-Splatting/feature-3dgs/語意分割模型（Semantic Segmentation Model）.md
tags: [type/howto, status/draft, area/3d-reconstruction, area/gaussian-splatting, area/ml]
---

# Feature-3DGS 個別物件偵測落地規劃

> 在 Feature-3DGS 上針對 3D 場景中「個別物件」的偵測與分割之技術規劃、步驟、目標。

## 1. 目標

實現針對 3D 場景中「個別物件」的偵測與分割，可用於：

- 物件級渲染（只顯示特定物體）
- 指定區域進行精細訓練／分析
- 將分割結果導出供下游 AI 使用

## 2. 技術流程

### (1) 訓練與重建階段

正常執行 Feature-3DGS 訓練流程，取得高品質 3D Gaussian Splatting 點雲。

### (2) 語意分割流程整合

**SAM 路線（推薦切入點）**：

- 將 3DGS 重建場景以多視角 2D 影像投影
- 用 Meta SAM 對影像做分割，產生多物件 mask
- 反投影（back-project）這些 mask 到點雲，為每個點分配語意 label（可用 segment-anything-3d 等 open source 實現）
- 在 Viewer 加入按鈕：「高亮 / 單獨渲染被 mask 物件」

**LSeg / CLIPSeg 路線（語意詞直接選物件）**：

- 利用 LSeg（或 CLIP + SAM）對每張 2D 圖像根據語意描述生成 mask
- 其他步驟同上

### (3) 互動式選取與自動化

- 擴充 viewer UI：用滑鼠點擊（SAM prompt）或語言輸入（LSeg）
- 自動保存分割結果：JSON mask、per-point label、或導出指定物件的 3DGS 子集

## 3. 所需工具與依賴

- **SAM (Segment Anything)**：Meta 原版 [segment-anything](https://github.com/facebookresearch/segment-anything) + PyTorch
- **segment-anything-3d**（可選）：專為 3D 點雲分割設計的 SAM 延伸
- **CLIP / LSeg**：如需語意 prompt，可使用 [LSeg](https://github.com/isl-org/lang-seg) 或 [CLIPSeg](https://github.com/timojl/clipseg)
- **Feature-3DGS**：主流程
- **SIBR viewer**：可能需調整或補充前端程式

## 4. 預期挑戰與可行性

- **2D-3D mask 映射 / 投影**：SAM 只處理 2D 影像，要精準將 mask 對應到 3D 點，需相機參數與點雲-影像對應（Feature-3DGS 通常可取得）
- **多視角合併與衝突處理**：同一物件在不同視角可能有不同 mask，需融合或一致性處理
- **效能與準確度**：點雲密度高時，分割後渲染與互動速度需優化

## 5. 分階段行動計畫

### 階段一：可行性驗證（PoC）

1. 用 3DGS 渲染輸出 N 張多視角 RGB 影像
2. 對每張影像用 SAM 做 segmentation，產生物件 mask
3. 根據相機內參，將 mask 投影回 3DGS 點雲（先做單一視角測試）
4. 渲染被 mask 標記的點雲，驗證分割結果

### 階段二：功能完善與自動化

5. 合併多視角分割結果，得高可信度 3D label
6. 擴充 Viewer UI，支援高亮／單獨顯示物件
7. 導出分割點雲、JSON label、或局部再訓練

### 階段三：語意分割升級（可選）

8. 整合 LSeg / CLIPSeg，支援「語言輸入」自動選物件

## 6. 技術參考資源

- [segment-anything (官方)](https://github.com/facebookresearch/segment-anything)
- [segment-anything-3d (3D extension)](https://github.com/Pointcept/segment-anything-3d)
- [CLIPSeg: Zero-Shot Segmentation using CLIP](https://github.com/timojl/clipseg)
- [LSeg: Language-driven Semantic Segmentation](https://github.com/isl-org/lang-seg)
- [SIBR Remote Viewer](https://github.com/EPFL-VILAB/SIBR_viewers)

---

## 簡明總結

1. **SAM 路線最容易上手、即時看到分割效果**，建議從這條線驗證
2. **要做語意分割或語言驅動選取，可再導入 LSeg / CLIPSeg**
3. **分割結果能直接幫助在 3DGS 中「對個別物件進行偵測、渲染、訓練或導出」**
4. **若需自動化或批次處理，可用 Agent Framework 把這套分割流程封裝成 tool**

## 相關筆記

- [[development-direction]] — Feature-3DGS 開發方向
- [[sam-limitations]] — SAM 的極限與實際效果
- [[lseg-clip]] — LSeg + CLIP 原理
