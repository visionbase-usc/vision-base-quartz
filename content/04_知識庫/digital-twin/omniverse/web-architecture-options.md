---
title: Omniverse + Web 整合架構選項
author: metaarchetech
created: 2026-04-19
updated: 2026-05-24
source: visustwin-vault
source_path: 02 Products/Visustwin/decisions/Web化-架構選項.md
tags: [type/note, status/done, area/digital-twin, area/omniverse, area/web, area/architecture]
---

# Omniverse + Web 整合架構選項

> 三套常見方案的技術比較，各有不同的整合範圍、開發量和長期維護成本。
> 適用於：已有 Omniverse Kit extension 體系，希望把分析 / 控制 / 報告 UI 搬到 web 端的場景。

---

## 前提假設（共同）

- Kit 端永遠保留做為 **RTX 渲染 + USD 資料服務器**（3D 視覺化品質不犧牲）
- Web 端負責：**UI 控制、計算邏輯展示、報告輸出、非 GPU 運算**
- 通訊協議共同基線：WebSocket / REST（Kit ↔ Web bridge）

---

## 方案 A：整合進現有 Web App

### 概念

在現有的 Next.js / React app 中新增分析功能 routes，擴展為全功能 app。

```
app/
├── (existing pages)
├── env-analysis/      ← 新增 EnvAnalysisModule
│   ├── solar/
│   ├── wind/
│   └── comfort/
├── bim/               ← 新增 BIMSustainabilityModule
├── ai-oracle/         ← 新增 AIOracleModule
├── safety/            ← 新增 SafetyModule
└── connections/       ← 新增 DataConnectionModule
components/
├── (existing components)
└── analysis/          ← 新增分析元件
lib/
├── solarGain.ts       ← 新增
├── windSolver.ts      ← 新增
└── kitBridge.ts       ← 新增 Kit WebSocket bridge
```

### Pros

- 零 repo 開銷，直接擴展現有代碼庫
- 共用已建立的 UI 元件 / 設計系統
- 已有 Next.js + React + TypeScript strict 環境
- 對使用者：一個 URL 涵蓋所有功能

### Cons

- 原 app 若定位是行銷或展示，加入重型分析工具會混合職責
- bundle size 大幅增長（charting、Three.js、D3 全部進去）
- URL 架構需要規劃（marketing vs tool）
- Lighthouse / SEO 分數可能受影響

### 工作量估算

| 項目 | 估計 |
|---|---|
| Route 架構調整 | S |
| EnvAnalysisModule | L |
| BIMSustainabilityModule | M |
| AIOracleModule | M |
| SafetyModule | M |
| Kit Bridge WS server | M |
| **總計** | **XL（3-4 個月）** |

### 適合情境

- 想要單一 URL 展示
- 團隊只有一個人維護前端
- 近期沒有 public-facing marketing 需求

---

## 方案 B：新 repo（推薦中期方案）

### 概念

建立全新的 studio repo，專門負責「操作 / 分析 / 工程工具」，與行銷展示分開。

```
studio-app/                     ← 新 repo
├── app/
│   ├── layout.tsx             （Sidebar + TopNav App Shell）
│   ├── dashboard/             （module 清單 + 狀態概覽）
│   ├── env-analysis/
│   ├── bim/
│   ├── ai-oracle/
│   ├── safety/
│   └── connections/
├── components/
│   ├── sidebar/
│   └── modules/
├── lib/
│   ├── solarGain.ts
│   ├── windSolver.ts
│   ├── kitBridge.ts          （Kit WebSocket client）
│   └── zoneStore.ts          （Zustand）
└── package.json              （共用 UI package 引用）
```

### Kit Bridge 架構

```
studio-app ──WS──▶ kitBridge.ts ──▶ Kit WebSocket Server
                                        │
                                        ├── scan_bim_stage()
                                        ├── calc_solar_gain()
                                        ├── get_zone_data()
                                        └── fly_to_camera()
```

Kit 端需新增一個 **WebSocket RPC server extension**（`ws.bridge`），暴露所有 Kit 功能為 JSON RPC endpoint。

### Pros

- 職責分離：行銷 vs 工具
- 可獨立部署（localhost or intranet）
- 技術棧自由（可選更適合 data app 的 UI 框架，如 shadcn/ui）
- 可以用 Next.js 15 app router 的 server actions 做 Kit RPC
- 未來共用 UI 套件抽出後，兩個 repo 都可引用

### Cons

