---
title: T-Box vs A-Box（描述邏輯基本分層）
author: metaarchetech
created: 2026-04-17
updated: 2026-05-24
source: visustwin-vault
source_path: 04 Knowledge/Lived/Ontology/01 T-Box vs A-Box.md
tags: [type/note, status/done, area/digital-twin, area/ontology, area/description-logic]
---

# T-Box vs A-Box

> Ontology 世界最基本的分層概念，來自 **描述邏輯 (Description Logic, DL)**。搞懂這兩個詞，就能判斷「這件事該放 ontology 檔，還是 USD / SQLite / MQTT 當下狀態」。

---

## 詞源

**T-Box** = **T**erminology **Box** = 術語盒 = 「字典」
**A-Box** = **A**ssertion **Box** = 斷言盒 = 「事實陳述」

DL 原本為 OWL / 知識圖譜設計，但概念可套到**任何**結構化資料系統：任何系統都有「規則」與「當下實例」兩層。

---

## 定義

### T-Box：定義類別、屬性、關係、約束

白話：這個世界**有哪些種類的東西**、**每種東西長什麼樣**。

Digital Twin 例子：
> `Room` 是一種 `Space`，每個 `Room` 必須屬於某個 `Floor`，可以包含 0 到多個 `Sensor` 與 `FurnitureItem`。每個 `Room` 有 `roomType` 屬性，取值是列舉 `{MasterBedroom, LivingRoom, Kitchen, Bathroom, ...}`。

T-Box 關心的：
- 類別階層 (`MasterBedroom ⊑ Room ⊑ Space`)
- 屬性定義域與值域 (`hasFloor: Room → Floor`)
- 基數限制 (`Room` 至少有 1 個 `hasFloor`)
- 不相交 (`Sensor ⊓ FurnitureItem ≡ ⊥`)

### A-Box：陳述具體個體與它們的關係

白話：**這個世界實際存在哪些個體**、**它們之間發生什麼事**。

Digital Twin 例子：
> `sampleHome_A.F2.Room_101` 是一個 `MasterBedroom`。
> `hue_bulb_007` 是一個 `LightFixture`，`hue_bulb_007.locatedIn = sampleHome_A.F2.Room_101`。
> `sampleHome_A.F2.Room_101.area = 16.5` (平方公尺)。

A-Box 關心的：
- 個體身份 (個體 URN)
- 個體屬於哪些類別
- 個體之間的關係實例
- 個體的屬性值

---

## 為什麼要分

### 改動頻率不同

| | T-Box | A-Box |
|---|---|---|
| 變動頻率 | 很慢（半年~年） | 很快（每天~每秒） |
| 變動來源 | 設計決策、需求演進 | 新建案、新設備、移動家具、即時感測 |
| 改動影響 | 所有下游系統要配合 | 局部，通常只影響特定實例 |
| 適合儲存 | Git 版控的 schema 檔 | 資料庫 / USD / MQTT |

如果把 A-Box 塞進 ontology 檔，**git 會變成流水帳**，也無法查詢。

### 工具生態不同

T-Box 的工具是 **schema editor / validator** （Protégé、pydantic、LinkML）。
A-Box 的工具是 **資料庫 / triple store / 即時訊息匯流排** （SQL、Neo4j、Oxigraph、MQTT broker）。

---

## Digital Twin 實際分佈（以 VisusTwin 為案例）

A-Box 不是單一儲存，而是 **多系統視角**：

```
                 T-Box (單一真相)
                 ontology repo
                          |
        +-----------------+-----------------+
        |                 |                 |
    Design-time      Catalog           Runtime
    A-Box            A-Box             A-Box
    (USD 場景)       (SQLite)          (MQTT / in-mem)
        |                 |                 |
  幾何 + prim path    供應商 / 產品       即時讀值
  場景設計            catalog DB         感測器、狀態
  Omniverse          REST API 服務     短壽命，不持久
```

三個 A-Box **共用 T-Box 定義的類別與 URN**，但各自儲存自己的實例資料。

### 舉例：一盞燈

同一盞燈「主臥的吸頂燈」，在三個系統裡：

| 視角 | 實體形式 | 持久性 |
|---|---|---|
| Design-time (USD) | `/World/SampleHome_A/F2/Room_101/Light_001` prim + mesh | 跟場景一起存 |
| Catalog (SQLite) | `products` 表一列，`supplier_id=Philips, sku=Hue-A60` | 跟專案一起存 |
| Runtime (MQTT) | topic `<ns>/siteA/showroom/f2/mbr/light_001/state` → `{ on: true, brightness: 0.6 }` | 秒級，消失 |

三個 A-Box 要「對得上」 → 靠 **URN** 黏合（見 [[02-identity-urn-design]]）。

---

## 業界實情：95% 只做 T-Box

學術 ontology 論文愛做 A-Box triple store（「讓我們 SPARQL 查遍整個知識庫」），但產業實戰中：

- **資料量太大**：一棟商辦可能有數萬感測點，持續產生 observation，塞 triple store 會爆
- **查詢不符需求**：SPARQL 在 point-in-time 查詢好，但時間序列、BI 儀表板、即時訊息各有專用資料庫更優
- **運維成本**：多一套 triple store 就多一個要維護的元件

主流做法：
- Ontology 只負責 **T-Box + URN 規則 + mapping spec**
- 下游各用各的儲存（time-series DB、SQL、USD）
- 需要跨系統語意查詢時，**ad-hoc 整合層** 或 **GraphQL federation** 解決，不強迫統一 triple store

想做 A-Box 整合時，可考慮 Oxigraph（嵌入式 RDF store）當輕量方案，不要一開始就立基在它上面。

---

## 快速決策：新資料該放哪

問自己三個問題：

1. **會用到的其他系統多嗎？** 只有 1 個 → A-Box，留在那個系統。多個 → 需要 T-Box 定義共通語彙。
2. **改這個會影響幾個人？** 只影響本週工作 → A-Box。影響其他工程師 / 其他項目 → T-Box。
3. **半年後還會長這樣嗎？** 會 → T-Box。不會 → A-Box。

---

## 相關

- [[00-industry-landscape]]
- [[02-identity-urn-design]]
- [[05-tbox-v1-class-list]]
- Description Logic 原典：Baader et al., *The Description Logic Handbook*, Cambridge 2003
- OWL 2 Primer (T-Box / A-Box 正式定義): https://www.w3.org/TR/owl2-primer/
