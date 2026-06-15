---
title: A-數據之門-3 — TD 專案總覽 / 資料夾 / 踩坑全紀錄（交付文件）
zone: A
author: changchen0913
created: 2026-06-09
updated: 2026-06-09
status: deliverable
tags:
  - 專案/寶鋪
  - 數據之門
  - zone-A
  - touchdesigner
  - 結構
  - 踩坑
related:
  - "[[A-數據之門-2]]"
  - "[[A-數據之門-TD搭建計畫]]"
---

# A-數據之門-3 — 專案總覽 / 資料夾 / 踩坑（交付文件）

> [!info] 快照說明
> 本檔為 **2026-06-09** 對實機（`MD_TOOL_POP.67.toe`，TD 099 2025.32460）的即時掃描整理。專案持續編輯中，**以實機為準**。
> 設計腳本與硬體規劃見 [[A-數據之門-2]]；本檔聚焦「**目前 TD 實作長怎樣 + 資料夾 + 踩過的坑**」。

---

## 1. 這是什麼 / 用途

寶舖 showcase **A 區「數據之門」**的 TouchDesigner 實作。一個 **7 分鏡的沉浸式體驗**：訪客進場 → 偵測輪廓/環境數據 → 數據泡泡圍繞身體 → 集氣匯聚 → 外包動畫。

- 體驗用 **Scene Changer 切分鏡**，每一幕一個畫面來源。
- 感測（輪廓/blob/環境數值/年齡）由第三方 **Motion_detection_tool（MD_TOOL）** + YOLO/InsightFace 提供。
- 最終由 **`FINALPLAY` 視窗**輸出（顯示 Scene Changer 的當前場景）。

---

## 2. 磁碟資料夾總覽

