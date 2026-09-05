---
title: Portable Agent Kit
tags:
  - agent
  - codex
  - hcl-domino
status: public
---

# Portable Agent Kit

一個可公開、可攜、以 Obsidian 管理的 Agent 快速設定工具箱，同時收錄 HCL Notes/Domino 開發筆記。

這不是需要安裝或啟動的應用程式。它是一組可按需取用的規則、Skill 與技術筆記：先放入最小共用規則，再依任務載入一份相關文件；不要把整個 repo 一次塞進 Agent context。

> [!warning] 公開內容邊界
> 這是 public repo。每次推送前都要做秘密掃描與 staged diff 人工審查；公司 DXL、NSF、內網設定、分析報告與真實識別資料不得放入此 repo。

## 先選擇導入方式

| 需求 | 要取用的內容 | 不需要取用 |
|---|---|---|
| 一般新專案 | `AGENTS.project.example.md`，有 UI 才加 `DESIGN.example.md` | 公司 DXL 規則、全部 Skills、MCP |
| 公司電腦安全預設 | `company-safe.config.toml`；需寫入時才另用 `company-edit.config.toml` | DXL 規則、repo 的 `.codex/config.toml` |
| 公司 DXL 排查 | 公司安全 profile + `AGENTS.company-dxl.example.md` | Skills、hooks、MCP、Plugin、自動 import |
| VS Code Copilot／HCL R12 規範初始化 | `58-VS-Code/hcl-notes-r12-copilot-toolkit` 的六份範本與 SETUP.md | Codex profile、MCP、安裝腳本、自動匯入 |
| 完整知識庫 | 保留本 repo 作唯讀參考，按任務讀一份相關文件 | 不要把整個 repo 塞入 context |

本 repo 不需要傳統安裝。將需要的 Markdown 或 config 範本複製到正確層級即可；不要整包覆蓋公司既有 Codex 設定。

Copilot 使用者請從 [HCL R12 Copilot Setup Toolkit](58-VS-Code/hcl-notes-r12-copilot-toolkit/README.md) 開始；範本可獨立下載使用，既有專案先盤點再合併。

## 一分鐘使用方式

### 如果你是使用者

1. 先讀 [[10-minute-agent-setup|10 分鐘 Agent 設定]]；需要完整流程時讀 [[codex-project-bootstrap|Codex 專案初始化與接入指南]]。
2. 在 Codex CLI 執行 `/init`，或將 `00-Quickstart/templates/AGENTS.project.example.md` 複製成專案根目錄的 `AGENTS.md`，再補上真實 coding style、建置、測試與完成條件。
3. 只挑本次任務需要的內容：一般 coding 載入 `karpathy-guidelines`，容易過度工程時再加 `ponytail-minimal-coding`；需要嚴格、顯式啟用的 action-first 輸出時用 `i-have-adhd`，較輕量的精簡說話方式可用 `adhd-friendly-communication`。其他文件仍依目錄按需讀取。
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

## 公司只導入「資安＋DXL」（建議）

公司環境若已經有自己的 coding style、build、test、Skills 或管理政策，只加入以下兩層，不要安裝本 repo 其他部分。

### 1. 安裝唯讀安全 profile

先確認 `$CODEX_HOME`；未設定時通常是 `~/.codex`。如果目的地已有同名檔案，先人工比較，不要直接覆寫：

```bash
if test -e ~/.codex/company-safe.config.toml; then
  echo "EXISTS: review before replacing"
else
  cp /path/to/portable-agent-kit/00-Quickstart/templates/company-safe.config.toml \
    ~/.codex/company-safe.config.toml
fi
```

Windows PowerShell 的預設目的地是 `$env:USERPROFILE\.codex\company-safe.config.toml`；同樣先用 `Test-Path` 確認，不要以 `Copy-Item -Force` 覆寫既有 profile。

需要修改時才另外安裝 `company-edit.config.toml`，而且只能在乾淨 branch、worktree 或可還原副本使用。完整邊界見 [[codex-company-safety|Codex 公司電腦防誤刪設定]]。

### 2. 只在 DXL 範圍加入 Agent 規則

如果 DXL 是獨立 repo，而且根目錄還沒有 `AGENTS.md`：

