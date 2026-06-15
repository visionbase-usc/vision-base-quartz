---
title: Augmented ML Agent — 3DGS 重建 + 語意分割 multi-agent pipeline
author: metaarchetech
created: 2026-05-24
updated: 2026-05-24
source: visustwin-vault
source_path: 04 Knowledge/Lived/AI-Frameworks/Augmented ML Agent.md
tags: [type/note, status/draft, area/ai-frameworks, area/3d-reconstruction]
---

# Augmented ML Agent

把「自動訓練 3DGS」的工作拆成多個專業 agent，讓任務可分工、模組可組合，串成 **ML 3D 重建 + 語意分割 + QA 優化 + 自動匯出**的全流程 AI pipeline。

## 一、Agent Team 架構總覽

| Agent 角色 | 主要責任 | 可用工具/行為 |
|---|---|---|
| **Orchestrator** | 規劃全流程、下指令、紀錄日誌 | 調度下游 agent、記錄 meta data |
| **Data Prep Agent** | 處理原始影像、資料驗證與預處理 | 影像格式轉換、檔案路徑管理、檢查 dataset |
| **Trainer Agent** | 執行 3DGS/4DGS 訓練、調超參數 | 調用 train.py、優化 batch、紀錄結果 |
| **Feature/Render Agent** | 產生特徵圖、執行各種視圖渲染 | render.py，特徵視圖 / RGB / 深度 |
| **Segmentation Agent** | 對 2D/3D 圖片做分割（SAM/LSeg/CLIPSeg） | segment.py、CLIPSeg pipeline |
| **Mapping Agent** | 將 2D 分割結果回映射到 3D 點雲 | 投影、標註、點雲處理 |
| **Semantic QA Agent** | 檢查語意分割品質、異常報告、輔助人工審查 | 評估 overlap / entropy、產生統計報表 |
| **Export/Deploy Agent** | 導出訓練結果、分割結果、USD/ply/報表 | 匯出檔案、同步雲端、送入下游工具 |

## 二、Pipeline 範例

1. **Orchestrator** 發起專案、分配任務
2. **Data Prep Agent** 整理 dataset、驗證品質
3. **Trainer Agent** 執行 3DGS 訓練，調參紀錄（可自動化 grid search）
4. **Feature/Render Agent** 產生各類 2D 圖片與特徵圖
5. **Segmentation Agent**
   - 先用 SAM 粗分割（協助 QA / 前處理）
   - 再用 LSeg / CLIPSeg 做語意分割，批次標註所有類別
6. **Mapping Agent** 把分割 label 映射回 3D 點雲，產生語意點雲
7. **Semantic QA Agent** 分析標註品質（不確定 / 重疊區），給出人工審查建議
8. **Export/Deploy Agent** 導出所有結果（點雲、分割結果、標註報表、USD/ply 檔），推送雲端或匯入後續流程

## 三、Agent 技術特徵與 Tool 配置

| Agent | Tool / Function | 自動化能力 |
|---|---|---|
| Data Prep Agent | Directory/FileTool、ImageConvertTool | 自動校對 / 整理 |
| Trainer Agent | Train3DGSModelTool（subprocess 調用） | 調參批次、效能優化 |
| Feature Agent | RenderViewTool、FeatureMapTool | 自動多視角渲染 |
| Segmentation Agent | SAMTool、LSegTool、CLIPSegTool | 自動 / 互動分割 |
| Mapping Agent | ProjectionTool（label 2D→3D） | 自動化點雲標註 |
| Semantic QA Agent | LogAnalyzer、OverlapChecker、EntropyStatsTool | 自動判別可疑區域 |
| Export Agent | ExportUSDTool、ExportPLYTool、SyncToCloudTool | 自動產生報表 / 結果 |

## 四、未來擴充

- **Agent 間訊息溝通**：根據 QA 結果自動重訓或調參
- **資源調度**：多 GPU / 多機訓練（Scheduler Agent）
- **即時 UI**：Streamlit / Web Dashboard，讓使用者用點擊 / 拖拉指定 agent 工作

## 一句話總結

> 不只是「自動訓練 3DGS」，而是打造一個**可隨需組合、任務可分工、多模組協作**的 ML 3D 重建 + 語意分割 + QA + 匯出全流程 AI pipeline。
