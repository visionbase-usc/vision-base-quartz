---
title: OTA 120 — 建築5.0 的智慧靈魂
version: v6 (8頁·內容驅動結構)
date: 2026-04-12
status: 草稿 · 待確認後匯出 .pptx
partners:
  - 寶舖建設
  - WELLTEK
  - NVIDIA Omniverse
tags:
  - OTA120
  - DigitalTwin
  - Omniverse
  - Welltek
  - 簡報
---

> **2026-05-19 重新定位**：寶舖 = **主線 B（BIM/Digital Twin）的市場端落地**。不是 anchor case，但仍是落地專案，6/6 deliverable 仍交付。詳見 [[00 系統/Entity_Map]]。

> **2026-05-19 重新對位**：USD ontology 為**雙軌共用層** — 主線 A (Physical AI world model) + 主線 B (BIM/DT 空間表示)。不獨佔任一條。

> 每個 `## Slide X` = 一張投影片。設計語言：Midnight Executive — navy `#1E2761` 底、ice `#CADCFC` 文字與線條。
> 確認後告訴我「可以匯出」，即轉為 `.pptx`。

---

## Slide 1 — 封面

**版面**：全出血深色背景，左對齊主文，右下角三方標誌列

- **主標**：`OTA 120`
- **副標**：全透明 · 全健築 · 全生命週期
- **tagline**：建築5.0 的智慧靈魂
- **副線**：感知 · 建模 · 進化 · 體驗
- **英文副線**：Sensing · Modeling · Evolving · Experiencing
- **合作方**：寶舖建設 × WELLTEK × NVIDIA Omniverse
- **版號**：`v6 · 2026.04`　**標註**：CONFIDENTIAL

---

## Slide 2 — 核心承諾 · 全透明 × 全健築 × 全生命週期

**版面**：左欄三段承諾 + 右欄三個大數字卡片

**主標**：CENTENNIAL BUILDING PASSPORT · 建築不再只是不動產，而是持續進化的智慧載體

**三項承諾**

**全透明**
每一次維修、升級、工法均被記錄於 OpenUSD 數位孿生。業主、建商、下一代住戶都能查驗建築的完整履歷——不是靠紙本保固書，而是可驗證的資料層。

**全健築**
WELLTEK AIoT 感測器持續監測室內空氣、溫熱、水質、聲學、能耗，對應 WELL Building Standard 五大指標。住宅的健康狀態是即時量化的數字，不是銷售說詞。

**全生命週期**
設計 BIM 的幾何資料與竣工後的感測資料共享同一個空間座標系。建築從圖面到維運，是連續的數據，不是斷裂的檔案交接。

**三個大數字（右欄）**

| 數字 | 說明 |
|---|---|
| `120` | 年可追溯建築數位履歷 |
| `12` | 項 WELLTEK 即時感測維度 |
| `∞` | 次 OTA 無線軟體迭代 |

---

## Slide 3 — 感知層 · Welltek × AIoT × MQTT 資料管道

**版面**：左側 WELL 指標儀表板 + 右側資料管道流程圖

**主標**：SENSING LAYER · 建築的神經系統

**核心說明**：感知層是整個系統的資料源頭。WELLTEK 12-in-1 感測器產生的即時數據，經由標準 MQTT 協議進入系統，成為數位孿生的持續輸入，也是未來 ML 預測模型的訓練基礎。這不是概念中的「AIoT」，而是已落地的資料管道。

**資料管道（右側）**

```
WELLTEK 12-in-1 IoT Sensors
  └─ MQTT :1883
       └─ Mosquitto Broker
            └─ Node.js server
                 └─ SQLite  ←─ 歷史資料存檔
                      └─ REST :3000 / WebSocket :3001
                           └─ React Dashboard（即時顯示）
```

**五大感測維度 × WELL認證對應（左側）**

