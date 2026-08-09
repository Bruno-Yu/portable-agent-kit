---
title: Portable Agent Memory Architecture
tags:
  - agent/memory
  - context-engineering
status: public-review
---

# Portable Agent Memory Architecture

> [!abstract]
> 常駐越小越好；專案資訊進入專案才讀；任務參考按需載入；個人記憶永不進 public repo。

## 先分清四類資料

| 類型 | 內容 | 公開策略 |
|---|---|---|
| Instructions | Agent 應如何工作 | 通常可公開 |
| Project memory | 架構決策、慣例、限制 | 視專案而定 |
| Working state | 當前任務、待辦、進度 | 預設本機私有 |
| Personal memory | 身份、偏好、工作與生活資訊 | 不公開 |

## 三層載入

```text
對話啟動
  → 第一層：AGENTS.md + MEMORY.md 路由
    → 第二層：當前專案的 decisions / constraints
      → 第三層：任務需要時才讀 SOP、規格與參考資料
```

## 建議目錄

```text
AGENTS.md
memory/
├── MEMORY.md
├── project-decisions.md
├── references/
└── private/
    ├── user-profile.local.md
    ├── PENDING.local.md
    └── session-notes/
```

## 是否需要 Memory MCP

Markdown 分層先解決大多數需求。只有出現跨大量專案、語意檢索或多 Agent 共用的明確需求，才評估記憶 MCP，並先檢查資料保存位置、認證與刪除機制。
