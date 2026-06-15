---
title: Ontology Canonical Format 選型
author: metaarchetech
created: 2026-04-17
updated: 2026-05-24
source: visustwin-vault
source_path: 04 Knowledge/Lived/Ontology/04 Canonical Format 選型.md
tags: [type/note, status/done, area/digital-twin, area/ontology, area/format]
---

# Canonical Format 選型

> 「用什麼語言寫 ontology？」這問題沒有唯一答案。取捨要看團隊、使用場景、外部交換需求。這篇比較主流選項，並記錄一個 **以 Pydantic 為核心** 的實務選擇（VisusTwin 案例）。

---

## 候選人

### A. OWL / RDF Turtle

**代表**：Brick Schema（原始檔）、BOT、ifcOWL

**長相**：

```turtle
@prefix bot: <https://w3id.org/bot#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .

bot:Room a rdfs:Class ;
    rdfs:subClassOf bot:Space ;
    rdfs:label "Room" ;
    rdfs:comment "A space inside a storey bounded by walls." .
```

**優點**
- 正統語意網，支援 SPARQL 查詢、OWL reasoning
- 對齊其他 ontology 零摩擦
- 學術界共通語言

**缺點**
- 工程師不愛，學習曲線陡
- 工具鏈（Protégé、rdflib）重
- 沒有 runtime type checking，要靠 SHACL 另外驗
- 序列化給 JSON 不自然

**何時選**
- 主要對外發布 ontology 供他人互通時
- 團隊有語意網經驗

---

### B. JSON Schema

**代表**：REST / MQTT payload 驗證標準

**長相**：

```json
{
  "$id": "https://example.com/schemas/Room.json",
  "title": "Room",
  "type": "object",
  "properties": {
    "urn": { "type": "string", "pattern": "^urn:visustwin:" },
    "class": { "const": "Room" },
    "floor_urn": { "type": "string" },
    "area": { "type": "number", "minimum": 0 },
    "room_type": { "enum": ["MasterBedroom", "LivingRoom", "Kitchen"] }
  },
  "required": ["urn", "class", "floor_urn"]
}
```

**優點**
- 跨語言，Node.js / Python / Go 都有 validator
- MQTT / REST payload 天然契合
- IDE 自動補全支援好

**缺點**
- 繼承關係靠 `allOf` 等組合，語法吵
- 類別階層表達弱
- 沒有「行為」— 只是資料殼

**何時選**
- 主要是資料交換協議，不太需要類別語意
- 團隊跨語言

---

### C. Pydantic (Python)

**代表**：FastAPI 生態、現代 Python data layer

**長相**：

```python
from pydantic import BaseModel, Field
from typing import Literal

class Room(Space):
    """
    A room inside a floor.

    Aligns with: bot:Space, brick:Room.
    """
    class_: Literal["Room"] = Field(default="Room", alias="class")
    floor_urn: URN
    area: float = Field(..., ge=0, description="Floor area in m^2")
    room_type: Literal["MasterBedroom", "LivingRoom", "Kitchen"] | None = None
```

**優點**
- 開發體驗超好：type hints、IDE、自動補全
- Runtime validation 免費，不用另寫 validator
- 一行 `Room.model_json_schema()` 就有 JSON Schema
- 繼承用 Python class `class Room(Space)`，自然
- 與 FastAPI / SQLModel / SQLAlchemy 生態無縫
- Omniverse extension 本來就是 Python

**缺點**
- Python-only（JSON Schema 匯出後 Node 可讀，但源檔不行）
- 不是「語意網」成員，OWL / SPARQL 零支援（要自己 export）
- 缺 ontology 傳統概念（reasoning、inference）

**何時選**
- 主要消費者是 Python
- 團隊 Python-heavy
- 需要 runtime 驗證 + schema export

---

### D. LinkML (YAML 中性 DSL)

**代表**：生物醫學 ontology（NIH、Monarch Initiative）

**長相**：

```yaml
id: https://example.com/linkml/core
name: twin-core
prefixes:
  bot: https://w3id.org/bot#
  brick: https://brickschema.org/schema/Brick#

classes:
  Space:
    description: Abstract spatial entity
    class_uri: bot:Space

  Room:
    is_a: Space
    class_uri: bot:Space
    slots:
      - floor_urn
      - area
      - room_type

slots:
  area:
    range: float
    minimum_value: 0
    unit: m2
  room_type:
    range: RoomTypeEnum

enums:
  RoomTypeEnum:
    permissible_values:
      MasterBedroom:
      LivingRoom:
      Kitchen:
```

