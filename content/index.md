---
title: VISION BASE Lab
---

# VISION BASE Lab — 共用知識庫

**VISION BASE Lab** 共用知識庫（Obsidian Vault），透過 [Obsidian Git](https://github.com/Vinzent03/obsidian-git) 外掛自動與 [`visionbase-usc/vision_base_vault`](https://github.com/visionbase-usc/vision_base_vault) 同步。研究主軸：**Digital Twin + Physical AI**。

---

## 📚 重要文件

| 文件 | 對象 | 內容 |
|---|---|---|
| [00_共用/使用規範.md](00_共用/使用規範.md) | **人類成員** | 完整使用規範、共筆守則、命名規則 |
| [CLAUDE.md](CLAUDE.md) | **Claude Code** | 精簡版規範，AI 寫筆記時自動讀 |
| [00_共用/](00_共用/) | 全員 | 共用文件、索引 |
| [05_GitHub_Repos/](05_GitHub_Repos/) | 全員 | `visionbase-usc` 各 repo 的文件鏡像 |
| [06_CLAUDE_Skills/](06_CLAUDE_Skills/) | 全員 | Claude Code skills 文件鏡像 |

> 第一次來這個 vault？先讀 **[使用規範.md](00_共用/使用規範.md)**。

## 🗂 資料夾結構

```
vision_base_vault/
├── 00_共用/              全員共用文件、規範、索引
├── 01_個人/<你的名字>/   個人筆記、草稿
├── 02_專案/<專案名>/     各專案資料
├── 03_會議/              跨專案會議
├── 04_知識庫/            文獻 + 技術筆記（按領域分）
├── 05_GitHub_Repos/     visionbase-usc 各 repo 的文件鏡像
├── 06_CLAUDE_Skills/    Claude Code skills 文件鏡像
├── _templates/           新筆記模板
└── _attachments/         圖片、附件（<5MB）
```

每個資料夾內有 README 說明該區用途。

## 🔗 相關 Repo

| Repo | 內容 |
|---|---|
| [`vision_base_vault`](https://github.com/visionbase-usc/vision_base_vault) | 本 vault |
| [`claude-skills`](https://github.com/visionbase-usc/claude-skills) | VISION BASE Lab 的 Claude Code skills |

---

## 在新電腦上首次設定

### 1. 安裝必要工具

| 工具 | 用途 | 下載 |
|---|---|---|
| [Obsidian](https://obsidian.md) | 編輯 vault | 官網安裝 |
| [Git for Windows](https://git-scm.com/) | 版本控制 | 官網安裝 |
| [GitHub CLI (`gh`)](https://cli.github.com/) | GitHub 登入 | 官網安裝或 `winget install GitHub.cli` |

### 2. 登入 GitHub（一次性）

打開 PowerShell：

```powershell
gh auth login
```

選項：
- **What account?** → `GitHub.com`
- **Protocol?** → `HTTPS`
- **Authenticate Git with GitHub credentials?** → `Yes`
- **How would you like to authenticate?** → `Login with a web browser`
- 跳出一次性代碼，貼到瀏覽器授權

授權後跑一次：

```powershell
gh auth setup-git
```

讓 git 把 gh 當成憑證來源（之後 push / pull 不會再問密碼）。

### 3. Clone vault

選一個你想放的位置，例如 `C:\vision_base\`：

```powershell
git clone https://github.com/visionbase-usc/vision_base_vault.git
```

### 4. 在 Obsidian 開啟

1. 打開 Obsidian
2. **Open folder as vault** → 選剛剛 clone 下來的 `vision_base_vault` 資料夾
3. 設定（齒輪）→ **社群外掛** → 看到「限制模式」提示就 **關閉限制模式**
4. **Obsidian Git** 應該已經列在已安裝的外掛裡 → 確認開關是**開**的

### 5. 設定 Obsidian Git 外掛

設定 → Obsidian Git，建議值：

| 設定項 | 建議值 |
|---|---|
| **Vault backup interval (minutes)** | `5` |
| **Auto pull interval (minutes)** | `5` |
| **Pull updates on startup** | ✅ 開 |
| **Disable push** | ❌ 關 |
| **Commit message** | 預設（含時間戳）|

設定完就會在背景自動同步。狀態列右下角會顯示同步狀態。

---

## 日常使用

### ✅ 你只要做的事

**寫筆記**。其他都是自動的。

- 每 5 分鐘外掛會自動 commit + push
- 每 5 分鐘外掛會自動從 GitHub pull
- 開 Obsidian 時會先 pull 一次最新內容

### ⚠️ 注意事項

1. **避免兩台電腦同時編輯同一檔案** — 會產生 merge conflict
2. **長時間離線後**，先打開 Obsidian 讓它先 pull 再開始編輯
3. **大檔案（>50MB）請勿放入** — GitHub 有檔案大小限制；要放大檔請用其他雲端再貼連結

---

## 哪些東西**不會**被同步

`.gitignore` 排除了以下檔案，原因是它們是「每台電腦不一樣」的本機狀態：

```
.obsidian/workspace.json          # 你打開的分頁、視窗排版
.obsidian/workspace-mobile.json   # 手機版的分頁狀態
.obsidian/workspaces.json
.obsidian/cache                   # Obsidian 內部快取
.obsidian/plugins/obsidian-git/data.json  # Git 外掛的個人設定
.trash/                           # Obsidian 的垃圾桶
*.DS_Store / Thumbs.db / desktop.ini  # 作業系統垃圾檔
```

> 所以在新電腦上要重新設定一次 Obsidian Git 的 commit 間隔（如上述第 5 步）。

---

## 手動操作（備用）

如果自動同步出問題，可以手動處理：

### 從 PowerShell

```powershell
cd C:\vision_base\vision_base_vault

# 看狀態
git status

# 手動 pull
git pull

# 手動 push
git add .
git commit -m "更新筆記"
git push
```

### 從 Obsidian Git 外掛

按 `Ctrl+P` 打開命令面板，搜尋 `Obsidian Git:`，常用：

- `Obsidian Git: Commit all changes`
- `Obsidian Git: Push`
- `Obsidian Git: Pull`
- `Obsidian Git: Open source control view`（圖形化介面）

---

## 常見問題

### Q1：開啟 Obsidian 後跳出 merge conflict 怎麼辦？

代表你在某台電腦改了檔，還沒 push 之前，另一台電腦也改了同一個檔。

**處理方式**：
1. 命令面板 → `Obsidian Git: Open source control view`
2. 找到衝突的檔案，在編輯器裡會看到 `<<<<<<< HEAD` 之類的標記
3. 手動選你要保留的版本，刪掉 `<<<<<<<`、`=======`、`>>>>>>>` 三行標記
4. 命令面板 → `Obsidian Git: Commit all changes`

### Q2：外掛說 push 失敗 / 沒有權限？

```powershell
gh auth status
```

如果沒登入，重跑 `gh auth login`。如果有登入但 git 還是不行，再跑一次：

```powershell
gh auth setup-git
```

### Q3：我想還原到某個時間點的版本？

GitHub 上點 [Commits 列表](https://github.com/visionbase-usc/vision_base_vault/commits/main) 可以看每次同步的快照。要還原找我或自己用 `git checkout <commit-hash>`。

### Q4：手機上能用嗎？

可以，但要設定較麻煩：
- iOS：Obsidian Git 外掛 **不支援 iOS**（限制原因）。改用 [Working Copy](https://workingcopy.app/) app 配合 Obsidian。
- Android：Obsidian Git 外掛**部分支援**，需要在外掛設定裡填 GitHub username + Personal Access Token。

---

## 技術細節

- **Repo**：[`visionbase-usc/vision_base_vault`](https://github.com/visionbase-usc/vision_base_vault)（私人）
- **預設分支**：`main`
- **同步外掛**：[Obsidian Git](https://github.com/Vinzent03/obsidian-git) `v2.38.3`
- **外掛程式檔**放在 `.obsidian/plugins/obsidian-git/`，跟著 repo 走，新電腦 clone 就有

---

## 修改本說明

直接編輯本 vault 根目錄的 `README.md`，存檔後外掛會自動 commit & push。

GitHub 首頁也會同步更新（GitHub 會自動把 repo 根目錄的 `README.md` 顯示在 repo 首頁）。
