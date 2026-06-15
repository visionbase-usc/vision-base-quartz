---
title: 06_CLAUDE_Skills 索引
author: metaarchetech
created: 2026-05-24
tags: [type/note, status/done, area/governance]
---

# 06_CLAUDE_Skills

VISION BASE Lab 的 Claude Code skills **文件鏡像**。

> ⚠️ **這裡只有文件，沒有實際 skill 程式檔**。要安裝 skill，去 [`visionbase-usc/claude-skills`](https://github.com/visionbase-usc/claude-skills) repo 抓 `.skill` 檔。

## 為什麼有兩個地方？

| Repo | 內容 | 用途 |
|---|---|---|
| [`claude-skills`](https://github.com/visionbase-usc/claude-skills) | `.skill` 打包檔、SKILL.md、scripts、assets | 給人安裝、執行 |
| `vision_base_vault/06_CLAUDE_Skills/`（這裡）| 每個 skill 一份 doc | 給人讀、學、查 |

主檔在 `claude-skills`，本資料夾是**鏡像**。更新流程：先改 skill → 再同步 doc 到這。

## 目前有的 skill

| Skill | 用途 | 文件 |
|---|---|---|
| `vision-base-template` | 產生 VB 品牌識別的簡報模板（深色/淺色），含內容改寫工具 | [vision-base-template.md](vision-base-template.md) |
| `vision-base-vault` *（規劃中）* | 處理本 vault 編輯任務的專屬 skill | *(待建立)* |

## 怎麼安裝 skill

在 Claude Code 對話中：

```
/plugin install <你下載的 .skill 檔的完整路徑>
```

詳見 [`claude-skills` repo 的 README](https://github.com/visionbase-usc/claude-skills#安裝方式)。

## 想新增 / 修改 skill

走 `claude-skills` repo 的流程：
1. fork / 開 branch
2. 在 repo 根目錄新增資料夾
3. 用 skill-creator 的 `package_skill.py` 打包
4. 開 PR，找 [@anitoisa](https://github.com/anitoisa) review

完成後，**同時更新本資料夾的對應 doc**，讓 vault 內成員能查到。

## Skill 結構慣例

每個 skill 在 `claude-skills` repo 的結構：

```
<skill-name>/
├── SKILL.md          ← 觸發描述 + 使用流程（必要）
├── scripts/          ← 可執行 Python / Bash / Node
├── assets/           ← 圖像、字型、logo
└── references/       ← 給 Claude 讀的詳細規範
```

設計原則：可移植、品牌一致、單一目的、中英並陳。

## 相關資料夾

- [`05_GitHub_Repos/claude-skills.md`](../05_GitHub_Repos/claude-skills.md) — `claude-skills` repo 本身的總覽文件
