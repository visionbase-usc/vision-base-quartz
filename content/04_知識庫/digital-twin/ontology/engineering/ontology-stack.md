---
title: Ontology 工程堆疊
author: metaarchetech
created: 2026-04-16
updated: 2026-05-24
source: visustwin-vault
source_path: 04 Knowledge/Lived/Ontology-Engineering/Ontology 工程堆疊.md
tags: [type/note, status/done, area/digital-twin, area/ontology, area/engineering, area/architecture]
---

# Ontology 工程堆疊

> Ontology 不神秘，它是一個堆疊的頂層。本篇拆解 Layer 0 到 Layer 7 的關係，以及實戰選型。

---

## 一、概念堆疊（從 raw data 到 ontology）

```
Layer 7 · Domain Ontology（產業特化，如 FIBO / GoodRelations）
Layer 6 · Ontology（TBox — 類型、規則、推理）        ← 這才是「ontology」
Layer 5 · Knowledge Graph（ABox — 實例資料）
Layer 4 · Semantic Model（三元組 subject-predicate-object）
Layer 3 · Object Model（類別 + 方法）
Layer 2 · Relational / Graph Model（實體 + 關係）
Layer 1 · Schema（類型定義）
Layer 0 · Record / Row（資料本身）
```

---

## 二、必須分清楚的核心概念

### TBox vs ABox（描述邏輯核心區分）

| | TBox | ABox |
|---|---|---|
| 名稱 | Terminology Box | Assertion Box |
| 定義什麼 | **什麼是什麼** | **這個是那個** |
| 範例 | `Building 是 Asset 的一種` | `Building_A 是 Building` |
| 角色 | **Ontology 本身** | **資料本身** |

### Ontology vs Taxonomy vs Knowledge Graph

| 概念 | 結構 | 範例 |
|---|---|---|
| Taxonomy | 樹（單一階層） | 生物分類、圖書 Dewey |
| Schema | 欄位定義 | `CREATE TABLE` |
| Ontology | 圖 + 規則 + 推理 | OWL, Palantir Ontology |
| Knowledge Graph | Ontology + 實際資料 | Google KG, Wikidata |

### Open World vs Closed World 假設

- **Closed World**（SQL 世界）：沒在 DB 裡 = 假
- **Open World**（OWL 世界）：沒在 DB 裡 = 未知，不是假
- **選錯會 bug 一整年**。企業內通常 Closed World，跨組織交換必須 Open World。

---

## 三、工程實踐的 6 個 Layer

### Layer A · 儲存（Storage）

| 方案 | 代表產品 | 特性 |
|---|---|---|
| Triple Store (RDF) | Apache Jena, GraphDB, Virtuoso | 標準互通，效能較弱 |
| Property Graph DB | **Neo4j**, Dgraph, TigerGraph, ArangoDB | 效能好，語法人性化 |
| SQL + graph | Postgres + recursive CTE / ltree | 低成本啟動 |
| 文件 + 向量 | MongoDB / Elasticsearch + pgvector | AI 時代常搭配 |
| 純檔案 | YAML / JSON + NetworkX | 原型、Git 友善 |

### Layer B · Ontology 定義語言

| 語言 | 用途 | 難度 |
|---|---|---|
| **RDF / RDFS** | W3C 標準三元組 | 中 |
| **OWL** | 最強，支援公理與推理 | 高 |
| **JSON-LD** | JSON 的 Linked Data | 低 |
| **SHACL / ShEx** | 約束驗證（類 TypeScript） | 中 |
| **SKOS** | 簡單分類學 | 低 |
| **Schema.org** | Web 最廣泛的通用 ontology | 低 |
| **Pydantic / dataclass / Zod** | 程式內 ontology | 低 |
| **LinkML** | 新興 schema + evolution | 中 |

### Layer C · 查詢（Query）

| 語言 | 對應儲存 | 範例 |
|---|---|---|
| **SPARQL** | RDF | `SELECT ?b WHERE { ?b rdf:type :Building }` |
| **Cypher** | Neo4j | `MATCH (b:Building) RETURN b` |
| **GQL** | ISO 2024 新標準 | 各家開始支援 |
| **Gremlin** | TinkerPop | `g.V().hasLabel("Building")` |
| **GraphQL** | API 層（非 graph DB） | `{ building(id:1){ name } }` |

### Layer D · 推理（Reasoning）

- **OWL Reasoner**：HermiT, Pellet, ELK — 邏輯嚴謹但慢
- **Rule Engine**：Drools, Jess — 商業規則
- **Graph Algorithm**：PageRank, community detection, shortest path
- **LLM + function calling**：2023 後新做法，ontology 變成 tool schema，**最大的範式轉移**

### Layer E · 驗證 & 一致性

- **SHACL**：用 shapes 驗證資料符合 ontology
- **OWL consistency check**：邏輯一致性
- **Schema evolution**：LinkML / Git + SemVer

