---
repo: visustwin-welltek-twin
github_url: https://github.com/metaarchetech/visustwin-welltek-twin
branch: feature/dna-port-v2
latest_commit: 849aa30
latest_commit_msg: "feat: auto-cycle section tabs with stable layout"
latest_commit_date: 2026-04-18
status: doing
device_target: 中大螢幕（desktop monitor / 壁掛 ≥1920px）
stack: Node.js + Express · MQTT (Mosquitto) · SQLite (better-sqlite3) · React + Vite · WebSocket · Docker
open_prs: []
updated: 2026-04-18
---

# welltek-twin

樣品屋供應商資料系統。即時接收供應商與產品資料的展場管理系統，透過 MQTT 接收、SQLite 儲存、REST API 查詢，並以 WebSocket 即時推播給展演端（Unity / Unreal / TouchDesigner）。

掛牆大螢幕為主要裝置目標，介面可以有高資訊密度，視覺上不需要跟 WebController 保持相同的克制感。

## 最新進度

`feature/dna-port-v2` — 今日（2026-04-18）主要推進：

- `849aa30` — feat: auto-cycle section tabs with stable layout（當日最新）
- `3e5fdb0` — Fix pure-white backgrounds and add light-mode HUD theming
- `e1117d4` — feat: DNA v2 — desktop-first living architecture monitor

`master` 最新：`a2096de` — docs: add CLAUDE.md with device target and design DNA notes（2026-04-18）

## 已知 TODO

- Section tabs label 改成寶舖語彙（WELL v2 / FM 維養 / 五大階段 / 六大數據）
- Demo mode for Vercel
- `feature/dna-port-v2` merge 回 `master`
- 3D 展示（規劃中）— Three.js / @react-three/fiber

## 關鍵決策

- **裝置目標**：壁掛 / desktop ≥1920px，資訊密度高 OK
- **Vercel**：先做 demo mode 再部署

## 連結

- GitHub：[VisTwin/visustwin-welltek-twin](https://github.com/metaarchetech/visustwin-welltek-twin)
- Dev：`http://localhost:5174`（React + Vite frontend）
- API：`http://localhost:3001`（Node.js server）

## 返回

- [[02 產品/VisTwin/README|VisTwin]]
