---
title: Identity 與 URN 設計
author: metaarchetech
created: 2026-04-17
updated: 2026-05-24
source: visustwin-vault
source_path: 04 Knowledge/Lived/Ontology/02 Identity 與 URN 設計.md
tags: [type/note, status/done, area/digital-twin, area/ontology, area/identity]
---

# Identity 與 URN 設計

> 一個 ontology 沒做身份系統 ≈ 沒做。因為**同一個東西在多個系統裡叫不同名字**就是跨系統整合的第一號痛點。

---

## 現狀：多套 ID 併存

Digital Twin 平台典型場景下，一個「主臥」在多個子系統裡的身份：

| 系統 | 身份形式 | 範例 |
|---|---|---|
| Omniverse USD | prim path | `/World/SampleHome_A/F2/Room_MBR` |
| MQTT | topic 片段 | `<ns>/siteA/showroom/f2/mbr` |
| Catalog SQLite | PK integer | `rooms.id = 42` |

問題：
- 無法只憑其中一個找到另外兩個
- 每改一個系統的 naming rule，另外兩個要手動同步
- 新加一個子系統（例如某 OSC controller 的 address）又多一套
- **語意比對** （「這個 42 跟那個 mbr 是同一個嗎？」）變成每個工程師要重新解的問題

---

## URN 解法

給每個「跨系統可識別」的實體一個 **canonical URN**，各系統把 URN 當作 business key。

### 格式

```
urn:<ns>:{site}:{building}:{unit}:{room}:{entity_type}:{entity_id}
```

規則：
- 全小寫，只允許 `[a-z0-9_-]`
- 階層用 `:` 分隔，與 URN spec 對齊
- 缺層用 `_` 佔位（例如 site 級的設備沒 room 可填）
- `entity_type` 使用 ontology 類別名的 kebab-case（`Sensor` → `sensor`）
- `entity_id` 供應商給的 serial，沒有就 UUID7 或 nanoid

### 範例（以 namespace `visustwin` 為案例）

| 實體 | URN |
|---|---|
| 案場 A | `urn:visustwin:siteA:_:_:_:site:siteA` |
| 案場 A 棟 A 的樣品屋 | `urn:visustwin:siteA:buildingA:showroom:_:sample-home:home1` |
| 樣品屋二樓 | `urn:visustwin:siteA:buildingA:showroom:_:floor:F2` |
| 樣品屋主臥 | `urn:visustwin:siteA:buildingA:showroom:mbr:room:MBR` |
| 主臥的照度感測器 | `urn:visustwin:siteA:buildingA:showroom:mbr:sensor:lux-007` |
| 主臥的展示材質選項 | `urn:visustwin:siteA:buildingA:showroom:mbr:material-option:wood-a` |

**注意**：URN 本身不是 URL，不會去「解析到資源」，它只是個**唯一字串標識符**。要從 URN 找到實體資料，要靠 adapter 層。

---

## 各載體的 Adapter 規則

### USD (Omniverse)

每個 prim 的 `customData` 必有 `<ns>:urn` key：

```python
prim.SetCustomDataByKey("visustwin:urn", "urn:visustwin:siteA:buildingA:showroom:mbr:room:MBR")
prim.SetCustomDataByKey("visustwin:class", "Room")
```

詳見 [[../usd/usd-customdata-conventions]]。

### MQTT

Topic pattern 由 URN 直譯：

```
urn:visustwin:siteA:buildingA:showroom:mbr:sensor:lux-007
    ↓ (把 `:` 換成 `/`，去掉 `urn:` 前綴)
visustwin/siteA/buildingA/showroom/mbr/sensor/lux-007
```

有 payload 時：

```
Topic:   visustwin/siteA/buildingA/showroom/mbr/sensor/lux-007/state
Payload: { "urn": "urn:visustwin:siteA:...:lux-007", "value": 245.3, "ts": "2026-04-17T10:23:45Z" }
```

