---
title: Recommended MCP Strategy
tags:
  - agent/mcp
  - security
last_verified: 2026-08-09
status: public-review
---

# Recommended MCP Strategy

## 選擇順序

```text
Agent 內建能力 → CLI → Skill → MCP
```

只有需要標準化外部整合、權限邊界或跨 client 重用時才加入 MCP。

## 建議 Profile

| Profile | 建議 |
|---|---|
| Minimal | 不裝 MCP |
| Browser debug | [Chrome DevTools MCP](https://github.com/ChromeDevTools/chrome-devtools-mcp) |
| Browser workflow | [Playwright MCP](https://github.com/microsoft/playwright-mcp) |
| Large codebase | 有明確效益後才評估 CodeGraph／語意索引 |
| Memory | Markdown 分層不足後才評估，預設不裝 |

Chrome DevTools MCP 與 Playwright MCP 通常二選一，不應因「可能有用」同時常駐。

## 現行傳輸

MCP 標準傳輸是：

- `stdio`
- Streamable HTTP

舊 HTTP+SSE 僅作相容，不應再列為新設定的預設。參考 [MCP Transport specification](https://modelcontextprotocol.io/specification/2025-06-18/basic/transports)。

## 安全基線

- Token 只從環境變數或系統秘密儲存載入
- 本機服務只綁 `127.0.0.1`，不要綁 `0.0.0.0`
- 遠端 Streamable HTTP 必須驗證 Origin 並使用認證
- 專案 `.mcp.json` 只能提交無秘密的模板
- 明確限制檔案系統、瀏覽器與帳號權限
