---
version: alpha
name: Example Product
description: Replace these example tokens with values verified from the product or approved design source.
colors:
  primary: "#1A1C1E"
  secondary: "#5F6368"
  accent: "#9F3526"
  surface: "#F7F5F2"
  on-surface: "#1A1C1E"
  error: "#B3261E"
typography:
  headline-lg:
    fontFamily: Public Sans
    fontSize: 2rem
    fontWeight: 600
    lineHeight: 1.2
    letterSpacing: -0.02em
  body-md:
    fontFamily: Public Sans
    fontSize: 1rem
    fontWeight: 400
    lineHeight: 1.6
  label-md:
    fontFamily: Public Sans
    fontSize: 0.875rem
    fontWeight: 600
    lineHeight: 1.2
rounded:
  sm: 4px
  md: 8px
  full: 9999px
spacing:
  xs: 4px
  sm: 8px
  md: 16px
  lg: 24px
  xl: 32px
components:
  button-primary:
    backgroundColor: "{colors.accent}"
    textColor: "#FFFFFF"
    typography: "{typography.label-md}"
    rounded: "{rounded.md}"
    padding: 12px
  card:
    backgroundColor: "{colors.surface}"
    textColor: "{colors.on-surface}"
    rounded: "{rounded.md}"
    padding: "{spacing.lg}"
---

# Example Product Design System

> [!warning]
> 這是結構範例，不是可直接套用的品牌。複製後必須用現有產品、核准設計稿或設計 token 替換範例值，並刪除這段提示。

## Overview

描述品牌個性、主要受眾、資訊密度與畫面應帶來的感受。當 token 沒有涵蓋某個決策時，本節提供判斷方向。

## Colors

- **Primary:** 用於標題與核心文字，提供最高閱讀對比。
- **Secondary:** 用於次要文字、邊框與 metadata。
- **Accent:** 每個畫面只用於最重要的操作或品牌焦點。
- **Surface／On Surface:** 成對使用，所有組合必須符合專案的 accessibility 標準。
- **Error:** 只用於錯誤狀態，不以顏色作為唯一提示。

## Typography

定義 heading、body、label 的角色，不要依畫面臨時新增字級。內容層級由語意與 token 決定，不用任意放大文字製造重要性。

## Layout

使用 spacing tokens 建立一致節奏。說明 mobile／desktop grid、最大內容寬度、gutter、safe area 與 breakpoint 策略。

## Elevation & Depth

說明使用 border、tonal layer 或 shadow 表達層級的方式。避免同一層級混用多套陰影。

## Shapes

說明 corner radius、icon stroke 與容器形狀。相同功能的元件應沿用相同 shape token。

## Components

- **Buttons:** 定義 primary、secondary、disabled、hover、focus 與 loading 狀態。
- **Inputs:** 定義 label、helper、error、focus 與 disabled 狀態。
- **Cards:** 定義 padding、border／elevation 與可互動狀態。
- **Navigation:** 定義 active state、層級與 responsive behavior。

## Do's and Don'ts

- Do 重用 tokens 與既有元件，並驗證 normal、hover、focus、disabled 與 error 狀態。
- Do 檢查鍵盤操作、focus visibility、文字縮放與對比度。
- Don't 在元件中加入未記錄的 arbitrary color、font size、spacing 或 radius。
- Don't 用顏色作為狀態的唯一訊號。
- Don't 在沒有核准來源時猜測 logo、字型或品牌資產。
