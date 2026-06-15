---
title: Model Context Protocol（MCP）
author: metaarchetech
created: 2026-05-24
updated: 2026-05-24
source: visustwin-vault
source_path: 04 Knowledge/Lived/Model Context Protocol.md
tags: [type/note, status/draft, area/ai-frameworks]
---

# Model Context Protocol（MCP）

**MCP** 是 Anthropic 於 2024 年提出的開放協議，目的是讓 LLM agent 能用統一介面對接外部資源（檔案、API、資料庫、工具）。在多 agent / multi-tool 架構下，MCP 扮演「agent ↔ 資源」的標準語言。

## 重點

- **協議分層**：把「agent 如何思考」與「agent 如何拿到資料 / 工具」分開
- **Server / Client 模型**：MCP server 暴露資源；agent client 用標準訊息調用
- **可移植**：同一個 MCP server，不同模型 / 不同 agent 框架都能用

## 官方資源

- 文件：https://modelcontextprotocol.io/introduction

## 相關

- [[langchain-mcp-migration]] — 把 LangChain 平台演化為 MCP 相容的路線圖
