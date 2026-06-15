---
title: Ontology 對齊既有標準
author: metaarchetech
created: 2026-04-17
updated: 2026-05-24
source: visustwin-vault
source_path: 04 Knowledge/Lived/Ontology/03 對齊既有標準.md
tags: [type/note, status/done, area/digital-twin, area/ontology, area/standards]
---

# 對齊既有標準

> [[00-industry-landscape]] 是鳥瞰圖，這篇是**逐欄位對照表**。目的：讓 Digital Twin 類別 docstring 能正確標註 `Aligns with: bot:Space`，未來要 export OWL 時不出洋相。

---

## BOT (Building Topology Ontology)

- 官方網址：https://w3c-lbd-cg.github.io/bot/
- namespace：`https://w3id.org/bot#`
- 風格：**極簡**，只定義拓撲骨架
- RDF 檔約 80 行 Turtle

### 核心類別

| BOT IRI | 中文 | Digital Twin 對應 |
|---|---|---|
| `bot:Zone` | 區域（抽象父類） | `Entity` （root） |
| `bot:Site` | 案場 / 基地 | `Site` |
| `bot:Building` | 建物 | `Building` |
| `bot:Storey` | 樓層 | `Floor` |
| `bot:Space` | 空間（房間、走廊、中庭） | `Space` / `Room` |
| `bot:Element` | 建築元件（牆、門、設備） | `Element` / `Device` |
| `bot:Interface` | 元件介面（牆之間的連接） | *暫不對應* |

### 核心關係

| BOT 屬性 | 語意 | Digital Twin |
|---|---|---|
| `bot:hasBuilding` | Site 含 Building | `Site.buildings: list[Building]` |
| `bot:hasStorey` | Building 含 Floor | `Building.floors: list[Floor]` |
| `bot:hasSpace` | Storey 含 Space | `Floor.spaces: list[Space]` |
| `bot:hasElement` | Zone 含 Element | `Space.elements: list[Element]` |
| `bot:containsZone` | 空間包含關係 | 透過 parent URN 推導 |
| `bot:adjacentZone` | 鄰接關係 | *選配，`Space.adjacent: list[URN]`* |

### 借用方式

- 類別結構**完全對齊**，名稱刻意相同（`Space`、`Floor`、`Building`）便於對照
- 屬性名**稍改**（用 Python-friendly 的 snake_case 複數，不用 `hasX` 前綴）
- docstring 寫：`Aligns with bot:Space (W3C LBD)`
- 未來 OWL export 時加 `rdfs:subClassOf bot:Space`

### 與 IFC 的關係

BOT 是 IFC 的「簡化版拓撲子集」。若要完整對應 BIM，改用 ifcOWL — 但複雜度飆升十倍。Digital Twin 一般場景**不需要 IFC 複雜度**，BOT 夠用。

---

## Brick Schema

- 官網：https://brickschema.org/
- namespace：`https://brickschema.org/schema/Brick#`
- GitHub：https://github.com/BrickSchema/Brick
- 風格：**感測點位 (points) 導向**，類別深且結構化

### 類別階層（節選）

```
brick:Class
├── brick:Equipment
│   ├── HVAC_Equipment
│   │   ├── Air_Handler_Unit (AHU)
│   │   ├── Chiller
│   │   └── ...
│   ├── Lighting_Equipment
│   └── ...
├── brick:Location
│   ├── Floor
│   ├── Room
│   └── Zone
├── brick:Point
│   ├── Sensor
│   │   ├── Temperature_Sensor
│   │   ├── Illuminance_Sensor
│   │   ├── CO2_Sensor
│   │   └── Occupancy_Sensor
│   ├── Setpoint
│   ├── Command
│   ├── Status
│   └── Parameter
└── brick:Tag
```

### Digital Twin 對應

| Digital Twin | Brick |
|---|---|
| `Sensor(kind="lux")` | `brick:Illuminance_Sensor` |
| `Sensor(kind="temperature")` | `brick:Temperature_Sensor` |
| `Sensor(kind="humidity")` | `brick:Humidity_Sensor` |
| `Sensor(kind="occupancy")` | `brick:Occupancy_Sensor` |
| `Actuator(kind="light")` | `brick:Luminaire` + `brick:Lighting_Command` |
| `Room(type="MasterBedroom")` | `brick:Bedroom` （Brick 細分度不如自家應用層）|

### 借用策略

- **不要**全盤繼承 Brick 類別樹（它太細，建物用不到 100 種 Equipment）
- **借用** point/sensor kind 命名：`brick:Illuminance_Sensor` → 寫 `kind="illuminance"`（改 snake_case）
- docstring 標 `Aligns with brick:Illuminance_Sensor`

---

## SOSA / SSN (W3C)

- 官網：https://www.w3.org/TR/vocab-ssn/
- namespace：`http://www.w3.org/ns/sosa/`
- 風格：**觀測 (observation) 導向**，動態資料模型

### 核心類別