`C:\Users\frncs\Desktop\寶鋪\數據之門\`

| 項目 | 說明 |
|---|---|
| **`MD_TOOL_POP.67.toe` / `.toe`** (37.7 MB) | ⭐ **數據之門主專案檔**（本檔描述的對象） |
| `aurora.26.toe` / `aurora.toe` (104 KB) | 另一支**音訊分析**專案（三頻段粒子環，獨立，非數據之門） |
| `Backup/` (51 個 .toe) + `備份/` | 自動/手動備份 |
| `TDYolo_v2.tox` (37 MB) | YOLO 元件（人體偵測，被 MD_TOOL 內嵌） |
| `det_10g.onnx` (16.5 MB)、`genderage.onnx` (1.3 MB) | InsightFace 模型（偵測 + 年齡/性別） |
| `face_analyzer.py`、`diag.py` | 臉部分析 / 診斷腳本 |
| `python_libs/` | 打包的 Python 套件（**opencv/cv2、numpy、onnxruntime、protobuf**）給 YOLO/臉部分析用 |
| `YOLO/`（含 `.venv`） | YOLO 環境 |
| `touchdesigner-mcp-td/` | **TD MCP server** 模組（讓 Claude 連 TD 的橋） |
| `Neo_佩綺_…哇！原來人的.mp3`、`…歡迎來到寶舖.mp3` | **分鏡2 AI 語音**素材 |
| `install.bat` / `fix_libs.bat` / `cleanup_old_v2.bat` / `README_setup.md` | 安裝/維護腳本與說明 |

---

## 3. TD 專案總覽 — 場景對應分鏡

整個 show 用 **Scene Changer** 在 6 個場景間淡切，每個場景 = 根目錄一個 null/TOP 當畫面來源：

| 鍵 | 場景 | 來源 TOP | 來源內容 | 對應分鏡 |
|---|---|---|---|---|
| `1` | **S1** | `/result` | ← `Motion_detection_tool/out1`（輪廓/粒子）| 進場 |
| `2` | **S2** | `/wave` | ← `/step2audioAnalysis/final`（聲波視覺化）| 前言（AI 語音波形）|
| `3` | **S3** | `/SCAN` | ← `/container3/out1` | 掃描 |
| `4` | **S4** | `/add1` | = `/result` + `/container1`（輪廓 + 數據泡泡）| 數據生成 |
| `5` | **S5** | `/converge` | ← `/container4/out1` | **集氣匯聚**（黑底+泡泡聚中心）|
| `6` | **S6** | `/REXVID` | ← `/container2/out1` | 外包動畫 |

> 各分鏡內容做在獨立 container：`container1`=數據泡泡互動、`container2`=外包影片、`container3`=掃描、`container4`=集氣、`step2audioAnalysis`=語音波形。

---

## 4. Scene Changer 切換骨架

| 節點 | 角色 |
|---|---|
| `/scene`（base COMP）| **場景庫**：S1~S6，每個是 container，內含 `select1`(引用來源 TOP) + `out1` |
| `/sceneChanger` | TD palette **Scene Changer 元件**，`Scenes COMP`=`/scene`；輸出 `out1`=當前場景 |
| `/sceneChangerUi` | Generate UI 產的控制面板（**半成品 / 已棄用**，見踩坑 #14）|
| `/sceneKeys`（keyboardin）+ `/sceneKeys_callbacks` | **鍵盤 1~6 → `SceneChange(鍵-1)`** 切場景（含淡切）|
| `/FINALPLAY`（window）| **正式輸出視窗**，winop = `/sceneChanger` |

**切換方式**：按鍵盤 `1`~`6`，或呼叫 `op('/sceneChanger').SceneChange(index)`。

---

## 5. 各 Container 詳解

### `/container1` — 分鏡4 數據泡泡互動（最複雜）

輸入：`in1` ← `Motion_detection_tool/out2`；`flip1`；`nvbackground1`（NV 去背，取得人形遮罩）

| 子系統 | 節點 | 做什麼 |
|---|---|---|
| **身體輪廓** | `trace1`→`join1`→`geo1`(line1 材質)→`render1`(cam1) | 紅色人體輪廓線 |
| **blob 框** | `datto1`(每 blob u/v/width)→`math1`→`null2`→`geo2`(line2 材質,矩形框 instance 在 blob 中心)→`render2`(cam2) | 跟著人的方框 |
| **blob 數據標籤** | `null3`→`labelFix`(x×1280/405 修正)→`chopto1`→`convert1/2`/`merge`→`null4`→`text1`(Text TOP **spec DAT** 定位多文字)；`dataPick`(Evaluate DAT,每 blob 隨機取一筆數據) | 框中心顯示隨機數據 |
| **矩陣字** | `gen_chars`(模組)→`text2`(30×114 二進位+感測器名)→`comp2`(text2 × nvbackground,遮在身上) | 身體裡的數據雨 |
| **常駐數據泡泡** | `bubMap`(Lissajous 環 + converge lerp)→`bubRename`→`bubPos`→`bubData`/`bubHeader`/`bubSpec`→`bubText`(Text TOP spec DAT)；`labelAtlas`(8 行數據,**來源=`Motion_detection_tool/merge1`,靠 index 對應**)；`bubConverge`(cv,由 container4 驅動) | 8 顆數據泡泡繞圈飄、可匯聚 |
| **合成輸出** | `comp1`(render1+render2+text1+bubText)+`comp2`→`add1`→`null5`→`out1` | |
| 其他 | `RES`(constant,集中控制解析度) | |

### `/container4` — 分鏡5 集氣匯聚

| 節點 | 作用 |
|---|---|
| `bg`(黑底 constant) + `bubbles`(select ← `/container1/bubText`) → `comp1`(over) → `out1` | 黑底 + 只有常駐泡泡 |
| `cvTarget`(constant,= 1 當 `sceneChanger.Currentscene==4`) → `cvLag`(平滑 ramp) → 驅動 `/container1/bubConverge` 的 cv | **切到 S5 自動匯聚到中心，切走自動散開** |

> 「身體/矩陣淡黑」由 Scene Changer 從 S4 淡切到 S5 時自動發生（S5 只有黑底+泡泡）。

---

## 6. 其他關鍵節點

| 節點 | 角色 |
|---|---|
| `/Motion_detection_tool`（MD_TOOL）| 第三方感測工具。`out1`/`out2`(輪廓輸出)、`VIDEO`(影像源)、`light1`(程序人形數據體)、`merge1`(**8 環境數值**:CO2/HCHO/PM2.5/PM10/TVOC/光照/溫/濕,但頻道名亂)、`TDYolo_v2`(人體偵測)、`Device`(webcam 選擇器) |
| `/STAGE` | 牆面 render context：`WALL_CANVAS`(寬畫布)、`crop_*`(三分區)、`IN_LEFT/FRONT/RIGHT/FLOOR`(四面)、`WRAP_SRC` |
| `/WALL_WINDOW`（window）| 顯示 `/STAGE/WALL_CANVAS`（三牆拉伸輸出）|
| `/perform`（window）| 顯示 `/UI_Screen`（FPS/UI 監看）|
| `/FloorFX` | 地面效果 |
| ⚠️ `/SHOW` | **已刪除**（原本的狀態機/scene 表/data_sim，現由 Scene Changer 取代；刪除曾連帶弄壞 labelAtlas 引用，見踩坑 #10）|

---

## 7. 踩過的坑 + 解決方法

### A. TD 實作坑（本次製作）

| # | 坑 | 解法 |
|---|---|---|
| 1 | **Instance Scale 把常數當頻道名** — geo 的 Instance Scale X/Y 填數字,會被當成「要去 instanceop 找的頻道名」→ 找不到 → 忽略(scale=1)。warning：`instance attribute '0.24' could not be found` | 用**頻道**(如 'width')縮放,或改用**幾何本身尺寸**(rectangleSOP 的 Size) |
| 2 | **Instancing 開關預設關** — 沒開,geo 會把一個 quad 渲染成滿尺寸鋪滿畫面 | 打開 geo 的 **Instancing** toggle |
| 3 | **inCHOP「number of channels does not match」** — `specifynum` 設了固定頻道數,餵不符就輸出空 | `specifynum` 關掉,接受實際頻道數 |
| 4 | **pointSOP `dovel` 引用不存在的點屬性 `v`** → 報錯 | 沒用到就把 `dovel` 關掉 |
| 5 | **scriptDAT 的 `.text` 透過 MCP 不可寫**（"operator is not editable"）| 改用可寫的：chopexec/keyboardin 回呼、Text TOP 的 `text` 運算式;或人工在 UI 編輯 |
| 6 | **Text TOP 的 border 是整張圖、不是每個 spec 元素** — 不能用 border 對每個文字畫框 | 框要另解（見 #7）|
| 7 | **框(3D 相機渲染)對齊文字(像素空間)極難** — cam2 ortho 的置中/長寬比換算超易錯,中心對、邊緣歪,還會「分頭飄動」 | **讓框跟文字是同一個物件/同一渲染路徑**才保證對齊同動;別用兩條不同空間硬對 |
| 8 | **泡泡飄動「一起動 / 折線 / 抖」** — noiseCHOP 相鄰 sample 在同 period 內相關 → 一起動;roughness/harmonics → 抖;太慢 → 折線 | 取樣點拉開(各自獨立)+ 降 roughness;或**乾脆用正弦/Lissajous 取代 noise**(保證平滑曲線、各自相位、不抖) |
| 9 | **menuSource 引用已刪 op → `'NoneType' has no attribute 'par'`** — 參數選單來源是 Python,指的 op 沒了。**這種錯 `node.errors()` 抓不到**,只在節點 info 的「Script errors」顯示 | 清掉或修正該參數的 menuSource |
| 10 | **刪 `/SHOW` 連帶弄壞引用** — `labelAtlas` 引用 `/SHOW/data_sim`,/SHOW 被刪 → 報錯 | 刪/改被引用的節點前先查 reference;此例改指 `/Motion_detection_tool/merge1` |
| 11 | **兩個 TD instance + MCP port 衝突** — 一個 port 只能跑一個 webserver,9981/9982 兩專案搶 | 要連哪個,就把那個的 `mcp_webserver` 開在 client 連的 port(關掉另一個的)|
| 12 | **假數據單位硬寫錯** — constant 的單位字串硬寫死(PM10 顯示成 PM2_5)| 從**乾淨數值 + 單位對照表**生成文字,別直接餵硬寫字串 |
| 13 | **`merge1` 頻道名亂**(ppm/ppm1/PM2_5/PM2_6/PM2_7,舊命名 bug)| 暫時靠 **index 順序**對應(⚠️ 改順序會跑掉,建議之後把 constant 頻道名改乾淨)|
| 14 | **Scene Changer 雷區** | ① Generate UI 是**空殼**(空 chopexec + A/B/C 佔位,點了沒反應);② 場景必須是 **COMP(非裸 TOP)且有 `out` TOP**;③ `Scenes COMP` 指向**裝場景的容器**;④ 面板要用 **Panel 檢視**才看得到按鈕(網路檢視看不到,widget 尺寸 0);⑤ `SceneChange(index)` 本身就含淡切 |
| 15 | **Force-cook 不反映即時** — audio/feedback/particle/lag/chopexec/噪聲動畫,用 force-cook 抓的像素不可靠 → 盲測判斷常出錯 | 動態系統靠**肉眼 / 實際播放**驗證,別只信像素取樣 |

### B. 規劃 / 硬體坑（承自 [[A-數據之門-2]]）

| 坑 | 解法 |
|---|---|
| **D555 走 Ethernet/DDS 進不了 TD**(TD RealSense 是 USB 取向)| 用 Python(`pyrealsense2`+DDS)抓幀 → **Spout** 同機橋進 TD;`pyrealsense2`+DDS 有未解 bug(librealsense #14532),需 source build 加 `-DBUILD_WITH_DDS=ON`;退路走 **ROS 2** node |
| **depth over DDS 鎖 30Hz**（60Hz 拿不到）| 延遲預算用 30fps 估 |
| **消費級顯卡(4080/3070)無硬體 genlock** → 跨機/跨視窗瞬切(爆白)會差一幀 | 爆白/硬切做成 **3–4 幀(80–120ms)急速 ramp**,別 1 幀硬切 |
| **跨機/跨層串 TOP/NDI 影像** → 延遲、卡頓 | 只傳算好的**資料**,兩端各自重建畫面 |
| **Web Render TOP 延遲過高**（CEF 無真 GPU 加速）| 不要用它當輸出橋;走原生或共享 GPU 材質(Spout)|
| **投影機 input lag 是延遲最大單點** | 選低延遲機型 + 遊戲模式 + 關所有後處理 |
| **抖動比絕對延遲更傷** | 鎖 fps、固定平滑窗口 |

---

## 8. 交付備註

- 主檔：`MD_TOOL_POP.67.toe`（資料夾另有 51 份 Backup）。
- 切場景：鍵盤 `1`~`6`（`/sceneKeys`）。
- 環境數據目前來源 = `Motion_detection_tool/merge1`（模擬值）；接真 MQTT 時改指該處即可。
- 相機源（D555）尚未驗證打通（見 B 區第一條），為**最關鍵前置**。
- `aurora.toe` 是另一支音訊分析專案，與本 show 無直接關係。
