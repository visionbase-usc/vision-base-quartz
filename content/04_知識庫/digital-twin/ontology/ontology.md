---
title: Digital Twin Ontology 知識庫
author: metaarchetech
created: 2026-04-17
updated: 2026-05-24
source: visustwin-vault
source_path: 04 Knowledge/Lived/Ontology/README.md
tags: [type/note, status/done, area/digital-twin, area/ontology]
---

# Digital Twin Ontology

> Digital Twin 平台的 **語意骨架** 知識庫。
> 目標：讓跨系統（BIM、感測、3D 場景、應用層）的「房間」「設備」「特徵」指向**同一個東西**，而不是各自為政。

---

## 閱讀順序

這套筆記從「產業全景」走到「具體實作決策」，建議第一次讀按順序看完。熟了以後可以當工具書，挑需要的章節跳讀。

1. [[00-industry-landscape]] — 誰用 IFC / Brick / DTDL，為什麼，適不適合 Digital Twin 場景
2. [[01-tbox-vs-abox]] — 描述邏輯兩個基本層，和為什麼工業界 95% 只做 T-Box
3. [[02-identity-urn-design]] — 多套 ID 併存的痛，URN 格式怎麼定
4. [[03-aligning-with-standards]] — BOT / Brick / SOSA / IFC / ifcOWL / DTDL / SAREF 細節
5. [[04-canonical-format-selection]] — LinkML / Pydantic / JSON Schema / OWL，案例為什麼選 Pydantic
6. [[05-tbox-v1-class-list]] — 落地的類別階層與欄位（以 VisusTwin 為案例）
7. [[../usd/usd-customdata-conventions]] — Ontology 如何在 Omniverse 場景裡存活

---

## 一頁總結 (TL;DR)

如果只想記五條決策：

| # | 決策 | 理由 |
|---|---|---|
| 1 | Canonical = **Pydantic** | Python-heavy 團隊，JSON Schema 一行匯出 |
| 2 | 標準對齊用 **結構借鏡**，不繼承 | 保留橋梁不綁外部生存 |
| 3 | **URN** 做核心 `urn:<ns>:{site}:{building}:{unit}:{room}:{type}:{id}` | 跨 USD / MQTT / SQL 統一身份 |
| 4 | **T-Box Only**，不做 A-Box triple store | 95% 工業界這樣做 |
| 5 | USD 先用 **customData dict**，不做 IsA schema | 降成本，穩定後再升級 |

---

## Ontology Engineering（工程實踐）

進階：把 ontology 當系統設計工程實踐看 → [[engineering/engineering]]

- [[engineering/ontology-stack]] — Layer 0 到 Layer 7 概念堆疊 + 三條主流技術路徑
- [[engineering/palantir-ontology-system]] — Palantir AIP 的三路匯流架構

---

## 相關

- [[../usd/usd]] — USD / OpenUSD schemas
- [[../omniverse/omniverse]] — Omniverse 實作端
- [[../bim/bim]] — BIM 整合（涉及 IFC）
- [[../../tools/ai-frameworks/ai-frameworks]] — LLM × ontology
