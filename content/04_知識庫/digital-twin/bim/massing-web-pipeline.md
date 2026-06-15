---
title: Massing Web Pipeline 設計
author: metaarchetech
created: 2026-04-19
updated: 2026-05-24
source: visustwin-vault
source_path: 02 Products/Visustwin/decisions/Web化-Massing-Pipeline.md
tags: [type/note, status/done, area/digital-twin, area/bim, area/web]
---

# Massing Web Pipeline 設計

> 原則：**整併只在上游（幾何來源）發生；下游模擬模組全部獨立。**
>
> 適用場景：Omniverse Kit extension family 內，多個分析插件（風場 / 日照 / 碳排）各自維護一份建築量體配置 → Web 化時需要一個共用的 Massing 上游層。

---

## 為什麼需要這一層？

### 問題診斷

Kit extension family 裡，**建築幾何配置常被重複定義**：

```python
# solar_heatmap / demo_preset.py
PRESETS = {
    "medium": [
        {"name": "Tower_A", "cx": -1000, "cy": -120, "wx": 1500, "wy": 1200, "hz": 4800},
        {"name": "Tower_B", "cx":  -800, "cy": 1130, "wx": 1200, "wy": 1000, "hz": 4300},
        {"name": "Tower_C", "cx":  1800, "cy":-1020, "wx": 1400, "wy": 1100, "hz": 4500},
    ]
}

# wind_tunnel / solver.py  ← 同樣的資料！
# !!  KEEP IN SYNC WITH solar_heatmap.demo_preset.py  !!
PRESETS = {
    "medium": [
        {"name": "Tower_A", "cx": -1000, "cy": -120, "wx": 1500, "wy": 1200, "hz": 4800},
        ...
    ]
}
```

每個下游 ext 都在維護自己的幾何版本，導致：
- **數據不一致風險**：兩份不同步就會讓日照和風場分析算的是不同建築
- **新增分析模組要複製貼上**：PMV、碳排新模組都要自己定義一次幾何
- **BIM 幾何簡化邏輯散落**：USD 掃描 → 簡化量體的邏輯沒有可重用介面

### 解決方案：Massing Pipeline

抽出一個共用層，**統一負責「生成 / 轉換建築量體」**，讓所有下游模擬模組消費同一份輸出。

---

## Data Flow 架構圖

```
                    ┌─────────────────────────────────────────┐
                    │          MASSING PIPELINE               │
                    │                                         │
  ┌──────────────┐  │  ┌──────────────┐  ┌────────────────┐  │
  │  test-preset │──┼─▶│   Adapter    │  │   Geometry     │  │
  │  (SMALL/MED/ │  │  │   Layer      │──▶   Cache        │  │
  │   LARGE)     │  │  │              │  │   (output)     │  │
  └──────────────┘  │  └──────────────┘  └───────┬────────┘  │
                    │                            │           │
  ┌──────────────┐  │  ┌──────────────┐          │           │
  │  bim-adapter │──┼─▶│  Simplifier  │          │           │
  │  (USD/BIM    │  │  │  (USD→massing│          │           │
  │   geometry)  │  │  │   reduction) │          │           │
  └──────────────┘  │  └──────────────┘          │           │
                    └────────────────────────────┼───────────┘
                                                 │
                    ┌────────────────────────────▼───────────────────────┐
                    │            DOWNSTREAM CONSUMERS（全部獨立）         │
                    │                                                     │
                    │  ┌──────────┐  ┌──────────┐  ┌──────────────────┐  │
                    │  │   CFD    │  │  Solar   │  │  Wind Analysis   │  │
                    │  │ WindTunnel│ │  Heatmap │  │  (Report)        │  │
                    │  └──────────┘  └──────────┘  └──────────────────┘  │
                    │                                                     │
                    │  ┌──────────┐  ┌──────────┐  ┌──────────────────┐  │
                    │  │   PMV    │  │  Carbon  │  │  BIM Inspector   │  │
                    │  │ Comfort  │  │  Tracker │  │  (viewer only)   │  │
                    │  └──────────┘  └──────────┘  └──────────────────┘  │
                    │                                                     │
                    │  ┌──────────┐  ┌──────────┐  ┌──────────────────┐  │
                    │  │  Solar   │  │  Light   │  │  AI Oracle       │  │
                    │  │  Report  │  │  Compass │  │  (tool calls)    │  │
                    │  └──────────┘  └──────────┘  └──────────────────┘  │
                    └─────────────────────────────────────────────────────┘
```

