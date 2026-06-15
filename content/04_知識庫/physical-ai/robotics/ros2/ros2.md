---
title: ROS2 / Nav2 / MoveIt
author: metaarchetech
created: 2026-05-24
tags: [type/note, status/draft, area/physical-ai, area/robotics, area/ros2]
---

# physical-ai / robotics / ros2

ROS2 生態相關筆記：版本、套件、與 Isaac / KUKA 的接點。

## 範圍

- **ROS2 distros**：Humble（22.04 LTS，baseline）、Jazzy（24.04 LTS，未來）
- **Nav2**：移動機器人導航 stack
- **MoveIt 2**：機械手臂路徑規劃
- **ros2_control**：硬體抽象層
- **KUKA ROS2 driver**：kroshu `kuka_drivers`、`kuka_eki_hw_interface`
- **Isaac ROS 2 Bridge**：`omni.isaac.ros2_bridge`

## 目前檔案

（尚未有專屬筆記；ROS2 相關目前散在 [[../../linux-lab-setup-plan]] §2.2、§2.5 與 [[../kuka/kuka-loop-architecture-options]] §1.3 §1.4）

## 待補

- [ ] ROS2 Humble 安裝與 workspace 結構
- [ ] MoveIt 2 + KUKA 整合範例
- [ ] Isaac Sim ↔ ROS2 Bridge 的 topic / service 對應
- [ ] Nav2 設定（如果有移動載具）

## 相關

- 上一層：[[../../..|physical-ai/]]
- KUKA 端：[[../kuka/kuka|robotics/kuka/]]
- Isaac 端：[[../../isaac/isaac|isaac/]]
