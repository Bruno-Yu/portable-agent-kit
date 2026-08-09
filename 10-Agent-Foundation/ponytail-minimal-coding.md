---
title: Ponytail Minimal Coding
tags:
  - agent/principles
  - minimal-coding
source: https://github.com/DietrichGebert/ponytail
last_verified: 2026-08-09
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

## 來源

- [DietrichGebert/ponytail](https://github.com/DietrichGebert/ponytail)
