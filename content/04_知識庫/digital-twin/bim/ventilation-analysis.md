---
title: 通風分析報告插件架構研究
author: metaarchetech
created: 2026-04-29
updated: 2026-05-24
source: visustwin-vault
source_path: 02 Products/Visustwin/Plugins/ventilation.report/architecture-research.md
tags: [type/howto, status/done, area/digital-twin, area/bim, area/omniverse, area/ventilation]
---

# 通風分析報告插件架構研究

> 本研究剖析 Omniverse Kit 環境下、一組「環境分析報告插件」的共用骨架（matplotlib + pop-out window pattern），作為新插件 scaffold 的對齊基準。
>
> 研究背景：建築物通風（換氣）分析報告插件，需與既有日照、風場分析報告維持結構一致。

---

## 既有 report 插件（2 個 reference）

| Ext 角色 | 用途 | 對應 sim plugin |
|---|---|---|
| Solar Report | 太陽熱能分析報告（SUN 家族） | Solar Heatmap |
| Wind Analysis | 風場 + 行人舒適度報告（AIR 家族） | Wind Tunnel |

兩者結構**幾乎完全鏡像**。Solar Report 的 README 明確寫：「結構完全對齊 Wind Analysis」、「Mirror of wind.analysis」。**Solar Report 是較新版本**（命名更整潔 — `xx.report`）且多了 CSV export — **建議當作 canonical reference**。

---

## 1. 統一檔案結構

```
<subject>.report/                              ← solar.report 用這個
<subject>.analysis/                            ← wind.analysis 是舊命名（不要學）
├── README.md                                  (5-6 KB，固定章節 — 見下節)
├── config/
│   └── extension.toml                         (~500 bytes)
├── output/                                    ← Generated reports (auto-created, 不 commit)
│   └── <subject>_<param>_<YYYYMMDD>_<HHMMSS>/
│       ├── chart_01.png
│       ├── chart_02.png
│       ├── ...
│       ├── summary.json
│       └── units.csv  (僅 solar.report)
└── <subject>/report/   或   <subject>/analysis/
    ├── __init__.py                            (1 行: from .extension import XReportExtension)
    ├── extension.py                           (~25 KB — IExt + 主控 UI panel)
    ├── engine.py                              (~21-25 KB — matplotlib charts + stats + CSV)
    └── report_window.py                       (~12 KB — 彈出視窗 + 分頁 + 統計列)
```

| 檔案 | solar bytes | wind bytes |
|---|---:|---:|
| README.md | 5,235 | 5,863 |
| extension.toml | 505 | 494 |
| extension.py | 25,090 | 26,692 |
| engine.py | 25,138 | 21,010 |
| report_window.py | 12,576 | 12,643 |
| __init__.py | 45 | 46 |

---

## 2. extension.toml 統一格式

```toml
[package]
title       = "<Subject> Report"
description = "<中文一句話描述> · 對齊 ... 的獨立報告模式"
version     = "0.1.0"
category    = "twin-extensions"
authors     = ["metaarchetech"]

[dependencies]
"omni.usd"                  = {}
"omni.kit.uiapp"            = {}
"omni.kit.pipapi"           = {}              ← 用來自裝 matplotlib
"<sim_plugin>"              = {}              ← 對應 sim plugin

[[python.module]]
name = "<subject>.report"
```

---

## 3. extension.py 統一架構

### 3.1 lifecycle

```python
class XReportExtension(omni.ext.IExt):
    def on_startup(self, ext_id):
        global _INSTANCE
        _INSTANCE = self                     # ← Dashboard discovery 三件套
        self._ext_id = ext_id
        self._window = None                  # ← 主控 UI window
        self._report_window = None           # ← pop-out report window
        # ...參數預設值...
        self._running = False
        self._status = "Ready"
        self._last_output_dir = None
        # ...UI refs(label / button list)...
        mgr = omni.kit.app.get_app().get_extension_manager()
        self._ext_dir = mgr.get_extension_path(ext_id)   # ← output/ 寫到這裡
        # 加進 Window 選單
        self._menu_path = "Window/<Subject> Analysis"
        self._menu = omni.kit.ui.get_editor_menu().add_item(...)

    def on_shutdown(self):
        # destroy windows + remove menu

    def _open_window(self):                  # ← Dashboard VIEW 按鈕進入點
        if self._window is None: self._build_ui()
        self._window.visible = True
```