---

## Input Adapter 介面設計

所有 adapter 產生相同輸出格式（Adapter Contract）：

```typescript
// massingPipeline.ts

export interface BuildingMass {
  id: string              // "Tower_A"
  cx: number              // 中心 X（公尺，Z-up）
  cy: number              // 中心 Y
  width: number           // 寬度（X 方向）
  depth: number           // 深度（Y 方向）
  height: number          // 總高
  angle: number           // 旋轉角（degree，Z 軸）
  floorCount: number      // 樓層數
  floorHeight: number     // 樓層高（公尺）
  tags: {
    usage?: "residential" | "commercial" | "mixed"
    material?: string     // 主要材質 keyword
    units?: UnitConfig[]  // 每層戶型配置（NE/SE/SW/NW）
  }
}

export interface MassingOutput {
  buildings: BuildingMass[]
  preset: string          // "small" | "medium" | "large" | "bim-import" | "custom"
  source: "test-preset" | "bim-adapter" | "manual"
  timestamp: number
}

export interface MassingAdapter {
  id: string
  label: string           // UI 顯示名稱
  generate(config: Record<string, unknown>): MassingOutput
}
```

### test-preset Adapter

```typescript
// adapters/testPreset.ts
export const testPresetAdapter: MassingAdapter = {
  id: "test-preset",
  label: "測試量體（參數化）",
  generate(config: { preset: "small" | "medium" | "large" }): MassingOutput {
    const raw = PRESETS[config.preset]   // 原本 demo_preset.py 的資料
    return {
      buildings: raw.map(b => ({
        id: b.name,
        cx: b.cx / 100,      // cm → m
        cy: b.cy / 100,
        width: b.wx / 100,
        depth: b.wy / 100,
        height: b.hz / 100,
        angle: b.angle,
        floorCount: Math.round((b.hz / 100) / 3.0),
        floorHeight: 3.0,
        tags: {}
      })),
      preset: config.preset,
      source: "test-preset",
      timestamp: Date.now()
    }
  }
}
```

### bim-adapter

```typescript
// adapters/bimAdapter.ts
export const bimAdapter: MassingAdapter = {
  id: "bim-adapter",
  label: "BIM 匯入（USD → 量體簡化）",
  generate(config: { scanData: BimScanResult }): MassingOutput {
    // 原本 bim_inspector/scanner.py 的幾何簡化邏輯抽出來
    // 把複雜 USD prim 樹簡化為 BuildingMass[]
    const buildings = simplifyBimToMassing(config.scanData)
    return {
      buildings,
      preset: "bim-import",
      source: "bim-adapter",
      timestamp: Date.now()
    }
  }
}
```

---

## 統一輸出 Schema（Geometry Cache）

```typescript
// 下游模組讀取的最終格式
export interface GeometryCache {
  version: number          // 版本號，下游可偵測快取是否需要刷新
  massing: MassingOutput
  // 衍生資料（由 Massing Pipeline 預計算，節省下游重複計算）
  bboxes: Record<string, BoundingBox>        // 每棟建築的 AABB
  facadeAreas: Record<string, FacadeAreas>   // NE/SE/SW/NW 各立面面積
  floorPlans: Record<string, FloorPlan[]>    // 每層平面
}
```

---

## 下游模組消費方式

下游模組**只讀取** Geometry Cache，不寫入：

