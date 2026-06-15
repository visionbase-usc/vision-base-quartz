---
title: vision-base-template skill
author: metaarchetech
created: 2026-05-24
tags: [type/howto, status/done, area/governance]
---

# vision-base-template

> 📌 **這是文件鏡像**。實際 skill 程式檔在 [`claude-skills/vision-base-template/`](https://github.com/visionbase-usc/claude-skills/tree/main/vision-base-template)。

## 用途

產生 VISION BASE 品牌識別套用的空白簡報模板（深色 / 淺色版），含內容改寫工具。

## 觸發

**預設觸發** — 安裝後在對話中提到任何簡報相關詞彙：
- 簡報 / pptx / 投影片 / deck / slides / 公版 / 模板 / 提案

Claude 都會自動套用 VB 公版，**不需要每次說「用 VB」**。

### 例外：明確 opt-out

如果這次工作不是 VB，明白講：「非 VB」/「一般簡報」/「給別的客戶」等。

## 安裝

1. 從 [`claude-skills` repo](https://github.com/visionbase-usc/claude-skills) 下載 `vision-base-template.skill`
2. 在 Claude Code 對話輸入：
   ```
   /plugin install ~/Downloads/vision-base-template.skill
   ```

## 環境需求

- **Python 3** + `python-pptx`（`pip3 install python-pptx`）
- **Avenir Next** 字型（macOS 內建；Windows/Linux 需另裝）
- **Noto Sans TC** 字型（skill 內含安裝腳本）

安裝字型：
```bash
python3 ~/path/to/vision-base-template/scripts/install_fonts.py
```

## 設計原則

> **初次只用黑白色；在需要重點強調時，給使用者品牌色選項，由使用者決定使用範圍與位置。**

執行流程：
1. 先產生**純黑白**版本
2. 問：「要不要加品牌色？要哪個？用在哪？」
3. 依使用者回答用 `VB_ACCENT=<色名>` 重新產生

## 6 色輔助色（品牌書定義）

| 等級 | 名稱 | Hex | `VB_ACCENT` |
|---|---|---|---|
| **Secondary**（優先建議）| 桃紅 | `#F21847` | `pink` |
| **Secondary**（優先建議）| 天藍 | `#3DA1D3` | `blue` |
| Tertiary | 橘 | `#F96419` | `orange` |
| Tertiary | 黃 | `#E6E143` | `yellow` |
| Tertiary | 紫 | `#5F4ADD` | `purple` |
| Tertiary | 綠 | `#6AD868` | `green` |

預設深色版、純黑白；用戶沒指定顏色時，Claude 會優先建議 pink → blue。

## 使用範例

```
幫我做一份「光沐 — 世界的阿里山」簡報，日期 2026.05
做一份提案投影片
我需要一個簡報模板
```

Claude 會自動：
1. 觸發 `vision-base-template`
2. 詢問深色 / 淺色
3. 產生 `VB_簡報公版_dark.pptx`（純黑白）
4. 問要不要加品牌色
5. 重新產生

## 維護者

[@anitoisa](https://github.com/anitoisa)

問題、bug、改進建議：在 [`claude-skills` repo](https://github.com/visionbase-usc/claude-skills/issues) 開 issue。
