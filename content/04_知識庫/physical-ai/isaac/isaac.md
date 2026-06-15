---
title: NVIDIA Isaac（Sim / Lab / ROS）
author: metaarchetech
created: 2026-05-24
tags: [type/note, status/done, area/physical-ai, area/isaac]
---

# physical-ai / isaac

NVIDIA Isaac 家族的安裝、使用、整合筆記。

## 範圍

| 元件 | 用途 |
|---|---|
| **Isaac Sim** | Omniverse-Kit-based 3D 物理模擬；吃 USD / URDF |
| **Isaac Lab** | RL 訓練框架，跑在 Isaac Sim 之上 |
| **Isaac ROS** | Jetson / edge 部署用的 perception + manipulation ROS 2 packages |
| **Isaac Manipulator** | Reference workflow，主 cover Pick & Place |
| **Isaac Cosmos** | World foundation models（v1 不必碰） |

## 目前檔案

- [[isaac-sim-installation]] — Isaac Sim / Isaac Lab Windows 安裝實戰，含 Omniverse Composer 整合、KUKA 整合、訓練 vs 實機同步比較

## 相關

- 上一層：[[../..|physical-ai/]]
- 與 Omniverse 的關係：[[../../digital-twin/omniverse/omniverse|digital-twin/omniverse/]]
- 在 lab Linux 機上裝 Isaac：[[../linux-lab-setup-plan]]
