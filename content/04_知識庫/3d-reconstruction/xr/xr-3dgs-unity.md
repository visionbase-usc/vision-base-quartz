---
title: XR + 3DGS Unity 整合
author: metaarchetech
created: 2026-05-24
updated: 2026-05-24
source: visustwin-vault
source_path: 04 Knowledge/Lived/XR/XR + 3DGS Unity.md
tags: [type/howto, status/done, area/3d-reconstruction, area/gaussian-splatting, area/xr, area/tools]
---

# XR + 3DGS Unity 整合

主要參考：[UnityGaussianSplatting](https://github.com/aras-p/UnityGaussianSplatting)（by Aras Pranckevičius）

## 概述

SIGGRAPH 2023 的論文 ["3D Gaussian Splatting for Real-Time Radiance Field Rendering"](https://repo-sam.inria.fr/fungraph/3d-gaussian-splatting/)（Kerbl 等）發表後，Aras 實作了即時可視化部分（讀取已產出的 splat model 檔）在 Unity 中。

本 repo 基於原始 paper；2023 底以後爆量出現的新 GS 研究均**不在**此專案範圍內。

> Status（2023-12）：作者未計畫進一步重大開發。

## 平台支援

僅在以下圖形 API 平台已知能運作：

- **PC**：Windows（D3D12 或 Vulkan）
- **Mac**：Metal
- **Linux**：Vulkan

注意事項：

- 部分 VR 裝置可用（HTC Vive、Varjo Aero、Quest 3、Quest Pro）；Apple Vision Pro 等可能不行（見 [issue #17](https://github.com/aras-p/UnityGaussianSplatting/issues/17)）
- OpenGL / OpenGL ES：不支援（[issue #26](https://github.com/aras-p/UnityGaussianSplatting/issues/26)）
- WebGPU：目前還缺少必要圖形特性（[issue #65](https://github.com/aras-p/UnityGaussianSplatting/issues/65)）
- 行動裝置：部分 iOS / Android 裝置無法運作（[#72](https://github.com/aras-p/UnityGaussianSplatting/issues/72)、[#112](https://github.com/aras-p/UnityGaussianSplatting/issues/112)）

## 使用方法

1. **clone 或下載 repo**，用 Unity 2022.3 開啟 `projects/GaussianExample`
2. 開啟 `GSTestScene` 場景
3. 必須使用 **DX12 或 Vulkan** — DX11 無法運作；行動／網頁未測試
4. **建立 GaussianSplat 資產**：選單 `Tools → Gaussian Splats → Create GaussianSplatAsset`
   - 將 `Input PLY/SPZ File` 指向你的 Gaussian Splat 檔案
   - 目前支援兩種格式：
     - **PLY**（原始 3DGS paper 格式；官方模型放在 `point_cloud/iteration_*/point_cloud.ply`）
     - **Scaniverse SPZ** 格式（<https://scaniverse.com/spz>）
   - 可選同一目錄或父目錄放 `cameras.json`
5. 選擇壓縮選項與輸出資料夾，按 **Create Asset**
   - 即使 "Very Low" 壓縮品質都有不錯的可用性，例如可在 8 MB 以下完成捕捉
6. 在 `GaussianSplatRenderer` 腳本的 GameObject 上，把 **Asset 欄位指向**建立好的資產
7. 腳本上提供多種除錯／視覺化控制，以及將遊戲相機移到資產相機位置的滑桿

### 物件變換

渲染會考慮 GameObject 的 transform matrix。官方 GS 模型大多會繞 X 軸旋轉約 -160 度並沿 Z 軸鏡像，所以範例場景的物件 transform 已預設好這個變換。

## 模型來源

GS 模型體積大，repo 不含模型檔。原始 [paper GitHub](https://github.com/graphdeco-inria/gaussian-splatting) 提供 [14 GB zip](https://repo-sam.inria.fr/fungraph/3d-gaussian-splatting/datasets/pretrained/models.zip) 的預訓練模型。

## 延伸文件

- [Render Pipeline Integration](https://github.com/aras-p/UnityGaussianSplatting/blob/main/docs/render-pipeline-integration.md)
- [Editing Splats](https://github.com/aras-p/UnityGaussianSplatting/blob/main/docs/splat-editing.md)

## 相關筆記

- [[depthkit]] — Depthkit 體積擷取
- [[../gaussian-splatting/gaussian-splatting|gaussian-splatting/]] — Gaussian Splatting MOC
- [[xr|xr/]] — XR MOC
