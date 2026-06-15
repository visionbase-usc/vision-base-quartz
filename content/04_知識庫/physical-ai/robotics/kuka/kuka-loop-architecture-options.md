---
title: KUKA Loop 架構選項（通用技術評估）
author: metaarchetech
created: 2026-04-29
updated: 2026-05-24
source: visustwin-vault
source_path: 01 Projects/visionbase/KUKA_Loop_Architecture_Options_v0.md
tags: [type/note, status/draft, area/physical-ai, area/robotics, area/isaac]
---

# KUKA Loop 架構選項（通用技術評估）

> 本檔為通用的「第一個 KUKA + sensor + Isaac + AI loop」架構選項菜單，列出可選技術組合與取捨。具體 lab 的網路、控制機 IP、機型決策不在此檔。

## 0. 文件定位

這是**選項菜單（menu of options）**，不是 lock 方案的決策文件。

實際 lab 需先回答 4 個關鍵問題後才能 lock：
1. KUKA 型號
2. Sensor 選型
3. 第一個情境
4. Sim vs Real

---

## 1. Loop 4 元素抽象架構

不論最後選哪台 KUKA、哪個場景，整個 Physical AI 迴圈都是 4 個 component：

```
[Sensor] → [Perception/AI 判斷] → [Planning] → [Actuation/KUKA]
   ↑                                                  ↓
   └────────────── Feedback loop ──────────────────────┘

   (Isaac Sim 在此扮演 sim 環境 / digital twin layer)
```

### 1.1 Sensor — 感知前端

- **選項**：RGB camera（cheap）、RGB-D（Realsense D435/D455）、LIDAR、Force-Torque 末端、IMU、結構光掃描
- **典型起手**：Realsense D455 + Jetson Orin 是社群最主流入門組

### 1.2 Perception / AI 判斷 — 「看懂」層

- **選項**：YOLO（物件偵測）、SAM 2（分割）、Grounding DINO（語意指向）、Foundation Pose、VLA 模型（RT-2、OpenVLA）、傳統 CV（OpenCV + ArUco）
- **典型起手**：YOLO + Foundation Pose 已能應付大多數 pick task；要走 LLM-based reasoning 才需 VLA

### 1.3 Planning — 路徑規劃

- **選項**：MoveIt 2（ROS）、cuMotion（NVIDIA GPU 加速）、Isaac Manipulator、KUKA.PRC、KUKA 內建 PTP/LIN
- **典型起手**：Pick & Place 走 Isaac Manipulator 內建 cuMotion；複雜建築 toolpath 走 KUKA.PRC
- **組合策略**：PRC 出粗 path → cuMotion 做避障 fine-tune

### 1.4 Actuation — 機器人執行層

- **選項**：KRL、RSI（Robot Sensor Interface, real-time）、FRI（Fast Research Interface, iiwa 才有）、EKI-XML（網路通訊, KRC4/5）、ROS 2 driver
- **典型起手**：KRC4/5 走 EKI-XML 從 PC 送指令；iiwa 走 FRI

---

## 2. KUKA 型號家族對應

不同 KUKA 機型，「最低成本接到 Isaac」的路徑差異很大。

| 系列 | 範例型號 | Controller | Isaac 整合方式 | 上手友善度 | 備註 |
|---|---|---|---|---|---|
| KR Agilus / Cybertech | KR3 R540, KR6 R900, KR16 R2010 | KRC4/KRC5 | EKI-XML + Python bridge | KRL 為主 | lab/小空間最常見 |
| KR Quantec | KR60-120, KR210 | KRC4/KRC5 | EKI-XML / RSI | KRL 為主 | 工業大型，建築場景對應 |
| LBR iiwa | iiwa 7/14 R820 | Sunrise Cabinet | **FRI 直接吃** + Isaac SDK | Java/Sunrise | cobot，力控強 |
| LBR iisy | iisy 3/11/15 | Sunrise OS（新） | 雲端 / Sunrise OS | 文件少 | 2024 後新 cobot |
| KR.AI（NVIDIA 合作） | Agilus AI 預載 Jetson | KRC5 + Jetson | **官方支援，最簡** | 取得性未知 | 2024 公布，可能難買 |

