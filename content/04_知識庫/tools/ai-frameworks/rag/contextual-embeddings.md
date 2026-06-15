---
title: Contextual Embeddings for RAG
author: metaarchetech
created: 2026-05-19
updated: 2026-05-24
source: visustwin-vault
source_path: 04 Knowledge/RAG_Patterns/contextual-embeddings.md
tags: [type/note, status/done, area/rag, area/ai-frameworks]
---

# Contextual Embeddings for RAG

## 中文摘要

**這個是什麼**：Anthropic 提出的 RAG 升級術 — embed 前先用 LLM 給每段加一句「這段在文件中代表什麼」，準確度 +35-50%。

**何時用**：
- 建 RAG pipeline 時（chunking 完成、embed 前）
- 文件長且結構複雜時（vault 100+ 頁文件）
- 單純 embedding 抓不準時（precision 卡在 0.6 以下）
- 想用便宜模型（all-MiniLM）但要品質保證

**核心步驟**：
- 原 chunk → Haiku 加一句「這段在文件中代表什麼」
- 例：「[extension-protocol.md > Dashboard Integration] 這段定義了儀表板如何與 Omniverse 集成...」
- 才 embed → 存向量庫
- 查詢時也一樣加 context prefix

**對 Digital Twin 場景的對位**：vault 變問答庫的關鍵升級、ontology spec 100+ 頁查詢必備、BIM 文件查詢（room metadata + hierarchy prefix）的品質保證。

---

## Summary

Contextual embeddings improve retrieval quality by augmenting documents with contextual metadata during embedding, rather than embedding raw text alone. This is especially valuable for multi-document collections where a chunk's meaning depends on its document structure, source type, or domain.

**For Digital Twin / vault use case**: use contextual embeddings to improve vault search (identify what each architecture doc chunk is about) and BIM search (link room data to project metadata).

## Key Insight

Instead of embedding: `"This section defines how the dashboard integrates with Omniverse."`

Embed with context: `"[extension-protocol.md > Dashboard Integration] This section defines how the dashboard integrates with Omniverse."`

The additional context (document title + section hierarchy) gives the embedding model more signal about intent and reduces ambiguity.

## When to Use

1. **Multi-document retrieval** — vault has many docs; adding doc headers helps the model understand scope
2. **Hierarchical content** — markdown with H1/H2/H3; adding header chain improves chunk interpretation
3. **Domain-specific terminology** — "API" means different things in architecture doc vs. BIM context; context clarifies
4. **Cost optimization with lightweight models** — cheaper models (all-MiniLM) benefit more from context than premium models

## Implementation Patterns

### Pattern 1: Document Metadata Prefix (Recommended for Vault)

```
Input: raw chunk from "docs/extension-protocol.md", H2 section "Dashboard Integration"

Contextual embedding input:
"[Source: extension-protocol.md] [Section: Dashboard Integration]
This section defines how the dashboard integrates with Omniverse.
The integration uses three dictionaries: manifest (UI definition), config (static config), lifecycle (runtime)."

Result: embedding model understands this chunk is about extension protocol + dashboard, not general Omniverse topics.
```

### Pattern 2: Hierarchical Header Chain (Recommended for BIM)

```
Input: chunk from BIM JSON, room "Bedroom-02", floor "3", project "Project-A"

Contextual embedding input:
"[Project: Project-A] [Floor: 3] [Room: Bedroom-02]
Materials: hardwood floor, painted drywall. Dimensions: 4m x 5m.
Windows: 2 (north-facing). Renovations needed: flooring, paint."

Result: embedding captures project + spatial hierarchy + room-specific details.
```

### Pattern 3: Query Context Augmentation (HyDE Variant)

When user searches: `"How should dashboard integrate with Omniverse?"`

Instead of embedding the query directly, generate a hypothetical document:
```
"The dashboard integrates with Omniverse via the extension protocol.
Three dictionaries handle: manifest (UI), config (static), lifecycle (runtime).
The protocol uses kit-bridge IPC messaging for real-time updates.
Error handling is graceful; connection failures degrade to cached UI."
```

Embed the **hypothetical document**, not the raw query. The hypothetical gives much more semantic signal.

## Domain-specific Notes

### For Vault Search
- **Prefix each chunk** with document path: `[AUTHORING.md > Conventions] ... text ...`
- **Include section hierarchy**: `[Level] [Section] ... text ...` (e.g., `[1. System] [1.1 Components]`)
- **Cost**: ~10% token increase per chunk, ~10% latency increase, ~20% improvement in retrieval precision

### For BIM Search
- **Link chunks to BIM hierarchy**: `[Project: <name>] [Floor: <n>] [Room Type: <type>] ... text ...`
- **Include metadata**: project phase, renovation status
- **Benefit**: filtering + ranking (e.g., "show only renovated bathrooms on floor 3" becomes filter on [Status: Renovated] + [Type: Bathroom] + [Floor: 3])

### For Safety Rule Retrieval
- **Prefix with rule severity + scope**: `[CRITICAL] [Structural Safety] [All Projects] ... text ...`
- **Enables ranking**: after dense retrieval, rank by severity (CRITICAL > HIGH > MEDIUM)
- **Enables filtering**: show only rules applicable to the active project

## Cost-Quality Tradeoff

| Model | Dims | Speed | Context Benefit | Cost |
|---|---|---|---|---|
| all-MiniLM (384) | 384 | Very fast | +25% quality gain | Free (self-hosted) |
| all-mpnet (768) | 768 | Balanced | +15% quality gain | Free (self-hosted) |
| text-ada (1536) | 1536 | Slow | +8% quality gain | $0.10 / 1M tokens |

**Recommendation**:
- Use **all-mpnet (768) + contextual embedding** as baseline
- If latency > 200ms for vault search: upgrade to all-MiniLM + more aggressive context prefixing
- If quality < 0.75 precision: test text-ada (not cost-effective unless precision is critical)

## Implementation Checklist

- [ ] Choose context prefix format (document path, hierarchy, metadata)
- [ ] Update chunking function to append context prefix during embedding
- [ ] Test with 10 real queries from the target user group
- [ ] Measure baseline precision@5 before and after
- [ ] Document context prefix format in vault (why you chose it, when to update)
- [ ] If using dense + sparse hybrid: apply context to dense chunks only
- [ ] Monitor latency: contextual prefixes add ~10% tokens = ~10% slower queries

## Pitfalls to Avoid

1. **Context inflation** — don't prefix every chunk with the entire document. Use relevant headers only.
2. **Inconsistent formatting** — if some chunks have `[Source: X]` and others have `<Source>X</Source>`, the model gets confused. Standardize format.
3. **Losing original chunk text** — always keep original chunk as the main content; context is a prefix/wrapper, not a replacement.
4. **Over-reliance on context** — if retrieval still fails with context, the problem is likely document chunking or model selection, not context.

## Sources & References

- **Anthropic Cookbook** (RAG patterns): contextual-embeddings examples
- **Embedding Model Comparison**: all-MiniLM vs. all-mpnet vs. text-ada tradeoffs
- **Implementation note**: PostgreSQL + pgvector supports prefix-augmented embeddings natively