**優點**
- **一源多輸出**：可產 Pydantic + JSON Schema + OWL + SHACL + SQL DDL
- YAML 好讀、好 diff
- 支援 IRI / prefix / ontology mapping 原生
- 與語意網和程式碼都相容

**缺點**
- 多一套 toolchain（`linkml`, `linkml-runtime`, `gen-pydantic`）
- 概念量比 Pydantic 大
- 團隊需要額外學習
- 生態比 Pydantic 小（2024 起才進入工程圈關注）

**何時選**
- 團隊中長期要維護 >100 類別的 ontology
- 需要產出多種正式 schema 格式
- 跨語言 + 正式語意網要求並存

---

### E. USD IsA API Schema

**代表**：OpenUSD 的自訂 schema 機制

**長相**：在 `plugInfo.json` + `.usda` 定義，`usdGenSchema` 生成 C++/Python 綁定

**優點**
- Omniverse 原生，在 USD 檔案裡直接驗證
- 可綁 UI attribute editor

**缺點**
- **超重工作流**：要 build tool、要發布 schema plugin
- 只在 Omniverse 環境有用，其他應用層吃不到
- 改 schema 要重 build
- 適合「已有穩定 ontology + 大量 USD 資產」的成熟階段

**何時選**
- Ontology 完全穩定（v2+）、使用者主要在 Omniverse
- 有專人負責 schema build

---

## 對照表

| | OWL | JSON Schema | Pydantic | LinkML | USD Schema |
|---|---|---|---|---|---|
| Runtime 驗證 | 否 | 是（via validator） | 是 | 是（生成 Pydantic） | 是（USD 端） |
| 語意網 / OWL export | 是 原生 | 否 | 否（要手寫） | 是 原生 | 否 |
| JSON Schema export | 否 | 是 原生 | 是 一行 | 是 原生 | 否 |
| Python 型別 / IDE | 否 | 是（生成） | 是 原生 | 是（生成） | 是（生成） |
| 跨語言 | 是 | 是是 | 透過 JSON Schema | 是是 | 否 |
| 學習曲線 | 高 | 低 | 低 | 中 | 高 |
| Tooling 成熟度 | 高（老） | 高 | 高 | 中 | 高（但重）|
| 團隊 Python-fit | 低 | 中 | **高** | 高 | 中 |
| 適合 v1 | 中 | 是 | **是** | 是 | 否 |

---

## 案例選擇（VisusTwin）：Pydantic

### 為什麼不是 LinkML（教科書答案）

LinkML 理論最漂亮，產業趨勢也好，但：
- 團隊小，多一套 toolchain 有學習成本
- 當前沒有 OWL export 需求（建商不會要）
- 跨語言需求只到 Python ↔ Node.js 層，Pydantic 產 JSON Schema 就夠
- 未來真需要時，Pydantic → LinkML 遷移不難（結構對應清楚）

**Pydantic 是實用主義選擇，LinkML 是未來選項。**

### 實際管線

```
twin_ontology/
└── *.py                 ← SOURCE OF TRUTH (Pydantic)
        ↓
     (Python import)     Omniverse extensions
        ↓
   model_json_schema()
        ↓
      schemas/*.json     ← Web / Node 端用
        ↓
    (未來) rdflib
        ↓
      twin.ttl           ← 若要對外發布 OWL 時
```

### 轉檔腳本

`scripts/export_json_schema.py`：

```python
from pathlib import Path
import json
from twin_ontology.spatial import Site, Building, Floor, Unit, Space
from twin_ontology.device import Sensor, Actuator, Observation
# ... all classes

OUT = Path(__file__).parent.parent / "schemas"
OUT.mkdir(exist_ok=True)

for cls in [Site, Building, Floor, Unit, Space, Sensor, Actuator, Observation]:
    schema = cls.model_json_schema()
    (OUT / f"{cls.__name__}.json").write_text(json.dumps(schema, indent=2))
```

跑 `python scripts/export_json_schema.py` → `schemas/Room.json` 給 Node 端用。

---

## 陷阱：Pydantic v1 vs v2

Omniverse Kit 環境可能混用。決策：**強制 Pydantic v2**（`pydantic >= 2.7`），`pyproject.toml` 寫死。v1 的 `dict()`, `json()`, `parse_obj()` 已棄用，別看網路舊教學。

---

## 相關

- [[05-tbox-v1-class-list]] — 實際 Pydantic 程式碼會長什麼樣
- [[../usd/usd-customdata-conventions]] — Pydantic 模型如何與 USD 配合
- Pydantic 官方文件：https://docs.pydantic.dev/latest/
- LinkML 官網：https://linkml.io/
