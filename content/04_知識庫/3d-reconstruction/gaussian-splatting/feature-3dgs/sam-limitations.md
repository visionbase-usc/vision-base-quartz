---
title: SAM 的極限與實際效果
author: metaarchetech
created: 2026-05-24
updated: 2026-05-24
source: visustwin-vault
source_path: 04 Knowledge/Lived/Gaussian-Splatting/feature-3dgs/SAM 的極限 & 實際效果.md
tags: [type/note, status/done, area/3d-reconstruction, area/gaussian-splatting, area/ml]
---

# SAM 的極限與實際效果

## SAM 是什麼，又不是什麼

- **SAM（Segment Anything）** 在 viewer 中產生的「feature map」彩色區塊，只是把特徵空間中**顏色／紋理相近**的像素聚成一塊，**沒有語意**（無法知道這是樹、車或背景）。
- SAM 沒有自帶語意分類訓練，無法自動「命名」物件。
- 對於 digital twin、BIM、下游 3D AI 任務，畫面上「顏色隨便分塊」的結果非常有限——這些任務需要的是明確標記「這是哪個物件」。

## 要做語意分割，必須用 LSeg / CLIPSeg

- **LSeg / CLIPSeg / Zero-shot Segmentation**
  - 使用 Vision-Language Model（CLIP）為每個像素／區域指派語意標籤（car、tree、road、person…）
  - 不是分「特徵相似區」，而是分「真實世界物件類別」——這對 3DGS 導出語意點雲、智慧城市建構、產線分析、AR/VR 都是關鍵需求
  - 可直接輸入自訂語意 prompt（例如「只顯示建築」、「選擇所有行人」）

## 下一步建議方向

> 直接把 pipeline 切到 LSeg / CLIPSeg 等語意分割模型，做自動化或半自動化語意標註。

理由：

- SAM 的分割只能粗切區塊，無法多物件精細語意分類
- LSeg / CLIPSeg 可直接產出「這是哪一類」的 mask 與分割圖
- 3DGS 取得語意 mask（搭配相機參數）後，可映射回每個點，讓 3D 點雲有語意標註

## 結論小抄

- **SAM 只能分連貫區塊，不知道是什麼東西（沒有語意）**
- **LSeg / CLIPSeg 能產生語意標註 mask，是 3DGS 結合多物件自動分割的主流解法**
- **Feature-3DGS 若要做真正的「語意物件偵測與分割」，必須額外串 LSeg 或同級技術**

---

## 只用 SAM + Viewer 能做到什麼？

### 各種視圖類型的功能

- **RGB** — 傳統彩色重建圖，無分割，用於原始視覺效果比對
- **Depth** — 每個點到相機的距離（灰階圖），可用於 3D 重建、碰撞、高度圖；無語意區分
- **Edge** — 顯示影像邊緣（物件輪廓），有助於找出「不同區塊」但不等同物件分割
- **Normal** — 每個點的法線方向（彩色），用於檢查表面結構、材質方向；無法區分語意
- **Curvature** — 顯示表面曲率（彎曲程度），高曲率區常代表物件邊緣，輔助分割但仍需人工介入
- **Feature Map** — 顯示模型學到的高維特徵；可作為分割依據，但**這些區塊沒有語意，只代表特徵相似度**
- **Show Input Points** — 顯示原始輸入點雲，可檢查資料品質與投影精度

### SAM 分割在 Viewer 中的實際效果

- **自動分割**
  - 可把特徵圖分成許多區塊（顏色／紋理／特徵相似）
  - 常見情形：一個物件被分成多塊，或多個物件合成一塊（完全沒有語意標籤）
- **互動式分割**
  - 在 viewer（或 2D 圖）上手動點擊，SAM 幫你分出該塊 mask
  - 適合「我要挑某一塊」的情境，無法自動分類所有東西
- **應用範圍**
  - 前處理：先用 SAM 切區塊，再手動選擇要的物件
  - 半自動標註：作為訓練語意分割的初始 mask
  - 資料檢查：觀察模型學到哪些特徵、空間是否被有效分割

### 侷限

- 無法做「自動物件語意分割」
- 無法直接用於物件統計、查詢、分析

---

## 應用方向總結（只用 SAM + Viewer 時）

| 應用 | 是否可行 | 補充 |
|---|---|---|
| 把場景粗略切成幾十至上百塊 | 可行 | 區塊對應 feature 相近區域 |
| 手動挑選 mask 做局部編輯／導出 | 可行 | 需人工判斷每個 mask |
| 批量產生 mask 做資料標註 | 可行 | 可大幅省人工框選時間，但需人工複審 |
| 自動分類所有語意物件 | 不可行 | 需 LSeg / CLIPSeg 或語意分割模型 |
| 直接做物件級統計、查詢 | 不可行 | mask 無語意 |

---

## 總結

- SAM + 各種 viewer 視圖主要是「特徵級」分割，不具語意，不會標註物件是什麼
- 最適合資料前處理、人工輔助標註、模型行為可視化與 QA
- 要做到「自動語意分割、多物件自動標註」，必須導入 LSeg 或類似模型

## 相關筆記

- [[semantic-segmentation-models]] — 語意分割模型落地規劃
- [[lseg-clip]] — LSeg + CLIP 原理
- [[sibr-remote-viewer]] — SIBR Viewer 操作說明