### 機型 trade-off：

- **KR Agilus + KRC4/5** → 走 EKI-XML，KRL 工程師上手快，Isaac 整合中等
- **iiwa** → FRI 真實時控制，是「Isaac 原生支援」最深的，但需要 Sunrise/Java
- **KR.AI** → 路徑最短但實際取得性 / 預算可能擋路

---

## 3. Isaac 棧三層解析

NVIDIA 的「Isaac」家族不是單一產品，是一組工具：

| 元件 | 用途 | First loop 是否需要 |
|---|---|---|
| **Isaac Sim** | Omniverse-based 3D 模擬，吃 USD/URDF；KUKA URDF 可直接導入 | **基本盤** |
| **Isaac Lab** | RL 訓練框架，跑在 Isaac Sim 之上 | 只有要 train policy 才需要 |
| **Isaac ROS** | Jetson/edge 部署用的 perception + manipulation ROS 2 packages | Real-time 部署需要 |
| **Isaac Manipulator** | Reference workflow，主要 cover Pick & Place | **First loop 起點** |
| **Isaac Cosmos** | World foundation models，給 sim 生成更真實的視覺/物理 | 進階，v1 不必碰 |

**First loop 推薦組合（共識基線）**：

- 開發階段：**Isaac Sim + Isaac Manipulator**（USD + cuMotion + Foundation Pose）
- 部署階段：**Isaac ROS** on Jetson + 把 Sim 訓練好的 perception 模型搬上去
- v1 不碰 Lab / Cosmos

---

## 4. 第一個 loop 場景 — 5 個典型選項對比

| # | 場景 | 動作描述 | 技術難度 | Demo 殺傷力 | 預估時程 |
|---|---|---|---|---|---|
| 1 | **Pick & Place 積木** | 視覺辨識 → 抓 → 放定點 | 低 | 中 | 3-4 週 |
| 2 | **跟隨人手** | 持續追蹤手部 → 末端跟隨 | 低-中 | 中 | 3-4 週 |
| 3 | **看圖建實體模型** | 讀 BIM 局部 → 用磚塊堆疊還原 | 中 | 高 | 6-8 週 |
| 4 | **缺陷標記** | 偵測牆面缺陷 → 末端噴頭標位置 | 中-高 | 極高 | 8-12 週 |
| 5 | **BIM 對齊驗證** | 真實場景 vs BIM → 標差異 | 高 | 極高 | 10-14 週 |

### 4.1 Pick & Place 積木

- Sensor: 單支 RGB-D 已足
- AI: YOLO + Foundation Pose
- 失敗模式: 抓不穩、漏判方向。可控
- 評語: 第一週熱身好題目，demo 不強

### 4.2 跟隨人手

- Sensor: RGB-D
- AI: MediaPipe Hand 或 RT-DETR
- 失敗模式: 速度跟不上、安全邊界
- 評語: cobot 展示（iiwa/iisy 適合），建築場景連接弱

### 4.3 看圖建實體模型

- Sensor: RGB camera + 機器人 TCP 已知座標
- AI: 影像分類（磚塊顏色）+ Grasshopper/PRC 算 toolpath
- 失敗模式: 累積誤差、堆疊穩定性
- 評語: 與 BIM/PRC 經驗最對胃口，視覺加分多

### 4.4 缺陷標記

- Sensor: RGB 高解析 + 距離計
- AI: 缺陷偵測模型（YOLO + 微調）或 anomaly detection
- 失敗模式: 缺陷定義模糊、模型 false positive
- 評語: 建築工地真實 use case；風險是需採缺陷資料 train

### 4.5 BIM 對齊驗證

