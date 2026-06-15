---
title: Digital Twin T-Box v1 類別清單（VisusTwin 案例）
author: metaarchetech
created: 2026-04-17
updated: 2026-05-24
source: visustwin-vault
source_path: 04 Knowledge/Lived/Ontology/05 VisusTwin T-Box v1 類別清單.md
tags: [type/note, status/done, area/digital-twin, area/ontology, area/t-box]
---

# Digital Twin T-Box v1 類別清單（VisusTwin 案例）

> 本篇是一份 v1 最終類別階層與欄位定義範例。實作時，Pydantic 程式碼依此表一對一展開。
> 本案例針對「樣品屋 / 智慧建築展示」場景，可作為 Digital Twin T-Box 設計的參考實例。

---

## 類別總覽

```
Entity                                (root, 抽象)
├── SpatialEntity                      (有位置的實體)
│   ├── Site                           案場 / 基地
│   ├── Building                       建物
│   ├── Floor                          樓層
│   ├── Unit                           單元（戶）
│   └── Space                          空間（房間、走廊）
│       └── Room                       有明確功能的房間（subtype of Space）
├── Device                             (設備)
│   ├── Sensor                         感測器
│   ├── Actuator                       致動器
│   └── Gateway                        資料閘道
├── Observation                        觀測事件（短壽命）
├── Showcase                           (展示導向)
│   ├── SampleHome                     整個樣品屋
│   ├── ShowcaseFeature                展示亮點（單一可解說項目）
│   ├── MaterialOption                 建材選項
│   └── FurnitureItem                  家具 / 陳設
├── AnalysisScenario                   (分析場景)
│   ├── WindScenario                   風場模擬
│   └── SolarScenario                  日照模擬
└── InteractionAsset                   (互動 / 敘事)
    ├── POI                            導覽興趣點
    ├── Route                          導覽路線
    └── ExhibitionBoard                展板
```

---

## 基底：Entity

所有類別繼承自 `Entity`，提供共通欄位。

| 欄位 | 型別 | 必填 | 說明 |
|---|---|---|---|
| `urn` | `URN` | 是 | 全域唯一身份（見 [[02-identity-urn-design]]）|
| `class_` | `str` | 是 | 類別名，配合 JSON 序列化的 `class` 鍵 |
| `display_name` | `str` | 是 | 展示用名稱，可含中文 |
| `description` | `str \| None` | | 長描述 |
| `tags` | `list[str]` | | 自由標籤 |
| `created_at` | `datetime` | 是 | 首次建立 |
| `updated_at` | `datetime` | 是 | 最後更新 |
| `metadata` | `dict[str, Any]` | | 開放擴充欄位（open-world 逃生艙） |

**Aligns with:** 無特定對應。設計哲學是所有 entity 都先是「有身份的東西」，才細分類別。

---

## SpatialEntity 分支

所有空間類別共同繼承 `SpatialEntity`，增加位置欄位：

| 欄位 | 型別 | 必填 | 說明 |
|---|---|---|---|
| `parent_urn` | `URN \| None` | | 階層父節點（Building 的 parent = Site）|
| `geometry_ref` | `str \| None` | | USD prim path 或其他幾何引用 |
| `bounds` | `AABB \| None` | | 軸對齊包圍盒（可選，算分析用）|
| `area` | `float \| None` | | 面積 m^2（若適用）|
| `volume` | `float \| None` | | 體積 m^3（若適用）|

### Site

- **Aligns with:** `bot:Site`, `IfcSite`
- **語意:** 案場 / 基地（建案）。
- **URN:** `urn:<ns>:{site_slug}:_:_:_:site:{site_slug}`
- **特有欄位:** `coordinate: GeoPoint | None`（WGS84），`address: str | None`

### Building

- **Aligns with:** `bot:Building`, `IfcBuilding`
- **URN:** `urn:<ns>:{site}:{building_slug}:_:_:building:{building_slug}`
- **特有欄位:** `floor_count: int`, `year_built: int | None`

### Floor

- **Aligns with:** `bot:Storey`, `IfcBuildingStorey`
- **URN:** `urn:<ns>:{site}:{building}:_:_:floor:{floor_slug}`（floor slug 建議 `F1`, `F2`, `B1`）
- **特有欄位:** `level_index: int`（樓層數字，地下為負），`elevation: float`（公尺）

### Unit

- **Aligns with:** 無直接對應（案例獨有；IFC 有 `IfcSpatialZone` 但不緊貼）
- **語意:** 樓層裡的「戶」— 一棟樓可切成 A 戶 B 戶
- **URN:** `urn:<ns>:{site}:{building}:{unit_slug}:_:unit:{unit_slug}`
- **特有欄位:** `unit_type: Literal["showroom", "real_sale"]`（是樣品屋還是實售戶）

