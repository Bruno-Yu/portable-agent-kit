---
title: Agent Platform Map
tags:
  - agent/config
status: public-review
---

# Agent Platform Map

| 內容 | 可攜核心 | 平台適配 |
|---|---|---|
| 專案規則 | `AGENTS.md` | `CLAUDE.md`、Copilot instructions、Cursor rules |
| Skills | `SKILL.md` 概念與來源 | 安裝路徑、plugin 格式、marketplace 不同 |
| MCP | server 定義與安全原則 | client 設定檔位置不同 |
| Memory | Markdown 路由與分層 | 自動載入規則不同 |

## 原則

先維護平台無關的內容，再做薄適配層。不要把同一份完整規則複製到四個平台檔案中。
