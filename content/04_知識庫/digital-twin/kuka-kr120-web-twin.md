---
title: KUKA KR 120 R2700 Web Digital Twin（robot_web）
author: metaarchetech
created: 2026-06-04
updated: 2026-06-04
source: visionbase-usc/VB_kuka_KR120_r2700
tags: [type/note, status/wip, area/digital-twin, area/web, area/robotics]
---

# KUKA KR 120 R2700 — Web Digital Twin

> 一個免 npm 的 **Three.js + FastAPI** 工業手臂 web 控制 / 視覺化介面,含正逆運動學、時間參數化軌跡、可達性分析,並打通 **Grasshopper → 即時匯入** 的工作流。
>
> Repo:**visionbase-usc/VB_kuka_KR120_r2700**(branch `main`)。本機:`C:\Users\User\Documents\chrono\robot_web\`。

---

## 1. 這是什麼

針對 **KUKA KR 120 R2700**(QUANTEC 2700 機構)的 web 3D 顯示 + 控制介面,概念上對標 visose/Robots。可以:

- 即時拖動 A1–A6 / TCP,看 FK 與 TCP 位姿
- 匯入 Rhino-Plane TCP 目標點 → 自動解 IK → 畫出路徑 → 播放
- 分析關節角、可操作度(manipulability)、加速度合規
- 從 **Grasshopper 一鍵推送**目標點直接進 viewer(免手動匯入)

> ⚠️ 模型來源:KR 120 R2700 無公開 URDF,採用 ROS-Industrial / kroshu 的 **kr210_r2700_2**(同 QUANTEC 2700 機構 → 運動學完全相同,只差負載)。STL mesh vendored 在 `meshes/kr210_r2700_2/visual/`。

## 2. 架構

```
robot_web/
├── backend/         Python / FastAPI(純 numpy 運動學)
│   ├── app.py            REST + WebSocket API
│   ├── robot_model.py    FK、damped-least-squares 數值 IK、極限、plane_to_matrix
│   ├── trajectory.py     梯形速度時間軸(PTP 同步 / LIN TCP 速度)
│   ├── configs.py        枚舉可達 IK 配置 + KUKA Status 位元
│   └── chrono_bridge.py  從 Project Chrono sim 串流關節角(demo/chrono 模式)
├── frontend/        Three.js(CDN import map,無 npm)
│   ├── js/main.js        主程式:場景、UI、IK、播放、WebSocket、渲染設定
│   ├── js/robot.js       建模
│   ├── js/analysis.js    KUKA|prc 風分析圖
│   └── assets/           官方 logo
├── grasshopper/     GhPython 元件(轉檔 + 推送)
├── meshes/          STL
└── samples/         測試路徑 JSON
```

## 3. 運動學與軌跡

- **IK**:多起點 damped-least-squares 數值解(無解析閉式)。匯入時逐點以前一點為 seed,路徑連續。
- **時間參數化**(`trajectory.py`):梯形速度,用真實關節 vel/accel 極限。
  - **PTP** = 同步(最慢軸決定時間),由速度倍率 % 控制(cycle time ∝ 1/override,accel ∝ override²)
  - **LIN** = 在 TCP 速度 (mm/s) 下走梯形;某軸超速時自動時間拉伸
- **可達配置 / KUKA Status**(`configs.py`):對一個 TCP 位姿枚舉所有可達 IK 解,各標 3-bit Status(s0 基本/過頂、s1 肘、s2 腕=A5 號)。此機 A2 極限(−140°…−5°)通常排除過頂 → 一般 8 取 4。

## 4. ⚠️ 加速度計算陷阱(曾是真 bug)

加速度 = 二階導數;在密集路徑上用相鄰取樣 `d²q/dt²` 會把 IK / JSON 捨入抖動放大約 `1/dt²`(≈28000×)→ 假性「超限」(低速時更糟,因雜訊地板與速度無關)。

**修法**(`computePathAccel()`):用固定**弧長視窗**,`acc = (∂²q/∂s²)·v²` —— 曲率在固定笛卡兒跨距上量測(抗雜訊)× v²(正確的速度² 縮放)。

> 產生「合規」測試路徑時,要用「**模擬真實匯入**」(FK→IK→弧長視窗 accel)驗證,**不能**用乾淨設計的關節角(會藏住雜訊)。`samples/test_verified_5000.json` 即完整驗證檔。

## 5. Grasshopper 整合(本專案特色)

**目標:** Rhino/GH 設計目標平面 → 一鍵進 web viewer。

### 5.1 JSON 格式(`/api/targets/import` 吃的)

```jsonc
{
  "units": "mm",                 // mm|cm|m|in,只縮放 origin
  "frame": "world",
  "base_transform": [[...4x4...]],   // 選填,world→base
  "targets": [
    { "origin": [x,y,z], "xaxis": [x,y,z], "yaxis": [x,y,z], "motion": "joint" },
    { "origin": [x,y,z], "xaxis": [x,y,z], "yaxis": [x,y,z], "motion": "linear", "speed": 250 }
  ]
}
```

- **Z 軸不用給**:後端自動 `Z = X × Y`(右手系)
- 軸不必正規化/正交:後端 normalize + Gram-Schmidt
- 不可達點不丟棄,標 `converged:false`

### 5.2 兩個 GhPython 元件(`grasshopper/`,Rhino 8 Python 3 / CPython)

| 元件 | 功能 | 輸入 → 輸出 |
|---|---|---|
| `VB_TargetExporter.py` | Rhino Planes → 上述 JSON;可寫檔 | `Planes/Motion/Units/BasePlane/Speed/Path/Write/Pretty` → `Json/Saved/Count/Info` |
| `VB_PushToRobot.py` | 把 JSON POST 進後端 inbox | `Json/Url/Send` → `Status/Seq/Ok` |

**GhPython 坑(踩過):**
1. 輸入是用「輸入腳名字」當變數讀;**腳本內的函式要把輸入當參數傳進去**(別靠全域查找,Rhino 8 CPython 注入方式不一定進 `globals()`)。
2. **第一個輸出腳是保留的 `out`(console)**,不綁回傳變數 → 值輸出要放在第 2 腳以後。
3. 輸出腳名稱**大小寫要一致**(`Ok` ≠ `OK`)。
4. CPython 用 `urllib.request`(不能用 IronPython 的 `System.Net`)。
5. `Planes` 設 **List Access**;單一物件要容忍(`_aslist`)。

### 5.3 後端推送收件匣(in-memory 單槽)

- `POST /api/targets/push`:存 payload + seq++,回 `{seq}`
- `GET /api/targets/inbox`:回 `{seq, data}`
- 前端 `pollGhInbox()` 每 1.5s 輪詢,seq 進位就跑 `importData()` 自動匯入(播放中暫停輪詢)
- 重啟後端會清空(單人工具)

## 6. 前端顯示行為(現況)

- **一律輕量繪製**:`computePath()` 不再呼叫 `/api/path` 逐段 IK(會在「不可達點跳接的長 LIN 段」上卡死主執行緒);改為**相鄰目標關節線性內插**(~30 取樣/秒、每段上限 200 步)→ 平滑播放、不卡。
- **目標點以單點顯示**(GPU Points,可達白/不可達紅),移除了 sphere 白球 + 接近箭頭。
- 匯入完顯示**匯入耗時**;不可達跳接以**非阻塞警告**提示(檢查 BasePlane/單位/工作範圍)。
- 取捨:關節內插平滑且必過每個解算姿態,但 LIN 段 TCP 不保證數學上完全直線(視覺接近)。要嚴格 LIN 直線需回後端逐段 IK(就是會卡的那條)。

## 7. 環境渲染

- ACES Filmic tone mapping、PMREM **RoomEnvironment IBL**(金屬反光)、**Reflector 鏡面地板** + `ShadowMaterial` 陰影層、漸層背景 + Fog。
- 浮動 **⚙ 顯示設定面板**(JS 動態建,免動 index.html):曝光 / 環境光(驅動 `envMapIntensity`,比 `scene.environmentIntensity` 跨版本可靠)/ 反射地板(調 Reflector color uniform)/ 點大小 / 霧 / 網格。
- 黑色基座圓盤上移到 z=0 之上(`makePedestal()` 加 `g.position.z = H`)。

## 8. 前端快取規則(重要)

`index.html` 以 no-cache 提供;CSS/JS 用 `?v=N` query。**改 JS 一定要 bump N**。`robot.js`/`analysis.js`/`pet.js` 由 `main.js` import 時也帶 `?v=N`,改它們要連 import 行一起 bump。目前 `main.js?v=45`。

> conda-run stdout 在非 ASCII(cp950)與 /tmp 路徑(git-bash vs Windows python)會卡 → 測試 JSON 寫 repo 目錄、純 ASCII。

## 9. 如何啟動

需 [[../tools/dev/dev|conda run]]:

```bash
conda run -n chrono python -m uvicorn app:app --app-dir robot_web/backend --host 127.0.0.1 --port 8000
```

開 `http://127.0.0.1:8000`。完整啟動 / 陷阱 / 待辦見 repo `README.md`(§1、§6、§9)。

## 10. 待辦

- 工具 / TCP offset 模型(目前無)
- CIRC / SPLINE 運動(目前僅 PTP / LIN)
- 解析閉式 IK(目前多起點數值)
- Project Chrono 整合(即時串流)、真實 KRC 連線(RWD)

## 跨領域連結

- Repo 文件鏡像:[[../../05_GitHub_Repos/VB_kuka_KR120_r2700|VB_kuka_KR120_r2700]]
- 屬 Lab 主軸 B **Digital Twin**:[[digital-twin]]
- 上游幾何來源(Rhino/GH)思路可參考 [[bim/massing-web-pipeline]]
- 未來 Chrono / Omniverse 串流參考 [[omniverse/omniverse]]