- 新 repo，需設置 CI/CD、環境變數、部署管線
- 需要額外維護成本
- 需要新建 Kit WS Bridge extension

### 工作量估算

| 項目 | 估計 |
|---|---|
| Repo 設置 + App Shell | S |
| Kit WS Bridge extension | M |
| EnvAnalysisModule | L |
| BIMSustainabilityModule | M |
| AIOracleModule | M |
| SafetyModule | M |
| **總計** | **XL（3-4 個月）** |

### 適合情境

- 想要清晰的產品定位（展示 vs 工具）
- 預計這個工具未來長期使用
- 有計劃抽出共用 UI 套件

---

## 方案 C：分散式 Islands（微服務 / 最小化侵入）

### 概念

不開新 repo，也不大改現有 app，而是把各功能**整合進現有 web 資產**，每個功能對應最相關的 repo。

| 功能 | 整合到哪裡（範例） |
|---|---|
| 環境分析（風場 / 日照 / 舒適）| 既有環境監控 app |
| BIM + ESG | 既有建築展示 app |
| AI Oracle | 既有展示 app（聊天 drawer） |
| 攝影機控制 + OSC | 既有 WebController |
| Vision Safety | 環境監控 app 或獨立 safety repo |
| 連線狀態 | 環境監控 app 的 connections tab |

### Pros

- 最低新代碼量（每個功能加在最相關的現有 repo）
- 零新 repo 管理成本
- 可以 incremental 推進（一次做一個功能）

### Cons

- 分散在多個 repo，使用者體驗破碎
- 無統一 URL，每個工具要去不同地方找
- 共用邏輯（solarGain、windSolver）會重複實作或需要 npm package
- 難以展示「完整平台」
- 長期維護複雜度高

### 工作量估算

| 項目 | 估計 |
|---|---|
| 各功能分散整合 | M × 6 |
| 共用邏輯複製或 npm 包 | M |
| **總計** | **L（2-3 個月，但碎片化）** |

### 適合情境

- 沒有資源做統一 app
- 各功能需要獨立部署到不同設備（壁掛 / iPad 等）
- 快速 demo 用，不考慮長期維護

---

## 方案比較矩陣

| 指標 | 方案 A（extend）| 方案 B（new repo）| 方案 C（islands）|
|---|:---:|:---:|:---:|
| 統一 UX | 是 | 是 | 否 |
| 職責清晰 | 部分 | 是 | 是 |
| 開發速度（初期）| 是 | 部分 | 是 |
| 長期維護 | 部分 | 是 | 否 |
| 展示效果 | 是 | 是 | 否 |
| Repo 管理成本 | 是 | 部分 | 是 |
| 共用元件 | 是 | 部分（需設置）| 部分（各自設置）|
| 未來 UI 套件 | 是（受益最多）| 是 | 部分（各自引用）|

---

## Kit 端保留架構（共同）

無論選哪個方案，Kit 端都需要新增 **WebSocket Bridge**：

```
ws.bridge（新 extension）
├── WebSocket server（ws://localhost:9001）
├── RPC dispatcher
│   ├── bim.scan() → scanner.py
│   ├── solar.gain(params) → gain_calculator.py
│   ├── wind.solve(params) → solver.py
│   ├── zone.get_all() → zone_registry.py
│   ├── camera.fly_to(path) → camera_fly.py
│   ├── scene.set_room(name) → osc_controller actions
│   └── lighting.blend(value) → osc_controller actions
└── Event streamer（push to web）
    ├── mqtt.message → web
    ├── vision.detection → web
    └── elements.zone_updated → web
```

---

## 一般選型建議

- **短期（0-3 個月）**：採用 **方案 A**，在既有 app 加一個 `/studio` route group，快速迭代。
- **中期（3-12 個月）**：當功能穩定後，**分拆為方案 B**（把 `/studio` 系列提取到獨立 repo），原 app 回歸純展示用途。
- **方案 C** 只在以下情況考慮：資源極度有限 + 不需要統一展示 + 各設備部署需求差異大。

---

## 待決策點（通用）

1. **Kit WS Bridge extension** 是否現在就啟動？（影響其他所有方案的可行性）
2. **RTX 畫面** 要不要做 WebRTC streaming？（嵌入式 RTX viewport 的 web 化前提）
3. **短期用方案 A 還是直接方案 B**？
4. **共用 UI 套件**何時抽出？（影響多 repo 設計系統共用成本）

---

## 相關

- [[omniverse-web-feasibility]] — 主評估文件
- [[../bim/massing-web-pipeline]] — Massing Pipeline 設計
