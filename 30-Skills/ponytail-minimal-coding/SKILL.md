---
name: ponytail-minimal-coding
description: 在 coding、修 bug、重構、review、選套件或 API 設計時，優先採用已存在、標準函式庫或平台原生的最小可靠方案，避免過度工程。不要用於一般寫作；也不可用來省略安全、資料保護、必要錯誤處理、測試或 accessibility。
license: MIT
---

# Ponytail Minimal Coding

像熟悉 codebase 的資深工程師一樣節省：先理解真正流程，再選擇能可靠完成需求、維護成本最低的方案。Lazy 是減少不必要實作，不是減少理解與驗證。

## Decision ladder

依序判斷，第一個足以可靠完成需求的選項就是停止點：

1. 需求是否真的需要存在？若只是推測的未來需求，不實作並簡短說明。
2. Codebase 是否已有 helper、元件、型別或 pattern？先重用。
3. 標準函式庫是否足夠？
4. 瀏覽器、資料庫、作業系統或 framework 的原生功能是否足夠？
5. 已安裝的 dependency 是否足夠？不要為少量程式碼新增套件。
6. 少量直接程式碼是否比新抽象更清楚？
7. 以上皆不足時，才建立最小必要實作。

## Working rules

- 修 bug 要找共同根因與完整 caller flow；只在症狀點加 patch 可能產生更多程式與更多 bug。
- 不建立只有一個 implementation 的 interface、只包一個產品的 factory，或沒有實際變化需求的 config。
- 兩個方案同樣小時，選 edge cases 較正確、較符合現有慣例者。
- 已知會成為瓶頸的刻意簡化，要註明限制與何時升級；不要假裝限制不存在。
- 使用者要求完整版本、報告或 walkthrough 時照需求提供，不用「極簡」拒絕明確 scope。

## Never cut

不可省略 trust boundary 的輸入驗證、防止資料遺失的錯誤處理、安全控制、accessibility、法規要求或使用者明確指定的行為。非瑣碎邏輯保留能在邏輯破壞時失敗的最小驗證。

## Output

回報採用的最小方案、刻意未加入的部分，以及什麼具體條件出現時才值得升級。不要用冗長設計辯護重新製造複雜度。

## Source and license

本 Skill 是 [DietrichGebert/ponytail](https://github.com/DietrichGebert/ponytail/tree/2ed6c52c9d7e5e56942508591085fd45dea277d3) 的繁體中文、純 Markdown 適配，依其 [MIT License](LICENSE.md) 使用。未包含上游 hooks、scripts、MCP 或 plugin metadata。
