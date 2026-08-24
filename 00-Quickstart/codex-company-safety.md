---
title: Codex 公司電腦防誤刪設定
tags:
  - agent/quickstart
  - codex
  - security
last_verified: 2026-08-20
status: public-review
---

# Codex 公司電腦防誤刪設定

目標是讓 Codex 預設無法改動公司電腦內容，需要實作時也只把寫入範圍縮到單一 Git workspace。這份設定不取代公司 IT、資料分級、備份與端點管理政策。

> [!danger] 先知道真正的邊界
> `AGENTS.md`、prompt 與 Skill 是行為指示，不是技術隔離。真正限制檔案存取的是 Codex 的 OS sandbox；真正防止使用者以 flag 放寬限制的是管理員部署的 `requirements.toml`。

## 建議使用模式

| 工作 | Profile | 技術邊界 |
|---|---|---|
| 看程式、規劃、review、找問題 | `company-safe` | 唯讀；預設不能修改或刪除檔案 |
| 實作已核准的小改動 | `company-edit` | 只可寫目前 workspace；workspace 內仍可能被修改或刪除 |
| 公司統一管控 | `requirements.toml` | 禁止 full access、無批准模式、外連 MCP／Plugin 等被使用者覆寫 |

最安全的日常流程是「先唯讀，再在乾淨 branch、Git worktree 或可丟棄副本中短暫開啟 workspace write」。未納入 Git 的重要檔案不要放在可寫 workspace。

## 個人安裝：先用唯讀 Profile

將 `00-Quickstart/templates/company-safe.config.toml` 複製到 `$CODEX_HOME/company-safe.config.toml`。預設的 `$CODEX_HOME` 是 `~/.codex`。

若目的地已有同名 profile，先人工比較，不要直接覆寫。這個 profile 只有在命令列指定 `--profile company-safe` 時才載入；受信任專案的 `.codex/config.toml` 優先序較高，仍須用 `/debug-config` 確認沒有把唯讀模式改回 `workspace-write`。不要把 portable-agent-kit 自己的 `.codex/config.toml` 複製到公司 repo。

只從單一 repo 根目錄啟動，不要在家目錄、桌面或包含多個公司專案的父目錄啟動：

```bash
codex -C /path/to/one/repo --profile company-safe
```

啟動後先執行 `/status` 與 `/debug-config`，確認：

- sandbox 是 `read-only`。
- approval 是 `untrusted`，reviewer 是 `user`。
- workspace 只有預期的單一 repo，沒有額外 writable root。
- web search、Apps 與 remote plugin 沒有被其他設定重新啟用。
- `codex mcp list` 沒有啟用中的外部 MCP；個人 profile 無法用一條通用設定停用所有既有 MCP。

> [!warning]
> Profile 是使用者設定，不是強制政策；命令列參數或更高優先序設定仍可能改寫它，也不會自動移除使用者原本設定的 MCP。不要建立 `danger-full-access` profile，也不要使用 `--yolo` 或 `--dangerously-bypass-approvals-and-sandbox`。公司禁止 MCP 時，應由 IT 使用空的 managed `mcp_servers` allowlist 強制停用。

## 需要修改時：限縮到單一 Workspace

確認任務與 `git status` 後，將 `00-Quickstart/templates/company-edit.config.toml` 複製到 `$CODEX_HOME/company-edit.config.toml`，再從專用 branch 或 worktree 啟動：

```bash
git status --short
codex -C /path/to/one/repo --profile company-edit
```

這個 profile 使用 `workspace-write`、關閉 command network、排除 `/tmp` 與 `$TMPDIR` 的額外寫入根目錄，並以 `untrusted` 要求非已知安全命令先取得人工批准。

它能阻止 sandboxed commands 寫到 workspace 外，但**不能保證 workspace 內的檔案不被刪除**。因此仍要：

1. 只在已 commit 的乾淨 branch、獨立 worktree 或可還原副本工作。
2. 不把唯一一份文件、憑證、客戶資料或未追蹤的重要檔案放進 workspace。
3. 每次批准前讀完整 command、目標路徑與作用範圍；不批准 `rm`、`git clean`、`git reset --hard`、跨目錄移動或不明腳本。
4. 用 `/diff`、`git diff --check` 與 `git status --short` 檢查改動；commit、push、deploy 與資料遷移由人決定。
5. 任務結束後回到 `company-safe`。

## 公司強制管控：交由 IT 部署

`00-Quickstart/templates/company-requirements.example.toml` 是給 IT／資安審查的範本，會限制可選 approval、sandbox、web search、MCP、Plugin、Apps、Browser、Computer Use、remote control 與非管理 hooks。

它必須由管理員部署到系統層或企業 managed configuration 才有效；把檔案 commit 到 repo 或放進個人的 `~/.codex` 不會形成不可覆寫的限制。官方列出的系統位置是：

- macOS／Linux：`/etc/codex/requirements.toml`
- Windows：`%ProgramData%\OpenAI\Codex\requirements.toml`

部署前要在測試機執行目前版本的 Codex，使用 `--strict-config`、`codex doctor --summary`、`/status` 與 `/debug-config` 驗證；不同 client 與版本支援的管理鍵可能不同。

## Repo 本身的安全預設

本 repo 的 `.codex/config.toml` 使用：

- `workspace-write`：只允許目前 repo 寫入，不開整台電腦。
- `untrusted` + `user` reviewer：非已知安全命令交給人批准，不使用自動 reviewer。
- command network、web search、Apps 與 remote plugin 關閉。
- `/tmp`、`$TMPDIR` 不加入可寫根目錄，login shell 關閉。

這只保護在本 repo 啟動的 Codex session，且使用者仍可覆寫；跨 repo 的預設應使用 profile，公司硬性政策應使用 managed requirements。

## 還需要的裝置層保護

- 使用公司管理的標準權限帳號，不用管理員或 root 身分執行 Codex。
- 啟用公司核准的備份、磁碟加密、EDR／MDM 與資料遺失防護。
- 將不可修改的資料以唯讀掛載、獨立帳號 ACL 或隔離 VM／container 提供；不要只依賴 Agent 指示。
- 公司若禁止外連，除 Codex 服務本身的核准流量外，還要由防火牆／proxy 做 egress allowlist；sandbox command network 不涵蓋 web search、Apps、MCP、Browser 或 Computer Use。

## 官方依據

- [OpenAI Docs：Agent approvals & security](https://learn.chatgpt.com/docs/agent-approvals-security)
- [OpenAI Docs：Configuration Reference](https://learn.chatgpt.com/docs/config-file/config-reference)
- [OpenAI Docs：Managed configuration](https://learn.chatgpt.com/docs/enterprise/managed-configuration)

> 最後依官方文件核對：2026-08-20。Codex 更新快速，部署前以公司核准版本與 `codex --version` 為準。
