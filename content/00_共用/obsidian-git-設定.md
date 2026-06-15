---
title: Obsidian Git 多人協作設定
author: metaarchetech
created: 2026-05-28
tags: [type/howto, status/done, area/governance]
---

# Obsidian Git 多人協作設定

新加入 vault 的成員、或同步出問題的成員，請照這份把 Obsidian Git 設定對齊到 lab 的標準值。設定不一致會出現「我這邊看不到別人的更新」、「我的修改沒同步出去」、「一直跳 conflict」等問題。詳細的 vault 使用規範請參考 [[使用規範]]。

## 核心原則

多人共筆的 git 工作流靠三件事撐起來：**開啟時先 pull、編輯停下才 commit、commit 完就 push**。任何一環關掉都會造成同步落差。Obsidian Git 預設值幾乎全部關閉自動行為，所以一定要照下面手動打開。

## 套用設定（兩種做法擇一）

### A. 直接編輯 data.json（最快）

完全退出 Obsidian（Cmd+Q，不是只關視窗），用文字編輯器打開：

```
<vault>/.obsidian/plugins/obsidian-git/data.json
```

把下面這些欄位改成對應數值。其他欄位保留現有值不要動。

| 欄位 | 建議值 | 說明 |
|---|---|---|
| `commitMessage` | `"{{hostname}} backup: {{date}}"` | commit log 看得出是哪台機器同步上來的 |
| `autoCommitMessage` | `"{{hostname}} backup: {{date}}"` | 同上，auto commit 的訊息格式 |
| `autoSaveInterval` | `5` | 每 5 分鐘 auto commit |
| `autoPushInterval` | `5` | 每 5 分鐘 auto push |
| `autoPullInterval` | `5` | 每 5 分鐘 auto pull |
| `autoPullOnBoot` | `true` | 開啟 Obsidian 自動先 pull 一次 |
| `pullBeforePush` | `true` | push 前先 pull，避免被 reject |
| `syncMethod` | `"merge"` | 多人共筆用 merge 比 rebase 安全 |
| `differentIntervalCommitAndPush` | `true` | commit 跟 push 用分開的 timer |
| `autoBackupAfterFileChange` | `true` | 編輯停止後才 auto commit，避免打字打到一半被切走 |
| `showErrorNotices` | `true` | conflict / push 失敗要看得到通知 |
| `autoCommitOnlyStaged` | `false` | 共筆 vault 不要漏 commit |
| `disablePush` | `false` | 共筆 vault 不能關 push |

改完存檔，重開 Obsidian。

### B. 透過 Obsidian Git 設定 UI

打開 Obsidian → Settings → Community plugins → Obsidian Git → Options，找對應區塊：

**Automatic 區塊**：
- Split timers for automatic commit and sync → **ON**
- Auto commit interval → **5**
- Auto commit after stopping file edits → **ON**
- Auto push interval → **5**
- Auto pull interval → **5**
- Auto commit only staged files → **OFF**
- Specify custom commit message on auto commit → **OFF**
- Commit message on auto commit → `{{hostname}} backup: {{date}}`

**Commit message 區塊**：
- Commit message → `{{hostname}} backup: {{date}}`

**Pull/Push 區塊**：
- Pull before push → **ON**
- Sync method / Merge strategy → **merge**
- Auto pull on boot → **ON**
- Disable push → **OFF**

**Misc 區塊**：
- Show error notices → **ON**

## 驗證設定生效

重開 Obsidian 後：

1. **觀察狀態列**：底部會顯示分支與同步狀態。如果狀態列消失，請開啟設定裡的 Show status bar。
2. **看 commit log**：`Ctrl+P` → `Obsidian Git: Show history`，最近的 auto commit 訊息應該長得像 `<你電腦的 hostname> backup: 2026-MM-DD HH:mm:ss`。
3. **第一次 boot pull**：剛開啟 Obsidian 時應該看到「Pulled from remote」之類的 notice，這代表 `autoPullOnBoot` 有生效。

## 衝突處理

多人協作偶爾還是會撞到 merge conflict，發生時：

1. **不要慌、不要關 Obsidian**，先看狀態列或 notice 上的錯誤訊息。
2. **打開 Source Control view**：`Ctrl+P` → `Obsidian Git: Open source control view`，會看到衝突檔案標記。
3. **打開衝突檔案**：Obsidian 會把 `<<<<<<<`、`=======`、`>>>>>>>` 標記直接寫進檔案內容裡。手動編輯保留正確版本、刪掉標記。
4. **存檔 → commit → push**：`Ctrl+P` → `Obsidian Git: Commit all changes` → 訊息寫「resolve conflict: <檔名>」→ 再 push。
5. **如果處理不來**，先在群組講一聲、停止編輯，請 admin (`@metaarchetech`) 從 GitHub web 端處理。

## 衝突預防（最重要）

設定救不了人為衝突，請遵守 [[使用規範]] 裡的協作規則：

- `01_個人/<你>/` 只有你自己編，避免任何衝突
- `02_專案/<project>/` 由專案負責人主寫
- 改 `00_共用/`、`03_會議/`、`04_知識庫/` **先在群組講一聲**
- 同一個檔案不要兩個人同時開著編
- 開始編輯前，手動 `Ctrl+P` → `Obsidian Git: Pull` 一次最安全（即使有 auto pull）

## 排錯

**問題：本地看不到其他人的更新**
→ 檢查 `autoPullInterval` 是不是 0、`autoPullOnBoot` 是不是 false。或在 Terminal `cd` 到 vault 跑 `git pull` 看實際狀態。

**問題：本地改了但其他人看不到**
→ 檢查 `autoPushInterval` 是不是 0、`disablePush` 是不是 true。在 Terminal `cd` 到 vault 跑 `git status` 確認有沒有 unpushed commits。

**問題：每幾分鐘跳一次「Specify commit message」框**
→ `customMessageOnAutoBackup` 或 UI 上的「Specify custom commit message on auto commit」設成 ON 了，請改 OFF。

**問題：編輯到一半被 commit 切走**
→ `autoBackupAfterFileChange` 沒開（對應 UI 的 Auto commit after stopping file edits），請改 ON。

**問題：push 一直被 reject**
→ 多半是 `pullBeforePush` 沒開，遠端有別人新 push 的 commit。改 ON，或手動跑一次 `git pull --rebase origin main` 再 push。
