
> 最後依官方文件核對：2026-08-09。Codex CLI 更新很快；公司電腦請以 `codex --help` 和 `codex --version` 的輸出為準。

這份手冊的預設立場是：**可在受信任的 repository 內修改檔案，但每當要執行可能有副作用的指令時先詢問。** 先建立可驗證的工作流程，再視需要放寬權限。

官方文件：

- [Codex CLI 快速開始](https://learn.chatgpt.com/docs/codex/cli)
- [CLI 完整指令參考](https://learn.chatgpt.com/docs/developer-commands?surface=cli)
- [設定檔](https://learn.chatgpt.com/docs/config-file/config-basic)
- [AGENTS.md 指引](https://learn.chatgpt.com/docs/agent-configuration/agents-md)
- [權限與 Sandbox](https://learn.chatgpt.com/docs/agent-approvals-security)

## 0. 跟公司 IT／資安先確認的四件事

在工作程式碼進入 Codex 前，先取得清楚答案：

1. 可否使用公司核准的 ChatGPT／Codex 帳號，以及適用的資料處理政策。
2. 公司 proxy、防火牆或端點管理是否允許 Codex 登入與更新；若不允許，請 IT 使用公司核准的安裝與更新途徑。
3. 哪些 repo、資料夾和機密資料可以提供給 agent；**不要把密碼、API key、私鑰或客戶資料貼進 prompt、`AGENTS.md` 或可提交的設定檔。**
4. 是否有裝置層級或組織層級政策。組織可以用 managed configuration 限制 sandbox 或 approval 設定；這是預期行為，不要繞過。

### 這份手冊的適用範圍

- 主要面向在 macOS / Linux 終端操作本機 Git repository 的工程師；Windows、WSL 與遠端主機可用的選項可能不同。
- 假設你只被允許使用 Codex CLI，不依賴桌面 App、IDE extension、cloud task 或未核准的外掛。
- 它不是公司資料政策的替代品。當內部規範比這份手冊嚴格時，**永遠以公司規範、受管理設定和 repo 的 `AGENTS.md` 為準**。

### 建議的權限決策

| 工作類型 | 建議 sandbox | approval | 為什麼 |
| --- | --- | --- | --- |
| 看架構、找問題、盤點測試 | `read-only` | 預設即可 | 不需要改檔，風險最低。 |
| 一般 bug fix、補測試、改文件 | `workspace-write` | `on-request` | 可以改目前 repo，但重要 shell 操作仍由你核可。 |
| 需要讀／寫相鄰的共用 library | `workspace-write` + `--add-dir <path>` | `on-request` | 只擴大到必要目錄，不開整台機器。 |
| CI 的純分析或 report | `read-only` | 不依賴人工批准 | 只能在受信任、隔離的 runner 執行。 |
| 發版、資料遷移、infra、刪除資料 | 先由人主導 | 人工核可 | 先讓 Codex 規劃、review 與產生命令；不要交給無人監管流程。 |

## 1. 一次性初始設定

### 安裝與登入

macOS / Linux 的官方安裝器：

```bash
curl -fsSL https://chatgpt.com/codex/install.sh | sh
```

進入專案目錄後登入並啟動：

```bash
cd /path/to/repo
codex login
codex
```

沒有瀏覽器可開時可用裝置碼登入：

```bash
codex login --device-auth
```

請勿把 API key 寫入 shell history、repo 或 `.codex/config.toml`。若公司採 API key 或 access token，由公司的祕密管理／CI 機制提供，並依官方 `codex login --with-api-key` 或 `--with-access-token` 流程輸入 stdin。

### 選擇正確的登入方式

| 情境 | 建議方式 | 指令 | 注意事項 |
| --- | --- | --- | --- |
| 個人互動式開發、公司提供 ChatGPT workspace | ChatGPT 登入 | `codex login` | 使用 ChatGPT workspace 的權限、RBAC 與保留／駐留設定。 |
| 無 GUI、localhost callback 被擋 | Device code | `codex login --device-auth` | 此流程為 beta；是否可用由帳號／workspace 管理設定決定。 |
| 私有 CI runner、程式化工作 | API key | `printenv OPENAI_API_KEY \| codex login --with-api-key` | 依 API 組織計費與資料政策；不要用在不受信任的 runner。 |
| Enterprise 的受控自動化 | Codex access token | `printenv CODEX_ACCESS_TOKEN \| codex login --with-access-token` | 必須先由 Enterprise 管理員允許；用於受信任的腳本或私有 CI。 |

可用 `codex login status` 確認目前認證方式，完成共享裝置工作後以 `codex logout` 清除。Codex 的本機認證快取在 OS credential store 或 `~/.codex/auth.json`；後者等同密碼，絕不可提交、貼到 ticket 或傳給同事。

若公司的 TLS proxy 使用私有根憑證，請 IT 提供 PEM bundle 與核准的設定方式。官方支援在登入前設定 `CODEX_CA_CERTIFICATE=/path/to/corporate-root-ca.pem`；不要自行略過憑證驗證。

### 安裝後健康檢查

```bash
codex --version
codex login status
codex doctor --summary
codex features list
```

`codex doctor` 會檢查安裝、設定、認證、Git、終端與 session 的健康狀況；無法登入或設定不生效時先跑它。可用 `codex update` 更新支援 self-update 的版本。

### 建議的個人預設

建立或編輯 `~/.codex/config.toml`：

```toml
# 對一般公司 repo 的保守預設
approval_policy = "on-request"
sandbox_mode = "workspace-write"
web_search = "cached"
model_reasoning_effort = "medium"
cli_auth_credentials_store = "keyring"
```

- `on-request`：需要時才由你批准執行命令；不等同於自動批准所有操作。
- `workspace-write`：讓 Codex 在目前工作區工作，而不是取得整台電腦的寫入權限。
- `cached`：使用搜尋快取；要查即時外部資料才在單次任務加 `--search`。
- 不建議在全域設定硬寫 model 名稱，因為可用模型受帳號與公司 workspace 政策影響；用 `/model` 或 `codex --model ...` 看當下可用選項。
- `keyring`：明確要求使用作業系統的憑證儲存庫；若公司環境不支援，請讓 `auto` 依官方預設回退，而不要手動複製認證檔。

設定優先序由高到低是：命令列 flag／`-c`、受信任專案的 `.codex/config.toml`、profile、`~/.codex/config.toml`、系統設定、內建預設。若公司電腦有管理政策，實際可用範圍可能更窄；在 CLI 輸入 `/debug-config` 可查看來源與限制。

### 哪種設定該放哪裡？

| 需求 | 放置位置 | 例子 | 不該放什麼 |
| --- | --- | --- | --- |
| 個人、跨 repo 的安全預設 | `~/.codex/config.toml` | approval、sandbox、web search、credential store | 公司或 repo 特有建置命令。 |
| 可信 repo 的共用行為 | `.codex/config.toml` | repo 可用的 sandbox／MCP／模型推理預設 | 憑證、provider auth、機器通知或 telemetry 設定。 |
| 專案作法與驗證標準 | `AGENTS.md` | 測試、lint、架構禁令、完成條件 | token、密碼與長篇歷史。 |
| 特殊工作模式 | `~/.codex/<name>.config.toml` | 深度 code review、低風險文件工作 | 所有人的共同設定。 |
| 只做一次的調整 | flag 或 `-c key=value` | 本次改成 read-only、提高推理量 | 長期偏好。 |

範例：建立只在「深度 review」時用的 profile。Profile 會疊在個人設定之上，但仍低於專案設定與命令列 flag。

```toml
# ~/.codex/deep-review.config.toml
model_reasoning_effort = "high"
approval_policy = "on-request"
sandbox_mode = "read-only"
```

```bash
codex --profile deep-review "Review this branch against main. Do not modify files."
```

單次覆蓋時，優先用專用 flag；沒有專用 flag 才用 `-c`。`-c` 的值是 TOML，不是 JSON：

```bash
codex --sandbox read-only "只分析，不修改檔案"
codex -c model_reasoning_effort='"high"' "釐清這個 race condition 的根因"
```

### 每個 repo 的最低限度設定

在 repo 根目錄建立並提交 `AGENTS.md`；它是給 agent 的專案作業說明，不是放機密的地方。可先在 Codex 內輸入 `/init` 產生草稿，再精簡成真實指令，例如：

```md
# AGENTS.md

## 開發方式

- 安裝依賴：`npm ci`
- 快速驗證：`npm test -- --runInBand`
- 完整檢查：`npm run lint && npm run typecheck && npm test`

## 變更規則

- 先讀取相關程式、測試與現有慣例，再修改。
- 不修改 lockfile、CI 或部署設定，除非任務明確要求。
- 新增或改變行為時，補足對應測試。
- 完成時回報修改檔案、執行過的檢查與未驗證項目。
```

設定只限此 repo 時，可用受信任 repo 的 `.codex/config.toml`。不要把 token、proxy 密碼或內網 URL 的憑證提交進去。

### `AGENTS.md` 的載入規則與維護原則

Codex 會在目前工作目錄往上找指引，直到 project root；離目前目錄越近的指引越具體。可在子目錄放 `AGENTS.override.md` 來覆蓋局部規則，例如支付服務有不同測試命令。專案設定只會在 repo 被信任時載入；不信任的 repo 會跳過 `.codex/` 層、project hook 和 project rule。

建議將檔案控制在約一頁，結構固定：

```md
# AGENTS.md

## 範圍

- `apps/web/` 是前端；`services/api/` 是 API；不要跨服務重構，除非任務明確要求。

## 指令

- 快速測試：`pnpm test --filter <package>`
- 完整驗證：`pnpm lint && pnpm typecheck && pnpm test`

## 變更規則

- 先閱讀相鄰測試與現有模式；避免為單一用途抽象化。
- 不要修改 migration、production config 或 lockfile，除非任務明確要求。
- 禁止讀取、輸出或提交 `.env`、憑證與客戶資料。

## 完成定義

- 行為變更須有回歸測試；回報實際執行的命令和失敗項目。
```

驗證是否載入正確規則：在 repo 根目錄啟動 Codex，要求它「列出目前載入的 instruction sources」，再以 `/status`、`/debug-config` 確認 workspace 與設定層。不要把 lint 格式細節塞進 `AGENTS.md`；交給 CI。若同一錯誤反覆發生兩次，才新增一條短、可執行、能驗證的規則。

## 2. 每日安全工作流程

```text
1. git status                         # 確認自己在哪個 branch、是否已有未提交修改
2. codex --sandbox read-only "..."   # 先理解、盤點或規劃
3. /plan                              # 複雜任務先取得計畫
4. codex                              # 在 workspace-write + on-request 下實作
5. /diff                              # 看每個改動
6. 執行專案檢查                       # test / lint / typecheck
7. /review 或 codex review ...        # 第二次檢查
8. git diff / git status              # 由人決定 commit 與 push
```

### 交給 Codex 前的 60 秒檢查

```bash
pwd
git status --short
git branch --show-current
git log -1 --oneline
```

然後先在 CLI 內查看 `/status`。確認四件事：目前目錄是否正確、sandbox 是否符合任務、approval 是否沒有被意外設成 `never`、以及是否有既有未提交修改。若 working tree 本來就不乾淨，prompt 裡要明講「只處理 `<files>`；不要改其他既有變更」。

### 實作期間的審核節點

1. **探索後：** 讓 Codex 說明會改哪些檔、為何、怎麼驗證。
2. **第一個 patch 後：** 用 `/diff` 或 `git diff --check` 看意外變更與空白錯誤。
3. **測試後：** 確認實際命令、exit status、未跑的檢查與原因。
4. **提交前：** `codex review --uncommitted`，再由人讀完重要 diff。
5. **push 前：** 看 `git status` 與目標 branch；Codex 可以建議 commit 訊息，但是否 commit / push 由人決定。

開始任何可能修改程式碼的工作前，建議先確認乾淨基線或至少記住既有修改：

```bash
git status
git diff
git switch -c feat/short-description
```

## 3. 最常用 CLI 指令速查

| 目的 | 指令／範例 | 說明 |
| --- | --- | --- |
| 進入互動介面 | `codex` | 在目前目錄啟動 TUI。 |
| 指定工作目錄 | `codex -C /path/to/repo` | 不必先 `cd`；`-C` 會設定 agent 的工作目錄。 |
| 直接帶任務啟動 | `codex "先說明此 repo 的架構；不要修改檔案"` | 適合單一明確任務。 |
| 純讀取探索 | `codex --sandbox read-only "找出付款流程與測試入口，不要修改檔案"` | 新 repo 的首選。 |
| 一般實作 | `codex --sandbox workspace-write --ask-for-approval on-request "修正 #123；先提出計畫，完成後跑相關測試"` | 建議日常寫入模式。 |
| 暫時增加可寫目錄 | `codex --add-dir /path/to/shared-lib "..."` | 比 `danger-full-access` 更小的授權範圍。 |
| 改模型／推理量 | `codex -m <available-model> -c model_reasoning_effort="high" "..."` | 僅此次覆蓋；model 名稱以本機可用清單為準。 |
| 要即時網路資料 | `codex --search "依官方文件確認套件升級方式"` | 只在必要時開 live search，外部內容一律當不可信。 |
| 繼續上一個 session | `codex resume --last` | 預設只找目前目錄；加 `--all` 可找所有目錄。 |
| Review 未提交修改 | `codex review --uncommitted` | 檢視 staged、unstaged 與 untracked 內容。 |
| Review 與主線差異 | `codex review --base main` | PR 前很實用；base 名稱依公司慣例調整。 |
| Review 一個 commit | `codex review --commit <SHA>` | 對已存在的 commit 做檢視。 |
| 查問題 | `codex doctor --summary` | 檢查安裝、登入、設定、Git、終端。 |
| 登入狀態 | `codex login status` | 自動化或排錯時確認認證是否存在。 |
| 登出共享電腦 | `codex logout` | 移除儲存的憑證。 |
| 看 MCP 連線 | `codex mcp list` | 列出已設定的外部工具；先審查再使用。 |
| 看 feature 狀態 | `codex features list` | 查看功能成熟度與生效狀態。 |
| 更新 CLI | `codex update` | 只有支援自動更新的安裝版本可用。 |

### 不要當預設的指令

```bash
codex --dangerously-bypass-approvals-and-sandbox
codex --yolo
```

這兩個名稱相同，會繞過 approval 與 sandbox。官方只建議在**已另外強化隔離**的環境使用；一般公司筆電與一般 repo 不該使用。

### 重要指令的細節與判斷點

#### `codex resume`、`fork`、`archive`

- `codex resume --last`：延續目前 repo 最近的互動 session。跨目錄時可加 `--all`，但先確認它使用的工作目錄。
- `codex fork --last`：保留原本對話，從同一脈絡開分支比較兩種作法；適合「修法 A 或 B」而不是要同時改同一批檔案。
- `codex archive <SESSION>`：把完成的 session 從日常清單收起來，但保留記錄；比永久刪除更適合工作紀錄。

#### `codex review`

四種 target 只能選一種，不要混用：

```bash
codex review --uncommitted                 # 目前工作樹
codex review --base main                   # 相對 main 的 branch diff
codex review --commit 0123456789abcdef     # 單一 commit
codex review "特別檢查權限升級與資料遺失風險" # 自訂 review 指令
```

這是輔助 review，不取代測試、CI 或需要領域知識的人類審查。要求它只報高影響問題時，通常可降低雜訊：

```bash
codex review --uncommitted \
  "只回報可重現的正確性、安全性、資料完整性和測試缺口問題；每項標明檔案與影響。"
```

#### `codex mcp`

MCP 會讓 Codex 存取 repo 外的工具或資料，因此比一般 prompt 需要更嚴格的審查：

```bash
codex mcp list
codex mcp get <server-name> --json
```

新增任何 MCP 前，確認 owner、資料範圍、讀／寫能力、OAuth scope、機密如何傳遞，以及公司是否核准。先從一個確實省去人工流程的唯讀整合開始；不要一開始就串 Slack、GitHub、資料庫與部署系統。

#### Shell completion 與 help

```bash
codex --help
codex review --help
codex completion zsh > "${fpath[1]}/_codex"
```

`codex completion` 將 completion script 印到 stdout；最後一行只適用於已確認 `fpath` 的 zsh 環境。公司電腦的 shell 設定受管時不要任意寫入，先問 IT 或只使用 `codex --help`。

## 4. CLI 內最有用的 slash commands

| 指令 | 用法 |
| --- | --- |
| `/status` | 看 model、權限、工作區、token／context 狀態。 |
| `/permissions` | 在同一 session 收緊或放寬權限。 |
| `/model` | 以選單選擇帳號可用的 model 與推理量。 |
| `/plan` | 複雜、含糊或跨多檔的工作先規劃。 |
| `/init` | 產生 `AGENTS.md` 草稿。 |
| `/mention path/to/file` 或 `@` | 把特定檔案帶入後續對話。 |
| `/diff` | 顯示 Git diff（含未追蹤檔）。 |
| `/review` | 檢視目前工作樹。 |
| `/compact` | 長對話壓縮早期內容，保留重點。 |
| `/resume`、`/fork`、`/new` | 延續、分支或開始新的工作脈絡。 |
| `/mcp` | 列出可呼叫的 MCP 工具；`/mcp verbose` 看診斷。 |
| `/ps`、`/stop` | 查看／停止本 session 的背景終端。 |
| `/debug-config` | 排查「為何設定沒生效」或受管理政策限制。 |

在 Codex 工作中按 `Tab` 可排隊下一個 prompt；以 `!` 開頭可在目前權限政策下執行本機 shell 指令。不要因為可以 `!` 就把高風險命令交給 agent；仍應自己審核。

## 5. Prompt 範本：讓結果可驗證

把任務寫成「目標、上下文、限制、完成條件」四段，通常比一句「幫我修」可靠。

```text
目標：修正使用者匯出 CSV 時日期時區錯誤。
上下文：從 @src/export/csv.ts、@src/export/csv.test.ts 開始；問題只出現在 UTC+8。
限制：不要更動 API 格式或新增相依套件；遵循 AGENTS.md。
完成條件：補一個能在修正前失敗的測試，跑相關測試與 lint，最後列出變更與結果。
```

常用請求：

```text
先只讀取並說明這個 bug 的可能根因、涉及檔案與最小修法；不要修改檔案。
```

```text
先給我分步計畫和風險；等我確認後才實作。完成後務必跑 <check command>，不要略過失敗。
```

```text
請 review 目前 diff，只回報會造成功能錯誤、安全問題、資料遺失或測試缺口的項目；每項附檔案與理由。
```

### 六個常見工作情境

#### A. 接手陌生 repository（先不修改）

```bash
codex --sandbox read-only \
  "先讀取 AGENTS.md、README、package/build 設定與測試入口。\
整理：(1) 架構與資料流、(2) 本機啟動與驗證指令、(3) 高風險區域、\
(4) 我應該先讀的 5 個檔案。不要修改檔案，也不要執行網路或寫入操作。"
```

#### B. 修一個可重現 bug

```text
目標：修正 <bug>。
重現：執行 `<exact command>` 時，預期 <expected>，實際 <actual>。
範圍：從 @<file> 和 @<test> 開始；不要重構無關程式。
流程：先用 read-only 分析根因與最小修法，等我確認後再實作。
完成：新增會在修正前失敗的測試，執行 `<test command>` 和 `<lint command>`。
```

#### C. 小型功能開發

```text
先讀現有相似功能和測試，提出 3–6 步計畫與預計修改檔案。
限制：維持既有 public API；不新增 dependency；資料庫 schema 不變。
確認計畫後才實作。每個可檢查節點回報 diff 與驗證結果。
```

#### D. 重構但不能改變行為

```text
先列出目前行為、public contract、涵蓋不足的測試與風險。
以小步驟重構；每步都執行最相關的測試。不要順手改 formatting、lockfile 或無關模組。
最後比較前後行為，列出未能驗證的部分。
```

#### E. CI 失敗排查

```text
根據下列 CI log 找出最可能根因。先不要改檔案。
請提出最小可驗證的修法、需要在本機跑的命令，以及可能只是環境差異的證據。
<貼上已去除 token、客戶資料與內網 URL 的 log>
```

#### F. 文件與 release note

```text
只根據目前 repository 的實際程式、測試和變更內容更新文件。
不要臆測尚未發布的功能。完成後列出每個主張對應的原始檔或測試。
```

### 提問時避免的模糊語句

| 不夠具體 | 較可靠的說法 |
| --- | --- |
| 「幫我修 bug」 | 「重現命令是…；預期／實際是…；先分析根因，不要改檔。」 |
| 「把它重構好」 | 「將函式拆成兩個單責任函式，public API 不變，測試需維持通過。」 |
| 「跑所有測試」 | 「先跑 `<fast test>`；若通過再跑 `<full test>`；若太久先回報預估與範圍。」 |
| 「直接做完」 | 「可改 `src/` 與 `tests/`；不可動部署、migration、lockfile；完成條件是…」 |

## 6. `codex exec`：給可重複、非互動的任務

`codex exec`（縮寫 `codex e`）適合 CI 或已穩定的腳本流程；它不需要人在終端裡對話。第一次不要拿它執行會發版、刪除資料或直接 push 的工作。

```bash
codex exec --sandbox read-only \
  "檢查目前分支相對 main 的變更，列出高風險回歸；不要修改任何檔案。"
```

把最後摘要寫入檔案，供後續腳本使用：

```bash
codex exec --sandbox workspace-write \
  --output-last-message ./codex-summary.txt \
  "更新文件以反映現有程式行為，並跑文件檢查。"
```

如需讓其他程式解析進度，可加 `--json`（輸出 JSONL）。請明確指定 `--sandbox`，並在 CI 使用限制權限的 runner；不要用 `--yolo` 取代隔離。

### CI／automation 的最低安全基線

1. 只在私有、受信任的 runner 執行；不要把能寫 repo 的 Codex endpoint 暴露到公網或 PR fork。
2. 初期只允許 read-only 任務，例如 diff 摘要、文件一致性檢查與 code review。
3. 明確指定 `--sandbox read-only` 或 `--sandbox workspace-write`，不要依賴不透明的預設值。
4. 給 task 有限的目錄與明確完成條件；腳本中的 prompt 也要遵守 repo `AGENTS.md`。
5. 若要機器解析結果，用 `--json`；若只要最後摘要，用 `--output-last-message <path>`。不要讓下游程序從自由文字猜測是否成功。
6. 對寫入工作，至少保留 human review、branch protection 和既有 CI。Codex 產出的 commit 不應自動繞過它們。
7. 對無須保存對話紀錄的短任務可用 `--ephemeral`；它不會把 session rollout 檔持久寫到本機。仍須按照公司資料保存政策處理輸入、輸出與 CI log。

一個安全的「每日 diff 摘要」例子：

```bash
codex exec --sandbox read-only --json \
  "以目前分支相對 main 的 diff 為範圍，列出測試缺口與高風險回歸。\
不要修改檔案；沒有證據不要推測。"
```

## 7. 常見問題與排查

| 現象 | 先做什麼 | 常見原因／下一步 |
| --- | --- | --- |
| `codex` 找不到或啟動失敗 | `codex --version`、`codex doctor --summary` | 安裝不完整、PATH 沒更新或端點管理阻擋；交給 IT 依核准流程安裝。 |
| 登入後仍無法使用 | `codex login status`、再跑 `codex login` | workspace / seat / RBAC、proxy、瀏覽器 callback 或公司強制登入方式。無 GUI 優先試 `--device-auth`。 |
| 公司 proxy 出現 TLS 錯誤 | 請 IT 檢查根 CA 與 proxy | 使用核准的 `CODEX_CA_CERTIFICATE`；不要加不安全的 TLS 略過選項。 |
| 設定看似被忽略 | CLI 輸入 `/debug-config` | CLI flag、專案／profile／user 層覆蓋，或公司 managed requirements 限制。 |
| Codex 不能改檔 | `/status`、`/permissions` | 正在 read-only、workspace 判斷錯誤、目錄不在 writable root，或公司政策禁止。先用 `-C`／`--add-dir` 精準修正。 |
| `.codex/config.toml` 沒生效 | 確認 repo 是否 trusted | 不信任 repo 時，專案設定、hook、rule 會被跳過；不要為了載入設定而盲目信任陌生 repo。 |
| 指令一直要求批准 | 確認 sandbox、approval 與命令副作用 | 這通常是保護機制，不是故障。減少任務範圍或在真正必要時逐次批准。 |
| session 太長、回答開始失焦 | `/compact` 或 `/new` | 一個 coherent outcome 一個 session；真正分支時用 `/fork`。 |
| 要回報支援 | `codex doctor --json` | 報告為 redacted，但仍須先依公司規範審閱再外傳。 |

### 給公司 IT 的最小資訊包

遇到無法自行排除的安裝／登入問題時，提供下列**不含 credential、原始碼或客戶資料**的內容：

```bash
codex --version
codex login status
codex doctor --summary
```

再附上：OS 與 shell、是否使用 VPN／proxy、錯誤發生的時間、是否為 browser login 或 device code、是否只在某 repo 發生。若 IT 要求完整診斷，先問清楚資料處理位置與保存期限。

## 8. 公司端的維護建議

- 每天：`git status` → Codex → `/diff` → 專案檢查 → `/review` → 人工 commit。
- 每個 repo：維護短而準確的 `AGENTS.md`，特別是建置、測試、lint、部署禁令與「完成」定義。
- 每次遇到重複錯誤：更新 `AGENTS.md` 或把已穩定的流程做成 skill，而不是每次重打長 prompt。
- 每次設定衝突：先跑 `/debug-config`；不要猜測是哪一層覆蓋設定。
- 每次受限：使用最小的放寬方式（單次 flag、`--add-dir`、專案設定），不要直接開全機權限。
- 共用或離職交接的電腦：完成後 `codex logout`，並按公司規範移除本機 repo／暫存資料。

## 9. 建議的第一週採用節奏

1. **第 1 天：只讀。** 用 `--sandbox read-only` 熟悉「解釋 repo、找測試、提出計畫」。
2. **第 2–3 天：小改動。** 只做有明確測試的 bug fix；維持 `workspace-write + on-request`。
3. **第 4 天：補 AGENTS.md。** 將真正有效的 build/test/lint 與禁令寫進 repo。
4. **第 5 天：review 流程。** 對自己的 diff 跑 `codex review --uncommitted`，但由人決定是否採納。
5. **穩定後才自動化。** 先手動重複成功數次，再把純分析／文件／檢查工作交給 `codex exec` 或 CI。

---

### 遇到問題時的最短排查路徑

```bash
codex --version
codex login status
codex doctor --summary
codex features list
```

接著在 CLI 內執行 `/status` 與 `/debug-config`。把 `codex doctor --json` 的**已遮蔽**診斷資訊附給公司 IT／OpenAI 支援前，仍請先依公司資安規範檢查。
