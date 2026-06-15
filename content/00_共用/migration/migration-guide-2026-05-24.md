---
title: Visustwin → vision_base_vault 搬遷規範（給 agent 讀）
author: metaarchetech
created: 2026-05-24
tags: [type/howto, status/done, area/governance]
---

# Visustwin vault → vision_base_vault 搬遷規範

> 本檔是 2026-05-24 一次性大型搬遷工作所有 agent 都要讀的「正典」。任何 agent 動工前先看完本檔，再看自己分到的清單。
> 搬遷完成後本檔轉為歷史紀錄，仍留在 `00_共用/migration/` 供日後查考。

## 1. 來源 / 目標

- **來源 vault（read-only）**：`C:\Visustwin\metaarchetech-vault\`
  - bash 路徑：`/sessions/amazing-inspiring-cannon/mnt/metaarchetech-vault/`
- **目標 vault**：`C:\vision_base\vision_base_vault\`
  - bash 路徑：`/sessions/amazing-inspiring-cannon/mnt/vision_base/vision_base_vault/`

**只複製，不刪源檔**。搬遷後源 vault 維持原狀，由使用者日後決定何時清理。

## 2. 檔名規則

- **一律改為英文 kebab-case**。中文檔名翻成英文（保留語意），中間用 `-` 連接。
- 數字前綴保留（例：`00 產業現況...` → `00-industry-landscape.md`）。
- 移除空白、移除 emoji、副檔名統一 `.md`。

範例：

| 來源 | 目標 |
|---|---|
| `Omniverse XR Design.md` | `xr-design.md` |
| `00 產業現況 與 生態圖譜.md` | `00-industry-landscape.md` |
| `🛠 XR 人體動作模擬導入機器學習 計畫.md` | `2025-xr-motion-ml-plan.md` |

## 3. Front-matter 規格（每個搬過去的檔都要有）

```yaml
---
title: <筆記標題（中文 OK）>
author: metaarchetech
created: <原檔的 created 或 2026-05-24>
updated: 2026-05-24
source: visustwin-vault
source_path: <相對於來源 vault 的原路徑>
tags: [type/<note|howto|paper>, status/<draft|review|done>, area/<x>, area/<y>]
---
```

規則：

- **author** 一律填 `metaarchetech`（原作者）
- **source / source_path** 一律填，方便日後追溯
- **tags**：至少含 `type/...` + `area/...`。area 可以多個（例：`area/omniverse` + `area/digital-twin`）
- 原本 front-matter 有 `tags: [..., visustwin]` 之類**公司專屬 tag 一律刪除**
- 原本沒有 front-matter 的檔，新增以上 5+ 行
- 原本有但欄位不齊的，補齊不衝突的欄位

## 4. Wikilink 改寫規則

來源 vault 用很多種寫法，**目標 vault 統一為短名 wikilink**：

```
[[short-name]]              ← 一般引用，依檔名
[[short-name|顯示文字]]     ← 要客製顯示文字
```

**不**使用：
- `[文字](相對路徑.md)` ── Markdown link 風格
- `[[完整/長/路徑/file|文字]]` ── 全路徑 wikilink（除非有檔名衝突）
- 直接散在文末的 `[[Omniverse]]` 之類「裸 wikilink」（無上下文）── 整段刪掉或併入正文

**特殊處理**：
- 來源檔常見開頭一行只寫 `[[Omniverse]]` 之類的 breadcrumb ── 一律**移除**（front-matter 的 `area` tag 已涵蓋此資訊）
- 連到 Visustwin vault 自有檔的 wikilink（例：`[[Vault Home]]`、`[[Entity_Map]]`、`[[Semantor*]]`）── **刪除整個 link**，把連結文字改成純文字或整句刪掉

## 5. 去專屬化清單（B 標記檔必做）

凡是內文出現以下字眼，原則上要改寫：

| 來源用語 | 改成 |
|---|---|
| `VisusTwin` / `VisTwin` | 視情況：若是技術泛指 → 改「Digital Twin」；若是公司專指且文意需要 → 改「以 VisusTwin 為例」並標註為案例 |
| `Visionbase` / `visionbase`（指實踐 lab）| `VISION BASE Lab` 或 `實踐 VISION BASE Lab` |
| `Semantor` | 刪除整段，或改「某 NLP-to-ontology agent」 |
| `metaarchetech 公司` `公司端` `商業化` `主線 A / 主線 B` | 一律刪除這類公司治理語言 |
| `寶舖` `寶鋪` `welltek` | 刪除整段（客戶案例不搬） |
| `VB 老大政治分寸` 等內部黑話 | 刪除 |

**判斷原則**：搬過去的內容是給「VISION BASE Lab 學術成員」讀的技術筆記，不是 VisTwin 公司產品文件。技術內容全留，公司語境一律去掉。

## 6. 操作步驟（每個檔）

1. **Read** 來源檔
2. 判斷檔名 → 改 kebab-case
3. 拆解原 front-matter（如有）→ 重組為第 3 節規格
4. 處理開頭的 breadcrumb wikilink（移除）
5. 改寫內文：去專屬化 + 改寫所有 wikilink
6. **Write** 到目標路徑
7. **不**刪源檔

## 7. 各 area 子分類速查

```
04_知識庫/
├── digital-twin/
│   ├── omniverse/         Omniverse Kit / Extensions / XR / Connectors
│   ├── ontology/          Ontology 理論 + USD customData + T-Box/A-Box
│   │   └── engineering/   Ontology engineering 案例（Palantir 等）
│   ├── usd/               OpenUSD schemas / workflows
│   ├── bim/               Revit / IFC / BIM 整合
│   └── concepts/          Digital Twin 概念、定義、產業案例
│
├── physical-ai/
│   ├── isaac/             Isaac Sim / Isaac Lab / Isaac ROS
│   ├── robotics/
│   │   ├── kuka/          KUKA 機械手臂、KRL、控制器
│   │   └── ros2/          ROS2 / Nav2 / MoveIt
│   └── concepts/          Physical AI / Embodied AI 概念
│
├── 3d-reconstruction/
│   ├── gaussian-splatting/
│   │   └── feature-3dgs/  Feature-3DGS 相關
│   ├── neural-rendering/  NeRF 系列
│   ├── photogrammetry/    SfM / RealityCapture / Metashape
│   └── xr/                Depthkit / XR 擷取與呈現
│
├── tools/
│   ├── ai-frameworks/     LangChain / MCP / Agents
│   │   └── rag/           RAG 模式
│   └── dev/               Git / Github / IDE / 工作流
│
└── papers/                跨領域 paper 摘要集中地（一篇一檔）
```

## 8. 失誤回報

agent 工作中若遇到：
- 來源檔內容已不可讀（亂碼、空檔）
- 分類有疑義（跨多個 area）
- 內文需要超過 30% 改寫才能去專屬化
→ **跳過該檔，記到 agent return 報告的「待人工處理」清單**，不要硬搬。

## 9. 安全網

- 不刪源檔
- 不改源 vault 的任何檔
- 若目標路徑已有同名檔，**停下來回報**，不要覆蓋
