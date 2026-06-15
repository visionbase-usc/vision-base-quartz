---
title: KUKA 控制 / 監控可行性
author: metaarchetech
created: 2026-04-29
updated: 2026-05-24
source: visustwin-vault
source_path: 01 Projects/visionbase/kuka-control-feasibility.md
tags: [type/note, status/draft, project/visionbase-lab, area/physical-ai]
---

# KUKA 控制 / 監控可行性

2026-04-29 進駐 VISION BASE Lab 後的研究筆記。**結論:正常的監控與外部控制都可以自寫,完全不需要破任何安全機制。**

---

## 結論先行

| 議題 | 結論 |
|---|---|
| 監控(讀變數 / 狀態) | ✅ 完全自寫得起來,介面是公開協定 |
| 外部控制(讓機器人動作) | ✅ 透過 EKI + KRL,合規方式自寫 |
| 是否需要繞 safety | ❌ 不需要,也不應該 |
| 是否需要付費 license | ⚠️ 看深度:基本監控免費,RSI / 進階整合要付 |
| 八十萬等級設備能否 vibe coding | 可以,但要嚴格分階段,真機部分由人類主導 |

---

## 監控介面(讀資料)

### 1. KUKAVARPROXY(免費,社群)

- 第三方 server 跑在 KRC,開 TCP `7000`
- 客戶端可讀 / 寫 KRL 變數(`$POS_ACT`、`$AXIS_ACT`、自訂變數)
- 適合做即時 dashboard、KPI 監控
- 安裝需要 KRC 端有人配合(放 .exe 進去)

### 2. EKI(EthernetKRL Interface)

- KSS 8.x+ **內建**,不額外付費
- 在 KRC 上跑 socket server,配 XML config 收發訊息
- KRL 端用 `EKI_Init` / `EKI_GetXxx` / `EKI_Send` 操作
- **可讀可寫**,所以也是控制路徑

### 3. OPC UA

- KRC5 標配,KRC4 加價購(`OPC UA Server` option)
- 工業最標準的監控協定,接得進 ignition / Grafana / Node-RED
- 唯讀(預設),寫入要加 client 認證
- 推薦做長期生產監控

### 4. RSI(Robot Sensor Interface)

- **付費 license**,4ms 控制週期
- 真正用來做即時反饋控制(force / vision feedback loop)
- **單純做監控用不到**,寫起來複雜很多

---

## 外部控制流程(讓機器人動)

需要 4 個前置條件,缺一不可:

1. **模式切到 EXT(External Automatic)**:KCP(教導器)上實體切換,需有鑰匙
2. **KRC 端寫 KRL loop**:接 EKI 訊號,把外部訊息翻譯成動作指令
3. **訊號協定先約定**:外部送 JSON / XML,KRC 端 parse
4. **Safety 不會被繞過**:E-stop / 速度限制 / 工作區限制全部仍生效

```
[外部 PC]                     [KRC]
   │                            │
   │  TCP / EKI socket          │
   ├───────────────────────────▶│
   │  {"cmd":"moveJ","p":...}   │
   │                            │
   │                  KRL loop:
   │                  - parse msg
   │                  - PTP / LIN 動作
   │                            │
   │◀───────────────────────────┤
   │  {"status":"ok","pos":...} │
```

---

## 不能繞的東西(法規 + 設計,不是技術障礙)

| 項目 | 原因 |
|---|---|
| Safety controller | 獨立 PLC,跟 KRC 隔離,符合 ISO 13849 |
| E-stop | 硬體電路,軟體看不到 |
| 物理鑰匙開關(模式切換) | KCP 上的鑰匙,沒有遠端等價物 |
| License-gated 介面(RSI / 進階 OPC UA write) | KUKA 商業模式 |
| 直接控馬達繞過 interpolator | 不存在 API,要拆機改硬體(離譜) |

> 想繞這些 = 違法 + 危險。**沒有任何理由要繞。**

---

## Vibe Coding 的安全紀律

八十萬等級設備,公安考量,LLM 不能主導物理動作流程。建議分 5 個 Phase:

### Phase 1 — Mock 開發(離線)

- 純前端 + mock API,所有「機器人狀態」都是假的
- 設計 dashboard、操作流程、UX
- LLM 全程主導 OK

### Phase 2 — 模擬器(離線)

- **KUKA Office Lite** — 官方 KRC 模擬器,跑 KRL、模擬 EKI socket
- **Isaac Sim** — NVIDIA 物理模擬,搭 ROS 2 + KUKA URDF,可驗證碰撞 / 路徑規劃
- LLM 跟模擬器互動 OK,寫 KRL / XML / Python 都可以
- 找錯的成本 = 0

### Phase 3 — T1 模式真機(慢速)

- 真機上電,KCP 切 T1 模式(降速 250 mm/s,人手按住 enable key)
- 跑 KRL 程式,人類盯著
- LLM 可建議 KRL 改動,但**改動由人類審 + 鍵入**
- 每一步動作前先 dry-run

### Phase 4 — T2 模式真機(全速,工程師)

- T2 模式,全速但仍需 enable key
- 整合 EKI,跑外部控制流程
- 同樣**人類在現場主導**,LLM 純諮詢

### Phase 5 — EXT 模式(無人值守)

- 模式切 EXT,真正 production 部署
- **這個階段絕不在 vibe coding session 中切換**
- 由人類工程師親自完成,有完整 SOP / 風險評估文件後才上

---

## Isaac Sim 替代分析

| 替代得了 | 替代不了 |
|---|---|
| 物理模擬(碰撞、慣性) | KRC 控制器邏輯(KSS 系統的內部行為) |
| 視覺模擬(渲染、相機) | KRL 解譯器的真實行為(只有 Office Lite 模擬到一定程度) |
| 路徑規劃 / Inverse Kinematics | Safety controller 的回應 |
| 多機器人 + AMR 協作場景 | 真機 IO 訊號時序(EKI 會卡 ms 級延遲) |

→ **要做 AI / CV 整合**(YOLO 偵測物件 → 機器人抓取):用 Isaac Sim + ROS 2 跑完整 pipeline,真機只在 Phase 4-5 上場
→ **純監控 dashboard**:Isaac 用不到,直接接 OPC UA / KUKAVARPROXY 開發即可

---

## 接下來在 VISION BASE Lab 的執行步驟

1. 確認 KRC 是 KSS 8.x 還是 9.x,以及是 KRC4 還是 KRC5(影響可用介面組合)
2. 確認 KRC 是否在另一個網段 / VLAN(目前 `192.168.0.0/24` 只有 `0.82` 是 Windows 工作站,真正控制器不在這個網段)
3. 取得 KCP 鑰匙存放位置與借用流程
4. 申請 / 確認 OPC UA option(若 KRC4 + 沒有,要先評估是付費還是用 KUKAVARPROXY 替代)
5. Phase 1 監控網頁先 mock,等 KRC 接上再切真實資料源(可在 [[visionbase-monitor]] 加 `/kuka` 路由)

---

## 返回

- [[network-inventory]]
- [[visionbase-monitor]]
- [[kuka-loop-architecture-options-v0]]
