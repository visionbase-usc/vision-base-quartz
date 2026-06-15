---
title: Omniverse 應用 Web 化技術評估
author: metaarchetech
created: 2026-04-19
updated: 2026-05-24
source: visustwin-vault
source_path: 02 Products/Visustwin/decisions/Omniverse-Web化-評估.md
tags: [type/note, status/done, area/digital-twin, area/omniverse, area/web]
---

# Omniverse 應用 Web 化技術評估

> 本筆記討論 Omniverse Kit extension 體系如何**逐步搬到 Web 端**的技術評估。內容以一組 Digital Twin Kit extension family 為案例，提煉「哪些功能適合 web、哪些必須留在 Kit、合併的依據」。
>
> 評估方式：原始碼靜態分析（`extension.toml` + `.py`），未做 runtime 驗證。

---

## 執行摘要

針對 18 個 Kit extension，可重新以實際狀態分為五類（T0–T4）：

| 分類 | 數量 | 說明 |
|---|:---:|---|
| **T0 — 核心保留** | 6 | 成熟功能、獨立身份、優先 port |
| **T1 — 合併模組** | 7 | 工作流相連，web 端合併為 5 個 Feature Module |
| **T2 — 已被取代** | 2 | 既有 web 資產已做掉，Kit 端退役 |
| **T3 — 刪除** | 1 | 太陽方位舊插件，重複實作，無獨立價值 |
| **T4 — 存封 stub** | 2 | Skeleton 階段、未整合基建 |

**整併後：18 Kit ext → 7 Web Feature Modules**

```
Dashboard · WindSimModule · SolarAnalysisModule · BIMReviewModule
ConsoleModule · PresentationModule · SafetyMonitor
```

**整併原則**（T0 層）：幾何來源上游整合進 Massing Pipeline；T1 層按工作流三條件合併（同工作流 + 共用資料來源 + 相似 UI 骨架同時成立）。

**關鍵技術發現**：

1. 風場 solver 內部以 **numpy（CPU）** 實作，不是 GPU Warp 路徑 → 可直接移植 TypeScript RK4
2. 「光羅盤」插件完整複製太陽計算模組，無獨立價值（T3 刪除）
3. 健康/濕氣模組 docstring 明寫 `Phase 1 (skeleton)`，risk map tab 是 placeholder（T4）
4. MQTT bridge / OSC controller 已被既有 web 端取代（T2）

---

## Extension 分類總表（脫敏後）

| Extension 角色 | 功能摘要 | 技術依賴（非 web 部分） | Web 難度 | 工作量 | 整併去向 |
|---|---|---|:---:|:---:|---|
| AI Oracle | LLM 建築顧問 + 工具編排 | _INSTANCE pattern 呼叫其他 ext | Medium | M | **T1** → ConsoleModule |
| BIM Inspector | USD Stage BIM 完整度掃描 + 碰撞偵測 | pxr USD 遍歷 + displayColor 寫入 | Medium | M | **T0** 獨立（幾何簡化 → Massing Pipeline）|
| Camera Travel | USD 攝影機 SLERP 飛行動畫 | UsdGeom.Camera + viewport API | Hard | S/M | **T1** → PresentationModule |
| Dashboard | Extension 總覽 / 開關 + theme | omni.kit.app ExtensionManager | Easy | S | **T0** → App Shell |
| Dev REPL | File-based Python REPL（開發工具）| Kit Python exec context | Medium | S | **T1** → ConsoleModule（Kit WS Bridge 取代 file IPC）|
| Elements Core | 跨插件 Zone 資料 singleton | carb.events pub-sub | Easy | — | **T4** 存封（web 端改 Zustand store）|
| ESG Tracker | 具現化碳足跡 + EEWH/LEED 認證 | pxr USD 材質掃描 + 體積計算 | Medium | M | **T1** → BIMReviewModule |
| Exhibition Board | 第二螢幕展覽看板 + 嵌入 Viewport | omni.kit.viewport（RTX 渲染）| Hard | L | **T1** → PresentationModule（WebRTC）|
| Light Compass | 36 方向照度玫瑰圖 + 太陽軌跡弧 | USD 3D prim 動畫 | — | — | **T3 刪除**（複製太陽計算，3D 裝飾無獨立價值）|
| Moisture & Health | PMV/PPD 熱舒適 + 材質濕度風險 | 無（純 Python math） | Easy | — | **T4** 存封（skeleton，risk map 是 placeholder）|
| MQTT Bridge | WebSocket → Carb event bus | websocket-client | Easy | **0** | **T2** ✓ 既有 web 已取代 |
| OSC Controller | OSC UDP → USD scene 動作分發 | pxr USD 操作 + omni.timeline | Easy | **0** | **T2** ✓ 既有 web 已取代 |
| Solar Heatmap | 3 棟×N 層×4 戶太陽得熱計算 | carb.settings + USD scene 熱圖 | Easy/Med | M | **T0** 獨立（demo_preset → Massing Pipeline）|
| Solar Report | 5 張 matplotlib 圖表 + CSV/JSON 匯出 | matplotlib | Easy | M | **T1** → SolarAnalysisModule |
| Sunlight Studio | NOAA 太陽位置 + UsdLux 場景照明 | UsdLux.DistantLight + RTX | Medium | S | **T0** 獨立（提供太陽位置；唯一正確來源）|
| Vision Detector | YOLO WS client + Viewport 偵測 overlay | omni.ui overlay | Medium | M | **T0** → SafetyMonitor |
| Warp WindTunnel | 位勢流風場（**numpy**）+ 3D 粒子 | USD BasisCurves + Warp dep | Medium | L | **T0** 獨立（PRESETS → Massing Pipeline）|
| Wind Analysis | 風場報告 + Davenport 舒適度評估 | 依賴 windtunnel solver | Easy | M | **T1** → WindSimModule |

