
## 專案初始化

在 Codex CLI 的專案目錄執行：

```text
/init
```

它會在目前目錄產生 `AGENTS.md` scaffold，作用類似其他 Coding Agent 的專案初始化命令。產生後仍需依 README、manifest、CI 與現有設定補上真實 build/test 指令、coding style、架構邊界與 Definition of Done；完整流程見 [[codex-project-bootstrap|Codex 專案初始化與接入指南]]。

官方說明：[OpenAI Docs：Custom instructions with AGENTS.md](https://learn.chatgpt.com/docs/agent-configuration/agents-md)

``` bash
# 對話控制

/goal <text> # 設定/更新本次工作目標的指令，以這個目標作為優先準則，常用在: 開始一個任務、途中需求變更、想約束修改範圍時

/clear # 清空目前對話/上下文（重新來一輪）

/history # 回顧你或工具做過什麼決策、找回先前的指令或關鍵輸出

/help # 顯示所有斜線指令與用法

# 上下文

/add <path|glob> #把檔案加入上下文

/drop <path> #移出上下文

# 檢視檔案內容

/open <file> #顯示檔案內容

# 變更流程

/plan #輸出修改計劃/步驟

/diff #顯示建議的變更差異

/apply #套用變更到工作區

/undo 或 /revert # 測回最近一次套用/回到上一個狀態

```

- 想用 Codex 的 /goal 功能，但不知道怎麼開？
	1. 先確保你的 Codex 是最新版本 (v0.128.0)。
	2. 打開 config.toml 這個設定檔。
	3. 在 [features] 區塊裡，加入這行：`goals = true`
	4. 存檔後重啟 Codex。
	5. 現在你就可以輸入 /goal 了。

- GPT 產圖
	- 走 API ($0.19/張 / 速度穩 / 完全控制 prompt) 合適大批量精準
	-  走 ChatGPT 訂閱（codex exec + $imagegen）訂閱內 0 邊際 / 內建 image_gen tool (可藉由 cc 或其他 ai 進行呼叫)
		``` bash
		codex exec --full-auto --skip-git-repo-check --cd /tmp '...$imagegen'
		```
