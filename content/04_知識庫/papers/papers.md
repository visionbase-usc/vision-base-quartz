---
title: Papers 摘要集中地
author: metaarchetech
created: 2026-05-24
tags: [type/note, status/done, area/governance]
---

# Papers

跨領域論文摘要集中地。**一篇論文 = 一個檔**，三句話 + 我的看法。

## 為什麼集中放這裡

很多論文同時跨多個 area（例：用 Gaussian Splatting 做 Digital Twin），與其決定要放哪個子資料夾、不如全部放這、靠 tag 交叉查詢。

## 命名規則

`<first-author-surname>-<keyword>-<year>.md`

範例：
- `mildenhall-nerf-2020.md`
- `kerbl-3dgs-2023.md`

## Front-matter 必填

```yaml
---
title: <paper title>
author: metaarchetech
created: YYYY-MM-DD
tags: [type/paper, status/done, area/<x>, area/<y>]
paper_title: <原論文標題>
paper_authors: <原作者列表>
paper_year: YYYY
paper_url: <arxiv / DOI 連結>
---
```

## 內容模板

```
## 三句話摘要
1.
2.
3.

## 主要貢獻


## 我的看法 / 對 VISION BASE 的意義


## 相關
- [[相關筆記 1]]
- [[相關筆記 2]]
```

## 大檔 PDF

PDF 本體**不放 vault**（>5MB），放外部雲端、本檔貼連結。