---

## 五類分級（T0–T4）

```
T0 核心保留 (6)
  Dashboard · WindTunnel · Solar Heatmap · Sunlight Studio
  BIM Inspector · Vision Detector

T1 合併 (7) → 5 個 Feature Module
  Wind Analysis ─────────────▶ WindSimModule     (+ WindTunnel T0)
  Solar Report ──────────────▶ SolarAnalysisModule (+ Heatmap + Sunlight T0)
  ESG Tracker ───────────────▶ BIMReviewModule   (+ BIM Inspector T0)
  AI Oracle + Dev REPL ──────▶ ConsoleModule
  Camera Travel + Exhibition ▶ PresentationModule

T2 退役 (2)
  MQTT Bridge ───────────────▶ 既有 web stack 已做
  OSC Controller ────────────▶ 既有 web stack 已做

T3 刪除 (1)
  Light Compass ─────────────▶ 太陽方位舊插件，複製主算法

T4 存封 (2)
  Moisture & Health ─────────▶ Phase 1 skeleton
  Elements Core ─────────────▶ zone registry 未整合，web 端 Zustand 取代
```

---

## 整併矩陣

```
Kit Extensions (18)              Web 輸出
─────────────────────────────────────────────────────────────

【Phase 0 清理 — 不帶進 web】
Light Compass ──────────────────▶ 刪除（T3）
MQTT Bridge ────────────────────▶ 退役（T2）
OSC Controller ─────────────────▶ 退役（T2）
Moisture & Health ──────────────▶ 存封（T4，skeleton）
Elements Core ──────────────────▶ 存封（T4，→ Zustand）

【Massing Pipeline 基礎層】
demo_preset (Solar Heatmap)    ┐
PRESETS dict (WindTunnel)      ├──▶ Massing Pipeline
BIM 幾何簡化 (BIM Inspector)   ┘    (testPresetAdapter + bimAdapter)
                                    ↓ 統一 BuildingMass[] 給所有下游

【7 Web Feature Modules】
Dashboard ──────────────────────▶ 1. Dashboard（App Shell + routing）
WindTunnel + Wind Analysis ─────▶ 2. WindSimModule
Solar Heatmap + Sunlight Studio ┐
+ Solar Report                  ├──▶ 3. SolarAnalysisModule
BIM Inspector + ESG Tracker ────▶ 4. BIMReviewModule
AI Oracle + Dev REPL ───────────▶ 5. ConsoleModule（Kit WS Bridge）
Camera Travel + Exhibition ─────▶ 6. PresentationModule（WebRTC opt.）
Vision Detector ────────────────▶ 7. SafetyMonitor
```

---

## 架構建議

### Kit 端保留職責

