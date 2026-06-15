---
title: Physical AI 總覽
author: metaarchetech
created: 2026-05-19
updated: 2026-05-24
source: visustwin-vault
source_path: 01 Projects/Physical_AI/README.md
tags: [type/note, status/draft, area/physical-ai, area/robotics]
---

# Physical AI 總覽

## 一句話
Physical AI = 讓 AI 走出螢幕，與真實物理世界互動。核心是把感知（sense）、推理（reason）、行動（act）三段在實機（或高擬真模擬）上閉環。

## 我們關注的技術堆疊

| 層 | 技術 | 角色 |
|---|---|---|
| 機器人作業系統 | **ROS2**（Humble baseline） | sensor / planner / driver 中介 |
| 大型機械手臂 | **KUKA**（KR Agilus / iiwa 系列） | actuation 端 |
| 模擬與訓練 | **NVIDIA Isaac**（Sim / Lab / ROS） | sim-to-real、RL policy、edge 部署 |
| World model | **NVIDIA Omniverse** | USD-based scene、digital twin layer |
| 跨層共用 | **USD ontology** | physical world model 的資料表示 |

## 為什麼是這組合
- ROS2 + KUKA：工業手臂 + 開源生態最成熟的對接點（EKI-XML / kuka_drivers）
- Isaac Sim：吃 USD / URDF，KUKA URDF 可直接導入，cuMotion 路徑規劃 GPU 加速
- Omniverse：USD scene 共享層，連通 BIM / Digital Twin 與 Physical AI 兩個方向

## 開放問題
- 第一個 PoC 場景範圍（pick & place / 跟隨 / BIM 對齊驗證 …）
- KUKA 機型最終選定（直接決定 actuation 介面：KRL / RSI / FRI / EKI）
- Milestone 時程
- Sim-first vs Real-first vs Hybrid 路線取捨

## 相關筆記
- [[linux-lab-setup-plan]] — lab Linux 機環境建置（Phase 0）
- [[kuka-loop-architecture-options]] — KUKA loop 4 元素架構選項
- [[isaac-sim-installation]] — Isaac Sim 安裝實戰
