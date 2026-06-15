---
title: KUKA 機械手臂
author: metaarchetech
created: 2026-05-24
tags: [type/note, status/done, area/physical-ai, area/robotics]
---

# physical-ai / robotics / kuka

KUKA 機械手臂相關：機型家族、控制器、KRL、介接協定、loop 架構選項。

## 範圍

- KUKA 機型家族（KR Agilus / KR Quantec / LBR iiwa / iisy / KR.AI）
- 控制器世代（KRC4 / KRC5 / Sunrise Cabinet）
- KRL 程式語言
- 介接協定：EKI-XML、RSI、FRI、KUKAVARPROXY、OPC UA
- 與 ROS2 driver / Isaac Sim 整合的架構選擇

## 目前檔案

- [[kuka-loop-architecture-options]] — KUKA + sensor + Isaac + AI loop 4 元素架構選項菜單；機型對應、Isaac 棧解析、場景對比、sim-vs-real trade-off
- [[dabai-llm-pet]] — 大白：LLM 驅動的 KR 120 R2700「機械手臂寵物」；本地 LLM 生成動作、皮克斯式生命引擎、VLM 看影片學動作

## 相關

- 上一層：[[../../..|physical-ai/]]
- ROS2 介接：[[../ros2/ros2|robotics/ros2/]]
- 模擬端：[[../../isaac/isaac|isaac/]]
- lab Linux 機這側 setup：[[../../linux-lab-setup-plan]]
