---
title: Photogrammetry 知識庫
author: metaarchetech
created: 2026-05-24
updated: 2026-05-24
tags: [type/note, status/draft, area/3d-reconstruction, area/photogrammetry]
---

# Photogrammetry

攝影測量學（Photogrammetry）相關技術筆記。透過多視角影像重建 3D 結構，包括 Structure-from-Motion（SfM）、Multi-View Stereo（MVS）、Dense Reconstruction 等流程，以及商業軟體（RealityCapture、Metashape）的工作流。

## 範圍

- **SfM 與 MVS**：COLMAP、OpenMVG / OpenMVS、Bundle Adjustment
- **商業工具**：RealityCapture、Metashape、3DF Zephyr
- **資料前置**：相機標定、特徵點偵測、影像匹配
- **資料後處理**：點雲清理、紋理映射、Mesh 化、LOD
- **應用**：BIM 既有建築掃描、文資保存、場域數位孿生

## 待整理筆記

本資料夾目前為骨架，尚未有具體筆記。建議優先補上：

- COLMAP 入門與 CLI 使用
- RealityCapture vs Metashape 比較
- 攝影測量 → 3DGS 訓練（用 SfM 的 sparse cloud 當初始化）

## 跨領域連結

- Gaussian Splatting（多視角影像也常用於 3DGS 訓練）：[[../gaussian-splatting/gaussian-splatting|gaussian-splatting/]]
- BIM 整合：[[../../digital-twin/bim/bim|digital-twin/bim/]]
- USD 工作流：[[../../digital-twin/usd/usd|digital-twin/usd/]]
