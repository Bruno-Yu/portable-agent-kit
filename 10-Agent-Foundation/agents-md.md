---
title: AGENTS.md 開放格式
tags:
  - agent/config
source: https://agents.md/
last_verified: 2026-08-18
status: public-review
---

# AGENTS.md 開放格式

`AGENTS.md` 可視為「給 Coding Agent 讀的 README」，集中記錄開發環境、測試方式、專案慣例與完成條件。

## 建議章節

- Project purpose 與必要文件路由
- Toolchain 與確切 setup／dev／build 指令
- Lint、format、typecheck、unit、integration／E2E 指令
- 無法從 formatter 或程式碼推知的 coding style
- Architecture boundaries 與禁止依賴方向
- UI 任務的 `DESIGN.md` 路由
- Privacy、security 與高影響操作邊界
- Definition of Done 與驗證回報格式

指令只能來自 README、manifest、lockfile、Makefile／Taskfile、CI 與現有設定。找不到時標記待確認，不要猜一個看似合理的 command。

## Codex 初始化

Codex CLI 可在專案目錄執行 `/init` 產生 `AGENTS.md` scaffold。它相當於初始化起點，不是完成品；產生後仍需補上真實 build/test 指令、非預設 coding style、架構邊界與完成條件。

完整步驟與可複製提示詞：[[codex-project-bootstrap|Codex 專案初始化與接入指南]]。

可攜模板：

- `00-Quickstart/templates/AGENTS.project.example.md`
- `00-Quickstart/templates/DESIGN.example.md`

## 維護策略

共用規則只維護一份。`CLAUDE.md`、Copilot instructions、Cursor rules 等平台檔案只補平台差異，避免規則分叉。

保持根目錄 `AGENTS.md` 精簡，讓它像目錄一樣指向架構、coding conventions、設計與測試文件。大型 repo 可在子目錄加入 `AGENTS.md` 或 `AGENTS.override.md`；Codex 會由 project root 往目前目錄合併，較近的規則優先。

將可機械執行的 style 交給 formatter、linter、type checker、測試與 CI。只有 Agent 無法自行推知、或重複犯錯的規則才放進 `AGENTS.md`。

## 來源

- [agentsmd/agents.md](https://github.com/agentsmd/agents.md)
- [OpenAI Docs：Custom instructions with AGENTS.md](https://learn.chatgpt.com/docs/agent-configuration/agents-md)
- [OpenAI Docs：Codex best practices](https://learn.chatgpt.com/guides/best-practices)
