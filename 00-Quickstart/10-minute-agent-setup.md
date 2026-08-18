---
title: 10 分鐘 Agent 設定
tags:
  - agent/quickstart
  - setup
status: public-review
---

# 10 分鐘 Agent 設定

## 1. 放入共用規則

Codex CLI 可先在專案根目錄執行 `/init`，再依 [[codex-project-bootstrap|Codex 專案初始化與接入指南]] 補齊。沒有 CLI 時，將 `00-Quickstart/templates/AGENTS.project.example.md` 複製成新專案的 `AGENTS.md`。

不要直接保留 scaffold 的猜測值。從 README、manifest、lockfile、Makefile／Taskfile、CI、linter 與測試設定填入真實內容。

## 2. 填入專案契約

至少確認：

- setup、dev、build、lint、format、typecheck 與 test 指令
- 非預設 coding style 與 architecture boundaries
- dependency、migration、deploy 與資料操作的確認邊界
- Definition of Done 與 skipped checks 的回報方式

## 3. 視需要加入設計規範

有 UI、品牌、dashboard、簡報或視覺一致性需求時，將 `00-Quickstart/templates/DESIGN.example.md` 複製成 `DESIGN.md`，再用現有產品或核准設計來源替換範例 token。

在 `AGENTS.md` 要求 UI 任務先讀 `DESIGN.md`；token 是精確值，Markdown 說明使用理由。格式目前是 `alpha`，不要把範例當成專案品牌。

## 4. 選擇行為 Profile

- 一般開發：[[karpathy-principles]]，或載入 `30-Skills/karpathy-guidelines/SKILL.md`
- 容易過度工程的任務：再加入 [[ponytail-minimal-coding]]，或載入 `30-Skills/ponytail-minimal-coding/SKILL.md`
- 需要先講結論、短步驟與一次一件事：載入 [[adhd-friendly-communication]] 對應的 `30-Skills/adhd-friendly-communication/SKILL.md`
- 高風險或複雜任務：要求先列成功標準、風險與驗證方法

Karpathy 原始整理是 4 條，本 repo 採用的是原始 4 條加上社群擴充 8 條的完整 12 條版本。ADHD-friendly 是使用者明確選擇的溝通格式，不應用來推測或記錄健康狀況。

## 5. 建立最小記憶

從 `20-Agent-Memory/templates/` 複製：

- `MEMORY.example.md` → 記憶路由
- `PROJECT_MEMORY.example.md` → 專案決策
- `PENDING.example.md` → 本機工作狀態
- `user-profile.example.md` → 本機個人偏好

實際個人資料應存成 `*.local.md` 或放入 `memory/private/`，不要 commit。

## 6. 只安裝真正需要的工具

依序判斷：內建工具、CLI、Skill、MCP。瀏覽器自動化通常只需選一套；記憶 MCP 不列為預設。

## 7. 驗證

給 Agent 一個小任務，檢查它是否：

- 實作前說明假設
- 能說出 repo 的 build、test、coding style 與設計規則來源
- 修改範圍精準
- 沒有額外造抽象或安裝不必要套件
- 會執行測試並明確回報結果
