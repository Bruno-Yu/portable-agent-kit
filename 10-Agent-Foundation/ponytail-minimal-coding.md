---
title: Ponytail Minimal Coding
tags:
  - agent/principles
  - minimal-coding
source: https://github.com/DietrichGebert/ponytail
last_verified: 2026-08-19
status: public-review
---

# Ponytail Minimal Coding

Ponytail 用「最懶但可靠的資深工程師」視角，阻止 Agent 為簡單需求引入不必要的套件、包裝與抽象。

## 決策階梯

1. 這段程式真的需要存在嗎？
2. Codebase 已經有可重用的實作嗎？
3. 標準函式庫能完成嗎？
4. 平台原生功能能完成嗎？
5. 已安裝的相依套件能完成嗎？
6. 一行或少量程式能安全完成嗎？
7. 最後才新增最小必要實作。

> [!warning] 不等於 Code Golf
> 不得為了縮短程式碼而刪除驗證、錯誤處理、安全、可讀性或無障礙需求。

## 效能數字怎麼引用

官方 benchmark 有特定模型、repo、任務與樣本限制。公開筆記應連到原始測試，不把 stars 或效能數字寫成永久事實。

## 本 repo 的離線版本

使用 `30-Skills/ponytail-minimal-coding/SKILL.md`。本機版本只保留決策階梯與不可犧牲的安全邊界，是純 Markdown，不包含上游 plugin 的 Node.js lifecycle hooks、scripts 或 Ponytail MCP。

上游專案採 MIT License，但完整 plugin 具有可執行面；公司環境若只需要「少寫但可靠」的判斷方式，不應直接安裝整包 plugin。

## 來源

- [DietrichGebert/ponytail](https://github.com/DietrichGebert/ponytail)