```
Omniverse Kit（保留）
├── RTX 渲染 + USD Stage（唯一事實來源）
├── 場景控制執行器（USD 操作 / camera fly / lighting）
├── ws.bridge（新 extension，暴露 WebSocket RPC API）
│   └── 接收 web 指令 → 轉發到 Kit Python 函式
└── 現有 extensions（降級為 headless 服務，可去除 omni.ui window）
```

### Web 端負責

```
Web（新 / 現有資產）
├── UI 控制（所有輸入 / 設定 / 參數調整）
├── 純計算邏輯（solar / wind / PMV / carbon，移植 TypeScript）
├── 報告輸出（圖表 / CSV / PDF，Recharts / Plotly / jsPDF）
├── IoT 數據顯示（MQTT / Vision WebSocket 直連）
└── AI 問答（LLM API 直連，tool calls 透過 ws.bridge）
```

---

## 整備階段建議（技術視角）

### Phase 0（清理優先）：T2 退役 + T3 刪除 + T4 存封

| 任務 | 對象 | 行動 |
|---|---|---|
| T3 刪除 | Light Compass | 從 extension 目錄移除；確認 Sunlight Studio 已含所有計算 |
| T2 退役 | MQTT Bridge | `extension.toml` 加 `[deprecation]`；kit apps cfg 移除載入 |
| T2 退役 | OSC Controller | 同上；確認既有 web 完整覆蓋所有 OSC route |
| T4 存封 | Moisture & Health | 備份 `comfort.py`（PMV 計算）→ 抽出進 SolarAnalysisModule 熱舒適分頁 |
| T4 存封 | Elements Core | 不 port，ZoneRegistry 概念改寫為 `zoneStore.ts` |

### Phase 1：計算核心 + Massing Pipeline 基礎

目標：移植計算邏輯 + 建立 Massing Pipeline，消除 `demo_preset.py` / `PRESETS` 重複。

| 任務 | 工作量 | 輸出 |
|---|---|---|
| 移植 `demo_preset.py` → `testPresetAdapter.ts` | S | 統一 BuildingMass[]，消除重複 |
| 建立 `massingStore.ts`（Zustand）| S | 下游模組統一讀取點 |
| 移植 `sun_calculator.py` → `solarCalc.ts` | S | TypeScript NOAA 演算法，單元測試 |
| 移植 `gain_calculator.py` → `solarGain.ts` | S | 每戶太陽得熱計算 |
| 移植 `solver.py`（numpy flow）→ `windSolver.ts` | M | 位勢流求解器（RK4） |
| 移植 `comfort.py` → `pmvCalc.ts` | S | PMV/PPD 計算器 |
| 移植 `carbon.py` → `carbonCalc.ts` | S | 碳排計算 + 材質資料庫 JSON |
| 建立 `zoneStore.ts`（Zustand）| S | 跨 module zone 數據共享 |

### Phase 2：獨立下游 Module UI 建構

目標：在選定架構上建立各下游模組（全部獨立，不合併）。

| 任務 | 工作量 | 輸出 |
|---|---|---|
| App Shell（Sidebar + routing）| S | 框架骨架 |
| Solar Heatmap module（讀 massingStore）| M | React UI + solarGain + Recharts |
| CFD WindTunnel module（讀 massingStore）| M | React UI + windSolver + 2D 風場圖 |
| PMV Comfort module（讀 zoneStore + MQTT）| S | React UI + pmvCalc + WS |
| BIM Inspector module UI（read-only）| M | React table |

### Phase 3：Kit Bridge + 即時連線

目標：建立 Kit WS Bridge，讓 web 端真正連線到 Kit 執行場景操作。

| 任務 | 工作量 | 輸出 |
|---|---|---|
| `ws.bridge` Kit extension | M | WebSocket RPC server |
| AIOracleModule（web 版 chat UI）| M | LLM API + tool calls via bridge |
| SafetyModule（Vision WS overlay）| M | Canvas bounding box overlay |
| PresentationModule（camera control）| S | 攝影機切換按鈕 |
| Exhibition WebRTC（可選）| L | RTX 串流接收（需 NVIDIA Streaming SDK） |

---

## 相關

- [[web-architecture-options]] — 架構選項 A / B / C 比較
- [[../bim/massing-web-pipeline]] — Massing Pipeline 設計
- [[omniverse]]
