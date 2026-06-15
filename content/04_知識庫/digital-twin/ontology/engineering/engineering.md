---
title: Ontology Engineering
author: metaarchetech
created: 2026-04-16
updated: 2026-05-24
source: visustwin-vault
source_path: 04 Knowledge/Lived/Ontology-Engineering/README.md
tags: [type/note, status/done, area/digital-twin, area/ontology, area/engineering]
---

# Ontology Engineering

> Ontology 作為系統設計的工程實踐。從概念堆疊、技術選型，到實戰應用。
> 核心觀點：**Ontology 在 2023 年後從學術概念變成工程印鈔機** — 原因是 LLM × Function Calling 出現。

---

## 起點筆記（建議按順序讀）

1. [[ontology-stack]] — Layer 0 到 Layer 7 + 三條主流實戰路徑
2. [[palantir-ontology-system]] — Palantir AIP 的三路匯流架構（Chad Wahlquist 投影片拆解）

---

## 核心概念

- **TBox vs ABox** — 類型 vs 實例（見 [[ontology-stack]] / [[../01-tbox-vs-abox]]）
- **Open World vs Closed World** — 假設選擇的後果（見 [[ontology-stack]]）
- **Derived Property** — ML 輸出如何成為 Ontology 一等公民（見 [[palantir-ontology-system]]）
- **Ontology 三路匯流** — Data × Logic × Action（見 [[palantir-ontology-system]]）

---

## 技術選型軸線

| 路徑 | 技術堆疊 | 適用場景 |
|---|---|---|
| 學術標準派 | OWL + Protégé + Jena + SPARQL | 跨組織、公共資料 |
| 企業務實派 | Neo4j + Cypher + Pydantic | 90% 商業系統 |
| AI 原生派 | Postgres + Pydantic + LLM function calling | 2026 新創主流 |

---

## AI 時代的新關連

- [[../../../tools/ai-frameworks/ai-frameworks]] — LangChain / MCP / Agents
- LLM 工具整合 / Model Context Protocol
- Foundation Model × Ontology Mapping

---

## 待撰寫（TODO）

- [ ] TBox vs ABox 獨立筆記（已部分覆蓋於 [[../01-tbox-vs-abox]]）
- [ ] Open World Assumption 深度討論
- [ ] Neo4j vs Jena vs Pydantic 實戰選型對比
- [ ] LLM × Ontology Function Calling 實戰 pattern
