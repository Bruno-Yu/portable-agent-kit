---
title: Portable Agent Kit
tags:
  - agent
  - codex
  - hcl-domino
status: public-review
---

# Portable Agent Kit

一個可公開、可攜、以 Obsidian 管理的 Agent 快速設定工具箱，同時收錄 HCL Notes/Domino 開發筆記。

> [!warning] 發布狀態
> 目前是本機 public-review 版本。通過秘密掃描、連結檢查與人工 staged diff 前，不要推到公開 GitHub。

## 從這裡開始

- [[10-minute-agent-setup|10 分鐘 Agent 設定]]
- [[karpathy-principles|Karpathy-inspired 工程原則]]
- [[ponytail-minimal-coding|Ponytail Minimal Coding]]
- [[memory-architecture|Agent 記憶架構]]
- [[recommended-skills|推薦 Skills]]
- [[recommended-mcp|推薦 MCP]]
- [[Codex-Index|Codex 索引]]
- [[VS-Code-Index|VS Code 索引]]
- [[HCL-Domino-Index|HCL Notes/Domino 索引]]

## 設計原則

1. 共用核心放在 `AGENTS.md`，平台專屬設定保持精簡。
2. 預設少裝工具：內建能力 → CLI → Skill → MCP。
3. 公開 repo 只存規則、模板與去識別化技術知識；個人記憶與工作狀態留在本機。
4. 第三方 Skill、Plugin、MCP 都視為供應鏈程式碼，安裝前先審查。

## 內容地圖

| 目錄 | 用途 |
|---|---|
| `00-Quickstart` | 最短設定路徑與發布檢查 |
| `10-Agent-Foundation` | Agent 行為原則與跨平台格式 |
| `20-Agent-Memory` | 可攜式記憶架構與公開模板 |
| `30-Skills` | 精選 Skill 與安全檢查 |
| `40-MCP` | MCP 選型、設定與安全邊界 |
| `50-Platforms` | Codex、Claude Code、Copilot、Cursor 對照 |
| `55-Codex` | Codex CLI、Hooks、指令與使用案例 |
| `58-VS-Code` | VS Code、GitHub Copilot 與開發環境設定 |
| `60-HCL-Domino` | LotusScript、Formula、安全、版控、REST API |

## 授權與來源

目前沒有對整個 repo 套用單一授權。第三方內容、官方文件摘要與外部專案各自受原始授權約束；公開前請參閱 [[PUBLICATION-CHECKLIST]]。