```bash
cp /path/to/portable-agent-kit/00-Quickstart/templates/AGENTS.company-dxl.example.md \
  /path/to/company-dxl-project/AGENTS.md
```

如果公司 repo 已有 `AGENTS.md`、coding style 與 build/test 規則，**不要覆寫它**。將 DXL 範本放到 DXL 子目錄成為較近的規則，並從該子目錄啟動：

```bash
cp /path/to/portable-agent-kit/00-Quickstart/templates/AGENTS.company-dxl.example.md \
  /path/to/company-project/path/to/dxl/AGENTS.override.md

codex -C /path/to/company-project/path/to/dxl --profile company-safe
```

如果該子目錄也已有 `AGENTS.md` 或 `AGENTS.override.md`，人工合併以下章節即可，不要新增第二份互相競爭的檔案：

- `Default mode`
- `Hard safety boundaries`
- `Sensitive information`
- `DXL policy`
- `Audit workflow`

較近目錄的規則會覆蓋較上層規則；因此既有 build/test/coding style 應保留，而 DXL 子樹內若發生衝突，以「唯讀、不 import、不 compile、不 sign、不外連」為準。完整步驟見 [[codex-company-dxl-start|公司 DXL 專案接入 Codex]]。

### 3. 啟動後驗證

先執行：

```text
/status
/debug-config
```

接著給 Codex：

```text
列出目前載入的 instruction sources，摘要本次 sandbox、允許讀取範圍、
DXL source of truth 與禁止事項。只做唯讀盤點，不建立或修改檔案。
```

確認 sandbox 是 `read-only`，且沒有啟用 web search、Apps、remote plugin 或未核准 MCP，才開始排查。

### 只導入這兩部分時的注意事項

- 不要把本 repo 的 `.codex/config.toml` 複製到公司專案；它是維護本知識庫使用的 `workspace-write` 設定，不是公司 DXL 唯讀 profile。
- `--profile company-safe` 不會刪除公司既有使用者設定，但受信任專案內的 `.codex/config.toml` 優先序較高，可能覆蓋 profile；所以每次都要看 `/debug-config`。
- `AGENTS.md` 是行為指示，不是 sandbox。真正的唯讀邊界來自 profile／OS sandbox；不可覆寫的限制需要 IT 部署 managed `requirements.toml`。
- `company-requirements.example.toml` 只供 IT／資安審查，不要由一般使用者放進 repo 或個人設定冒充公司政策。
- 不需複製 `30-Skills`、`40-MCP`、hooks、Plugins、記憶模板或本 repo 的根目錄 `AGENTS.md`。
- 不要把公司 DXL/NSF 複製進 portable-agent-kit；公司專案與這個 public repo 分開保存。
- 第一階段維持 Tier A：NSF 是 canonical，DXL／ODP 只讀。要 round-trip 必須另外建立 compile、sign、rollback 與行為驗證流程。
- Profile 只在明確帶入 `--profile company-safe` 時生效；公司若要禁止 `--yolo` 或統一停用 MCP，需由 IT 管理設定與網路邊界。

## 如果你是 Agent

進入這個 repo 或使用其中內容時：

1. 先完整讀取根目錄 `AGENTS.md`，把它當作本 repo 的工作規則。
2. 從使用者任務判斷需要的最小文件；不要遞迴讀取整個 vault。
3. 修改前先讀目標文件、直接引用它的索引，以及同目錄的既有寫法。
4. 優先順序使用：內建能力 → 已核准的本機 CLI → repo 內 Skill → 經審查的外部工具。不要自行安裝套件或啟用 MCP。
5. 只修改完成任務所需的檔案，保留繁體中文、Obsidian wikilink 與既有 frontmatter 慣例。
6. 完成後執行可用的 deterministic checks，並明確列出已驗證、未驗證與變更檔案。
7. 若使用者只要求公司資安與 DXL 支援，只路由到 `codex-company-safety.md`、`codex-company-dxl-start.md` 與 `AGENTS.company-dxl.example.md`；不要建議安裝其餘 Skills、MCP、hooks 或 Plugins。

## 常見任務範例

### 建立新專案的 Agent 規則

在 Codex CLI 執行 `/init`，或複製 `00-Quickstart/templates/AGENTS.project.example.md`，再給 Agent：

