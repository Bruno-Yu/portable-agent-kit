---
title: Codex 專案初始化與接入指南
tags:
  - agent/quickstart
  - codex
  - setup
last_verified: 2026-08-18
status: public-review
---

# Codex 專案初始化與接入指南

目標是讓 Codex 第一次進入 repo 就知道如何建置、測試、遵循 coding style，以及 UI 任務該讀哪一份設計規範；不要把整個知識庫塞進 context。

## 建議結果

```text
your-project/
├── AGENTS.md                  # 每次都要遵守的專案規則與文件路由
├── DESIGN.md                  # 有 UI／品牌需求時才需要
├── README.md                  # 給人類的安裝與操作方式
├── docs/                      # 架構、API、決策等詳細文件
└── .codex/config.toml         # 選用；只放 repo-specific Codex 設定
```

`AGENTS.md` 應保持精簡，扮演目錄與執行契約；詳細架構與 API 文件放在 `docs/`，讓 Agent 按任務載入。

## 方法一：使用 Codex `/init`

在 Codex CLI 進入專案根目錄後執行：

```text
/init
```

Codex 會在目前目錄產生 `AGENTS.md` scaffold。完成後不要直接接受預設內容，繼續要求它依 repo 證據補齊：

```text
請初始化這個 repo 的 Agent 開發環境，只修改文件，不修改產品程式碼。

1. 讀取 README、套件 manifest、lockfile、Makefile／Taskfile、CI workflow、linter、formatter 與測試設定。
2. 用實際存在的指令補齊 AGENTS.md：setup、dev、build、lint、format check、typecheck、unit test、integration／E2E。
3. 記錄無法從程式碼直接推知的 coding style、架構邊界、禁止事項與 Definition of Done。
4. 不要猜指令、不要安裝套件、不要執行 deploy；不確定的欄位標成「待確認」並附證據位置。
5. 若 repo 有 UI，先從現有 token、CSS、theme、元件與截圖整理 DESIGN.md；不要自行發明品牌規範。
6. 最後列出修改檔案、採用的證據、可執行的驗證與仍待人工確認項目。
```

## 方法二：複製可攜模板

不使用 CLI 時，複製：

- `00-Quickstart/templates/AGENTS.project.example.md` → 專案根目錄 `AGENTS.md`
- `00-Quickstart/templates/DESIGN.example.md` → 專案根目錄 `DESIGN.md`（只有 UI 專案需要）

先替換範例值，再刪除不適用章節。未知內容寧可標示待確認，不要讓 Agent 猜測。

## 如何填建置方法

只記錄可由 repo 證明的指令：

| 欄位 | 優先證據 |
|---|---|
| Runtime／版本 | `.tool-versions`、`.nvmrc`、`global.json`、`pyproject.toml`、CI image |
| 安裝 | lockfile、README、CI install step |
| 開發 | `package.json` scripts、Makefile、Taskfile、專案文件 |
| Build | CI build step、既有 build script |
| Lint／format | linter、formatter 設定與 CI step |
| Test | test runner 設定、CI、既有 scripts |
| E2E | Playwright／Cypress 等設定與 pipeline |

將「快速局部驗證」與「完整驗證」分開，避免每個小改動都跑最昂貴的整套流程，也避免只跑單一測試就宣稱全部通過。

## 如何寫 coding style

優先讓 formatter、linter、type checker 與測試成為可執行規則。`AGENTS.md` 只補充工具無法表達或 Agent 容易誤判的內容，例如：

- 專案刻意偏離語言預設的規則。
- 命名、錯誤格式、logging 與非同步處理慣例。
- 業務邏輯、資料存取與 UI 元件的目錄邊界。
- generated code、migration 與 snapshot 的修改方式。
- 新依賴、breaking change、資料庫變更所需的人工確認。

不要重寫整份 ESLint、Prettier、Ruff 或 EditorConfig 規則；指出設定檔與必要例外即可。

## 何時加入 `DESIGN.md`

UI、簡報、品牌頁、dashboard 或需要長期保持視覺一致時加入。依 Google Labs 的 draft specification，`DESIGN.md` 包含兩層：

1. YAML frontmatter：精確且具規範性的 color、typography、spacing、rounded、component tokens。
2. Markdown：設計理由、使用情境、元件狀態與 Do／Don't。

現有程式碼、設計 token 與核准素材優先於範例。若 token 與實作衝突，先回報 drift，不要默默平均兩者。格式目前為 `alpha`，升版前應重新核對官方規格。

在 `AGENTS.md` 加入路由規則：

```md
- UI、CSS、元件、簡報或品牌相關任務開始前，先讀取 `DESIGN.md`。
- YAML design tokens 是精確值；Markdown 說明使用理由。不得自行新增任意色碼、字級或間距。
- 若 `DESIGN.md` 與現有產品衝突，停止並列出差異，取得決定後再修改。
```

## 選用：公司離線 Codex 設定

只有在公司政策允許、repo 已被信任且確實需要 project override 時，才建立 `.codex/config.toml`：

```toml
approval_policy = "on-request"
sandbox_mode = "workspace-write"
web_search = "disabled"

[features]
apps = false
remote_plugin = false
```

不要把 token、MCP credentials、內網位址或個人偏好 commit 進專案設定。組織的 managed configuration 與 `requirements.toml` 優先於 repo 設定。

## 驗證接入

1. 從 repo root 啟動新的 Codex session。
2. 要求 Codex：「列出目前載入的 instruction sources，並摘要 build、test、coding style 與 UI 設計規則。」
3. 從子目錄再測一次，確認較近的 `AGENTS.override.md`／`AGENTS.md` 是否正確覆蓋根目錄規則。
4. 給一個小改動，確認 Agent 會先讀相關文件、只做局部修改並執行正確驗證。
5. 每當相同錯誤重複出現，再把一條可執行、可驗證的規則加入最接近的 `AGENTS.md`。

## 來源

- [OpenAI Docs：Custom instructions with AGENTS.md](https://learn.chatgpt.com/docs/agent-configuration/agents-md)
- [OpenAI Docs：Codex best practices](https://learn.chatgpt.com/guides/best-practices)
- [OpenAI Docs：Config basics](https://learn.chatgpt.com/docs/config-file/config-basic)
- [Google Labs：DESIGN.md specification](https://github.com/google-labs-code/design.md/blob/main/docs/spec.md)
