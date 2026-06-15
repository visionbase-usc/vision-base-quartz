---
title: BIM 整合知識庫
author: metaarchetech
created: 2026-05-24
updated: 2026-05-24
source: visustwin-vault
tags: [type/note, status/done, area/digital-twin, area/bim]
---

# BIM 整合

> Revit、IFC、BIM × Digital Twin 整合的技術筆記。

## 內容

| 筆記 | 用途 |
|---|---|
| [[massing-web-pipeline]] | Massing Pipeline 設計：從 BIM 幾何 → 共用建築量體格式 → 下游模擬 |
| [[ventilation-analysis]] | 通風 / 換氣分析報告插件架構研究（matplotlib + Omniverse extension）|

## 跨領域

- [[../omniverse/omniverse-revit-2026]] — Revit 2026 → Omniverse 資料流
- [[../ontology/03-aligning-with-standards]] — IFC / BOT / ifcOWL 對齊
- [[../usd/usd]] — USD / OpenUSD（BIM 幾何轉檔目標格式）

## 標準

- **IFC** — buildingSMART 大標準，建築 / BIM 主流交換格式
- **ifcOWL** — IFC 的 OWL 翻版
- **BOT** — Building Topology Ontology（W3C LBD，輕量拓撲）
- **gbXML** — 建築能耗模擬用，Revit 能匯出
