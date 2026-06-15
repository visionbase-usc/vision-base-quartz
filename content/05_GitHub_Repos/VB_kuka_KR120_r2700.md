---
title: VB_kuka_KR120_r2700 repo
author: metaarchetech
created: 2026-06-04
tags: [type/note, status/wip, area/governance, area/digital-twin, area/robotics, project/visionbase-lab]
---

# VB_kuka_KR120_r2700

KUKA **KR 120 R2700** 六軸工業手臂的 **web digital twin** — Three.js 前端 + FastAPI 後端,含正逆運動學、時間參數化軌跡、可達性分析,並打通 **Grasshopper → 即時匯入** 工作流。免 npm(CDN import map),概念上對標 visose/Robots。

## 基本資訊

| 項目 | 值 |
|---|---|
| **URL** | [github.com/visionbase-usc/VB_kuka_KR120_r2700](https://github.com/visionbase-usc/VB_kuka_KR120_r2700) |
| **類型** | 內部 Web App(機器人控制 + 視覺化 / digital twin) |
| **可見性** | 私人 |
| **預設分支** | `main` |
| **本機路徑** | `C:\Users\User\Documents\chrono\robot_web\` |
| **開發 URL** | `http://127.0.0.1:8000` |
| **機型** | KUKA KR 120 R2700(QUANTEC 2700 機構) |

## 技術棧

| 層 | 選用 |
|---|---|
| 後端 | FastAPI + 純 numpy(FK、damped-least-squares 數值 IK、極限) |
| 軌跡 | 梯形速度時間軸(PTP 同步 / LIN TCP 速度) |
| 前端 | Three.js(CDN import map,無 npm) |
| 渲染 | ACES tone mapping、RoomEnvironment IBL、Reflector 鏡面地板、Fog |
| 串流 | WebSocket `/ws`(`chrono_bridge.py` 從 Project Chrono sim 串關節角) |
| CAD 整合 | GhPython 元件(Rhino 8 / CPython)→ JSON → 後端 push inbox |

## 模型來源

KR 120 R2700 無公開 URDF → 採 ROS-Industrial / kroshu 的 **kr210_r2700_2**(同 QUANTEC 2700 機構 → 運動學完全相同,只差負載)。STL vendored 在 `meshes/kr210_r2700_2/visual/`。

## 主要能力

- 即時拖動 A1–A6 / TCP,看 FK 與 TCP 位姿
- 匯入 Rhino-Plane TCP 目標 → 多起點 IK → 畫路徑 → 平滑播放
- 可達配置枚舉 + KUKA Status 位元(s0 過頂 / s1 肘 / s2 腕)
- 分析圖:關節角、可操作度 `w=|det J|`、加速度(弧長視窗法)
- **Grasshopper 一鍵推送**(`/api/targets/push` + `/api/targets/inbox`,前端 1.5s 輪詢自動匯入)

## Grasshopper 元件(`grasshopper/`)

| 元件                     | 功能                              |
| ---------------------- | ------------------------------- |
| `VB_TargetExporter.py` | Rhino Planes → import JSON(可寫檔) |
| `VB_PushToRobot.py`    | JSON POST 進後端 inbox             |

## 維護者

- 開發:[@metaarchetech](https://github.com/metaarchetech)

## 相關文件

- [[../04_知識庫/digital-twin/kuka-kr120-web-twin|技術筆記(04_知識庫)]] — 架構、IK/軌跡、加速度陷阱、GhPython 踩坑、渲染細節、待辦
- repo `README.md`(§1 啟動、§6 陷阱、§9 待辦)為完整自足交接文件

## 相關 repo

- [`visionbase-monitor`](visionbase-monitor.md) — 同為內部 Web App、套 VB 品牌深色主題
- [`vision_base_vault`](vision_base_vault.md) — 本筆記所在 vault
