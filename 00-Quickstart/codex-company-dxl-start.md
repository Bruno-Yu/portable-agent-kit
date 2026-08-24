---
title: 公司 DXL 專案接入 Codex
tags:
  - agent/quickstart
  - codex
  - hcl-domino
  - dxl
last_verified: 2026-08-24
status: public-review
---

# 公司 DXL 專案接入 Codex

目標是先完成一件事：Codex CLI 從公司 DXL 專案根目錄啟動時，能自動讀到安全邊界與基本排查流程。第一版不需要 Skill、hook、MCP 或自動匯入。

> [!important] `AGENTS.md` 的作用範圍
> Codex 會從專案根目錄到目前工作目錄載入 `AGENTS.md`。本 kit 的根目錄規則不會自動套用到另一個公司 repo；公司專案本身必須有一份根目錄 `AGENTS.md`。

## 第一次接入

1. 公司 DXL 專案與本 public repo 分開保存；不要把公司 DXL、NSF、內網設定或分析報告放進 `portable-agent-kit`。
2. 若公司 DXL 是獨立 repo 且沒有既有規則，將 `00-Quickstart/templates/AGENTS.company-dxl.example.md` 複製到公司專案根目錄並命名為 `AGENTS.md`。
3. 若公司 repo 已有根目錄 `AGENTS.md`、coding style、build 或 test 規則，不要覆寫。將範本放到 DXL 子目錄成為 `AGENTS.override.md`，並從該子目錄啟動 Codex；如果該子目錄也已有規則，人工合併安全與 DXL 章節。
4. 只替換範本內 `<...>` 欄位；不知道的值保留 `待確認`，不要猜測。
5. 安裝 [[codex-company-safety|Codex 公司電腦防誤刪設定]] 的 `company-safe` profile。
6. 從單一 DXL repo 或其 DXL 子目錄啟動 Codex，不要從包含多個專案的父目錄啟動：

```bash
codex -C /path/to/one/company-dxl-project --profile company-safe
```

7. 啟動後先執行 `/status` 與 `/debug-config`，確認 sandbox 是 `read-only`，且 web search、Apps、remote plugin 與未核准 MCP 均未啟用。

> [!warning] 不要把 kit 的 project config 帶進公司 repo
> 本 repo 的 `.codex/config.toml` 是維護知識庫用的 `workspace-write` 設定，不屬於 DXL 最小安裝。公司 DXL 只取用 user profile 與 DXL `AGENTS.md` 範本。

## 建議的第一個提示詞

```text
先列出目前載入的 instruction sources，再摘要 AGENTS.md 中的：
1. 工作模式；
2. 禁止事項；
3. DXL source of truth；
4. 本次允許讀取的目錄。

接著只做唯讀盤點：列出設計元素種類、數量、主要語言與可能的外部相依。
不要修改檔案，不要 import、sync、compile、sign、deploy，不要使用網路、MCP、Plugin 或安裝套件。
先在對話中回報盤點結果，不建立報告檔。
```

這個驗證通過後，再交付一個小型排查任務。初期應維持「一次一個專案、一次一類問題」，不要直接批次處理全部 NSF。

## 第一版工作模式

- `Tier A`／`待確認`：NSF 是真實來源；DXL／ODP 只是唯讀文字鏡像。
- Codex 可以讀取、搜尋、比較、盤點與提出驗證方法。
- Codex 不可以直接修改 export，也不可以匯回 Designer。
- 找到問題時，先輸出檔案、設計元素、行號、證據、風險與人工驗證方式。
- 需要寫入報告或程式碼時，另開乾淨 branch／worktree，明確批准後才使用 `company-edit`。
- 是否升級成可 round-trip 的 `Tier B`，留到 compile、sign 與行為基準建立完成後再決定。

DXL 的完整風險背景見 [[HCL Domino 12 NSF 版控與 AI 輔助開發規劃（v2）]]。第一輪不要求把整份文件放入 context；只有碰到版本遷移、重編、簽章或 source-bytecode drift 時才讀相關章節。

## 驗收條件

第一次接入只要確認以下四件事：

- Codex 能說出載入的是公司專案根目錄 `AGENTS.md`。
- Codex 預設把 DXL 視為唯讀輸入。
- Codex 不會自行使用網路、MCP、Plugin、import、compile 或 sign。
- Codex 只盤點使用者指定的單一 repo，不搜尋父目錄或其他公司專案。

通過後再觀察重複出現的排查步驟；流程穩定時才建立共用 DXL Skill，確定性檢查成熟時才加入 hook。

## 官方依據

- [OpenAI Docs：Custom instructions with AGENTS.md](https://learn.chatgpt.com/docs/agent-configuration/agents-md)
- [OpenAI Docs：Agent approvals & security](https://learn.chatgpt.com/docs/agent-approvals-security)