```typescript
// 任何下游模組（例如 solarHeatmap.ts）
import { useMassingStore } from '@/stores/massingStore'

function SolarHeatmapPanel() {
  const { cache } = useMassingStore()

  // 直接用統一格式
  const results = cache.massing.buildings.flatMap(bldg =>
    Array.from({ length: bldg.floorCount }, (_, floor) =>
      calcFacadeGain(bldg, floor, sunPosition)
    )
  )
  // ... render
}
```

**Zustand store 串接事件**：當 Massing Pipeline 更新時，所有下游 store 透過 subscribe 收到通知：

```typescript
// massingStore.ts
const useMassingStore = create<MassingStore>((set, get) => ({
  cache: null,
  activeAdapter: "test-preset",
  setMassing: (output: MassingOutput) => {
    const cache = buildGeometryCache(output)
    set({ cache })
    // 下游 stores 監聽 massingStore 的 cache 變更，自動 invalidate 自己的計算結果
  }
}))
```

---

## 跟原本 Extension 的對應關係

| 原 Kit Extension 角色 | 原職責 | 移入 Massing Pipeline | 留在原 Module |
|---|---|---|---|
| Solar Heatmap | `demo_preset.py` 建築配置 | 是 — test-preset adapter | 計算 + UI |
| Wind Tunnel | `solver.py` PRESETS dict | 是 — test-preset adapter（消除重複）| 求解器 + 視覺化 |
| BIM Inspector | USD 幾何掃描 + 簡化 | 是 — bim-adapter（幾何簡化部分）| BIM 查詢 / 報告 UI |
| Elements Core | ZoneRegistry singleton | 部分（幾何定義 + bbox） | zone 數據聚合 → Zustand |
| Wind Analysis | 讀 wind tunnel 建築配置 | 是 — 改讀 massingStore | 求解器報告 |
| Moisture & Health | 間接用 zone data | — | 計算 + UI（改讀 massingStore）|
| ESG Tracker | USD 材質體積 | 是 — bim-adapter 的材質 tag | 碳排計算 + UI |
| Solar Report | 讀 `demo_preset` | 是 — 改讀 massingStore | 報告 + 圖表 |

---

## 為什麼下游不合併

| 模組 | Solver 類型 | UI 特殊需求 | 獨立理由 |
|---|---|---|---|
| CFD WindTunnel | 位勢流 + curl noise | 時間軸 + 向量層 + 粒子密度 | solver 本質不同 |
| Solar Heatmap | 幾何光線投影 | 樓層熱圖 grid + 方位排名 | 時間維度（LIVE / DAILY）|
| Wind Analysis | 同 CFD | comfort 等級地圖 + 向量場 | 報告格式不同 |
| Solar Report | 積分掃描 | 5 種 matplotlib 圖表 | 報告格式不同 |
| PMV Comfort | Fanger thermal model | 儀錶板 + 感測器疊加 | 需即時 MQTT 輸入 |
| Carbon/ESG | 碳排因子查表 | 認證門檻 + 熱圖 | 認證邏輯獨立 |
| BIM Inspector | USD 遍歷 | LOD 表格 + 碰撞清單 | 純 viewer，不做模擬 |

**合併 = 萬能糊**：把 CFD solver 和 PMV 計算塞進同一個 React component，只會讓兩個都難維護。

---

## 開發優先順序建議

```
Week 1-2:  test-preset adapter（移植 demo_preset.py → TS）
           massingStore（Zustand）
           → 所有下游模組切換到讀 massingStore，消除重複

Week 3-4:  bim-adapter（抽出 BIM Inspector 的幾何簡化邏輯）
           GeometryCache 完整 schema
           → BIM 匯入後直接驅動所有模擬模組

Later:     更多 adapter（DXF 匯入、手動繪製量體等）
```

---

## 相關

- [[../omniverse/omniverse-web-feasibility]] — 整體 Web 化評估
- [[../omniverse/web-architecture-options]] — 架構方案 A/B/C
- [[bim]] — BIM 整合知識庫
