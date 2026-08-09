---
title: 10 分鐘 Agent 設定
tags:
  - agent/quickstart
  - setup
status: public-review
---

# 10 分鐘 Agent 設定

## 1. 放入共用規則

將 repo 根目錄的 `AGENTS.md` 複製到新專案，再加入該專案的建置、測試與完成條件。

## 2. 選擇行為 Profile

- 一般開發：[[karpathy-principles]]
- 容易過度工程的任務：再加入 [[ponytail-minimal-coding]]
- 高風險或複雜任務：要求先列成功標準、風險與驗證方法

## 3. 建立最小記憶

從 `20-Agent-Memory/templates/` 複製：

- `MEMORY.example.md` → 記憶路由
- `PROJECT_MEMORY.example.md` → 專案決策
- `PENDING.example.md` → 本機工作狀態
- `user-profile.example.md` → 本機個人偏好

實際個人資料應存成 `*.local.md` 或放入 `memory/private/`，不要 commit。

## 4. 只安裝真正需要的工具

依序判斷：內建工具、CLI、Skill、MCP。瀏覽器自動化通常只需選一套；記憶 MCP 不列為預設。

## 5. 驗證

給 Agent 一個小任務，檢查它是否：

- 實作前說明假設
- 修改範圍精準
- 沒有額外造抽象或安裝不必要套件
- 會執行測試並明確回報結果
