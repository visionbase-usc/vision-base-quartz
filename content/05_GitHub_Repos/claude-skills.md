---
title: claude-skills repo
author: metaarchetech
created: 2026-05-24
tags: [type/note, status/done, area/governance]
---

# claude-skills

VISION BASE Lab 的 **Claude Code skills 集中倉庫**。把 skill 安裝到自己的 Claude Code，就能在對話中呼叫團隊共用的工作流。

## 基本資訊

| 項目 | 值 |
|---|---|
| **URL** | [github.com/visionbase-usc/claude-skills](https://github.com/visionbase-usc/claude-skills) |
| **類型** | Claude Code skills (`.skill` 打包檔 + 原始檔) |
| **可見性** | 私人 |
| **預設分支** | `main` |
| **維護者** | [@anitoisa](https://github.com/anitoisa) |

## 目前 skill 清單

| Skill                    | 用途                  | 觸發                   | vault 內文件                                                              |
| ------------------------ | ------------------- | -------------------- | ---------------------------------------------------------------------- |
| `vision-base-template`   | VB 品牌簡報模板（深/淺色）     | 簡報 / pptx / 投影片 / 公版 | [vision-base-template.md](../06_CLAUDE_Skills/vision-base-template.md) |
| `vision-base-vault`（規劃中） | 編輯本 vault 的專屬 skill | vault / 共筆 / 筆記      | (待建立)                                                                  |
|                          |                     |                      |                                                                        |

## 安裝方式

從 [repo 列表](https://github.com/visionbase-usc/claude-skills) 下載 `<skill-name>.skill` 檔，然後在 Claude Code 對話中：

```
/plugin install ~/Downloads/<skill-name>.skill
```

## 設計原則

- **可移植** — 腳本只引用 skill 內 assets
- **品牌一致** — 任何 VB skill 都遵循品牌書
- **單一目的** — 一個 skill 處理一件事
- **中英並陳** — SKILL.md 觸發描述涵蓋雙語

## 新增 / 修改 skill 的流程

1. 在 repo fork / 開 branch
2. 在根目錄新增資料夾（或修改現有 skill）
3. 用 skill-creator 的 `package_skill.py` 重新打包 `.skill`
4. 開 PR，找另一位團隊成員 review 過再 merge 到 `main`

完成後**同步更新 vault 的 `06_CLAUDE_Skills/<skill>.md`** doc。

## 相關 vault 文件

- [`06_CLAUDE_Skills/`](../06_CLAUDE_Skills/) — 每個 skill 的詳細鏡像 doc

## 相關 repo

- [`vision_base_vault`](vision_base_vault.md) — 配套的共筆 vault
