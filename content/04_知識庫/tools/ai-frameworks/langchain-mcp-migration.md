---
title: LangChain 平台 MCP 化的演進路線
author: metaarchetech
created: 2026-05-24
updated: 2026-05-24
source: visustwin-vault
source_path: 04 Knowledge/Lived/AI-Frameworks/Langchain 平台 MCP 化.md
tags: [type/howto, status/draft, area/ai-frameworks, area/omniverse]
---

# LangChain 平台 MCP 化

把現有「LangChain agent + REST API」操控 Omniverse 的架構，逐步演化為 **Model Context Protocol（MCP）相容**的標準協議層。讓任何外部 agent（不限於內部 LLM）都能用統一協議調用 3D 場景資源。

## 1. 起點：典型 LangChain 平台架構

- **前端**：Streamlit / Web Dashboard
- **後端**：FastAPI + LangServe / LangChain agent
- **能力**：API 已支援語意查詢與代碼生成；模擬 / 真實 Omniverse 操作；REST API 初步標準化、前後端可分離

這個結構已經有「協議」的雛型。只要讓**內容調用與控制能力**更標準化、對外部 agent 友善，就能 MCP 化。

## 2. 什麼是「MCP 化」

讓**任何外部智能 agent**（內部 LLM、LangChain agent、其他 AI 代理、自動化腳本）都能用統一 API 操控 Omniverse 資源。

明確劃分三層：

1. **內容資料層**：USD、場景、物件
2. **指令協議層**：標準 API，讓 agent 調用
3. **智能 agent 層**：LLM、LangChain、rule engine 配合 / 解釋 / 自動決策

## 3. 演進步驟

### A. API 標準化

- 將所有 Omniverse 內容操作（CRUD、查詢、屬性變更、場景指令）包裝成統一標準 API
  - 例：`/list_nodes`、`/get_node_info`、`/add_primitive`、`/set_material`、`/run_script`
- 用 **OpenAPI / Swagger** 或 **gRPC** 描述介面，提升語意可見度

### B. Schema / Context 定義

- 對每個內容物件（USD node、Material、Layer）設計明確 schema
- 結合 **JSON Schema** 或自定義 schema 標註每個 API input / output

### C. Agent 層與協議分離

- 現有 LangChain / LangServe agent 繼續服務 UI
- 未來允許外部 agent（不一定是 LLM，也可以是自動化腳本、其他智能代理）透過協議層存取
- 多包一層「MCP Server」（REST / WebSocket / gRPC 都可），專責處理指令並呼叫 Omniverse 操作

### D. 認證與權限管理（可選）

- MCP server 層可加 token-based、OAuth、API key 安全機制

## 4. 演進路線圖

1. 把和 Omniverse 操作有關的 API 做 schema 明確化（自動生成 API doc）
2. 所有場景、內容、指令都標準化 API 名稱與 input / output
3. 允許多種 agent 存取同一 MCP 層（不限定 LangChain / Streamlit）
4. 逐步補齊物件、屬性、動畫等 CRUD / 查詢指令
5. 文件與 API doc 對外公開，鼓勵第三方（不一定是 LLM agent）也能用 MCP 操控 Omniverse 資源

## 5. README 範例片段

```markdown
## MCP 標準協議支援

本平台支援 Model Context Protocol（MCP）設計理念，將 Omniverse 操作指令、
內容查詢標準化為 API 協議層，支援多種智能 agent 及自動化工具對接。

### API 標準介面範例

| Endpoint | Method | 說明 | 範例參數 |
|---|---|---|---|
| /list_nodes | GET | 列出場景所有物件 | scene_id |
| /get_node_info | GET | 查詢單一物件資訊 | node_id |
| /add_primitive | POST | 新增幾何體到場景 | type, position, ... |
| /set_material | POST | 設定物件材質 | node_id, material_id |
| /run_script | POST | 執行 Python 腳本 | script_content |
```

## 6. 小結

- LangChain agent 與 MCP 並非互斥；前者是 agent 層的具體實作，後者是 agent ↔ 資源的協議
- 平台逐步 MCP 化，就是讓一切內容操作變得「可標準調用、可自動化」
- 既能保有「AI 智能決策」（LangChain agent / LLM prompt）層，也讓任何組織能自己寫 agent 直接對 MCP 下指令

## 相關

- [[model-context-protocol]] — MCP 規範本身