| 感測維度 | 即時值範例 | WELL 指標 |
|---|---|---|
| 空氣品質 PM2.5 / CO₂ | PM2.5: 8 µg/m³ | WELL Air |
| 溫熱舒適 Temp / RH | 26.2 °C · 58% RH | WELL Thermal |
| 用水水質 TDS | TDS: 142 ppm | WELL Water |
| 聲學環境 dBA | 38 dBA | WELL Sound |
| 能源消耗 kWh | 3.2 kWh/hr | WELL Light |

**底部標語**：`12 in 1 Sensor · 1 Hz Sampling · 24/7 Uptime · WELL認證資料基礎`

---

## Slide 4 — 建模層 · BIM × OpenUSD × 空間資料整合

**版面**：上方生命週期時間軸 + 下方兩欄技術說明

**主標**：MODELING LAYER · 單一真相來源

**生命週期時間軸**（權重比例呈現「維運才是主戰場」）

```
設計 BIM ─10%─→ 施工監管 ─30%─→ 交屋確認 ─5%─→ 維運傳承 ─55%─→
```

**左欄 · 空間資料如何整合**

BIM 幾何（IFC / USD）匯入 Omniverse 後轉為 OpenUSD Stage，建立毫米級精準的空間座標系。感知層的每一筆感測資料都帶有這個座標系的位置參數——PM2.5 數值不只是一個數字，它知道自己在建築的哪個角落。

- IFC / Revit → USD 轉換 → Omniverse Nucleus 儲存庫
- 管線位置、設備座標與感測器部署位置對齊
- 維修紀錄附加於 USD Prim，不可竄改

**右欄 · 為什麼用 OpenUSD**

USD（Universal Scene Description）是 NVIDIA 與 Pixar 共同推動的 3D 世界標準，支援跨工具互通（Revit / Rhino / Omniverse / UE5），並透過 `carb.eventBus` 實現 Omniverse Kit 擴充模組的即插即用。

`Built on OpenUSD — Industry Standard for 3D Worlds`

---

## Slide 5 — 進化層 · OTA × 全生命週期 × 數位孿生持續學習

**版面**：中央 OTA 循環圖示 + 左右兩欄說明文字

**主標**：EVOLUTION LAYER · 可進化的建築不會老化

**核心敘事**：傳統建築交屋後就開始折舊。OTA 120 的邏輯相反——建築交屋後，數據才開始真正累積，系統才開始真正學習。

**OTA 更新機制（左欄）**

「OTA」不只是軟體更新的比喻，而是真實的運作邏輯：
- 感測資料持續寫入 SQLite，建立設備健康基線
- 異常模式被 Node.js server 偵測後觸發維護通知
- 數位孿生中的設備狀態隨物理世界同步更新
- 管理端可透過 React Dashboard 下發配置變更，推送至建築系統

**數位孿生持續學習（右欄）**

當前資料管道的設計，是以未來 ML 模型為前提建構的：
- SQLite 歷史資料 = 訓練資料集的原始來源
- 感測維度的時間序列 = 預測性維護的特徵工程基礎
- OpenUSD Stage 的空間語意 = 空間 AI 的結構化輸入

今天的資料管道，是明天 AI Digital Twin 的地基。

**OTA UPDATE 狀態示意**

- `SCHEDULED` — 硬體維護通知 · 提前 72 小時預警
- `AVAILABLE` — 軟體效能優化 · 無需停機即可套用
- `COMPLETED` — 空調系統校正 · 附完整操作紀錄

---

## Slide 6 — 體驗層 · UE5 × iPad × 空間即介面

**版面**：左側 iPad UI 截圖 + 右側 UE5 主螢幕截圖，中間通訊協議標示

**主標**：EXPERIENCE LAYER · 空間本身就是最好的介面

**核心說明**：體驗層不是獨立的展示系統，而是整個感知—建模—進化管道的前端輸出。iPad 上顯示的 PM2.5 數值來自 WELLTEK 感測器；UE5 場景的燈光反應了數位孿生的當前狀態。

