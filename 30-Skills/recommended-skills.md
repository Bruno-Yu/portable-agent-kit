---
title: Recommended Agent Skills
tags:
  - agent/skills
last_verified: 2026-08-19
status: public-review
---

# Recommended Agent Skills

先從少量、可解釋的 Skill 開始，不要一次安裝整個排行榜。

## Repo 內建行為 Skills

這三個版本只包含 Markdown 文件，不含 scripts、hooks、MCP、套件安裝或外部資料呼叫：

| 用途 | Skill | 何時使用 |
|---|---|---|
| Coding Agent 基線 | [[karpathy-guidelines/SKILL\|karpathy-guidelines]] | 寫 code、修 bug、重構與 review；原始 4 條加社群擴充 8 條 |
| 防過度工程 | [[ponytail-minimal-coding/SKILL\|ponytail-minimal-coding]] | 容易新增套件、抽象或 boilerplate 的 coding task |
| 精簡溝通 | [[adhd-friendly-communication/SKILL\|adhd-friendly-communication]] | 使用者明確要求 ADHD-friendly、focus mode、短步驟或一次一件事 |

它們可以逐次明確載入，也可以把需要長期套用的工程原則摘要進專案 `AGENTS.md`。不要把個人健康資訊 commit 到公開設定。

## 外部候選

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

## 行為 Skills 來源審查

2026-08-19 檢查：

- `doggy8088/andrej-karpathy-skills` 的原始版本是 4 條，不是 Andrej 親自發布的 8 條；本 repo 將後來常見的社群擴充 8 條清楚標開，合計 12 條。
- `DietrichGebert/ponytail` 採 MIT License，但完整 repo 包含 Node.js lifecycle hooks、scripts、plugin metadata 與 MCP。公司環境只收錄自行審閱過的純 Markdown 適配，不安裝上游執行面。
- `pat-eason/adhd-mode` Gist 確實是精簡說話方式，但未標示明確授權，因此不複製原文；本 repo 的版本獨立撰寫。
- `UditAkhourii/adhd` 採 MIT License，但它是多框架創意發散工具，不是本次要找的 speaking-style Skill。Standalone `SKILL.md` 可提示詞運作；CLI／library 則使用 Node.js、Agent SDK 與多次模型呼叫。本 repo 不因名稱相同就混合兩種用途。

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
