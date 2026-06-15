---
title: Linux Lab 環境建置計畫
author: metaarchetech
created: 2026-05-20
updated: 2026-05-24
source: visustwin-vault
source_path: 01 Projects/Physical_AI/Linux_Lab_Plan.md
tags: [type/howto, status/draft, area/physical-ai, area/robotics, area/isaac]
---

# Linux Lab 環境建置計畫

> 本檔聚焦軟體架構 + 作業系統 setup checklist。硬體規格與預算不在本檔範圍。

## 0. 文件定位

這是 lab Linux 機 **Phase 0（環境就緒）** 建置計畫。Phase 0 完成 = lab 機具備跑通第一個 KUKA loop 所需的全部軟體基礎。本檔只到環境就緒，第一個 loop 的場景選擇 / sim-vs-real 路線屬 [[kuka-loop-architecture-options]] 的決策範圍。

---

## 1. 硬體規格

⏳ **待補**。硬體規格與硬體架構由現場後續提供；本檔不臆測機型、GPU、記憶體、網路拓撲。

收到硬體架構後需回頭校驗的軟體決策：
- GPU 世代 → NVIDIA driver 分支 + CUDA 版本 + Isaac Sim 最低需求是否滿足
- 是否多機（工作站 + Jetson edge）→ 是否需要 §2.3 的 Isaac ROS / Jetson 部署支線
- 網路是否與 KRC 控制器同網段 → 影響 §2.5 KUKA 介接層的部署位置

---

## 2. 軟體架構（baseline，actionable）

### 2.1 作業系統選擇

| 選項 | 配套 | 生態成熟度 | 建議 |
|---|---|---|---|
| **Ubuntu 22.04 LTS (Jammy)** | ROS2 Humble + Isaac Sim 4.x | Isaac Sim / ROS2 / KUKA ROS driver 主流驗證組合，社群文件最厚 | baseline |
| Ubuntu 24.04 LTS (Noble) | ROS2 Jazzy | 生態剛跟上，Isaac Sim 對 24.04 支援陸續到位但社群踩雷紀錄仍少 | 暫不用；待 24.04 + Jazzy 生態與 22.04 同厚再評估遷移 |

**建議理由**：第一個 loop 的目標是「最短路徑跑通 sense→reason→act」，不是追新。22.04 + Humble + Isaac Sim 4.x 是 NVIDIA 官方範例、KUKA ROS2 driver、社群教學三方都對齊的組合，踩雷成本最低。

> baseline：**Ubuntu 22.04 LTS**

### 2.2 ROS2 版本

- baseline：**ROS2 Humble Hawksbill**（綁 22.04，LTS 支援至 2027-05，與 KUKA 社群 driver 對齊）
- 24.04 對應為 Jazzy Jalisco —— 與 §2.1 一起延後

### 2.3 NVIDIA stack

由底到上：

| 層 | baseline 選擇 | 備註 |
|---|---|---|
| GPU Driver | proprietary 分支（依 GPU 世代，收到硬體後鎖版本） | ⏳ 待硬體；不要用 nouveau |
| CUDA | CUDA 12.x（對齊 Isaac Sim 4.x 需求） | 隨 driver 一併鎖 |
| Isaac Sim | **Isaac Sim 4.x**（Omniverse Kit-based 3D 物理模擬，吃 USD/URDF） | First loop 基本盤 |
| Omniverse Kit | 隨 Isaac Sim 帶；USD Composer 視需要另裝 | scene 編輯 / USD 檢視 |
| Isaac Lab | 視需要（只有要 train RL policy 才裝） | v1 不必碰 |
| Isaac ROS | 視需要（Jetson/edge 即時部署才裝） | ⏳ 視硬體是否含 Jetson |

開發階段 Isaac Sim + Isaac Manipulator，v1 不碰 Lab / Cosmos（詳見 [[kuka-loop-architecture-options]]）。

### 2.4 USD 工具鏈

- **OpenUSD**（usd-core / usdview）—— USD 檢視與除錯
- **Omniverse USD Composer**（選裝）—— scene 組裝
- Rhino / Grasshopper（KUKA.PRC）出 path → USD 互通屬 loop 設計細節，不在 Phase 0 範圍

### 2.5 KUKA 介接層

依既有 protocol 分析，正常監控與外部控制都可自寫，不需破任何安全機制。Linux 機這側要準備的介接元件：

| 介面 | 用途 | License | Linux 機這側準備 |
|---|---|---|---|
| **EKI (EthernetKRL)** | 可讀可寫，主控制路徑 | KSS 8.x+ 內建 | socket client（Python / ROS2 node）+ XML 協定約定 |
| KUKAVARPROXY | 讀寫 KRL 變數，做 dashboard | 免費（社群） | TCP `7000` client；KRC 端需有人放 .exe |
| OPC UA | 標準監控，接 Grafana/Node-RED | KRC5 標配 / KRC4 加價 | OPC UA client |
| RSI | 4ms 即時反饋控制 | **付費** | 單純監控用不到，v1 不碰 |
| ROS2 driver | ROS2 生態整合（kuka_eki / kroshu kuka_drivers） | 開源 | ROS2 Humble 套件，走 EKI-XML 底層 |

> KUKA Office Lite（KRC 模擬器）是 Windows 程式，不在 Linux 機上跑。Phase 2 離線模擬若要用 Office Lite，需另一台 Windows 機 / VM；Linux 機這側用 Isaac Sim 做物理模擬。

### 2.6 開發工具

- VS Code + Cursor（IDE）
- Git
- **Docker + NVIDIA Container Toolkit**（GPU 容器；Isaac ROS / 隔離環境）
- Python 3.10（對齊 Humble / Isaac Sim 4.x）+ conda 或 mamba 管環境

