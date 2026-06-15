---
title: Digital Twin Ontology 產業現況與生態圖譜
author: metaarchetech
created: 2026-04-17
updated: 2026-05-24
source: visustwin-vault
source_path: 04 Knowledge/Lived/Ontology/00 產業現況 與 生態圖譜.md
tags: [type/note, status/done, area/digital-twin, area/ontology, area/standards]
---

# 產業現況 與 生態圖譜

> 數位孿生 ontology 的現實不是「一個標準打天下」，而是 **分領域壟斷**。各垂直自己有 de facto standard，互不相讓，任何一個宣稱「通用」的其實都只在自己地盤強。

---

## 領域 × 標準對應表

| 領域 | 事實標準 | 主管機構 | 誰在用 | 與 Digital Twin 的關係 |
|---|---|---|---|---|
| **BIM / 建築設計** | IFC + ifcOWL | buildingSMART | Autodesk Revit、ArchiCAD、政府採購 | 建商給 .rvt / .ifc 時接 |
| **智慧建築營運** | Brick Schema | UC Berkeley / LBNL | Google、Microsoft、Siemens 商辦 | **感測器、HVAC、照明最實用** |
| **物聯網感測** | SOSA / SSN | W3C | 學術界、少數電信 | 配 Brick 用 |
| **通用建築拓撲** | BOT | W3C Linked Building Data CG | 歐洲 BIM+web 研究 | 輕量容器，補 IFC 不足 |
| **家電 / 設備** | SAREF / SAREF4BLDG | ETSI (EU) | 歐盟法規要求的互通家電 | 未來接 Matter/Thread 可考慮 |
| **雲端孿生 PaaS** | DTDL | Microsoft | Azure Digital Twins 客戶 | 雲鎖定，除非用 Azure |
| **幾何場景** | OpenUSD | Pixar / Alliance for OpenUSD | NVIDIA Omniverse、Apple Vision Pro | **已在用** |
| **通用網頁語意** | Schema.org | Google / 搜尋業者 | SEO、結構化資料 | 與建築孿生無關 |

---

## 各標準詳述

### IFC (Industry Foundation Classes)

- 建築 / BIM 的大標準，EXPRESS schema 定義，檔案副檔名 `.ifc`
- **ifcOWL** 是 IFC 的 OWL 翻版，讓 BIM 模型能用 SPARQL 查
- **超重**：類別數千個、版本不穩（IFC2x3 / IFC4 / IFC4.3 互有差異）
- 適用：政府案、大型建設公司、Revit 匯出管線
- **對 Digital Twin 案例：** 不直接用，但建商若給 IFC 模型，要有能力萃取空間拓撲（Building → Storey → Space）

### Brick Schema

- UC Berkeley 2016 起的 open-source 專案，專攻 **商辦智慧樓宇的感測器與點位** (points)
- 類別清晰：`Sensor` / `Setpoint` / `Command` / `Status` / `Parameter`，每個又細分溫度、濕度、CO2 等
- 用 Turtle (RDF) 定義，但也有 Python 驗證器
- **實戰最成熟** 的建築 ontology，大廠在用
- **對 Digital Twin 案例：** 第一順位對齊對象。`Sensor` 在 docstring 標注 `≈ brick:Sensor`

### SOSA / SSN (W3C)

- **SOSA** (Sensor, Observation, Sample, Actuator) 是「最小核心」
- **SSN** (Semantic Sensor Network) 是 SOSA 的完整版，含 `System`、`Deployment`、`Procedure`
- 與 Brick 配合：Brick 說「這個點位是什麼」(identity)，SOSA 說「這個點位此刻觀測到什麼」(observation)
- **對 Digital Twin 案例：** 配 Brick 用，`Observation` 的概念直接借來當 runtime telemetry payload

### BOT (Building Topology Ontology)

- W3C Linked Building Data Community Group，刻意寫得**超簡**
- 只管拓撲：`Site` / `Building` / `Storey` / `Space` / `Element` 和 `containsZone` / `hasElement` 等關係
- 優點：輕、穩、跟 ifcOWL / Brick / SOSA 可以拼
- **對 Digital Twin 案例：** 空間階層結構直接對齊 BOT

