---
title: Foundation Model Distillation 與 Ontology Mapping 名詞釐清
author: metaarchetech
created: 2026-05-24
updated: 2026-05-24
source: visustwin-vault
source_path: 04 Knowledge/Lived/AI-Frameworks/Foundation Model Distillation Ontology Mapping.md
tags: [type/note, status/draft, area/ai-frameworks, area/ontology]
---

# Foundation Model Distillation 與 Ontology Mapping

兩個常一起出現但意義不同的術語整理。

## 1. Foundation Model Distillation

**中文：基礎模型蒸餾 / 基礎模型知識萃取**

- **定義**：從大型基礎模型（GPT、BERT、CLIP 等）中提取知識，再將這些知識壓縮進較小的模型中，使後者在保留部分表現力的同時具有更高的效率與可部署性。
- **應用場景**：
  - 資源受限設備上部署 AI（手機、邊緣裝置）
  - 快速訓練專用模型（醫療、法務領域的小模型）
  - 縮減模型推理成本
- **相關術語**：
  - 模型壓縮（Model Compression）
  - 知識蒸餾（Knowledge Distillation）

## 2. Ontology Mapping

**中文：本體映射 / 知識本體對應**

- **定義**：將兩個不同知識系統（Ontology）之間的概念對齊或對映，使它們可以互相理解與交換資料。
- **應用場景**：
  - 異質資料整合（IoT 資料、企業資料庫）
  - 跨系統知識共享（醫學與生物資訊對接）
  - 語意網（Semantic Web）技術中的概念統一
- **相關術語**：
  - 本體（Ontology）：知識的結構化描述，像是有層級與關聯的知識圖譜
  - Semantic Alignment（語意對齊）

## 兩者交集

當基礎模型用於 ontology mapping 任務（如自動對齊兩個醫療 schema）時，distillation 就會出現：把大模型對 mapping 的能力蒸餾到小模型，讓 mapping pipeline 能在低成本環境跑。