### 3.2 主控 UI 板（`width=340, height=520`）

從上而下垂直佈局：

```
┌──────────────────────────────┐
│       SOLAR ANALYSIS         │  ← build_header() — BG_HEADER 深底白字
│   Environmental Assessment   │
├──────────────────────────────┤
│ Status text(10px C_MID)      │  ← _set_status() 在每個 phase 更新
│ ──────────────────────────── │  ← _section_divider()
│ Scene Parameters             │  ← _section_label()
│ Preset  [SMALL][MEDIUM][LARGE] │  ← toggle buttons, active = C_ACCENT
│ Mode    [DAILY][LIVE]          │
│ Glass Ratio  [──●──] 0.60     │  ← _slider_float()
│ Floor H      [──●──] 3.0 m    │
│ ──────────────────────────── │
│ Ground Grid                  │
│ Grid Nx      [──●──] 40       │  ← _slider_int()
│ Grid Ny      [──●──] 40       │
│ ──────────────────────────── │
│ [Generate Analysis Report]   │  ← 主按鈕，C_GREEN bg，height=36
│ [View Last Report]           │  ← 副按鈕，藍灰 0xFF335577，只有當 _last_output_dir 存在才顯示
└──────────────────────────────┘
```

### 3.3 Generate 流程（async pipeline）

```python
async def _generate_async(self):
    self._running = True
    app = omni.kit.app.get_app()
    try:
        # Phase 1: load preset / params
        self._set_status("Loading preset...")
        await app.next_update_async()
        from <sim>.<module> import ...    # ← lazy import sim plugin
        # ...load...

        # Phase 2: compute (1+ steps，每步更新 status，讓 UI 不凍)
        self._set_status("Computing N units...")
        await app.next_update_async()
        # ...重型計算...

        # Phase 3: ensure matplotlib (auto-install via pipapi 首次執行)
        if not _ensure_charting_deps(self._set_status):
            return
        await app.next_update_async()
        from <subject>.report.engine import generate_report

        # Phase 4: write output to timestamped dir
        timestamp = time.strftime("%Y%m%d_%H%M%S")
        output_dir = os.path.join(self._ext_dir, "output",
                                  f"<subject>_<param>_{timestamp}")
        result = generate_report(..., output_dir=output_dir)
        self._last_output_dir = output_dir

        # Phase 5: pop out report window
        self._open_report(result["paths"], result["stats"], output_dir)

    except Exception as e:
        traceback.print_exc()
        self._set_status(f"Error: {e}", error=True)
    finally:
        self._running = False
```

### 3.4 matplotlib 自裝模板（關鍵 helper）

```python
def _ensure_charting_deps(status_cb=None):
    try:
        import matplotlib
        return True
    except ImportError: pass
    try:
        if status_cb: status_cb("First run: installing matplotlib (~30s)...")
        import omni.kit.pipapi
        omni.kit.pipapi.install("matplotlib", module="matplotlib")
        import matplotlib
        return True
    except Exception as e:
        carb.log_error(f"[<subject>.report] matplotlib install failed: {e}")
        if status_cb: status_cb(f"matplotlib install failed: {e}", error=True)
        return False
```

### 3.5 Design System 整合（雙寫法）

```python
try:
    from dashboard.theme import (
        BG_WIN, BG_HEADER, BG_INPUT, LINE_LIGHT,
        C_WHITE, C_PRIMARY, C_SECONDARY, C_TERTIARY, C_ACCENT, C_ERROR,
        FONT_TITLE, FONT_H2, FONT_BODY, FONT_LABEL, FONT_CAPTION,
        SP_S, SP_M, SP_L, MARGIN, RADIUS,
        H_HEADER, H_SLIDER, H_BUTTON, H_DIVIDER, H_SECTION_SP,
        LABEL_W, SLIDER_STYLE,
        btn_primary, btn_secondary,
        build_header, build_section_title, build_divider,
    )
except ImportError:
    # ...完整 inline fallback，所有 token 都用 hardcoded 值再實作 helpers...
```