**延遲指標**：`< 80 ms  iPad → UE5 端對端控制延遲`

**控制管道（中間標示）**

```
iPad Safari → React UI
  └─ WS :9000 ──→ bridge.js
       ├─ OSC / UDP ──→ UE 5.6 :8000   (場景 · 燈光 · 動畫)
       └─ OSC / UDP ──→ Omniverse :8001 (孿生狀態同步)
```

**四大居住情境（感測資料驅動，非純展示）**

| 情境 | 觸發 WELLTEK 指標 | UE5 場景回應 |
|---|---|---|
| HEALTH 健康 | PM2.5 ↓ · CO₂ ↓ | 清晨自然光 · 植栽動態 |
| ECO 節能 | 能耗曲線即時下降 | 遮陽聯動 · 溫控調適 |
| CINEMA 影音 | 照度 ↓ · 噪音 ↓ | 燈光漸暗 · 聲學模式 |
| SLEEP 睡眠 | CO₂ ↓ · 溫熱舒適達標 | 月光氛圍 · 全靜音 |

---

## Slide 7 — 技術架構 · 四層真實整合系統

**版面**：navy 底，四層水平堆疊，ice blue 箭頭，右側 Security 垂直帶貫穿全層

**主標**：SYSTEM ARCHITECTURE · FOUR-LAYER INTEGRATION

**Layer 1 · 感知層 Sensing**

```
WELLTEK 12-in-1 IoT Devices
  └─ MQTT :1883 ──→ Mosquitto Broker ──→ Node.js server ──→ SQLite
```

**Layer 2 · 資料層 Data**

```
Node.js server
  ├─ REST API :3000 ──→ React Dashboard (歷史查詢 · 報表)
  └─ WebSocket :3001 ──→ React Dashboard (即時推播)
```

**Layer 3 · 數位孿生層 Digital Twin**

```
Omniverse Kit
  ├─ mqtt.bridge        ← 接收 Layer 1 感測資料
  ├─ osc.controller     ← 接收 Layer 4 iPad 指令
  ├─ exhibition.board   ← 展示場景狀態管理
  ├─ warp.windtunnel    ← 物理模擬運算
  └─ dashboard          ← 視覺化輸出
       └─ carb.eventBus ──→ OpenUSD Stage (BIM + Sensor 共享座標系)
```

**Layer 4 · 體驗層 Experience**

```
UE 5.6 ArchVizExplorer (Lumen GI)
  +
iPad Safari → React UI
  └─ WS :9000 ──→ bridge.js
       ├─ OSC / UDP ──→ UE :8000
       └─ OSC / UDP ──→ Omniverse :8001
```

**右側垂直帶**：`Security / Identity · TLS · Auth Token · Audit Log (不可竄改)`

---

## Slide 8 — 封底 · 建築5.0 的起點

**版面**：全出血深色背景，居中大標，底部 CTA + 三方標誌

**主視覺大字**：`建築5.0`

**主標**：全透明 · 全健築 · 可進化

**副標**
寶舖建設 × WELLTEK × NVIDIA Omniverse
聯手定義台灣豪宅的下一個百年標準

**CTA**：`下一步 · Let's build the first one together.`

**聯絡資訊**：（email / 窗口 placeholder）

**標註**：CONFIDENTIAL

---

# 匯出備註

- Slide 3、Slide 6 需要實拍或截圖素材（iPad UI 近拍、UE5 主螢幕）；未備妥時以灰框佔位。
- 技術架構圖（Slide 7）的 code block 為內容規格，建議由設計師以向量圖重製。
- Roadmap 與 KPI 已從本版移除（內容不足以獨立成頁）；如後續需要可補充為附錄頁。

---

## Related

- [[寶舖品牌概念對照表]] — 寶舖建設品牌語言 × OTA120 對應
- [[01 專案/OTA120/MOC|OTA120 MOC]]
