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

這不是需要安裝或啟動的應用程式。它是一組可按需取用的規則、Skill 與技術筆記：先放入最小共用規則，再依任務載入一份相關文件；不要把整個 repo 一次塞進 Agent context。

> [!warning] 發布狀態
> 目前是本機 public-review 版本。通過秘密掃描、連結檢查與人工 staged diff 前，不要推到公開 GitHub。

## 一分鐘使用方式

### 如果你是使用者

1. 先讀 [[10-minute-agent-setup|10 分鐘 Agent 設定]]；需要完整流程時讀 [[codex-project-bootstrap|Codex 專案初始化與接入指南]]。
2. 在 Codex CLI 執行 `/init`，或將 `00-Quickstart/templates/AGENTS.project.example.md` 複製成專案根目錄的 `AGENTS.md`，再補上真實 coding style、建置、測試與完成條件。
3. 只挑本次任務需要的內容：開發原則放在 `10-Agent-Foundation`、記憶模板放在 `20-Agent-Memory`、Skill 放在 `30-Skills`、平台筆記放在 `50-Platforms` 之後的目錄。
4. 將選定的 Skill 目錄複製到 Agent 平台支援的 project-level 或 user-level skills 目錄；若平台沒有 Skill discovery，就在提示詞中要求 Agent 先完整讀取該 `SKILL.md`。
5. 先用一個小任務驗證 Agent 是否遵守規則，再用於正式工作。

最小設定通常只需要：

```text
你的專案/
├── AGENTS.md
├── DESIGN.md        # 有 UI／品牌需求時才加入
└── 原本的專案檔案
```

不要一開始就複製所有記憶模板、Skill 與 MCP 設定。只有需求出現時才加入。

### 如果你是 Agent

進入這個 repo 或使用其中內容時：

1. 先完整讀取根目錄 `AGENTS.md`，把它當作本 repo 的工作規則。
2. 從使用者任務判斷需要的最小文件；不要遞迴讀取整個 vault。
3. 修改前先讀目標文件、直接引用它的索引，以及同目錄的既有寫法。
4. 優先順序使用：內建能力 → 已核准的本機 CLI → repo 內 Skill → 經審查的外部工具。不要自行安裝套件或啟用 MCP。
5. 只修改完成任務所需的檔案，保留繁體中文、Obsidian wikilink 與既有 frontmatter 慣例。
6. 完成後執行可用的 deterministic checks，並明確列出已驗證、未驗證與變更檔案。

## 常見任務範例

### 建立新專案的 Agent 規則

在 Codex CLI 執行 `/init`，或複製 `00-Quickstart/templates/AGENTS.project.example.md`，再給 Agent：

```text
先完整讀取 AGENTS.md，再從 README、manifest、lockfile、CI、linter 與測試設定
補上這個專案真實的 setup、dev、build、lint、typecheck、test、coding style 與完成條件。
只修改文件，不安裝套件、不執行 deploy、不猜測找不到的指令。
```

有 UI 時，再複製 `00-Quickstart/templates/DESIGN.example.md` 為 `DESIGN.md`，要求 Agent 從既有 theme、CSS、token 與元件整理內容，不得自行發明品牌規範。

### 離線處理 PowerPoint

使用 [[offline-pptx/SKILL|offline-pptx]]：

```text
先完整讀取 30-Skills/offline-pptx/SKILL.md，再處理 <input.pptx>。
全程離線，不得使用 MCP、下載素材、啟用外部連結或覆寫原檔；
輸出到 <output.pptx>，並回報內容與視覺驗證結果。
```

### 離線處理 Excel

使用 [[offline-xlsx/SKILL|offline-xlsx]]：

```text
先完整讀取 30-Skills/offline-xlsx/SKILL.md，再處理 <input.xlsx>。
全程離線，不得執行巨集、更新外部資料、安裝套件或覆寫原檔；
輸出到 <output.xlsx>，並回報公式、外部連結與隱藏內容的檢查結果。
```

### 查詢 HCL Notes/Domino

先從 [[HCL-Domino-Index|HCL Notes/Domino 索引]] 選擇主題，再只讀取對應筆記。回答時區分 repo 已記錄的內容、推論與仍需查證的版本差異。

## 安全邊界

- 不要把 token、cookie、私鑰、內網 URL、客戶資料或個人記憶放進這個公開 repo。
- 不要因文件推薦就直接安裝第三方 Skill、Plugin 或 MCP；先讀完整內容、授權、scripts、hooks、網路與檔案權限。
- 公司禁止外連時，使用 `30-Skills/offline-pptx`、`30-Skills/offline-xlsx` 這類純 Markdown Skill，並只允許已核准的本機工具。
- 外部 Office 檔案視為不受信任輸入；不要啟用巨集、ActiveX、外部資料連線或解除 Protected View。
- 公開前依 [[PUBLICATION-CHECKLIST]] 執行秘密掃描、連結檢查與 staged diff 人工審查。

## 從這裡開始

- [[10-minute-agent-setup|10 分鐘 Agent 設定]]
- [[codex-project-bootstrap|Codex 專案初始化與接入指南]]
- [[agents-md|AGENTS.md 開放格式]]
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