> **Why**：每個 ext 都能獨立運作，即使 dashboard 沒載入也不會炸。Plugin 之間 dashboard 改 token，所有 report ext 都跟著動。

---

## 4. engine.py 統一架構

### 4.1 imports + 樣式常數（必須一致才能版面對齊）

```python
import json, os
from typing import List, Tuple
import numpy as np

import matplotlib
matplotlib.use("Agg")                          # ← Kit 環境必須（非互動）
import matplotlib.pyplot as plt
import matplotlib.patches as mpatches
from matplotlib.colors import LinearSegmentedColormap, BoundaryNorm

# 兩個 ext 共用的圖表常數（改了會破壞「報告系列視覺一致性」）
BG_COLOR   = "#1a1a2a"      # 深藍背景
TEXT_COLOR = "#e0e0e0"      # 淺灰文字
GRID_COLOR = "#2a2a3a"      # 格線
BLD_EDGE   = "#ffffff"      # 建築輪廓白
BLD_FACE   = (0.4, 0.4, 0.4, 0.35)  # 半透明灰填
FIG_W, FIG_H = 10, 8        # 10×8 inch
DPI = 150
WATERMARK = "<sim model name> · <param> · Preliminary Reference Only"
```

### 4.2 統一函式分區

| 區塊 | solar.report 函式 | wind.analysis 函式 |
|---|---|---|
| Helpers | `_setup_fig` `_draw_preset_buildings` `_add_watermark` `_save` | `_setup_fig` `_draw_buildings` `_draw_wind_arrow` `_add_watermark` `_save` |
| Custom colormaps | 6 段 LinearSegmentedColormap（冷→熱） | 同 |
| Chart 1 | `plot_ground_irradiance` | `plot_speed_map` |
| Chart 2 | `plot_shadow_hours` | `plot_amplification` |
| Chart 3 | `plot_comfort_map` | `plot_comfort` |
| Chart 4 | `plot_unit_gain`（grouped bar） | `plot_pressure` |
| Chart 5 | `plot_orientation_profile`（雙 panel） | `plot_vectors`（quiver） |
| extra | — | `plot_section` `plot_side_view`（垂直截面） |
| Stats | `compute_stats` → dict | `compute_stats` → dict |
| Export | `write_units_csv` ⭐（僅 solar） | — |
| Entry | `generate_report(...)` | `generate_report(...)` |

> **5 張圖是默契**（每個 report ext 提供 5 張），wind 多了截面圖到 7 張。

### 4.3 `generate_report()` 統一介面

```python
def generate_report(...,                       # subject-specific data inputs
                    output_dir,
                    ...) -> dict:
    """
    Returns:
        {
            "paths": {                         # ← 給 ReportWindow 用，key = tab id
                "ground":      "<output_dir>/ground_irradiance_map.png",
                "shadow":      "<output_dir>/shadow_hours_map.png",
                "comfort":     "<output_dir>/comfort_map.png",
                "unit_gain":   "<output_dir>/unit_gain_3d.png",
                "orientation": "<output_dir>/orientation_profile.png",
                "summary":     "<output_dir>/summary.json",
            },
            "stats": { ... }                   # ← summary.json 內容
        }
    """
    os.makedirs(output_dir, exist_ok=True)
    paths = {}
    paths["ground"] = plot_ground_irradiance(..., os.path.join(output_dir, "ground_irradiance_map.png"))
    paths["shadow"] = plot_shadow_hours(..., os.path.join(output_dir, "shadow_hours_map.png"))
    # ... etc ...
    stats = compute_stats(...)
    with open(os.path.join(output_dir, "summary.json"), "w", encoding="utf-8") as f:
        json.dump(stats, f, indent=2, ensure_ascii=False)
    paths["summary"] = ...
    if csv_export:                              # solar 才有
        write_units_csv(..., os.path.join(output_dir, "units.csv"))
        paths["csv"] = ...
    return {"paths": paths, "stats": stats}
```

### 4.4 summary.json 格式範例（wind）