| SOSA IRI | 語意 | Digital Twin |
|---|---|---|
| `sosa:Sensor` | 感測器（硬體） | `Sensor` |
| `sosa:Actuator` | 致動器 | `Actuator` |
| `sosa:Platform` | 乘載平台（閘道器、控制箱） | `Device` / `Gateway` |
| `sosa:Observation` | 一次觀測事件 | `Observation` |
| `sosa:FeatureOfInterest` | 被觀測的對象（例如「房間的溫度」） | 透過 `observation.target_urn` 表達 |
| `sosa:ObservableProperty` | 被觀測的屬性（溫度、照度本身） | 透過 `sensor.kind` |

### 為什麼配 Brick 用

- **Brick 說**：「這個點位是 Illuminance_Sensor，位於 Room X」（**靜態身份**）
- **SOSA 說**：「`obs_001` 是 `sensor_007` 在 `2026-04-17T10:23:45Z` 對 `feature_room_x_illuminance` 的觀測，值為 245 lux」（**動態事件**）

兩個拼起來 = 完整的 smart building 感測模型。

### Observation 範例

```python
class Observation(Entity):
    sensor_urn: URN          # sosa:madeBySensor
    target_urn: URN          # sosa:hasFeatureOfInterest
    observed_property: str   # sosa:observedProperty, e.g. "illuminance"
    result: float            # sosa:hasSimpleResult
    ts: datetime             # sosa:resultTime
```

---

## IFC / ifcOWL

- 官網：https://technical.buildingsmart.org/
- ifcOWL：https://technical.buildingsmart.org/standards/ifc/ifc-formats/ifcowl/
- 風格：**極重**，萬能但艱澀

### 與 Digital Twin 的距離

- **不在 T-Box 階段對齊**
- 當建商給 `.ifc` 檔時，用 IfcOpenShell 萃取 → 映射到 BOT → 進自家 T-Box
- 只對齊到 **Site/Building/Storey/Space** 這幾個頂層類別，以下深度用自有類別
- IFC 的 MEP（機電管路）、結構分析細節暫不吃

### 常用對應

| IFC | BOT | Digital Twin |
|---|---|---|
| `IfcSite` | `bot:Site` | `Site` |
| `IfcBuilding` | `bot:Building` | `Building` |
| `IfcBuildingStorey` | `bot:Storey` | `Floor` |
| `IfcSpace` | `bot:Space` | `Space` / `Room` |
| `IfcWall` | `bot:Element` | *不展開* |
| `IfcDoor` | `bot:Element` | *不展開* |

---

## SAREF / SAREF4BLDG

- 官網：https://saref.etsi.org/
- 風格：**家電互通**，歐盟偏好

### Digital Twin 對應

| SAREF | Digital Twin |
|---|---|
| `saref:Device` | `Device` |
| `saref:Sensor` | `Sensor`（與 Brick/SOSA 重疊，用 Brick 對就好）|
| `saref:Actuator` | `Actuator` |
| `saref4bldg:Building` | `Building`（與 BOT 重疊）|

**結論**：**不優先對齊 SAREF**。未來若接 Matter/Thread 家電，只需 export 時多加一組 rdfs 對映。

---

## DTDL (Microsoft Azure)

- GitHub：https://github.com/Azure/opendigitaltwins-dtdl/
- 風格：JSON-LD，類別定義清楚

### 不對齊理由

- Azure 生態鎖定
- 類別 naming 偏 Microsoft 風格，不照 W3C 慣例
- 離開 Azure 無工具支援

### 什麼時候看

- 客戶要求 Azure Digital Twins 整合時 — 寫一層 export
- 當學習範例：`DTDL` 的 Smart Places Ontology（基於 RealEstateCore）文件寫得好

---

## 周邊標準（可選）

### CityGML

- OGC 標準，城市級 GIS
- 類別：`Building`、`CityObjectGroup`、`Road`
- **適合都市孿生**，不是單棟建物
- 未來若做社區 / 多案場整合可考慮

### Schema.org (RealEstateListing)

- 不是 ontology，是搜尋標記
- 建案網站 SEO 用，可產出 JSON-LD 給 web demo 用
- namespace：`https://schema.org/`

### FIWARE Smart Data Models

- 歐盟推，跨領域（Building、Energy、Mobility）
- 用 JSON Schema + NGSI-LD
- **不主動對齊**，但若接 FIWARE broker 有現成 schema 可借

---

## 對齊優先順序

```
必對齊 (寫死 docstring)
  ├── BOT              空間拓撲
  ├── Brick            感測點位類別
  └── SOSA             觀測事件模型

可選對齊 (export-time 才做)
  ├── ifcOWL           BIM 匯入時
  ├── SAREF            家電互通時
  └── DTDL             Azure 客戶要求時

不對齊 (除非明確需求)
  ├── CityGML
  ├── FIWARE Smart Data Models
  └── Industry Ontology Foundry (IOF)
```

---

## 相關

- [[00-industry-landscape]]
- [[05-tbox-v1-class-list]] — 每個類別都會標 `Aligns with: ...`
- [[../omniverse/omniverse-revit-2026]] — IFC → USD 流程