### Layer F · 編輯 & 視覺化

- **Protégé**（Stanford，學術標準）
- **WebProtégé**（線上協作）
- **TopBraid Composer**（企業級）
- **Gephi / Cytoscape / yEd**（圖視覺化）

---

## 四、2026 年實戰的三條主流技術路徑

### 路徑 1 · 學術標準派（重、慢、互通強）

```
OWL + Protégé → Apache Jena → SPARQL endpoint → HermiT reasoner
```

- 用途：公共資料、跨組織交換、學術研究
- 不適合：商業快速迭代

### 路徑 2 · 企業務實派（Palantir 類型）

```
Neo4j / Dgraph → Cypher + GraphQL API → Pydantic schema → 自建 reasoner
```

- 用途：企業 KG、Palantir 風格產品
- **90% 商業 ontology 系統長這樣**

### 路徑 3 · AI 原生派（2023 後爆發）

```
Postgres + pgvector → Pydantic ontology → LLM function calling → LangGraph agent
```

- 用途：LLM 驅動應用、原型、RAG + KG 混合
- **2026 新創最常用的組合** — LLM 處理模糊語意匹配，正式 ontology 處理嚴謹邏輯

---

## 五、工程上必遇到的 6 個硬議題

1. **IRI/URI 命名**：每個 Object 要全域唯一 ID
    - 例：`https://example.com/ontology/Building#A123` 或 `urn:ex:building:a123`

2. **版本控制**：ontology 會演化
    - Git + SemVer，或 LinkML 官方工具

3. **Schema Mapping**：外部資料（CSV、SQL、BIM）映射到 ontology
    - R2RML, Ontop, Mastro（RDB → RDF）

4. **Performance**：圖查詢可能很慢
    - Index、物化視圖、快取策略

5. **Multi-tenancy**：多客戶隔離
    - Named graphs / subgraph / 每租戶獨立 DB

6. **跨服務一致性**：微服務架構下的 ontology 同步
    - Event sourcing, CDC, Schema Registry

---

## 六、最小可行 ontology（10 分鐘寫完）

不用 Neo4j、不用 Protégé，純 Python：

```python
# ontology.py — TBox（類型定義）
from pydantic import BaseModel
from typing import Literal

class Entity(BaseModel):
    id: str              # IRI
    type: str            # Class

class Building(Entity):
    type: Literal["Building"] = "Building"
    name: str
    floor_count: int
    developer_id: str    # Link

class Floor(Entity):
    type: Literal["Floor"] = "Floor"
    building_id: str     # Link → Building
    number: int

# storage.py — ABox（實例儲存）
import networkx as nx
G = nx.DiGraph()

# actions.py — Actions
def create_building(b: Building):
    G.add_node(b.id, **b.model_dump())

def add_floor(f: Floor):
    G.add_node(f.id, **f.model_dump())
    G.add_edge(f.building_id, f.id, type="hasFloor")

# query.py — Query
def get_floors(building_id: str):
    return [G.nodes[n] for _, n, d in G.out_edges(building_id, data=True)
            if d["type"] == "hasFloor"]

# reason.py — 2026 AI 原生推理
import anthropic
def ask(question: str):
    client = anthropic.Anthropic()
    return client.messages.create(
        model="claude-opus-4-6",
        tools=[{"name": "get_floors", "input_schema": {...}}],
        messages=[{"role": "user", "content": question}]
    )
```

**這就是完整 ontology 系統** — Pydantic 當 TBox，NetworkX 當儲存，純函數當 Action，Claude 當 reasoner。

### 升級路徑

- NetworkX → Neo4j（百萬節點以上）
- Pydantic → LinkML（需要 schema evolution）
- 手刻 query → SPARQL / Cypher（需要聲明式查詢）
- LLM-only reasoning → HermiT（需要邏輯嚴謹性）

---

## 七、三個工程品味

1. **Schema-first, not data-first**
    先定義 TBox，再灌 ABox。反過來做會噩夢。

2. **Actions 是一等公民**
    ontology 不是純資料模型，Action 要跟 Object 一起設計（這是 Palantir 學對的事）。

3. **LLM 不取代 ontology，是 ontology 的 UI**
    LLM 做語意匹配、ontology 做規則執行，兩者分工。

---

## 一句話

> **Ontology 在工程面不神秘。它是：類型系統（Pydantic/OWL）+ 圖資料庫（Neo4j/Jena）+ 規則引擎（Drools/LLM）+ 查詢介面（Cypher/SPARQL）+ 推理機制（Reasoner/LLM）的堆疊。**
>
> **2026 年的關鍵差異：下層全商品化，LLM 把「語意理解」成本降到趨近零。**

---

## 相關

- [[palantir-ontology-system]] — AIP 的三路匯流架構
- [[../01-tbox-vs-abox]]
- [[../04-canonical-format-selection]]
