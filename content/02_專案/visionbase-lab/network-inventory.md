---
title: VISION BASE Lab 區網盤點
author: metaarchetech
created: 2026-04-29
updated: 2026-05-24
source: visustwin-vault
source_path: 01 Projects/visionbase/network-inventory.md
tags: [type/note, status/draft, project/visionbase-lab, area/physical-ai]
---

# VISION BASE Lab 區網盤點(`192.168.0.0/24`)

掃描時間:2026-04-29 14:22 UTC(`scan.json` `scannedAt`),掃描耗時 17 秒。
共 **34 台裝置**,**29 在線 / 5 離線**。

資料源:
- `visionbase-monitor/.cache/scan.json`(PowerShell 掃描 + enrich,有 ownership / category)
- `network-scan/devices.json`(較早一輪的 raw 結果,vendor 字串較完整,用來補欄位)

> 備註:Personal 類(個人手機 / 隨機 MAC)的 MAC 略保留前 3 段,其餘照實列。

---

## 基礎建設(Infrastructure / Lab 財產)

### Router & Network Core

| IP | MAC | Vendor | Hostname | Type | Online | Ports |
|---|---|---|---|---|---|---|
| 192.168.0.1 | `10:7C:61:DB:D6:EC` | ASUSTek | `RT-AX1800S-D6EC.local` | ASUS Router | ✅ | 80, 8443 |
| 192.168.0.161 | `74:4D:BD:85:F7:B0` | Espressif | — | Router/Embedded(疑似 Bambu MQTT 站) | ✅ | 990, 6000, 8883 |

> `0.161` 雖被 PowerShell 標為 Router(TTL=255),但 OUI 是 Espressif,開放 990/6000/8883 與 0.127 / 0.217 同一 pattern,**比較可能是 Bambu Lab 3D 印表機的 MQTT 控制端**。待現場確認。

### NAS

| IP | MAC | Vendor | Hostname | Type | Online | Ports |
|---|---|---|---|---|---|---|
| 192.168.0.85 | `90:09:D0:5F:75:D9` | Synology | `VISIONBASE_NAS.local` | Synology NAS | ✅ | 80, 443, 445, 5000, 5001 |

### Printer

| IP | MAC | Vendor | Hostname | Type | Online | Ports |
|---|---|---|---|---|---|---|
| 192.168.0.7 | `F8:25:51:C5:26:53` | Seiko Epson | `EPSONC52653.local` | Epson Printer | ✅ | 80, 443, 631, 9100 |

---

## 攝影機(Lab 財產,共 14 台)

### TP-Link Tapo(3 台)

| IP | MAC | Hostname | Type | Online | Ports |
|---|---|---|---|---|---|
| 192.168.0.36 | `9C:53:22:2E:1D:DD` | — | TP-Link Tapo Camera | ✅ | 443, 554, 2020, 8800 |
| 192.168.0.107 | `9C:53:22:2E:10:D3` | — | TP-Link Tapo Camera | ✅ | 443, 554, 2020, 8800 |
| 192.168.0.223 | `30:DE:4B:A8:A0:1C` | — | TP-Link IP Camera(RTSP+ONVIF) | ✅ | 443, 554, 2020, 8800 |

### ACTi(11 台,連續 IP)

vendor 全部是 `ACTi Corporation`,皆為標準 IP camera ports `80 / 443 / 554`。

| IP | MAC | Online |
|---|---|---|
| 192.168.0.200 | `00:0F:7C:1B:79:DB` | ✅ |
| 192.168.0.201 | `00:0F:7C:1B:F5:8E` | ✅ |
| 192.168.0.202 | `00:0F:7C:1B:F5:CC` | ✅ |
| 192.168.0.203 | `00:0F:7C:1B:F5:AD` | ✅ |
| 192.168.0.204 | `00:0F:7C:1B:F5:32` | ✅ |
| 192.168.0.205 | `00:0F:7C:1B:F5:C8` | ✅ |
| 192.168.0.206 | `00:0F:7C:1B:F5:9A` | ✅ |
| 192.168.0.207 | `00:0F:7C:1B:F5:C3` | ✅ |
| 192.168.0.208 | `00:0F:7C:1B:F5:CF` | ✅ |
| 192.168.0.209 | `00:0F:7C:1B:F5:C1` | ✅ |
| 192.168.0.210 | `00:0F:7C:1B:F5:D4` | ✅ |

---

## 機械手臂(Lab 財產)

| IP | MAC | Vendor | Hostname | Type | Online | Ports | 備註 |
|---|---|---|---|---|---|---|---|
| 192.168.0.82 | `AC:19:8E:CC:3D:3C` | Intel Corporate | `VB-KUKA-Only.local` | Robot Arm Controller PC | ❌ | 445(SMB) | **這是 KUKA 旁的 Windows 工作站**,不是 KRC。真正的 KUKA 控制器(KRC4 / KRC5)目前不在此網段(可能未開機或在隔離 VLAN) |

詳見 [[kuka-control-feasibility]]。

---

## 3D 印表機(疑似 Bambu Lab,Lab 財產 / 待確認)

OUI + port pattern 命中(MQTT 8883 / 6000 / 990 — Bambu printer 的標準 cloud/LAN port)。

| IP | MAC | Vendor | Hostname | Online | Ports |
|---|---|---|---|---|---|
| 192.168.0.127 | `CC:8D:A2:09:B1:1C` | Espressif | `espressif` | ✅ | 990, 6000, 8883 |
| 192.168.0.161 | `74:4D:BD:85:F7:B0` | Espressif | — | ✅ | 990, 6000, 8883 |
| 192.168.0.217 | `28:2D:06:00:1F:DA` | AMPAK Technology | — | ✅ | 990, 6000, 8883 |

