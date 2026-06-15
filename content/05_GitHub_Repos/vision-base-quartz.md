---
title: vision-base-quartz repo
author: metaarchetech
created: 2026-06-15
tags: [type/note, status/done, area/governance]
---

# vision-base-quartz

VISION BASE Lab vault 的對外發佈站，用 [Quartz 5](https://quartz.jzhao.xyz) 把 [`vision_base_vault`](vision_base_vault.md) 的內容轉成靜態網站。內容鏡像到 `content/`，push 到 `main` 後由 GitHub Actions 自動 build + 部署到 GitHub Pages。

## 基本資訊

| 項目 | 值 |
|---|---|
| **URL** | [github.com/visionbase-usc/vision-base-quartz](https://github.com/visionbase-usc/vision-base-quartz) |
| **類型** | 靜態網站(Quartz 5 digital garden) |
| **可見性** | 公開 |
| **預設分支** | `main` |
| **部署** | GitHub Pages(GitHub Actions 自動部署 `main`) |
| **正式網址** | [visionbase-usc.github.io/vision-base-quartz](https://visionbase-usc.github.io/vision-base-quartz/) |

## 技術棧

| 層 | 選用 |
|---|---|
| 產生器 | Quartz `5.0.0` |
| 設定 | `quartz.config.yaml`(`pageTitle: VISION BASE Lab`、`locale: zh-TW`、`analytics: null`) |
| 內容 | `content/`(整個 vault 鏡像：00～06 + `_attachments`) |
| 部署 | `.github/workflows/deploy.yml`(`npx quartz build` → `upload-pages-artifact` → `deploy-pages`) |

## 隱私 / 收錄

- `content/robots.txt`(`Disallow: /`)→ 擋搜尋引擎自然收錄，但知道網址的人仍可直接瀏覽。
- 尚未加登入驗證。GitHub Pages 本身不支援 access control；若日後要做「Google 登入才能看」需搬到 Cloudflare Pages + Cloudflare Access。

## 更新流程

1. 在 vault 編輯內容。
2. 同步到本站 `content/`(目前為手動鏡像，非自動)。
3. push 到 `main` → GitHub Actions 自動重新 build + 部署。

> ⚠️ 本站 `content/` 與 [`vision_base_vault`](vision_base_vault.md) 是兩份；vault 更新後需同步過來才會反映到網站。

## 維護者

- 建置 / 維護：[@metaarchetech](https://github.com/metaarchetech)

## 相關 repo

- [`vision_base_vault`](vision_base_vault.md) — 內容來源 vault
- [`linktree`](linktree.md) — 對外連結首頁