```text
先完整讀取 AGENTS.md，再從 README、manifest、lockfile、CI、linter 與測試設定
補上這個專案真實的 setup、dev、build、lint、typecheck、test、coding style 與完成條件。
只修改文件，不安裝套件、不執行 deploy、不猜測找不到的指令。
```

有 UI 時，再複製 `00-Quickstart/templates/DESIGN.example.md` 為 `DESIGN.md`，要求 Agent 從既有 theme、CSS、token 與元件整理內容，不得自行發明品牌規範。

### 選擇行為 Skills

四個 Skill 都沒有 hooks、scripts 或 MCP；`i-have-adhd` 另外包含非執行的 Codex UI metadata 與 MIT License：

| 需求 | Skill | 建議用法 |
|---|---|---|
| 穩定的 Coding Agent 基線 | `30-Skills/karpathy-guidelines/SKILL.md` | 一般 coding、修 bug、重構與 review 預設使用 |
| 阻止過度工程 | `30-Skills/ponytail-minimal-coding/SKILL.md` | 需要壓低 dependency、抽象與 boilerplate 時疊加 |
| 嚴格 action-first 模式 | `30-Skills/i-have-adhd/SKILL.md` | 明確執行 `$i-have-adhd`；持續到使用者要求停止 |
| 先結論、短步驟、一次一件事 | `30-Skills/adhd-friendly-communication/SKILL.md` | 使用者明確要求 ADHD-friendly 或 focus mode 時使用 |

可直接告訴 Agent：

```text
先完整讀取 30-Skills/karpathy-guidelines/SKILL.md；
這個任務容易過度工程，再同時讀取 30-Skills/ponytail-minimal-coding/SKILL.md。
回答請套用 30-Skills/adhd-friendly-communication/SKILL.md，一次只給我目前要做的步驟。
```

`i-have-adhd` 的 Codex 用法：

```text
$i-have-adhd 請幫我完成目前任務。
```

它採顯式啟用，不會因 Agent 猜測而自動套用；要結束時說 `stop adhd mode` 或 `normal mode`。Karpathy 的原始整理是 4 條；本 repo 使用的是原始 4 條加社群擴充 8 條，共 12 條。ADHD-friendly 只代表溝通偏好，不是診斷或人物標籤。

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

- 公司電腦先讀 [[codex-company-safety|Codex 公司電腦防誤刪設定]]；`AGENTS.md`、prompt 與 Skill 不是 sandbox，不能單獨防止誤刪。
- 不要把 token、cookie、私鑰、內網 URL、客戶資料或個人記憶放進這個公開 repo。
- 不要因文件推薦就直接安裝第三方 Skill、Plugin 或 MCP；先讀完整內容、授權、scripts、hooks、網路與檔案權限。
- `karpathy-guidelines`、`ponytail-minimal-coding`、`adhd-friendly-communication` 是 repo 內純 Markdown 版本；`i-have-adhd` 只 vendoring 上游 `SKILL.md`、`agents/openai.yaml` 與 MIT License，不包含上游 hooks、scripts、extensions 或 plugin metadata。
- 公司禁止外連時，使用 `30-Skills/offline-pptx`、`30-Skills/offline-xlsx` 這類純 Markdown Skill，並只允許已核准的本機工具。
- 外部 Office 檔案視為不受信任輸入；不要啟用巨集、ActiveX、外部資料連線或解除 Protected View。
- 公開前依 [[PUBLICATION-CHECKLIST]] 執行秘密掃描、連結檢查與 staged diff 人工審查。

## 從這裡開始

- [[10-minute-agent-setup|10 分鐘 Agent 設定]]
- [[codex-project-bootstrap|Codex 專案初始化與接入指南]]
- [[codex-company-safety|Codex 公司電腦防誤刪設定]]
- [[codex-company-dxl-start|公司 DXL 專案接入 Codex]]
- [[agents-md|AGENTS.md 開放格式]]
- [[karpathy-principles|Karpathy-inspired 工程原則]]
- [[ponytail-minimal-coding|Ponytail Minimal Coding]]
- [[i-have-adhd/SKILL|I Have ADHD]]
- [[adhd-friendly-communication|ADHD-friendly Communication]]
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
