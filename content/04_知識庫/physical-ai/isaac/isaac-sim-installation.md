---
title: Isaac Sim / Isaac Lab 安裝（Windows）
author: metaarchetech
created: 2026-05-24
updated: 2026-05-24
source: visustwin-vault
source_path: 04 Knowledge/Lived/Omniverse/Isaac Sim.md
tags: [type/howto, status/draft, area/physical-ai, area/isaac]
---

# Isaac Sim / Isaac Lab 安裝（Windows）

參考：[Installation using Isaac Sim pip — Isaac Lab Documentation](https://isaac-sim.github.io/IsaacLab/main/source/setup/installation/pip_installation.html)

## 完整 Isaac Lab 建構步驟（Windows 系統）

### 前置準備檢查

1. 檢查系統是否有 Conda 或 Miniconda
2. 檢查 Python 版本（需要 3.10）
3. 檢查 GPU 驅動程式（NVIDIA）

### 環境設置階段

1. 創建 Isaac Lab conda 環境（使用項目提供的 `environment.yml`）
2. 激活環境

### Isaac Sim 安裝階段

1. 安裝 Isaac Sim 4.5.0（通過 pip）
2. 驗證 Isaac Sim 安裝是否成功

### Isaac Lab 建構階段

1. 使用 `isaaclab.bat` 腳本設置環境
2. 安裝 Isaac Lab 依賴套件
3. 安裝 Isaac Lab 擴展模組

### 驗證階段

1. 運行簡單的演示腳本驗證安裝
2. 測試基本功能（如四軸飛行器演示）

### 第一個應用程式

1. 運行內建的演示應用
2. 了解如何創建自己的應用

---

## 詳細執行命令

```bash
# 1. 檢查環境
conda --version
python --version
nvidia-smi

# 2. 創建環境
conda env create -f environment.yml -n isaaclab

# 3. 激活環境
conda activate isaaclab

# 4. 安裝 Isaac Sim
pip install isaacsim-rl

# 5. 設置 Isaac Lab
./isaaclab.bat --install

# 6. 測試安裝
./isaaclab.bat -p scripts/demos/quadcopter.py
```

---

## 潛在問題預警

- 下載 Isaac Sim 可能需要 5-10 GB 空間和較長時間
- GPU 記憶體不足可能導致某些演示無法運行
- 網路連線問題可能影響套件下載

## 需要確認的問題

1. 系統有 Conda 或 Miniconda 嗎？
2. GPU 是 NVIDIA 的嗎？
3. 完整版本還是精簡版本？
4. 磁碟空間是否足夠（建議至少 20 GB）？

---

## 補充：Isaac Sim 與 Omniverse Composer / Explorer 整合

Composer / Explorer 與 Isaac Sim 同屬 **Omniverse Kit** 底層、共用 **OpenUSD** 資料格式，整合途徑：

| 整合方式 | 說明 | 適用情境 |
|---|---|---|
| **USD 檔案共享** | 透過 Nucleus Server 共用 `.usd / .usda / .usdc` 檔案 | 互相修改場景、Asset 共享 |
| **Live Sync** | Composer / Isaac Sim 間即時同步 | 即時預覽、雙向互動場景 |
| **Extension 插件共享** | 可共享或自訂開發 Extension | 自動化流程、AI/Robot 整合控制 |
| **Action Graph / Python Script** | 將 Sim 邏輯以 USD schema 或 script 匯出 | Digital Twin、可視化模擬過程 |

### 透過 USD 場景同步

```python
omni.usd.get_context().save_as_stage("omniverse://localhost/Projects/my_scene.usd")
```
Composer 開啟此 USD，配合自訂 Extension 重播模擬。

### Live Sync 同步場景變動

- Isaac Sim 支援 Live Collaboration（需打開 `omni.kit.livestream` 等 Extension）
- Composer 也能作為 Live Session 一端
- 適合：一端做模擬（Isaac Sim），一端視覺化與互動設計（Composer）

---

## 補充：Isaac Sim 整合 KUKA 機械手臂

### 整合方式總覽

| 整合方式 | 適用情境 | 所需技術 |
|---|---|---|
| 載入 KUKA USD 模型 | 在 Isaac Sim 中直接操作機械臂 | USD 模型、Articulation 設定 |
| 與 ROS 連動控制 | 用 ROS 控制並模擬實體機械臂 | ROS2 Bridge、MoveIt / KUKA ROS |
| 模擬運動學與軌跡 | 路徑規劃、強化學習、碰撞測試 | Motion Planning、Isaac Gym |
| 實體控制器同步（KUKA Sunrise） | 搭配真實控制器測試 | TCP/IP 接口、自訂中介橋接程式 |

### 載入 KUKA 模型（iiwa / KR 系列）

1. 從 Content 瀏覽器匯入 `.usd / .urdf / .sdf / .fbx`（官方提供 `/Isaac/Robots/KUKA/iiwa.usd`）
2. `Create → Isaac → Robot → Articulation`，指定 root joint
3. Python 控制關節：
   ```python
   from omni.isaac.core import World
   kuka = world.scene.get_object("kuka")
   kuka.get_articulation_controller().apply_action(joint_positions)
   ```

### ROS 連接 KUKA

1. 啟用 `omni.isaac.ros2_bridge`
2. 匯入 ROS 接口描述（URDF）—— 例如 `iiwa_stack`，建議用 `ros2_control` 介面
3. 啟動 ROS 範例：
   ```bash
   ros2 launch iiwa_gazebo iiwa_gazebo.launch.py
   ros2 topic pub /iiwa/command std_msgs/Float64MultiArray "data: [0.5, -0.3, ...]"
   ```

### 控制實體機器人（Sunrise / TCP）

```python
import socket
s = socket.socket()
s.connect(("KUKA_IP", 30002))
s.send(b"MOVEJ 0.1 0.2 0.3 ...")
```

---

## 訓練 vs 實機同步：核心差別

| 項目 | 訓練用途（模擬、AI、RL） | 實機同步（控制真實 KUKA） |
|---|---|---|
| **主要目標** | sim 內任務學習、強化學習、運動學驗證 | 遠端控制、模擬同步、UI 驅動實機動作 |
| **執行平台** | 全部在 Isaac Sim 內 | Isaac Sim + 實體 KUKA + 通訊橋接 |
| **控制對象** | USD 模型（虛擬 KUKA） | 實體 KUKA 機械臂（如 KR3、iiwa） |
| **是否需要硬體** | 不需要 | 需要（KUKA Controller + ROS 或 Socket） |
| **AI 結合程度** | 高（路徑優化、RL、自主控制） | 中（可整合 AI 推理，重點在實機安全與同步） |
| **精準度要求** | 可放寬 | 需非常精準（誤差會導致撞擊／危險） |
| **典型工具** | Isaac Gym / RL Extensions / Motion Planner | ROS Bridge、KUKA Sunrise.OS、EtherCAT、Socket |

### 技術差異

| 技術面 | 模擬訓練 | 實機同步 |
|---|---|---|
| 控制訊號 | 直接控制 USD 模型的 Joint Drive | 轉換成 ROS message / Socket 指令 |
| 物理模擬 | 可開 PhysX 模擬碰撞與重量 | 不建議；控制交由實體硬體與演算法 |
| 速度與效率 | 可同時模擬上百支機械臂加速訓練 | 受實機動作時間與安全規範限制 |
| 故障容忍 | 模擬中出錯可 Reset | 實機出錯可能損壞設備或人員受傷 |

### 安全提醒

實機同步中，**任何來自模擬環境的錯誤訊號，都有可能導致實際設備出現危險行為**。建議：

- 先用 Isaac Sim 完整測試路徑與行為
- 使用 ROS 中的虛擬控制器模擬驗證
- 再逐步部署到 Sunrise.OS 或 KUKA 控制器