```json
{
  "wind_dir": 45.0,
  "ref_speed_ms": 10.0,
  "z_height_cm": 150.0,
  "max_speed_ratio": 3.07,
  "max_speed_ms": 30.7,
  "max_speed_location": [-178.0, 988.0],
  "pct_comfortable": 0.0,
  "pct_acceptable": 0.1,
  "pct_uncomfortable": 3.3,
  "pct_dangerous": 96.6,
  "avg_pedestrian_speed_ms": 11.1,
  "hotspot_count": 4
}
```

格式：**flat dict，key 用 snake_case**，所有 % 用 0-100 數字而非 0-1 fraction。

---

## 5. report_window.py 統一架構

### 5.1 class ReportWindow

```python
class ReportWindow:
    def __init__(self, image_paths, stats, output_dir, title_info=""):
        # paths: dict from generate_report()
        # stats: dict from generate_report()
        # output_dir: 給 _open_folder() 用
        # title_info: 顯示在 header 的副標
```

### 5.2 視窗規格

- 大小：**960 × 720**
- 用 `move_to_new_os_window()` → **獨立 OS 視窗**（可拖到第二螢幕）
- 標題列：`build_header()` 用 BG_HEADER 深底白字
- Layout：左側 sidebar tab（寬 ~120px）+ 右側大圖（顯示當前 tab 的 PNG）+ 底部 stats bar

### 5.3 方法

| 方法 | 用途 |
|---|---|
| `_build_ui()` | 整個視窗 layout + 預設選 tab 0 |
| `_build_stats_bar()` | 底部黑色橫條，從 `self._stats` 取若干 key 做 `_stat_label()` |
| `_stat_label(text, warn=False)` | 單一統計字段（warn 用 C_RED 顯眼） |
| `_select_tab(idx)` | 切換 tab，刷新右側圖 |
| `_show_image(idx)` | 把對應 PNG 載到 `ui.Image` |
| `_open_folder()` | Windows: `os.startfile(output_dir)` |
| `_close()` / `destroy()` | 關閉清理 |
| `_bring_to_front()` | Windows ctypes `SetForegroundWindow`（避免被主 Kit 視窗蓋住） |
| `pop_out_async()` | 主入口：`_build_ui` → `move_to_new_os_window` → `_bring_to_front` |

### 5.4 Tab 配置（僅在 `paths` 有對應 key 才出現）

solar.report：`Ground / Shadow / Comfort / Unit / Orientation`
wind.analysis：`Speed / Amplification / Comfort / Pressure / Vectors / Section / Side`

---

## 6. README.md 統一章節結構

```markdown
# <Subject> Report

**<中文一句副標 — 強調獨立報告 / pop-out>**

<2-3 句說明：屬於哪個家族、對應哪個 sim plugin、按 Generate 一次產生什麼>

**不共享 state** — report 會重新計算，與 sim plugin 可同時或獨立運作。

---

## Features

### For Users
- <Preset 選擇器，N 種>
- <模式切換>
- <關鍵參數可調（slider）>
- 一鍵 Generate → 進度狀態逐步顯示 → 自動 pop out 到獨立 OS 視窗
- <N 個分頁 tab>，左側 sidebar 切換，右側顯示大圖
- Open Folder 按鈕直接開啟輸出目錄（Windows）
- View Last Report — 不重算，直接重新開上次結果

### For Developers
- 結構完全對齊 `<other>.report`
- Matplotlib Agg backend（Kit 環境必備）
- Colormap 與 wind.analysis 視覺一致
- 匯出目錄命名：`output/<subject>_<param>_<YYYYMMDD>_<HHMMSS>/`

---

## N Charts

| # | Filename | Content |
|---|---|---|
| 1 | `<chart_1>.png` | <說明> |
| ... |

外加：
- `summary.json` — 完整統計
- `units.csv`（若有）
```

---

## 7. 報告格式 — 不是 PDF / HTML，是「PNG + JSON + CSV + Pop-out viewer」

**重要**：現有 report ext **沒有產生 PDF 或 HTML**。報告是：

- 數張 **PNG 圖**（matplotlib `savefig` DPI=150）
- 一份 **summary.json**（統計 dict）
- **（solar 才有）** **units.csv**（逐戶 row）
- 全部寫到 `output/<subject>_<param>_<timestamp>/` 資料夾
- 「報告」即「資料夾本身」，viewer = pop-out window 把這幾張 PNG 用 tab 切換顯示

