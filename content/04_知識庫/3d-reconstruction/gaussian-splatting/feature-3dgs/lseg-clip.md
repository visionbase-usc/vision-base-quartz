---
title: LSeg + CLIP 原理
author: metaarchetech
created: 2026-05-24
updated: 2026-05-24
source: visustwin-vault
source_path: 04 Knowledge/Lived/Gaussian-Splatting/feature-3dgs/LSeg_CLIP.md
tags: [type/note, status/done, area/3d-reconstruction, area/gaussian-splatting, area/ml]
---

# LSeg + CLIP

> **LSeg** = Language-driven Semantic Segmentation
> **CLIP** = Contrastive Language–Image Pretraining
>
> LSeg 整個架構建立在 CLIP 的基礎上，將 CLIP 的語意空間延伸到 pixel-level segmentation。

## 一句話總結

> **CLIP 是語意理解的基礎，LSeg 是把它變成逐像素語意的應用。**

---

## 對照表

| 類別 | CLIP | LSeg |
|---|---|---|
| 任務目標 | 將圖片與文字拉到同一語意空間 | 將每個「像素」拉到對應的文字語意空間 |
| 輸入 | 一張整圖 + 一段文字 | 圖片（每個像素） + 一組語意標籤文字 |
| 模型用途 | 類別辨識、語意匹配 | 語意 segmentation（甚至 zero-shot） |
| Text Encoder | CLIP 原本的文字 encoder（如 ViT-B/32） | **直接重用 CLIP 的文字 encoder** |
| Image Encoder | CLIP 的整圖 encoder | 改造為 per-pixel encoder（ViT、ResNet 等） |
| 特點 | 可詢問「這是貓嗎？」 | 可詢問「圖中哪些像 `grass`？」並畫出區域 |
| Zero-shot 能力 | 對整張圖分類 | 對像素 zero-shot 分類 |

---

## LSeg 如何用到 CLIP？

LSeg 把 CLIP 拆成兩部分使用：

1. **Text Encoder（CLIP 原本的文字模組）**
   - 將文字標籤（如 `"dog"`、`"tree"`、`"metal pole"`）轉成語意向量
2. **Image Encoder（改裝後的 CNN 或 ViT）**
   - 將圖片的每個像素轉成語意向量

接著透過 **contrastive learning** 把 pixel 向量對齊到文字向量，模型學會「這個像素看起來像 `tree`」、「那一區像 `sky`」。

---

## 實際執行 LSeg 時會看到的元件

- `clip` 模組（來自 openai/CLIP）：被 import 進來做文字嵌入
- `ViT` / `ResNet` 背骨：做 per-pixel feature embedding
- `CLIPSeg`、`CLIPTextEncoder`：LSeg 中的 component

---

## 相關筆記

- [[development-direction]] — Feature-3DGS 開發方向
- [[semantic-segmentation-models]] — 語意分割模型整合計畫
- [[lseg-unicode-fix-report]] — Windows 上 LSeg Unicode 編碼問題解決方案