Payload 也帶 `urn` 欄位 — 訂閱者可以不管 topic 結構，直接從 payload 認實體。

### Catalog SQLite

不動既有 PK，新增 `urn TEXT UNIQUE` 欄位：

```sql
ALTER TABLE suppliers ADD COLUMN urn TEXT UNIQUE;
ALTER TABLE products  ADD COLUMN urn TEXT UNIQUE;
CREATE INDEX idx_products_urn ON products(urn);
```

應用層在 insert / upsert 時填 URN，查詢可用 PK 或 URN。

### OSC

OSC address 直接從 URN 截段：

```
urn:visustwin:siteA:buildingA:showroom:mbr:light:hue-007
    ↓ (取 entity_type + entity_id)
/visustwin/light/hue-007/brightness → 0.6
```

若要全限定，用完整 URN：

```
/visustwin/entity  "urn:visustwin:siteA:...:hue-007"  0.6
```

---

## Migration 策略

舊系統（MQTT topic、catalog 既有資料、USD 既有場景）不可能一夜換完。分三階段：

### Phase 1：只規格化，新系統強制

- 寫死 URN 規則到 ontology doc
- 新寫的 extension、新 migration、新 data ingestion **一定用 URN**
- 舊系統維持不動

### Phase 2：加 URN 欄位

- SQLite 加 `urn` column
- 場景存檔時，extension 把現有 prim path 依規則轉 URN 寫進 `customData`
- MQTT：新 publisher 帶 payload `urn` 欄位；舊訂閱者忽略即可

### Phase 3：URN 當 primary key（長期）

- 新的跨系統 API 只接受 URN 查詢
- 舊 business logic 逐步改以 URN 而非內部 PK 為準
- 完全自動化後，舊 ID 保留為 internal only

---

## URN ↔ 程式碼 helper

可在 ontology Python package 提供：

```python
from twin_ontology import URN

# 解析
urn = URN.parse("urn:visustwin:siteA:buildingA:showroom:mbr:room:MBR")
urn.site         # "siteA"
urn.entity_type  # "room"
urn.entity_id    # "MBR"

# 構建
urn = URN.build(site="siteA", building="buildingA", unit="showroom",
                room="mbr", entity_type="sensor", entity_id="lux-007")
str(urn)  # "urn:visustwin:siteA:buildingA:showroom:mbr:sensor:lux-007"

# 轉 topic / osc
urn.to_mqtt_topic()   # "visustwin/siteA/buildingA/showroom/mbr/sensor/lux-007"
urn.to_osc_address()  # "/visustwin/sensor/lux-007"

# 繼承關係
urn.parent()          # urn:...:showroom:mbr:_:_ (Room 的父節點是 Unit)
urn.ancestors()       # [..., site, building, unit, room]
```

---

## 常見陷阱

### 不要把易變資訊放 URN

❌ `urn:visustwin:siteA:buildingA:showroom:mbr:supplier-007:...`（包供應商）
→ 以後換供應商 URN 就變了，**broken link 大災難**

✅ URN 只放**結構性位置身份**，供應商屬性放類別欄位。

### 不要拿展示名當 URN

❌ `urn:visustwin:案場A:A棟:樣品屋:主臥:...`
→ 中文、空格、大小寫變化會爆

✅ 用穩定 slug (`siteA`, `mbr`)，展示名放 `displayName` 欄位。

### 版本化 URN？不要

實體不是 API。房間不會有 v2，就是搬走或拆掉 → 發一個新 URN、舊的標 `deprecated: true`。

---

## 相關

- [[01-tbox-vs-abox]] — URN 是把 T-Box 與多個 A-Box 黏起來的膠水
- [[../usd/usd-customdata-conventions]] — USD 側 URN 寫入細節
- [[05-tbox-v1-class-list]] — 每個類別的 URN pattern
- RFC 8141 URN spec: https://www.rfc-editor.org/rfc/rfc8141
