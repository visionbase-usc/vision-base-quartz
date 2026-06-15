---
title: RAG Patterns 知識庫
author: metaarchetech
created: 2026-05-24
updated: 2026-05-24
tags: [type/moc, status/done, area/rag, area/ai-frameworks]
---

# RAG Patterns

Retrieval-Augmented Generation（RAG）的設計模式、調優技巧、實作筆記。

## 範圍

- **Chunking strategies**：document、semantic、hierarchical chunking
- **Embedding optimization**：contextual embeddings、HyDE、hybrid search
- **Retrieval**：dense / sparse、reranking、filtering
- **Evaluation**：precision@k、recall、faithfulness、answer relevancy

## 本資料夾筆記

| 筆記 | 重點 |
|---|---|
| [[contextual-embeddings]] | Anthropic Cookbook 的 RAG 升級術：embed 前先用 LLM 加 context prefix，準確度 +35-50% |

## 之後可能擴充

- HyDE（Hypothetical Document Embeddings）獨立筆記
- Hybrid search（BM25 + dense vector）
- Reranking（Cohere rerank、cross-encoder）
- Evaluation 框架（Ragas、TruLens）
