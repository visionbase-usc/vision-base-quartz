---
title: Isaac Lab RL 實機建置紀錄（Windows / RTX 3070）
author: visionbase-usc
created: 2026-06-17
updated: 2026-06-17
tags: [type/howto, status/done, area/physical-ai, area/isaac]
---

# Isaac Lab RL 實機建置紀錄（Windows / RTX 3070）

> 實際在實驗室電腦跑通的完整流程 + 踩坑修法。計畫版見 [[isaac-sim-installation]]。

## ⭐ 結論先講（最關鍵的一條）

**RTX 3070 上要即時 RTX 渲染（互動視窗），只有 NVIDIA Studio 537.58 驅動能穩定。**

- `591.86`：離屏渲染 OK，但即時視窗 → **D1 藍屏強制重啟**
- `610.62`：不藍屏，但**完全不能渲染**
- **`537.58 Studio`（官方對 Isaac Sim 4.5 驗證版）：渲染穩 + 不藍屏 ← 唯一可用（20 分鐘壓測過）**
- 判讀：事件檢視器 BugCheck `0x000000D1`（DRIVER_IRQL_NOT_LESS_OR_EQUAL）= 驅動層 bug 的鐵證。
- 下載：NVIDIA 官方 → GeForce RTX 3070 / Windows 11 / **Studio Driver** 537.58，**清潔安裝**。

## 最終可用環境

| 元件 | 版本 / 位置 |
|---|---|
| OS / GPU | Windows 11 / RTX 3070 8GB |
| NVIDIA 驅動 | **Studio 537.58**（31.0.15.3758）|
| conda env | `env_isaaclab`（Python 3.10，**用 conda-forge 建**，避開 Anaconda 商用 ToS）|
| Isaac Sim | pip `isaacsim==4.5.0.0` |
| PyTorch | `2.5.1+cu118` |
| Isaac Lab | `v2.1.1` @ `C:\omniverse\IsaacLab` |
| h5py | `3.11.0`（HDF5 1.14.2）|

## 踩坑與修法（精華）

| # | 症狀 | 修法 |
|---|---|---|
| 1 | 路徑含空格 → premake/build 出錯 | repo 放無空格路徑 `C:\omniverse\IsaacLab` |
| 2 | 繁中 cp950 讀 UTF-8 設定崩潰 | 所有指令前設 `PYTHONUTF8=1` |
| 3 | extscache 路徑 >260 字元，pip 解壓失敗 | 開 Windows 長路徑：`LongPathsEnabled=1`（registry，需管理員）|
| 4 | conda 預設 channel 要商用 ToS | 用 `-c conda-forge --override-channels` 建環境 |
| 5 | setuptools 81 移除 `pkg_resources`，flatdict 編譯失敗 | 降版 `setuptools==75.8.0` |
| 6 | 核心 `isaaclab` 沒裝進去 | `pip install -e source/isaaclab --no-build-isolation`（環境先裝 `toml`）|
| 7 | `qdldl.pyd` 載入 access violation（Kit 自帶舊 14.29 C++ runtime）| `sitecustomize.py` 啟動時搶載 System32 的 14.44 MSVC runtime |
| 8 | `h5py._errors` DLL 載入失敗 / Kit 卡死（h5py 的 HDF5 2.0 撞 omni.sensors 的 1.14）| **h5py 降到 3.11.0**（帶 HDF5 1.14.2，ABI 相容）|
| 9 | **即時渲染 D1 藍屏重啟** | **換 NVIDIA Studio 537.58**（見上）|
| 10 | Go2 USD `st` primvar 損壞警告 | 在 537.58 上僅警告、無害（錯誤驅動上才會觸發崩潰）|

## 訓練（headless，穩定）

每次開終端機先：

```powershell
conda activate env_isaaclab; cd C:\omniverse\IsaacLab; $env:PYTHONUTF8=1
```

訓練（範例 Go2 平地 1500 回合）：

```powershell
.\isaaclab.bat -p scripts\reinforcement_learning\rsl_rl\train.py `
  --task=Isaac-Velocity-Flat-Unitree-Go2-v0 --headless --num_envs 1024 --max_iterations 1500
```

**實測結果（RTX 3070）：**

| 任務 | 回合 | 耗時 | 最終 reward | episode length |
|---|---|---|---|---|
| Anymal-C 平地 | 1500 | 31 min | +20.9 | 940/1000 |
| Unitree Go2 平地 | 1500 | 28 min | +35.0 | 1000/1000 |
| Unitree Go2 崎嶇 | 1500 | 80 min | +12.0 | 888/1000 |

> 內建四足任務：Anymal B/C/D、Unitree Go2/Go1/A1、Spot。輪型無內建任務，需自建。
> 監看訓練：`python -m tensorboard.main --logdir logs\rsl_rl` → http://localhost:6006

## 看成果（三種，依需求）

1. **原生互動視窗（推薦，可滑鼠控相機）** — 需 537.58
   ```powershell
   .\isaaclab.bat -p scripts\reinforcement_learning\rsl_rl\play.py `
     --task=Isaac-Velocity-Flat-Unitree-Go2-Play-v0 --num_envs 32
   ```
   相機：**Alt+左鍵** 轉視角 ｜ **滾輪** 縮放 ｜ **Alt+中鍵** 平移 ｜ **按住右鍵+WASD** 飛行 ｜ 選中按 **F** 聚焦
2. **cv2 即時被動視窗** — headless 離屏渲染 → OpenCV 視窗（自訂腳本 `C:\omniverse\IsaacLab\scripts\reinforcement_learning\rsl_rl\play_live_cv2.py`）
3. **headless 錄影** — `play.py ... --headless --video`，輸出 MP4 到 `logs\rsl_rl\<task>\<ts>\videos\`

## 相關

- 計畫版：[[isaac-sim-installation]]
- 主題：[[isaac]]、[[physical-ai]]