---

## 3. 作業系統 setup checklist（Phase 0a–0e）

> 每一 Phase 完成才進下一 Phase。指令為 baseline 骨架；版本號待 §1 硬體確認後鎖定。

### Phase 0a — Ubuntu 安裝 + GPU driver

- [ ] 安裝 Ubuntu 22.04 LTS（Desktop；lab 機需 GUI 跑 Isaac Sim viewport）
- [ ] 系統更新：`sudo apt update && sudo apt full-upgrade`
- [ ] 安裝 proprietary GPU driver（**版本待 §1 硬體鎖定**）：`ubuntu-drivers devices` → `sudo ubuntu-drivers install` 或鎖定版本 `sudo apt install nvidia-driver-<ver>`
- [ ] **驗證**：重開機後 `nvidia-smi` 顯示 GPU 與 driver 版本

### Phase 0b — NVIDIA 運算棧（CUDA + Container Toolkit）

- [ ] 安裝 CUDA 12.x toolkit（依 driver 對應版本）
- [ ] 安裝 Docker + NVIDIA Container Toolkit
- [ ] **驗證**：`nvcc --version` 正常；`docker run --rm --gpus all nvidia/cuda:12.x-base nvidia-smi` 容器內看得到 GPU
- 文件：developer.nvidia.com/cuda-downloads；docs.nvidia.com/datacenter/cloud-native/container-toolkit

### Phase 0c — ROS2 Humble + 開發工具

- [ ] 安裝 ROS2 Humble（`ros-humble-desktop`）
- [ ] `source /opt/ros/humble/setup.bash` 寫入 shell rc；建 colcon workspace
- [ ] 安裝 VS Code / Cursor / Git；conda 或 mamba
- [ ] **驗證**：`ros2 run demo_nodes_cpp talker` ↔ `listener` 通；`colcon build` 空 workspace 成功
- 文件：docs.ros.org/en/humble/Installation.html

### Phase 0d — Isaac Sim / Omniverse

- [ ] 安裝 Isaac Sim 4.x（Omniverse Launcher 或容器版）
- [ ] 確認 USD 工具（usdview / usd-core）可開
- [ ] **驗證**：Isaac Sim 開得起，載入內建 KUKA URDF / 範例 scene 不報 GPU 錯
- 文件：docs.isaacsim.omniverse.nvidia.com

### Phase 0e — KUKA driver / 介接層

- [ ] 安裝 / 編譯 KUKA ROS2 driver（kroshu `kuka_drivers` 或 `kuka_eki_hw_interface`，依 §1 KRC 型號）
- [ ] 準備 EKI socket client 骨架 + XML 協定草案
- [ ] ⏳ KRC 端前置（KSS 版本 / 網段 / KCP 鑰匙）—— **lab 現場確認，非 Linux 機軟體任務**
- [ ] **驗證**：Linux 機對 KUKA Office Lite 或 Isaac Sim 模擬端跑通一次 EKI 收發（真機收發屬安全紀律 Phase 3+，見 §5）

---

## 4. KUKA 介接協定整理（baseline）

- 監控（讀）：KUKAVARPROXY（免費，TCP 7000）/ EKI / OPC UA / RSI（付費）
- 控制（動）：EKI + KRL，4 個前置缺一不可（EXT 模式、KRL loop、訊號協定、safety 不繞）
- 不能繞：Safety controller / E-stop / 物理鑰匙 / license-gated 介面 —— 法規 + 設計，非技術障礙

## 5. 5-Phase 安全紀律（baseline，不可協商）

八十萬等級設備，LLM **不主導物理動作流程**：

1. **Phase 1 Mock**（離線）— LLM 全程主導 OK
2. **Phase 2 模擬器**（離線）— Isaac Sim / KUKA Office Lite，LLM 互動 OK
3. **Phase 3 T1 真機**（慢速 250mm/s）— 改動由人類審 + 鍵入
4. **Phase 4 T2 真機**（全速，工程師現場）— LLM 純諮詢
5. **Phase 5 EXT**（無人值守）— 絕不在 vibe coding session 切換，人類工程師依 SOP 完成

> 本檔的 Phase 0e「驗證」一律對模擬端做；真機收發歸上述 Phase 3+。

## 6. Digital Twin 接口

- act 端閉環的數位語言是 BIM → USD ontology —— 與 BIM/DT 工作共用同一條 USD 場景圖
- lab Linux 機產出的 USD scene / KUKA URDF 應可被 Omniverse pipeline 消費，反之 BIM→USD 輸出可餵 Isaac Sim
- Phase 0 不需實作此接口，但 USD 工具鏈（§2.4）選型須保留兩邊互通性

---

## 7. 標記分布總表

| 類別 | 涵蓋段 |
|---|---|
| **baseline（可直接執行）** | §2.1 Ubuntu 22.04 / §2.2 Humble / §2.3 NVIDIA stack 結構 / §2.4 USD / §2.5 KUKA 介接層 / §2.6 工具 / §3 Phase 0a–0e checklist / §4 協定 / §5 安全紀律 / §6 雙軌接口原則 |
| **開放問題（待 loop 決策，不在本檔）** | 第一個 loop 場景、sim-vs-real 路線、sensor 選型 → 全在 [[kuka-loop-architecture-options]]，本檔不複製 |
| **等補** | §1 硬體規格/架構 / §2.3 driver+CUDA 版本鎖定 / §2.3 是否含 Jetson(Isaac ROS) / §3 Phase 0e KRC 現場前置 |

—— 文件結束 ——