- Sensor: LIDAR 或 RGB-D 點雲
- AI: 點雲對齊（ICP）+ 差異偵測
- 失敗模式: 對齊飄、coverage 不全
- 評語: Digital Twin 核心戰略最對應，但工程量大，v1 太重

---

## 5. Sim-first vs Real-first 真實 trade-off

| 維度 | Sim-first | Real-first | Hybrid |
|---|---|---|---|
| 起手成本 | 高（學 Isaac Sim） | 低（現成設備） | 最高 |
| 迭代速度 | 高（不用實機 reset） | 中 | 高 |
| 出錯成本 | 零（虛擬撞） | 高（實機 / 人員） | 低 |
| Sim-to-real gap | 永遠存在 | 無 | 由 real fine-tune 補 |
| 學習價值 | 高（NVIDIA 主流） | 低（若已會 KRL） | 高 |
| Demo 殺傷力 | 中（螢幕 demo） | **高**（真機在動） | **極高** |
| v1 工期 | 3-4 週純 sim | 4-6 週純 real | 8-12 週 |
| 後續 scale 能力 | **高**（sim 可複製） | 低 | **高** |

### 三種路線的 use case：

- **Sim-first**：適合先建立 Isaac 棧掌握度的 demo（給技術觀眾）
- **Real-first**：實機展示效果最強，但 v1 學到的東西難複製到第二台機器
- **Hybrid（Sim 跑通邏輯 + Real fine-tune）**：工程最佳，成本最高。v1 起來最慢，但 v2 之後會補回來

---

## 6. 決策必答清單

依重要性排序：

### 必答 5 題

1. **KUKA 型號 + Controller + 目前用的 SDK？**
   直接決定 §2 走哪一列、§1.4 actuation 用 KRL/RSI/FRI/EKI 哪個。

2. **5 個場景中，第一個 loop 要選哪個（或變形）？**
   決策因素 weight：demo 殺傷力 vs 學習曲線 vs 6-8 週可達性。

3. **Sim-first / Real-first / Hybrid？**
   與 demo 對象有關：給誰看？什麼時候要？

4. **Hardware 預算 ceiling？**
   Realsense（$300）vs 工業 LIDAR（$5K+）vs 力控末端（$10K+）差異極大。

5. **第一個 demo 目標時點 + 對象？**

### 次要 3 題

6. 執行人是全職還是 part-time？ — 直接乘除時程
7. Lab 空間：KUKA 在哪？電源、安全圍欄、網路？ — 影響 real-first 可行性
8. 是否要為 v1 預留 v2 scale path？ — 影響是否強推 hybrid

---

## 附錄 A：Sensible default 路徑

> **預設組合**：KR Agilus（KRC4/5）+ Realsense D455 + 場景 #3「看圖建實體模型」+ Hybrid（Sim 跑邏輯 → Real fine-tune）

理由：
- 用上 KUKA.PRC + Grasshopper 既有強牌
- BIM → 實體的故事直接接 Digital Twin
- 6-8 週可看到實體 demo
- Sim 階段建立 Isaac 經驗，real 階段給殺傷力
- 不需採購工業級 LIDAR / 力控末端，預算可控

這是 sensible default，不是推薦。等前 5 題回完再 lock。

---

## 附錄 B：典型技能 vs 要補

對於以 KUKA.PRC + Grasshopper + KRL 為背景的執行者：

| 元件 | 既有 | First loop 要補什麼 |
|---|---|---|
| KUKA.PRC | 熟練 | 與 Isaac path 整合（USD ↔ Rhino） |
| Grasshopper | 熟練 | 接點雲 → 幾何 pipeline |
| KRL | 熟練 | EKI-XML / RSI 從 PC 端送指令 |
| Python | 視個人 | Isaac SDK / OpenCV / YOLO 推理 |
| ROS 2 | 視個人 | 部署階段需要（Isaac ROS） |
| Isaac Sim | 新 | USD scene、URDF import、cuMotion |
| AI 模型微調 | 新 | （場景 #4 才需要） |

—— 文件結束 ——