### SAREF / SAREF4BLDG

- ETSI (歐洲電信標準協會) 的 Smart Applications REFerence ontology
- 定位在 **家電互通**，SAREF4BLDG 擴充到建築環境
- 歐盟部分法規會引用
- **對 Digital Twin 案例：** 建案需求不急，但若接 Matter / Thread / HomeKit 家電孿生值得看

### DTDL (Digital Twins Definition Language)

- 微軟為 **Azure Digital Twins** 發明的 JSON-LD 方言
- 類別系統清楚，有工業、智慧城市官方 model 資源
- **雲鎖定**：離開 Azure 幾乎沒生態
- **對 Digital Twin 案例：** 不用。但它的文件寫得非常好，值得當**學習範例**看「雲廠商怎麼做孿生 ontology」

### OpenUSD

- 嚴格講**不是 ontology**，是**場景描述 + schema 系統**，但提供 ontology 能掛進來的底座
- 可以用 **IsA API schema** 自定語意類別，或輕量用 **customData dict** 塞 metadata
- **對 Digital Twin 案例：** 第一版走 customData（見 [[../usd/usd-customdata-conventions]]），穩定後考慮升級 IsA schema

### Schema.org

- Google / Bing / Yahoo 為 SEO 發起，描述網頁結構化內容
- 與建築孿生沒啥關係，但若要讓 **建案展示頁** 被 Google 吃出豐富結果卡，可加 `RealEstateListing` 型別
- **對 Digital Twin 案例：** 周邊加值，非核心

---

## 冷門但值得知道的

- **FIWARE Smart Data Models** — 歐盟推的跨領域模型（含 Building、City、Energy 等），免費。有時當 SAREF 的替代。
- **CityGML** — 城市級 GIS，OGC 標準。適合做**都市級**孿生，不是單棟建物。
- **gbXML** — 建築能耗模擬用，Revit 能匯出。
- **Industry Ontology Foundry (IOF)** — 製造業孿生，共通 upper ontology 嘗試。走得慢。

---

## 殘酷事實

**沒有任何建商會跟你說「我們用 ifcOWL」**。真實工作流長這樣：

1. 建築師交 `.rvt` / `.ifc` → 轉 USD 進 Omniverse（**幾何**）
2. 自家小 ontology 標註「這是主臥、這是建材選項 A」（**語意**）
3. 感測器走 Brick / 自家 MQTT（**運行時**）
4. 自家 catalog DB 存供應商產品目錄（**商業**）

大型標準對應用面而言是「**需要時再對齊**」的目標，不是「一開始就全吃」的起點。

---

## Digital Twin（VisusTwin 案例）對齊策略

| 案例類別 | 結構對齊 | namespace 歸屬 |
|---|---|---|
| `Site` / `Building` / `Floor` / `Unit` / `Space` | BOT | 自有 namespace |
| `Sensor` / `Actuator` / `Observation` | Brick + SOSA | 自有 namespace |
| `SampleHome` / `ShowcaseFeature` / `MaterialOption` | *(無對應，案例獨有)* | 自有 namespace |
| `POI` / `Route` / `ExhibitionBoard` | *(無對應)* | 自有 namespace |
| `WindScenario` / `SolarScenario` | 鬆散對齊 SOSA Procedure | 自有 namespace |

**寫法慣例**：Python class docstring 第一行註明 `Aligns with: bot:Space, brick:Room`，但類別本身繼承自有 `Entity` base。未來需要 OWL export 時，rdflib serializer 補 `rdfs:subClassOf` 即可。

---

## 相關

- [[01-tbox-vs-abox]]
- [[03-aligning-with-standards]] — 本文是概覽，那篇是深入到欄位對應
- [[../omniverse/omniverse-revit-2026]] — IFC → USD 具體流程
- W3C BOT spec: https://w3c-lbd-cg.github.io/bot/
- Brick Schema: https://brickschema.org/
- SOSA/SSN: https://www.w3.org/TR/vocab-ssn/
- DTDL v3: https://github.com/Azure/opendigitaltwins-dtdl/