### Space / Room

- **Aligns with:** `bot:Space`, `IfcSpace`, `brick:Room`
- **語意:** `Space` 是所有內部空間通用父類；`Room` 是「有明確功能的房間」子類
- **URN:** `urn:<ns>:{site}:{building}:{unit}:{room_slug}:room:{room_slug}`
- **Room 特有欄位:**
  - `room_type: RoomType`（枚舉，下方）
  - `expected_occupancy: int | None`
  - `adjacent_room_urns: list[URN]`

**`RoomType` 枚舉：**
`MasterBedroom / Bedroom / LivingRoom / DiningRoom / Kitchen / Bathroom / Powder / Foyer / Corridor / Balcony / Study / UtilityRoom / Other`

---

## Device 分支

### Device（基底）

| 欄位 | 型別 | 說明 |
|---|---|---|
| `located_in_urn` | `URN` | 所在 Space 的 URN（SOSA `sosa:Platform` 類似）|
| `manufacturer` | `str \| None` | 製造商 |
| `model` | `str \| None` | 型號 |
| `serial_number` | `str \| None` | 序號 |
| `firmware_version` | `str \| None` | |

### Sensor

- **Aligns with:** `brick:Sensor`, `sosa:Sensor`
- **特有欄位:**
  - `kind: SensorKind`（枚舉）
  - `unit: str`（SI 單位或自訂）
  - `sampling_rate_hz: float | None`

**`SensorKind` 枚舉**（對齊 Brick）：
`illuminance / temperature / humidity / co2 / occupancy / motion / door_contact / sound_level / air_quality / power / custom`

### Actuator

- **Aligns with:** `brick:Equipment`, `sosa:Actuator`
- **特有欄位:**
  - `kind: ActuatorKind`
  - `state_schema: str`（指向 JSON Schema 文件）

**`ActuatorKind` 枚舉:**
`light / blind / hvac / door_lock / audio / display / custom`

### Gateway

- **Aligns with:** `sosa:Platform`
- **特有欄位:** `protocol: Literal["mqtt", "osc", "modbus", "bacnet"]`, `endpoint: str`

---

## Observation（動態資料）

- **Aligns with:** `sosa:Observation`
- **語意:** 一次感測事件。**一般不會 persist 進 ontology 實例資料庫**，而是進 time-series DB / MQTT payload。本類別只提供**格式契約**。

| 欄位 | 型別 | 必填 | 說明 |
|---|---|---|---|
| `sensor_urn` | `URN` | 是 | 發出觀測的 sensor |
| `target_urn` | `URN \| None` | | 觀測對象（如房間）；省略則代表 self |
| `observed_property` | `str` | 是 | 對應 sensor.kind |
| `result` | `float \| str \| bool` | 是 | 觀測值 |
| `unit` | `str \| None` | | SI 單位 |
| `ts` | `datetime` | 是 | 觀測時間 |
| `quality` | `Literal["ok", "estimated", "bad"]` | | 品質標記 |

---

## Showcase 分支（案例獨有）

### SampleHome

- **Aligns with:** 無（案例獨有）
- **語意:** 一間完整樣品屋（對應 Unit with `unit_type="showroom"`）
- **URN:** `urn:<ns>:{site}:{building}:{unit}:_:sample-home:{slug}`
- **特有欄位:**
  - `unit_urn: URN`（對應的 Unit）
  - `sqm: float`（權狀坪數換算）
  - `features: list[URN]`（ShowcaseFeature 列表）
  - `available_material_options: list[URN]`
  - `hero_pov_urn: URN | None`（主推的 POI）

### ShowcaseFeature

- **語意:** 要在導覽時強調的單一特色（例如「整面落地窗」「中島廚房」）
- **特有欄位:**
  - `category: Literal["architectural", "material", "smart", "layout", "view"]`
  - `priority: int`（導覽排序）
  - `anchor_urn: URN`（附著在哪個 Space / Element）
  - `narrative: str`（給導覽員或 TTS 的解說文本）
  - `media_refs: list[str]`（圖片 / 影片 URL）

### MaterialOption

- **語意:** 客戶可選的建材版本（木地板 A / B、磁磚 A / B）
- **特有欄位:**
  - `scope: Literal["floor", "wall", "ceiling", "cabinet", "countertop"]`
  - `applies_to_urns: list[URN]`（哪些 Space 可換）
  - `usd_material_path: str`（Omniverse 材質路徑）
  - `supplier_urn: URN | None`（連回 catalog 供應商）
  - `price_tier: Literal["standard", "upgrade", "premium"]`

