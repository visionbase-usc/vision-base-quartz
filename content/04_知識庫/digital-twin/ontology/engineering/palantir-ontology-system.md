---
title: The Palantir Ontology System
author: metaarchetech
created: 2026-04-16
updated: 2026-05-24
source: visustwin-vault
source_path: 04 Knowledge/Lived/Ontology-Engineering/The Palantir Ontology System.md
tags: [type/note, status/done, area/digital-twin, area/ontology, area/palantir, area/architecture]
---

# The Palantir Ontology System

> Palantir AIP 核心架構圖的拆解。**Ontology 不是資料模型，是三路匯流點（Data × Logic × Action）**。
>
> 來源：Palantir AIP Software Demonstration — Chad Wahlquist (FWD Deployed Architect)

## 一句話

傳統 Ontology = **被動的知識圖譜**（只存資料）。
Palantir Ontology = **主動的三路匯流點**（資料 + 推理 + 動作都在這裡對齊）。

---

## 核心架構圖（來自投影片）

```
┌──────────────────────────────────────────────────────────────┐
│                    ONTOLOGY 中央層                            │
│  企業所有 Object 在這裡定義、對齊、查詢、推理                 │
│                                                              │
│  範例 PLANT OBJECT:                                          │
│    Status: Running                                           │
│    Energy Consumption: 12000 kWh                             │
│    Cycle Time (Avg): 45.2                                    │
│    Issues: 17                                                │
│    Plant Uptime: 99.98%                                      │
│    Efficiency Score: 88                                      │
└──────────────────────────────────────────────────────────────┘
      ▲              ▲              ▲
      │              │              │
┌─────┴─────┐ ┌──────┴──────┐ ┌─────┴──────────┐
│   DATA    │ │    LOGIC    │ │   SYSTEMS OF   │
│  SOURCES  │ │   SOURCES   │ │     ACTION     │
│           │ │             │ │                │
│ Transactions │ Supervised ML │ ERP            │
│ IoT/Sensor│ │ Entity Res. │ │ SCM            │
│ Geospatial│ │ Unsup. ML   │ │ MES            │
│ Unstructured │ Optimizers │ │ Scheduling     │
│ Relational│ │ Forecast    │ │ Edge           │
│ More...   │ │ Rule-Based  │ │ More...        │
└───────────┘ └─────────────┘ └────────────────┘
```

---

## 三層拆解

### Layer 1 · Data Sources（事實輸入）

現實世界的事實進入 Ontology 的管道。

| 類型 | 範例 | 工程意義 |
|---|---|---|
| Transactions | 訂單、發票、金流 | 需要 streaming ingestion |
| IoT / Sensor | 溫度、振動、電耗 | 需要 time-series pipeline |
| Geospatial | GPS、衛星、LiDAR | 需要 PostGIS / Mapbox 類 stack |
| Unstructured | 文件、郵件、影像 | 需要 embedding + RAG |
| Relational | SQL、ERP 原生 | 需要 R2RML / CDC |

**工程需求**：一個強大的 connector / adapter 框架，把異質資料正規化到 Ontology schema。

### Layer 2 · Logic Sources（推理/決策輸入）

**這層是 Palantir 最特別的地方** — 把 ML 模型當成 Ontology 的一等公民。

| 類型 | 範例 | 輸出到 Ontology 的方式 |
|---|---|---|
| Supervised ML | 預測模型、分類器 | Derived Property |
| Entity Resolution | dedup、merge | Object 合併 |
| Unsupervised ML | 分群、異常偵測 | Cluster Label |
| Optimizers | 排程、路徑 | Action 建議 |
| Forecast Models | 時間序列預測 | Projected Property |
| Rule-Based Logic | 商業規則 | Derived Boolean |

**關鍵**：ML 模型輸出**不是 API 呼叫結果**，而是**直接寫入 Ontology** 作為 Derived Property。
對使用者來說，Object 上的 `Efficiency Score: 88` 跟 `Status: Running` 看起來一樣 — 背後一個是 ML 算的、一個是 sensor 讀的，但 Ontology 抹平了差異。

### Layer 3 · Systems of Action（動作實現層）

從 Ontology 出發的動作，落地到真實系統。

