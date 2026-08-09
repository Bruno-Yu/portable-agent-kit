---
title: Recommended Agent Skills
tags:
  - agent/skills
last_verified: 2026-08-09
status: public-review
---

# Recommended Agent Skills

先從少量、可解釋的 Skill 開始，不要一次安裝整個排行榜。

| 用途 | 資源 | 備註 |
|---|---|---|
| 工程原則 | [andrej-karpathy-skills](https://github.com/doggy8088/andrej-karpathy-skills) | 可使用 AGENTS.md 或 Skill 版本 |
| 防過度工程 | [Ponytail](https://github.com/DietrichGebert/ponytail) | 支援多種 Agent 形式 |
| 官方 Skill 範例 | [anthropics/skills](https://github.com/anthropics/skills) | 安裝前仍需審查 |
| 瀏覽器 CLI | [agent-browser](https://github.com/vercel-labs/agent-browser) | 日常操作可先於 MCP 評估 |
| Web 測試 | [Playwright MCP](https://github.com/microsoft/playwright-mcp) | 適合複雜瀏覽器流程 |
| Obsidian | [kepano/obsidian-skills](https://github.com/kepano/obsidian-skills) | Markdown、Bases、Canvas、CLI |

## Skills.sh

依官方 CLI 文件，通用安裝形式為：

```bash
npx skills add <owner>/<repo>
```

參考：[Skills CLI documentation](https://www.skills.sh/docs/cli)

## 安裝前檢查

- 閱讀完整 `SKILL.md`
- 檢查 scripts、hooks、MCP command 與安裝後動作
- 確認是否會讀取環境變數、瀏覽器狀態或整個檔案系統
- 優先 project scope；全域安裝需要更高信任
- 記錄來源 URL、commit/tag 與最後驗證日期

> [!warning]
> Marketplace 收錄與高安裝數不等於安全保證。
