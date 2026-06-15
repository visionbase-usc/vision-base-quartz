---
title: Feature-3DGS 知識庫
author: metaarchetech
created: 2026-05-24
updated: 2026-05-24
tags: [type/note, status/done, area/3d-reconstruction, area/gaussian-splatting]
---

# Feature-3DGS

Feature-3DGS 為 3D Gaussian Splatting 的延伸，在每個 Gaussian 上掛載語意特徵（feature embedding），使 3DGS 場景可進行語意分割、語言驅動編輯、物件偵測等下游任務。

本資料夾包含 Feature-3DGS 開發思路、相關模型（LSeg、CLIP、SAM）的技術筆記與整合計畫。

## 本資料夾內容

| 筆記 | 內容 |
|---|---|
| [[development-direction]] | Feature-3DGS 系統開發方向、可操作階段、產出檔案總覽 |
| [[lseg-clip]] | LSeg + CLIP 原理對照 |
| [[sam-limitations]] | SAM 的極限與實際效果（為何需要 LSeg） |
| [[semantic-segmentation-models]] | 個別物件偵測落地規劃（SAM / LSeg / CLIPSeg） |
| [[sibr-remote-viewer]] | SIBR Remote Gaussian Viewer 操作說明 |
| [[lseg-unicode-fix-report]] | Windows 上 LSeg Unicode 編碼問題的完整除錯報告 |

## 相關資源

- [Feature-3DGS 原始 paper](https://feature-3dgs.github.io/)
- [SAM (Segment Anything)](https://github.com/facebookresearch/segment-anything)
- [LSeg](https://github.com/isl-org/lang-seg)
- [CLIPSeg](https://github.com/timojl/clipseg)

## 跨領域連結

- 上層 MOC：[[../..|gaussian-splatting/]]
- 對應 XR 整合：[[../../xr/xr|xr/]]
