---
title: VISION BASE Lab
author: metaarchetech
created: 2026-04-29
updated: 2026-05-24
source: visustwin-vault
source_path: 01 Projects/visionbase/README.md
tags: [type/project, status/draft, project/visionbase-lab, area/physical-ai]
---

# VISION BASE Lab（實踐大學 visionbase 實驗室）

> 一句話描述：實踐大學 visionbase 實驗室——VISION BASE Lab 的硬體所在地，2026-04-29 起進駐並開始建置區網盤點與 KUKA 控制鏈。

## 基本資訊

| 項目 | 內容 |
|---|---|
| **單位** | 實踐大學 visionbase 實驗室 |
| **角色** | 學校端 lab、Physical AI 主軸的硬體基地 |
| **物理進駐日** | 2026-04-29 |
| **負責人** | @ |
| **成員** | @ |
| **狀態** | 進行中 |
| **Area** | physical-ai / digital-twin |

## 目標

- 打通 ROS2 + KUKA + NVIDIA Isaac + Omniverse 全棧
- 區網盤點與監控站建置（已完成首次掃描）
- 建立可重現的 Lab Linux 開發環境

## 子資料夾

| 資料夾 | 內容 |
|---|---|
| `會議/` | 本專案會議記錄 |
| `筆記/` | 技術筆記、進度（KUKA / 監控站等） |
| `refs/` | 相關文獻、外部參考（含 2025 早期計畫） |
| `_attachments/` | 本專案專用圖片（小檔） |

## 主要文件

### 現況（2026-W18 起）

- [[network-inventory]] — 區網盤點清單（34 台裝置 / 29 在線）
- [[visionbase-monitor]] — 裝置監控站文件
- [[kuka-control-feasibility]] — KUKA 控制可行性研究
- [[kuka-loop-architecture-options-v0]] — KUKA 控制 loop 架構選項

### 歷史參考（2025 年早期計畫）

- [[2025-cooperation-summary]] — 2025 年合作摘要
- [[2025-xr-exhibition]] — 2025 XR 展示開發
- [[2025-xr-motion-ml-plan]] — XR 人體動作模擬 + ML 計畫

## 網路環境

| 項目 | 值 |
|---|---|
| SSID | visionbase 學校網路 |
| 子網 | `192.168.0.0/24` |
| 開發機本機 IP | `192.168.0.220`（`DESKTOP-JDT5BJ8`，Windows） |
| Router | `192.168.0.1`（ASUS RT-AX1800S） |
| KUKA 控制 PC | `VB-KUKA-Only.local`（`192.168.0.82`） |
| Synology NAS | `VISIONBASE_NAS.local`（`192.168.0.85`） |

## 相關知識庫連結

- [[../../04_知識庫/physical-ai/physical-ai|Physical AI 知識庫]]
- [[../../04_知識庫/physical-ai/robotics/kuka/kuka|KUKA 知識]]
- [[../../04_知識庫/digital-twin/omniverse/omniverse|Omniverse 知識]]

## 備註

- 本專案資料原存於 Visustwin vault `01 Projects/visionbase/`，2026-05-24 搬遷至 VISION BASE vault
