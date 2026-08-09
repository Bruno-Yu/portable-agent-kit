
> **文件版本**：2026-05-28｜基於 VS Code 1.119 官方文件撰寫
> **官方來源**：https://code.visualstudio.com/docs/copilot/overview

---

## 目錄

1. [VS Code 基本介紹](#1-vs-code-基本介紹)
2. [GitHub Copilot 授權與啟用](#2-github-copilot-授權與啟用)
3. [Copilot Chat — 三種內建 Agent](#3-copilot-chat--三種內建-agent)
4. [可用的 LLM 模型](#4-可用的-llm-模型)
5. [Chat 工具：@ / # / /](#5-chat-工具----)
6. [Agent 執行權限層級](#6-agent-執行權限層級)
7. [設定範疇：User / Workspace / .code-workspace](#7-設定範疇-user--workspace--code-workspace)
8. [Copilot 自訂指令系統](#8-copilot-自訂指令系統)
9. [Prompt Files / Custom Agents / Skills / MCP / Hooks](#9-prompt-files--custom-agents--skills--mcp--hooks)
10. [快速命令與設定速查](#10-快速命令與設定速查)
11. [官方參考資源](#11-官方參考資源)

---

## 1. VS Code 基本介紹

### 1.1 介面組成

```
┌─────────────────────────────────────────────────────────────┐
│  Title Bar（標題列）—— 含 Chat 選單、Command Center          │
├────┬────────────────────────────────────────────┬───────────┤
│    │                                            │           │
│ A  │                                            │  Side     │
│ c  │           Editor Area（編輯區）             │  Panel    │
│ t  │         （主要程式碼編輯空間）               │ （次要    │
│ i  │                                            │  面板）   │
│ v  │                                            │           │
│ i  ├────────────────────────────────────────────┤           │
│ t  │      Panel（底部面板）                      │           │
│ y  │  Terminal / Output / Problems / Debug      │           │
│    ├────────────────────────────────────────────┴───────────┤
│ B  │              Status Bar（狀態列）                       │
│ a  │  語言 | Git 分支 | Copilot 狀態 | 行號:列號             │
│ r  │                                                        │
└────┴────────────────────────────────────────────────────────┘
```

| 區域 | 說明 |
|------|------|
| **Activity Bar**（左側圖示列）| 切換 Explorer、Search、Source Control、Extensions、**Copilot** 等面板 |
| **Side Bar**（側邊欄）| 顯示目前選取的面板內容 |
| **Editor Area**（編輯區）| 主要編輯區域，支援多 Tab、Split View、Diff View |
| **Panel**（底部面板）| Terminal、Output、Problems、Debug Console |
| **Status Bar**（最底列）| 顯示語言模式、Git 分支、**Copilot 狀態**、行號等 |

> **Copilot 圖示**：Activity Bar 有專屬的 Copilot 圖示，點選可開啟 Chat 面板。

### 1.2 常用快速鍵（Windows）

| 功能 | 快速鍵 |
|------|--------|
| 命令面板 | `Ctrl + Shift + P` |
| 快速開啟檔案 | `Ctrl + P` |
| 開啟設定 UI | `Ctrl + ,` |
| 開啟終端機 | `` Ctrl + ` `` |
| **開啟 Copilot Chat** | `Ctrl + Alt + I` |
| **開啟 Inline Chat（編輯器內）** | `Ctrl + I` |
| **開啟 Quick Chat** | `Ctrl + Shift + Alt + L` |
| 接受 Inline Suggestion | `Tab` |
| 拒絕 Inline Suggestion | `Esc` |

### 1.3 開啟設定

**方式一：圖形化 UI（推薦新手）**
> `Ctrl + ,` 或命令面板輸入 `Preferences: Open Settings (UI)`

**方式二：直接編輯 JSON**
> 命令面板輸入 `Preferences: Open User Settings (JSON)`

---

## 2. GitHub Copilot 授權與啟用

### 2.1 VS Code 1.119 已內建 Copilot

> ✅ 從 VS Code 1.119 起，GitHub Copilot 已**預裝整合**於 VS Code 本體中。
> **無需另外安裝任何擴充套件**，Activity Bar 上直接可見 Copilot 圖示。

### 2.2 前置需求

- **有效的 GitHub Copilot 訂閱**（以下任一）：
  - GitHub Copilot Individual（個人付費）
  - GitHub Copilot Business（企業付費）
  - GitHub Copilot Enterprise（企業進階）
  - GitHub Copilot Free（免費版，功能有限制）

> ⚠️ **注意（截至 2026-04）**：新 Copilot Pro / Pro+ 訂閱暫停開放，詳見官方公告。
> 參考：https://docs.github.com/copilot/concepts/usage-limits

### 2.3 登入步驟

1. 點選 Activity Bar 的 **Copilot 圖示**
2. 選擇 **Sign in to use GitHub Copilot**
3. 瀏覽器自動開啟 GitHub OAuth 頁面
4. 完成授權後回到 VS Code
5. Status Bar 右下角 Copilot 圖示恢復正常（無 ❌ 或 ⚠️）

### 2.4 驗證是否正常

- Status Bar 右下角 Copilot 圖示正常顯示（無錯誤）
- 在任意程式碼檔案輸入時，出現灰色的 **inline suggestion**
- 按 `Ctrl + Alt + I` 可成功開啟 Chat 面板

---

## 3. Copilot Chat — 三種內建 Agent

> **重要術語更新**：在 VS Code 1.119 中，Chat 模式稱為「**Agent**」而非「模式（Mode）」。
> 從 Chat 面板輸入框的 **agent picker 下拉選單** 切換。

### 3.1 Ask Agent（詢問代理人）

- **用途**：問問題、解釋程式碼、理解概念、查詢用法
- **特性**：
  - **不會修改任何檔案**
  - 不執行 terminal 指令
  - 適合學習、探索、取得建議
- **適用情境**：「這段程式碼是什麼意思？」「如何使用這個 API？」

### 3.2 Plan Agent（規劃代理人）

- **用途**：在動手修改前，先產生**結構化的逐步實作計畫**
- **特性**：
  - 先產生清晰的步驟清單供你審閱
  - 計畫確認後，**交由 Agent 代理人執行**
  - 大幅降低 AI 自動執行的不確定性
- **適用情境**：不確定 AI 會改哪些地方時，先用 Plan 確認計畫

### 3.3 Agent（代理人）

- **用途**：自主完成複雜的多步驟任務
- **特性**：
  - 自動呼叫工具（讀寫檔案、執行 terminal、搜尋程式碼等）
  - 發現錯誤時會**自我修正**並重試
  - 每個工具呼叫依**權限層級**決定是否需要確認
- **適用情境**：建立新功能、修 bug、設定專案環境、重構程式碼

### 3.4 Agent 比較表

| | Ask | Plan | Agent |
|---|:---:|:---:|:---:|
| 修改檔案 | ❌ | ✅（需確認）| ✅（依權限）|
| 執行 Terminal | ❌ | ✅（需確認）| ✅（依權限）|
| 先規劃後執行 | ❌ | ✅ | ❌ |
| 自我修正迭代 | ❌ | ❌ | ✅ |
| 適合情境 | 問答學習 | 複雜任務確認 | 自動化執行 |

> **官方文件**：https://code.visualstudio.com/docs/copilot/agents/overview

---

## 4. 可用的 LLM 模型

### 4.1 如何切換模型

在 Chat 輸入框**左側的模型名稱下拉選單**點選，即可選擇不同模型。
也可透過命令面板：`Language Models: Select Language Model`

另外，可開啟 **Language Models 管理頁面**查看完整模型清單與配額：
> 命令面板 → `Language Models: Manage Language Models`

### 4.2 模型清單（截至 2026-05-28）

以下為實際 VS Code 介面中顯示的可用模型：

| 模型 | Context Size | 能力 | 配額倍率 | 建議用途 |
|------|:-----------:|------|:-------:|---------|
| **Claude Haiku 4.5** | 200K | Tools + Vision | 0.33x | 快速任務、大量簡單問答 |
| Claude Sonnet 4.6 | 200K | Tools + Vision | 1x | 日常開發、Agent 模式 |
| **Claude Opus 4.6** | 200K | Tools + Vision | 3x | 複雜架構設計、深度推理（**最強但消耗多**）|
| **Gemini 2.5 Pro** | 173K | Tools + Vision | 1x | 多模態、程式碼分析 |
| **Gemini 3 Flash** *(Preview)* | 173K | Tools + Vision | 0.33x | 快速輕量任務 |
| **Gemini 3.1 Pro** *(Preview)* | 200K | Tools + Vision | 1x | 進階分析任務 |
| **GPT-4.1** ⚠️ | 128K | Tools + Vision | **0x** | 不消耗配額，預算考量 |
| **GPT-5 mini** | 192K | Tools + Vision | **0x** | 不消耗配額，輕量任務 |
| **GPT-5.2** ⚠️ | 400K | Tools + Vision | 1x | 超大程式碼庫分析 |
| **GPT-5.2-Codex** | 400K | Tools + Vision | 1x | 程式碼專用任務 |
| **GPT-5.3-Codex** | 400K | Tools + Vision | 1x | 程式碼專用任務 |
| **GPT-5.4** | 400K | Tools + Vision | 1x | 最新 GPT-5 旗艦版 |

> **說明**：
> - **⚠️ 符號**：官方標記為 deprecated/legacy（舊版），仍可使用但建議遷移
> - **配額倍率**：0x = 不消耗訂閱配額；0.33x = 消耗 1/3 配額；1x = 消耗標準配額；3x = 消耗 3 倍配額
> - **Tools**：支援 Agent 工具呼叫（讀寫檔案、執行指令等）
> - **Vision**：支援圖片輸入（截圖、UI 設計圖等）

### 4.3 模型選擇建議

```
日常開發         → Claude Sonnet 4.6
節省配額         → GPT-4.1 或 GPT-5 mini（0x，不消耗配額）
大型程式碼庫      → GPT-5.4（400K Context）
複雜架構/推理     → Claude Opus 4.6（最強，消耗 3x 配額）
快速輕量問答      → Claude Haiku 4.5 或 Gemini 3 Flash（0.33x）
```

---

## 5. Chat 工具：@ / # / /

### 5.1 Chat 參與者（以 `@` 開頭）

在 Chat 輸入框中輸入 `@` 可呼叫特定領域的專屬代理人：

| 參與者 | 說明 | 範例 |
|--------|------|------|
| `@workspace` | 理解整個工作區程式碼，回答跨檔案問題 | `@workspace 哪裡有呼叫 getUserById？` |
| `@vscode` | 回答 VS Code 設定、功能、擴充套件問題 | `@vscode 如何設定 ESLint？` |
| `@terminal` | 協助撰寫與解釋 Terminal 指令 | `@terminal 如何找出佔用 port 3000 的程序？` |
| `@github` | 存取 GitHub Issues、PR、Repository 資訊 | `@github 最新的 PR 有什麼變更？` |

> **官方文件**：https://code.visualstudio.com/docs/copilot/chat/copilot-chat

### 5.2 Chat 變數（以 `#` 開頭）

在 Chat 中使用 `#` 插入特定上下文資訊：

| 變數 | 說明 | 範例 |
|------|------|------|
| `#file` | 指定工作區中的一個檔案 | `#file:src/auth.ts 有什麼安全問題？` |
| `#selection` | 目前編輯器中選取的程式碼 | `請優化 #selection` |
| `#editor` | 目前開啟的編輯器完整內容 | `解釋 #editor` |
| `#codebase` | 整個程式碼庫（深度搜尋）| `#codebase 哪個檔案處理登入邏輯？` |
| `#sym` | 指定特定符號（函式、類別等）| `解釋 #sym:UserService` |
| `#terminalSelection` | Terminal 中選取的文字 | `解釋 #terminalSelection 的錯誤訊息` |
| `#terminalLastCommand` | Terminal 最後執行的指令與輸出 | `#terminalLastCommand 為何失敗？` |
| `#fetch` | 抓取指定 URL 的內容 | `#fetch:https://api.example.com/docs` |

### 5.3 Slash 指令（以 `/` 開頭）

在 Chat 輸入框輸入 `/` 可使用快捷指令：

| 指令 | 說明 |
|------|------|
| `/explain` | 解釋選取的程式碼 |
| `/fix` | 修復程式碼中的問題 |
| `/tests` | 為選取的程式碼產生測試 |
| `/doc` | 產生文件註解 |
| `/clear` | 清除目前對話紀錄 |
| `/help` | 顯示 Copilot 說明 |
| `/new` | 建立新的工作區或專案 |
| `/init` | 分析工作區並自動產生 `copilot-instructions.md` |
| `/instructions` | 開啟 Configure Instructions and Rules 選單 |
| `/create-instruction` | 透過 AI 產生指令檔案 |

---

## 6. Agent 執行權限層級

### 6.1 為什麼需要權限控制？

Agent 模式可執行影響系統的操作（修改檔案、執行 shell 指令），VS Code 讓你控制 Agent 的自主程度，從每次確認到完全自動。

### 6.2 三種權限層級

在 Chat 面板輸入框的**「permissions picker（權限選擇器）」**設定：

| 層級 | 圖示說明 | 行為 | 適合情境 |
|------|---------|------|---------|
| **Default Approvals** | ✅ Copilot uses your configured settings | 依 VS Code 設定決定。預設：唯讀/安全工具不需確認，其他需核准 | **推薦新手使用** |
| **Bypass Approvals** | ⚠️ All tool calls are auto-approved | 所有工具呼叫自動核准，不彈出確認視窗。Agent 仍可能詢問澄清問題 | 熟悉任務且信任 AI 時 |
| **Autopilot** *(Preview)* | 🤖 Autonomously iterates from start to finish | 所有工具自動核准 + 自動回應問題 + 持續執行直到任務完成，**完全自主** | 定義清楚的明確任務 |

> **設定持久化**：若要讓偏好的權限層級跨 session 持續，設定 `chat.permissions.default`
> **官方文件**：https://code.visualstudio.com/docs/copilot/agents/overview#_choose-a-permission-level

### 6.3 安全建議

```
新手 → Default Approvals（每次仔細確認 Agent 要執行的操作）
有經驗 → Bypass Approvals（已熟悉任務，不需每次點確認）
批次自動化 → Autopilot（任務定義明確，接受 AI 完全自主執行）
```

> ⚠️ **Autopilot 注意事項**：啟用後 AI 可自行執行 terminal 指令，請確保在安全環境下使用，避免不可逆的操作。

---

## 7. 設定範疇：User / Workspace / .code-workspace

### 7.1 三種設定範疇

#### 👤 User 設定（使用者層級）
- **位置（Windows）**：`%APPDATA%\Code\User\settings.json`
- **範圍**：此電腦上**所有**專案
- **適用**：個人偏好（字體、主題、個人 Copilot 設定）
- **開啟**：`Ctrl + Shift + P` → `Preferences: Open User Settings (JSON)`

#### 📁 Workspace 設定（工作區層級）
- **位置**：專案根目錄的 `.vscode/settings.json`
- **範圍**：只影響**此專案**，建議加入 Git 版控
- **優先權**：**覆蓋** User 設定
- **開啟**：`Ctrl + Shift + P` → `Preferences: Open Workspace Settings (JSON)`

#### 🗂 .code-workspace 檔案（多根工作區）
- **位置**：任意位置的 `*.code-workspace` 檔案
- **範圍**：定義包含**多個資料夾**的工作區（Monorepo / 前後端分離）
- **開啟**：`File > Open Workspace from File...`

**結構範例**：
```json
{
    "folders": [
        { "path": "./frontend", "name": "前端" },
        { "path": "./backend",  "name": "後端" }
    ],
    "settings": {
        "editor.fontSize": 14,
        "github.copilot.chat.codeGeneration.useInstructionFiles": true
    },
    "extensions": {
        "recommendations": ["dbaeumer.vscode-eslint"]
    }
}
```

### 7.2 設定優先順序

```
.code-workspace 設定  >  Workspace (.vscode/settings.json)  >  User 設定  >  預設值
```

### 7.3 .vscode 資料夾完整結構

```
.vscode/
├── settings.json      ← 工作區設定（建議加入 Git）
├── launch.json        ← Debug 執行設定
├── tasks.json         ← 自動化 Task 設定
├── extensions.json    ← 推薦擴充套件清單
└── mcp.json           ← MCP 工具伺服器設定（Copilot Agent 用）
```

---

## 8. Copilot 自訂指令系統

> 自訂指令（Custom Instructions）讓你告訴 Copilot **專案規範、程式碼風格與架構慣例**，
> 使全團隊的 AI 行為一致，不需每次手動說明背景。

### 8.1 指令類型總覽

| 類型 | 套用時機 | 適用場景 |
|------|---------|---------|
| **Always-on**（常態載入）| 自動套用於所有 Chat 請求 | 全專案規範、技術棧宣告 |
| **File-based**（條件載入）| 依檔案路徑 glob 或語意比對 | 語言特定規範、框架規則 |

### 8.2 `.github/copilot-instructions.md`（最重要）

**Always-on 指令**，自動套用於此工作區所有 Chat 請求。

**位置**：
```
專案根目錄/
└── .github/
    └── copilot-instructions.md
```

**啟用設定**（加入 `.vscode/settings.json`）：
```json
{
    "github.copilot.chat.codeGeneration.useInstructionFiles": true
}
```

**範例內容**：
```markdown
# 專案開發規範

## 技術棧
- 前端：React 18 + TypeScript + Vite
- 後端：.NET 8 + Dapper + Oracle
- 測試：xUnit + Vitest

## 程式碼規範
- 使用 async/await，禁止 .then() 鏈式呼叫
- 禁止使用 TypeScript 的 `any` 型別
- 所有公開 API 必須有 JSDoc 或 XML 文件

## 架構規範
- API 遵循 Controller → Service → Repository 分層
- 資料庫操作使用參數化查詢，禁止字串拼接

## 回應語言
- 請用繁體中文回答
```

> **💡 提示**：可用 `/init` 指令讓 AI 自動分析你的工作區並產生這份檔案

### 8.3 AGENTS.md（多 AI 工具共用）

適合同時使用多種 AI 工具（Copilot、Claude Code 等）的團隊，`AGENTS.md` 會被所有支援的工具讀取。

```
專案根目錄/
└── AGENTS.md
```

### 8.4 `.github/instructions/*.instructions.md`（細粒度條件指令）

針對**特定檔案類型**自動套用不同規範：

**目錄結構**：
```
.github/
└── instructions/
    ├── frontend/
    │   └── react.instructions.md
    ├── backend/
    │   └── api.instructions.md
    └── testing/
        └── unit-tests.instructions.md
```

**檔案格式**（YAML frontmatter + Markdown 內容）：
```markdown
---
name: 'API 開發規範'
description: '套用於所有後端 API C# 檔案'
applyTo: '**/api/**/*.cs'
---
# API 開發規範

- Controller 只做路由與參數驗證，商業邏輯放在 Service
- 所有 Oracle 查詢使用 Dapper 參數化（`:param` 格式）
- 錯誤以 ApException 拋出，包含 ErrorCode
- 所有 API 需加 [Authorize] 屬性
```

> **`applyTo` 格式**：標準 glob pattern（如 `**/*.ts`、`**/api/**/*.cs`）
> **未設定 applyTo**：不自動套用，但可在 Chat 中手動附加

### 8.5 指令優先順序

```
個人（User 層級）  >  Repository（.github/copilot-instructions.md）  >  組織層級
```

---

## 9. Prompt Files / Custom Agents / Skills / MCP / Hooks

### 9.1 Prompt Files（可重用提示詞範本）

預先撰寫好的 prompt 範本，可在 Chat 中快速呼叫。

**位置**：`.github/prompts/*.prompt.md`

**啟用設定**：
```json
{
    "chat.promptFiles": true
}
```

**範例**（`.github/prompts/code-review.prompt.md`）：
```markdown
---
mode: ask
description: 對選取程式碼進行完整 Code Review
---
請對以下程式碼進行完整的 Code Review，包含：
1. 程式碼品質與可讀性
2. 潛在的 Bug 或邊界情況
3. 效能問題
4. 安全性漏洞（OWASP Top 10）
5. 具體改善建議與範例程式碼

程式碼：
{{selection}}
```

**使用方式**：
- Chat 輸入框輸入 `/` → 從清單選取已建立的 prompt
- 或輸入 `/instructions` 開啟管理介面

### 9.2 Custom Agents（自訂代理人）

除了內建的 Ask / Plan / Agent 三種代理人，你可以建立**自訂代理人**，賦予特定角色、工具與指令（如安全審查員、架構規劃師等），儲存後可在 agent picker 直接選取。

**位置**：`.github/agents/*.agent.md`

```
.github/
└── agents/
    ├── security-reviewer.agent.md  ← 自訂安全審查代理人
    └── api-planner.agent.md        ← 自訂 API 規劃代理人
```

**檔案格式範例**（`.github/agents/security-reviewer.agent.md`）：

```yaml
---
name: 'Security Reviewer'
description: 針對安全漏洞審查程式碼（OWASP Top 10）
tools: ['search', 'read']  # 只允許唯讀工具，不修改程式碼
---
# 安全審查代理人
你是一位資安專家，負責審查程式碼的安全性。
專注於 OWASP Top 10 弱點，特別是 SQL Injection、XSS 與認證問題。
只提供建議，不直接修改程式碼。
```

> **💡 快速建立**：在 Agent 模式 Chat 中輸入 `/create-agent`，描述你想要的角色，AI 會自動產生 `.agent.md` 檔案。
> 或輸入 `/agents` 開啟 Configure Custom Agents 選單瀏覽現有代理人。

> **官方文件**：https://code.visualstudio.com/docs/copilot/customization/custom-agents

### 9.3 Skills（技能定義）

Skills 是預先撰寫的**領域知識包**，當使用者的問題符合技能描述時，Copilot 自動載入並依照其指引操作。無需明確呼叫指令，Copilot 依 `description` 自動比對並套用。

**建立步驟**：

1. 在 `.github/skills/{技能名稱}/` 目錄下建立 `SKILL.md`
2. 撰寫 frontmatter（`name` 與 `description`）
3. 在 body 填寫操作流程、完成條件與輸出格式
4. 存檔後 Copilot 即可自動識別並使用

**SKILL.md 文件結構**：

```yaml
---
name: my-skill
description: 一句話說明何時使用此技能（Copilot 依此判斷是否載入）
---

# My Skill

## 何時使用
- 適用情境 A
- 適用情境 B

## 工作流程
1. 步驟一
2. 步驟二

## 完成條件
- 條件 A
```

> **💡 放置位置與生效條件**：路徑 `.github/skills/{技能名稱}/SKILL.md`
> `description` 是觸發關鍵：描述越精確，Copilot 越能正確判斷是否載入。
> Skills 在 Copilot 收到相關任務時**自動讀取**，不需在 Chat 明確呼叫。

> **官方文件**：https://code.visualstudio.com/docs/copilot/customization/copilot-customization

### 9.4 MCP（Model Context Protocol）工具伺服器

讓 Copilot Agent 連接外部服務（資料庫、API、瀏覽器自動化等）。

**設定位置**：`.vscode/mcp.json`（建議加入 Git）

```json
{
    "servers": {
        "postgres": {
            "type": "stdio",
            "command": "npx",
            "args": ["-y", "@modelcontextprotocol/server-postgres"],
            "env": {
                "POSTGRES_CONNECTION_STRING": "postgresql://localhost/mydb"
            }
        },
        "playwright": {
            "type": "stdio",
            "command": "npx",
            "args": ["-y", "@playwright/mcp"]
        }
    }
}
```

**查看已設定的 MCP 伺服器**：
> 命令面板 → `MCP: List Servers`

> **官方文件**：https://code.visualstudio.com/docs/copilot/customization/mcp-servers

### 9.5 Hooks（鉤子）— 自動化觸發

在特定生命週期事件自動執行指令（如存檔時 lint、commit 前產生訊息）。

> **官方文件**：https://code.visualstudio.com/docs/copilot/customization/hooks

---

## 10. 快速命令與設定速查

### 10.1 常用命令面板指令（`Ctrl + Shift + P`）

| 指令 | 說明 |
|------|------|
| `Chat: Open Chat` | 開啟 Chat 面板 |
| `Chat: New Chat Session` | 開啟新的 Chat session |
| `Chat: Configure Instructions` | 管理自訂指令檔案 |
| `Chat: Open Customizations` | 開啟 Agent Customizations 編輯器 |
| `Chat: New Instructions File` | 建立新的指令檔案 |
| `Language Models: Manage Language Models` | 查看模型清單與配額 |
| `MCP: List Servers` | 查看已設定的 MCP 伺服器 |
| `Preferences: Open User Settings (JSON)` | 開啟 User 設定 JSON |
| `Preferences: Open Workspace Settings (JSON)` | 開啟 Workspace 設定 JSON |

### 10.2 推薦的 Workspace 設定（`.vscode/settings.json`）

```json
{
    // 啟用 copilot-instructions.md 等指令檔案
    "github.copilot.chat.codeGeneration.useInstructionFiles": true,

    // 啟用 Prompt Files 功能
    "chat.promptFiles": true,

    // 啟用 Next Edit Suggestions（預測下一步編輯位置）
    "github.copilot.nextEditSuggestions.enabled": true,

    // 依語言控制 Inline Suggestion 開關
    "github.copilot.enable": {
        "*": true,
        "markdown": true,
        "plaintext": false
    },

    // 預設權限層級（可選："default" / "bypassApprovals" / "autopilot"）
    "chat.permissions.default": "default"
}
```

---

## 11. 官方參考資源

> 📌 建議加入瀏覽器書籤

### VS Code 官方文件

| 資源 | 連結 |
|------|------|
| VS Code 官方文件首頁 | https://code.visualstudio.com/docs |
| 設定說明 | https://code.visualstudio.com/docs/getstarted/settings |
| 多根工作區說明 | https://code.visualstudio.com/docs/editor/multi-root-workspaces |
| 鍵盤快速鍵（Windows PDF）| https://code.visualstudio.com/shortcuts/keyboard-shortcuts-windows.pdf |

### GitHub Copilot in VS Code

| 資源 | 連結 |
|------|------|
| Copilot 概覽 | https://code.visualstudio.com/docs/copilot/overview |
| Chat 說明 | https://code.visualstudio.com/docs/copilot/chat/copilot-chat |
| Agents 概覽 | https://code.visualstudio.com/docs/copilot/agents/overview |
| 自訂指令說明 | https://code.visualstudio.com/docs/copilot/customization/custom-instructions |
| Prompt Files | https://code.visualstudio.com/docs/copilot/customization/prompt-files |
| Custom Agents | https://code.visualstudio.com/docs/copilot/customization/custom-agents |
| Skills（自訂技能） | https://code.visualstudio.com/docs/copilot/customization/copilot-customization |
| MCP 工具伺服器 | https://code.visualstudio.com/docs/copilot/customization/mcp-servers |
| Hooks | https://code.visualstudio.com/docs/copilot/customization/hooks |
| 設定完整參考 | https://code.visualstudio.com/docs/copilot/reference/copilot-settings |

### GitHub 官方文件

| 資源 | 連結 |
|------|------|
| GitHub Copilot 首頁 | https://docs.github.com/en/copilot |
| copilot-instructions.md 說明 | https://docs.github.com/en/copilot/customizing-copilot/adding-repository-custom-instructions-for-github-copilot |
| 訂閱方案比較 | https://docs.github.com/en/copilot/about-github-copilot/subscription-plans-for-github-copilot |
| 可用模型說明 | https://docs.github.com/en/copilot/using-github-copilot/ai-models/changing-the-ai-model-for-copilot-chat |

---

## 快速入門檢查清單

### 個人設定（每位成員各自完成）
- [ ] 確認 VS Code 版本為 1.119（`Help > About`）
- [ ] 點選 Activity Bar 的 Copilot 圖示，完成 GitHub 登入授權
- [ ] 確認 Status Bar 右下角 Copilot 圖示正常（無錯誤標示）
- [ ] 在任意 `.ts` 或 `.cs` 檔案測試 Inline Suggestion（按 `Tab` 接受）
- [ ] 按 `Ctrl + Alt + I` 開啟 Chat，嘗試 Ask Agent 提問

### 專案設定（Tech Lead 完成，加入 Git）
- [ ] 建立 `.github/copilot-instructions.md` 填入專案規範
- [ ] 在 `.vscode/settings.json` 加入 `useInstructionFiles: true`
- [ ] 建立 `.github/instructions/` 目錄與各語言/框架的指令檔案
- [ ] 建立 `.github/prompts/` 目錄，新增常用 prompt 範本
- [ ] 確認 `.vscode/settings.json` 和 `.github/` 已加入 Git

### 進階學習
- [ ] 嘗試 Plan Agent 處理一個新功能開發任務
- [ ] 嘗試 Agent + Default Approvals 自動修復一個 bug
- [ ] 閱讀官方 Agents 教學：https://code.visualstudio.com/docs/copilot/agents/agents-tutorial
