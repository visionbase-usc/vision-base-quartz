---
title: 05_GitHub_Repos 索引
author: metaarchetech
created: 2026-05-24
tags: [type/note, status/done, area/governance]
---

# 05_GitHub_Repos

VISION BASE Lab 在 [`visionbase-usc`](https://github.com/visionbase-usc) GitHub organization 底下所有 repo 的**文件鏡像**。

> ⚠️ **這裡只有文件**。實際程式碼、commit 歷史都在 GitHub。本資料夾是給 vault 內成員「快速查 repo 概況」用。

## 為什麼有這個資料夾

- 不用每次切到 GitHub 才能查某個 repo 是做什麼的
- 新成員 onboarding 時一目了然 lab 有哪些 repo
- vault 內的筆記、會議記錄可以直接 wiki link 到對應 repo 文件

## 維護

每加一個新 repo 到 org → 在這建一個對應的 `.md`。每個 repo 改重大方向 → 更新對應的 doc。

格式：每個 repo 一個 `.md`，內容包括 URL、用途、主要技術、維護者、相關連結。

---

## 目前 repo 清單

| Repo | 類型 | 可見性 | 用途 | 文件 |
|---|---|---|---|---|
| [`vision_base_vault`](https://github.com/visionbase-usc/vision_base_vault) | 共筆 | 私人 | Obsidian vault（就是這個！）| [vision_base_vault.md](vision_base_vault.md) |
| [`vision-base-quartz`](https://github.com/visionbase-usc/vision-base-quartz) | 對外 | 公開 | vault 的 Quartz 發佈站（GitHub Pages） | [vision-base-quartz.md](vision-base-quartz.md) |
| [`claude-skills`](https://github.com/visionbase-usc/claude-skills) | 工具 | 私人 | Claude Code skills（簡報公版等） | [claude-skills.md](claude-skills.md) |
| [`linktree`](https://github.com/visionbase-usc/linktree) | 對外 | 公開 | 對外連結首頁（visionbase.app） | [linktree.md](linktree.md) |
| [`VBwebsite`](https://github.com/visionbase-usc/VBwebsite) | 對外 | 私人 | VISION BASE 對外形象官網（建置中） | [VBwebsite.md](VBwebsite.md) |
| [`visionbase-monitor`](https://github.com/visionbase-usc/visionbase-monitor) | 內部 | 私人 | 區網裝置監控 + 攝影機視覺分析 | [visionbase-monitor.md](visionbase-monitor.md) |
| [`VBot`](https://github.com/visionbase-usc/VBot) | 自動化 | 私人 | VISION BASE Discord 機器人 | [VBot.md](VBot.md) |
| [`VB_kuka_KR120_r2700`](https://github.com/visionbase-usc/VB_kuka_KR120_r2700) | 內部 | 私人 | KUKA KR 120 R2700 web digital twin（FK/IK、軌跡、GH 推送） | [VB_kuka_KR120_r2700.md](VB_kuka_KR120_r2700.md) |
| [`RGBD_SLAM`](https://github.com/visionbase-usc/RGBD_SLAM) | 研究 | 私人 | （待補充，GitHub 描述為空） | [RGBD_SLAM.md](RGBD_SLAM.md) |
| [`chorono`](https://github.com/visionbase-usc/chorono) | 研究 | 私人 | （待補充，GitHub 描述為空） | [chorono.md](chorono.md) |

## 相關資料夾

- [`06_CLAUDE_Skills/`](../06_CLAUDE_Skills/) — `claude-skills` repo 內每個 skill 的詳細文件鏡像