---

## 電腦(8 台)

| IP | MAC | Vendor | Hostname | Type | Online | Ownership |
|---|---|---|---|---|---|---|
| 192.168.0.3 | `2E:62:98:**:**:**` | Apple(random) | `Mac` | Mac | ❌ | Personal |
| 192.168.0.91 | `BC:15:A6:00:49:C7` | Taiwan Jantek Electronics | — | Windows | ✅ | Unknown(疑似 ACTi NVR / 監控伺服器) |
| 192.168.0.92 | `BC:15:A6:00:49:D4` | Taiwan Jantek Electronics | — | Windows | ✅ | Unknown |
| 192.168.0.93 | `BC:15:A6:00:49:C1` | Taiwan Jantek Electronics | — | Windows | ✅ | Unknown |
| 192.168.0.94 | `BC:15:A6:00:50:17` | Taiwan Jantek Electronics | — | Windows | ✅ | Unknown |
| 192.168.0.178 | `F4:26:79:D8:2A:8E` | Intel Corporate | `DESKTOP-VISIONBASE09.local` | Windows PC | ✅ | Lab |
| 192.168.0.220 | — | — | `DESKTOP-JDT5BJ8` | Windows PC(本機) | ✅ | 開發機 |
| 192.168.0.237 | `9C:2F:9D:A4:91:A5` | Liteon Technology | `Yang-PC.local` | Windows | ❌ | Unknown |

> Jantek 4 台 IP 連續 + vendor 同 + 都只開 80 port + TTL=128 → 高度懷疑是 ACTi 11 台 NVR 旁邊的搭配 PC(Jantek 是 ACTi 攝影機常見的 OEM 合作)。需現場確認。

---

## 手機 / 平板(Personal)

| IP | MAC | Hostname | Type | Online |
|---|---|---|---|---|
| 192.168.0.83 | `02:A9:E0:**:**:**` | `iPad` | iPad | ❌ |
| 192.168.0.164 | `D2:36:FD:**:**:**` | `iPhone` | iPhone | ❌ |

---

## IoT(Espressif / Linux 嵌入式,4 台)

| IP | MAC | Vendor | Hostname | Type | Online | Ports |
|---|---|---|---|---|---|---|
| 192.168.0.100 | `A8:F7:E0:E5:FA:08` | — | — | Linux/Android/IoT | ✅ | 80, 443 |
| 192.168.0.127 | `CC:8D:A2:09:B1:1C` | Espressif | `espressif` | ESP32(也疑似 Bambu) | ✅ | 990, 6000, 8883 |
| 192.168.0.161 | `74:4D:BD:85:F7:B0` | Espressif | — | ESP32 / Bambu | ✅ | 990, 6000, 8883 |
| 192.168.0.217 | `28:2D:06:00:1F:DA` | AMPAK | — | Linux/Android/IoT(疑似 Bambu) | ✅ | — |

> `0.127 / 0.161 / 0.217` 同時被歸 IoT 也被歸 3D 印表機 — 是同一批裝置,類別重疊。

---

## 其他不明 / Apple 周邊(2 台)

| IP | MAC | Vendor | Hostname | Type | Online | 備註 |
|---|---|---|---|---|---|---|
| 192.168.0.151 | `EA:01:A1:**:**:**` | Apple(random) | — | Apple Device | ✅ | Personal,可能是 AirPods / Watch / 未命名 iPhone |
| 192.168.0.152 | `18:4A:53:20:11:C4` | Apple, Inc. | `VISIONBSEdeMini` | macOS / iOS Device | ✅ | Lab(hostname 含 VISIONBASE) |

---

## 漏掉 / 變動的裝置(較早一輪 vs 最新)

`network-scan/devices.json`(較早一輪)裡額外出現過:

| IP | MAC | Vendor | 備註 |
|---|---|---|---|
| 192.168.0.124 | `F8:4E:58:E4:44:BE` | Samsung | Samsung Android/IoT(7000, 8080),最新一輪不在 |
| 192.168.0.191 | `96:9D:7A:**:**:**` | Apple(random) | `MAC-CF698A`,AirTunes/AirPlay(5000, 7000),最新一輪不在 |

→ 都是個人 / Personal 裝置,人離開實驗室就不在。掃描結果本來就會浮動。

---

## 統計總表(分類交叉)

| 分類 | 全部 | 在線 | 離線 |
|---|---:|---:|---:|
| Camera | 14 | 14 | 0 |
| Computer | 8 | 6 | 2(0.3 Mac、0.237 Yang-PC) |
| IoT(含 Bambu 重疊) | 4 | 4 | 0 |
| Mobile | 2 | 0 | 2(0.83 iPad、0.164 iPhone) |
| Router | 2 | 2 | 0 |
| Robot Arm | 1 | 0 | 1(0.82 KUKA-Only) |
| NAS | 1 | 1 | 0 |
| Printer | 1 | 1 | 0 |
| Unknown(Apple 周邊) | 2 | 2 | 0 |
| **合計** | **34** | **29** | **5** |

| Ownership | 數量 |
|---|---:|
| Lab(學校 / 實驗室財產) | 21 |
| Personal | 4 |
| Infrastructure | 2 |
| Unknown(待釐清) | 7 |

---

## 返回

- [[visionbase-monitor]]
- [[kuka-control-feasibility]]