| 類型 | 範例 |
|---|---|
| ERP | SAP, Oracle, Microsoft Dynamics |
| SCM | 供應鏈系統 |
| MES | 製造執行系統 |
| Scheduling | 排程系統 |
| Edge | 邊緣運算設備 / PLC |

**關鍵設計**：Action 不是 HTTP call，是**透過 Ontology 中介的狀態轉換**。
Ontology 知道 `reassign_workorder()` 執行後應該改變哪些 Object 的哪些 Property，然後 Systems of Action connector 把改變同步到外部系統。

---

## 為什麼這個架構 2023 年才爆發

三個條件疊加才成立：

1. **LLM + Function Calling（2023）**
    AIP 讓 LLM 直接讀寫 Ontology。Ontology 變成 LLM 的 tool schema。沒有 LLM，這架構只是複雜的 ESB（企業服務匯流排）。

2. **Foundry Platform 成熟（2018-2022）**
    三路接入管線終於穩定。這段是 Palantir 燒了 US$1B+ 才完成的底層工程。

3. **企業 AI 需求爆發（2023-2025）**
    企業要「能動作的 AI」，不是「能聊天的 AI」。AIP 剛好踩在這個拐點。

**缺一不成立**。這也是為什麼 2010 年代 Palantir 一直被當成「高級顧問公司」 — 架構想法在，但工具鏈不足。

---

## PLANT OBJECT 的深層解讀（投影片左下）

這個範例雖小，但**展示了三路匯流的完整樣貌**：

| Property | 來源 | 意義 |
|---|---|---|
| Status: Running | Data Source（MES 直接讀） | 事實 |
| Energy Consumption: 12000 kWh | Data Source（IoT sensor aggregation） | 事實 |
| Cycle Time (Avg): 45.2 | Data + Logic（ML 計算平均，去除異常） | 衍生事實 |
| Issues: 17 | Logic + Action（Entity Resolution + 工單系統串接） | 互動狀態 |
| Plant Uptime: 99.98% | Logic（從 Status history 計算） | 衍生統計 |
| Efficiency Score: 88 | Logic（ML 模型輸出的綜合分數） | ML 推論 |

**對使用者是單一 Object，對工程是三路資料在 Ontology 層匯流**。

---

## 對比傳統架構

| 維度 | 傳統企業架構 | Palantir Ontology 架構 |
|---|---|---|
| 資料流向 | 單向（Data Lake → Model → App） | 三向匯流到 Ontology |
| ML 位置 | 獨立服務 | Ontology 的一等公民 |
| Action 位置 | App 裡的 API 呼叫 | Ontology 定義的狀態轉換 |
| 唯一真相 | 分散在多系統 | Ontology |
| LLM 使用 | 外掛 RAG | 原生 tool calling via Ontology |

---

## 工程實踐的關鍵挑戰

如果要複製這個架構，真正難的是三件事：

1. **Schema 對齊**：Data / Logic / Action 三側對 Object 定義要一致，否則翻譯成本吃掉一切
2. **權限模型**：每個 Property 讀寫權限要能溯源到三側任一個
3. **Derived Property 的及時性**：ML 算出的 Property 要在多久內更新？同步？非同步？事件驅動？

Palantir 燒了 US$3B 主要就是燒在這三件事的工程化上。

---

## 對 Digital Twin 應用的延伸

Digital Twin 可以延伸這三層，**並加上 Palantir 沒有的第四層**：

| Palantir 層 | Digital Twin 對應 |
|---|---|
| Data Sources | BIM (IFC) + USD + 工地 IoT + ERP + 採購系統 |
| Logic Sources | PhysicsNeMo + Earth-2 + Replicator + 自訓 ML |
| Systems of Action | CMMS + FM + 工單系統 + Dashboard |
| **3D Spatial Layer（獨特）** | **USD Stage + Omniverse 場景綁定** |

**關鍵差異**：每個 Digital Twin Object 有「**USD Prim 綁定**」這個維度，意味著每個 Object 都有空間對應。這是 Palantir Ontology 結構上做不到的。

---

## 相關

- [[ontology-stack]] — Layer 0-7 + 三條主流技術路徑
- [[engineering]]
- Palantir AIP 產品頁：https://www.palantir.com/platforms/aip/
- Palantir Ontology 官方文件：https://www.palantir.com/docs/foundry/ontology/overview
