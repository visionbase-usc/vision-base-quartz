---
title: Omniverse XR/VR Extension 整合
author: metaarchetech
created: 2026-05-24
updated: 2026-05-24
source: visustwin-vault
source_path: 04 Knowledge/Lived/Omniverse/Omniverse XR Integration.md
tags: [type/note, status/done, area/digital-twin, area/omniverse, area/xr]
---

# Omniverse XR/VR Extension 整合

## 1. Omniverse App Kit XR/VR 主要支援與 Extension 一覽

### 核心 XR/VR Extension

#### 1. CloudXR Extension

- **說明**：NVIDIA CloudXR 技術，能將 Omniverse 場景遠端串流到主流 XR 裝置（如 Meta Quest, Apple Vision Pro, Android/iOS 裝置等），實現高品質沉浸式體驗。
- **典型應用**：USD Composer、Machinima、USD Presenter。
- **參考**：[CloudXR 官方文件](https://docs.omniverse.nvidia.com/extensions/latest/ext_cloud-xr.html)

#### 2. omni.kit.xr.profile.vr Extension

- **說明**：提供基於 OpenXR 的 VR 裝置支援，是在 Omniverse Kit/Composer/Isaac Sim 中進行 VR 渲染與互動的核心。
- **重點提醒**：不同 Omniverse 版本、Isaac Sim 版本 API 可能有變動，需特別注意升級時的相容性。
- **參考**：[官方討論串](https://forums.developer.nvidia.com/t/how-to-use-omni-kit-xr-profile-vr-extension-with-isaacsim-4-5/322383)

#### 3. Kit XR Samples

- **說明**：NVIDIA 官方提供的 XR/VR 開發範例集，包含多種常見應用模板與程式碼，方便快速整合與測試 XR 能力。
- **參考**：[kit-xr-samples GitHub](https://github.com/NVIDIA-Omniverse/kit-xr-samples)

---

### 其他相關 Extension

#### 4. omni.add_on.openxr

- **說明**：精簡的 OpenXR Python 綁定，支援如 HTC Vive、Oculus 等主流 VR 裝置。
- **系統限制**：官方目前僅支援 Linux，如為 Windows 需自行嘗試或編譯。
- **參考**：[開發者討論](https://forums.developer.nvidia.com/t/external-extensions-openxr-compact-binding-for-creating-extended-reality-applications/192173)

#### 5. Convai Extension

- **說明**：整合 AI 虛擬角色與語音對話（非 XR 本體，但常與 XR 體驗搭配）。
- **參考**：[Convai Omniverse Extension 文件](https://docs.convai.com/api-docs/plugins-and-integrations/other-integrations/omniverse-extension)

---

## 2. 業界主流 XR/VR 裝置支援現況

| XR/VR 裝置 | OpenXR 支援 | CloudXR 串流 | 典型用途 |
|---|:-:|:-:|:--|
| Meta Quest 系列 | 支援 | 支援 | 工業、娛樂、設計 |
| HTC Vive | 支援 | 部分 | 工業、展示 |
| Valve Index | 支援 | 部分 | 專業/娛樂 |
| Apple Vision Pro | 計畫中 | 支援中 | 創新應用 |
| Android/iOS 手機 | - | 支援 | 手機 AR/VR |

- **重點**：Meta Quest 與 HTC Vive 仍為目前主流選擇，Apple Vision Pro 逐漸崛起。

---

## 3. 實作要點與最佳實踐

1. **啟用方式**：在 Omniverse Composer（或自訂 Kit App）中，透過 Window > Extensions 啟用相關 XR/VR Extension。
2. **OpenXR 設定**：若用 Windows/SteamVR，建議將 SteamVR 設為 OpenXR Runtime。Quest 系列則建議 Virtual Desktop 或 Oculus Link。
3. **API 變更注意**：升級 Omniverse 版本時需特別注意 XR API 的兼容性。
4. **雲端串流**：CloudXR 能極大降低本地端硬體需求，適合高畫質、多人協作場景。
5. **範例學習**：強烈建議參考 kit-xr-samples 之範例作為專案起手式。

---

## 4. 重要官方/社群資源

- [CloudXR Extension 官方文件](https://docs.omniverse.nvidia.com/extensions/latest/ext_cloud-xr.html)
- [kit-xr-samples 官方 GitHub](https://github.com/NVIDIA-Omniverse/kit-xr-samples)
- [VR/XR 技術交流討論串](https://forums.developer.nvidia.com/t/vr-xr-development-in-omniverse/304188)
- [Convai 官方文件](https://docs.convai.com/api-docs/plugins-and-integrations/other-integrations/omniverse-extension)

---

## 相關

- [[xr-design]] — 互動性與雙向通訊設計考量
- [[extensions]] — Extension 基本概念
- [[../3d-reconstruction/xr/xr]] — 3D 重建 × XR