### FurnitureItem

- **語意:** 陳列家具（不同於建材，是可搬動的物件）
- **特有欄位:**
  - `category: Literal["sofa", "bed", "dining_table", "chair", "lighting", "decor", "appliance"]`
  - `placed_in_urn: URN`（所在 Space）
  - `supplier_urn: URN | None`
  - `usd_asset_path: str`

---

## AnalysisScenario 分支

### AnalysisScenario（基底）

- **語意:** 「一次分析的參數設定」— **不是分析結果本身**。結果檔（PNG、JSON）指標留在 extension 本地（例如 `wind_analysis/output/`）
- **欄位:**
  - `target_urn: URN`（被分析的 Site / Building / Space）
  - `run_ts: datetime`
  - `output_dir: str`（相對路徑指向結果）
  - `parameters: dict[str, Any]`

### WindScenario

- **Aligns with:** 鬆散對應 `sosa:Procedure`
- **特有欄位:**
  - `wind_direction_deg: float`
  - `wind_speed_ms: float`
  - `reference_height_m: float`

### SolarScenario

- **特有欄位:**
  - `date: date`
  - `time_of_day: time`
  - `latitude: float`, `longitude: float`
  - `timezone: str`

---

## InteractionAsset 分支

### POI (Point of Interest)

- **語意:** 導覽上停留點
- **特有欄位:**
  - `located_at_urn: URN`（Space 或 ShowcaseFeature）
  - `camera_pose: CameraPose`（位置 + look-at + fov）
  - `dwell_seconds: float`
  - `narrative: str | None`

### Route

- **語意:** 有序的 POI 列表，構成一次完整導覽路徑
- **特有欄位:**
  - `poi_urns: list[URN]`（有序）
  - `total_duration_seconds: float`
  - `loop: bool`

### ExhibitionBoard

- **語意:** 3D 場景中的展板
- **特有欄位:**
  - `placed_in_urn: URN`
  - `content_schema: Literal["image", "markdown", "video", "comparison"]`
  - `content_refs: list[str]`

---

## URN 模板備忘表

| 類別 | URN 範例 |
|---|---|
| Site | `urn:visustwin:site-a:_:_:_:site:site-a` |
| Building | `urn:visustwin:site-a:tower-1:_:_:building:tower-1` |
| Floor | `urn:visustwin:site-a:tower-1:_:_:floor:F2` |
| Unit | `urn:visustwin:site-a:tower-1:showroom:_:unit:showroom` |
| Space / Room | `urn:visustwin:site-a:tower-1:showroom:mbr:room:MBR` |
| Sensor | `urn:visustwin:site-a:tower-1:showroom:mbr:sensor:lux-007` |
| Actuator | `urn:visustwin:site-a:tower-1:showroom:mbr:actuator:hue-007` |
| SampleHome | `urn:visustwin:site-a:tower-1:showroom:_:sample-home:home1` |
| ShowcaseFeature | `urn:visustwin:site-a:tower-1:showroom:lr:feature:floor-to-ceiling-window` |
| MaterialOption | `urn:visustwin:site-a:tower-1:showroom:_:material-option:oak-a` |
| FurnitureItem | `urn:visustwin:site-a:tower-1:showroom:mbr:furniture:bed-007` |
| POI | `urn:visustwin:site-a:tower-1:showroom:lr:poi:hero-view-01` |
| Route | `urn:visustwin:site-a:tower-1:showroom:_:route:default` |
| WindScenario | `urn:visustwin:site-a:_:_:_:wind-scenario:summer-ne-3ms` |

---

## 對應實作檔案結構（參考）

```
ontology_pkg/twin_ontology/
├── __init__.py           re-exports everything
├── core.py               Entity, URN (parse/build/to_mqtt_topic/to_osc)
├── spatial.py            SpatialEntity, Site, Building, Floor, Unit, Space, Room
├── device.py             Device, Sensor, Actuator, Gateway, Observation
├── showcase.py           SampleHome, ShowcaseFeature, MaterialOption, FurnitureItem
├── analysis.py           AnalysisScenario, WindScenario, SolarScenario
├── interaction.py        InteractionAsset, POI, Route, ExhibitionBoard, CameraPose
├── enums.py              RoomType, SensorKind, ActuatorKind...
└── usd_bridge.py         read/write customData helpers
```

---

## 相關

- [[02-identity-urn-design]] — URN 規則
- [[03-aligning-with-standards]] — 類別對齊細節
- [[04-canonical-format-selection]] — Pydantic 實作方式
- [[../usd/usd-customdata-conventions]] — 類別如何在 Omniverse 場景標註