> 沒看到 reportlab / fpdf2 / weasyprint / jinja2。如果之後要產 PDF，要新增 dependency，跟既有兩個 ext 不一致。**初版維持 PNG+JSON+CSV 對齊，有需要再加 PDF stage 在 engine.py 後面**。

---

## 8. 跟 sim plugin 的整合點

| Report ext | sim plugin | 整合方式 |
|---|---|---|
| Solar Report | Solar Heatmap | `from solar.heatmap.demo_preset import load_demo_preset`<br>`from solar.heatmap.unit_model import build_project_from_preset`<br>`from solar.heatmap.gain_calculator import compute_all_units_*, compute_ground_grid_*, _get_sun_position` |
| Wind Analysis | Wind Tunnel | `from wind.tunnel.solver import sample_velocity_grid, sample_velocity_section` |

**關鍵原則**：

- **report 不重跑模擬，而是 import sim plugin 的 solver / sampler / calculator**
- 必要時 report 自己**讀 USD stage** 取得當下幾何（wind.analysis 從 `/World/WindTunnel/Building_*` 讀）
- import 失敗就 fail-loud（寫入 `_set_status` + status bar 紅字）

---

## 9. 報告系列視覺常數（別動）

| 常數 | 值 | 用途 |
|---|---|---|
| `BG_COLOR` | `#1a1a2a` | 深藍背景 |
| `TEXT_COLOR` | `#e0e0e0` | 淺灰文字 |
| `GRID_COLOR` | `#2a2a3a` | 格線 |
| `BLD_EDGE` | `#ffffff` | 建築輪廓白 |
| `BLD_FACE` | `(0.4, 0.4, 0.4, 0.35)` | 建築半透明灰填 |
| `FIG_W, FIG_H` | `10, 8` (inch) | matplotlib figure size |
| `DPI` | `150` | savefig dpi |
| ReportWindow 大小 | `960 × 720` | pop-out 視窗 |
| Main UI 大小 | `340 × 520` | 主控板 |

任何新 report ext 都應原封照搬這幾個常數，**不要自由發揮**，否則「一系列報告印出來」會看起來像不同團隊做的。

---

## 10. 為新 plugin 對齊用的 checklist

新通風 report ext 要走的決策：

1. **命名**：`<ns>.ventilation.report`（對齊 solar.report 命名）
2. **module 路徑**：`<ns>/ventilation/report/`
3. **class 名**：`VentilationReportExtension`
4. **menu path**：`Window/Ventilation Analysis`（注意 Wind/Solar 用「Analysis」，不是「Report」）
5. **dependency**：對應 ventilation sim plugin
6. **資料源**：從 ventilation tracer / particles / scene import 三個模組（或讀 USD stage 上的 inlet prim 屬性）
7. **5 張圖建議**（對應 ventilation 性質設計）：
   - `01 inlet_layout.png` — 進氣口配置俯視 + 法向量箭頭 + 開口面積標
   - `02 streamlines_plan.png` — 粒子流軌跡俯視
   - `03 streamlines_section.png` — 垂直截面（沿法向）
   - `04 coverage_heatmap.png` — 空間覆蓋率（粒子穿透密度）
   - `05 stagnation_zones.png` — 滯留區分析（粒子密度低於閾值的角落）
8. **summary.json 欄位**（建議）：
   - `ach`（Air Changes per Hour 估算）
   - `inlet_count`
   - `total_inlet_area_m2`
   - `avg_jet_speed_ms`
   - `coverage_pct`（空間覆蓋率）
   - `stagnation_pct`（滯留區比例）
   - `total_particles`
   - `recycle_rate_per_min`
9. **CSV**：跟 solar.report 一樣加 `inlets.csv`
10. **README**：照第 6 節結構寫
11. **目錄命名**：`output/ventilation_<config>_<YYYYMMDD>_<HHMMSS>/`

---

## 相關

- [[massing-web-pipeline]] — 上游幾何來源
- [[../omniverse/omniverse-web-feasibility]] — Web 化整體評估
