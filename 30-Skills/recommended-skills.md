---
title: Recommended Agent Skills
tags:
  - agent/skills
last_verified: 2026-08-18
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

## 離線 Office Skills

公司環境禁止外連 MCP 時，優先使用 repo 內的純 Markdown skill：

| 用途 | Skill | 執行面 |
|---|---|---|
| PowerPoint | [[offline-pptx/SKILL|offline-pptx]] | 只有 `SKILL.md`；禁止 MCP、下載與巨集 |
| Excel | [[offline-xlsx/SKILL|offline-xlsx]] | 只有 `SKILL.md`；禁止 MCP、資料更新與巨集 |

兩者都只允許使用已安裝的本機工具；缺少相依套件時應停止並回報，不可自行連網安裝。

### 網路候選審查

2026-08-18 檢查 [anthropics/skills](https://github.com/anthropics/skills) 的 `pptx`、`xlsx`（commit `f379e5ad66e2febc1616cf8d6284666fecbe514e`）：

- 功能完整，但兩個 skill 都標示 proprietary，授權禁止向第三方散布，不可直接 vendoring 到本公開 repo。
- 除 `SKILL.md` 外還包含 Python、OOXML schema、LibreOffice subprocess/socket shim；不是純 Markdown skill。
- 文件在缺少套件時建議使用 `pip` 或 `npm` 安裝，不符合公司離線與固定供應鏈要求。
- 結論：只作研究來源，不安裝、不複製；本 repo 改採原創的最小離線 skill。

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
