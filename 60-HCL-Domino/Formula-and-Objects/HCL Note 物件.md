
## 📐 整體概念：Back-end vs. Front-end（後端 vs. 前端）

Notes 的物件模型分為兩個世界：

| 類型                     | 特色                              | 適用情境             |
| ---------------------- | ------------------------------- | ---------------- |
| **Back-end 物件**（無 UI）  | 直接操作資料庫、文件的資料，不需要開啟畫面           | 批次處理、Agent、背景作業  |
| **Front-end 物件**（含 UI） | 操作「使用者正在看的畫面」，與 Notes Client 互動 | 表單事件、按鈕、即時 UI 操作 |

> 名稱中有 **`UI`** 的（如 `NotesUIDocument`）就是 **Front-end 物件**，代表「畫面上可見的那一層」。

---

## 🌳 階層結構總覽

```
NotesSession                     ← 最頂層（後端入口）
│
├── NotesDatabase
│   ├── NotesView               ← 檢視（後端）
│   │   └── NotesDocument       ← 文件（後端）
│   │       └── NotesItem       ← 欄位資料
│   └── NotesForm               ← 表單設計（後端）
│
└── NotesWorkspace（後端，較少用）


NotesUIWorkspace                 ← 最頂層（前端入口）
│
└── NotesUIView                 ← 當前開啟的檢視（前端）
│
└── NotesUIDocument             ← 當前開啟的文件（前端）
    └── NotesUIField            ← 欄位的 UI 操作


```


---

## 🔍 各物件逐一說明

### 1. `NotesSession`（後端）

- **最頂層的後端入口點**
- 代表目前的 Notes 工作階段（Session）
- 可以取得：目前登入使用者、伺服器名稱、環境變數、開啟資料庫等
- 典型用法：`Dim s As New NotesSession`

---

### 2. `NotesWorkspace`（後端）

- 代表 Notes Client 的**工作台（Workspace）**
- 後端版本，功能較少，主要用來取得目前開啟的資料庫清單
- ⚠️ 在現代 Notes 中較少使用，多數場合已被 `NotesUIWorkspace` 取代

---

### 3. `NotesUIWorkspace`（**前端，有 UI**）

- **前端的最頂層入口**
- 可以操作 Notes Client 畫面，例如：
    - 開啟/關閉文件視窗
    - 取得目前畫面上開啟的文件（`CurrentDocument`）
    - 彈出對話框（`Prompt`）
- 典型用法：`Dim ws As New NotesUIWorkspace`

---

### 4. `NotesForm`（後端）

- 代表資料庫中**表單的設計定義**（不是執行中的畫面）
- 可以讀取表單名稱、欄位清單、別名等設計資訊
- ⚠️ 這是靜態設計物件，不能直接操作使用者填寫中的表單

---

### 5. `NotesView`（後端）

- 代表資料庫中的**檢視（View）或資料夾**
- 可以：查詢文件、取得欄位值、排序、全文搜尋等
- 適合做批次資料處理

---

### 6. `NotesUIView`（**前端，有 UI**）

- 代表使用者**畫面上正在看的那個 View**
- 可以取得選取的文件、重新整理畫面、操作 UI 狀態

---

### 7. `NotesDocument`（後端）

- 代表資料庫中**某一份文件的資料**
- 可以讀寫欄位值（`NotesItem`）、儲存、刪除、傳送郵件
- 不需要開啟 Notes Client 畫面即可操作

---

### 8. `NotesUIDocument`（**前端，有 UI**）

- 代表使用者**正在畫面上開啟、編輯中的文件**
- 可以做到：
    - 讀寫目前畫面的欄位值（即時反映在 UI 上）
    - 切換至編輯/閱讀模式
    - 觸發 Refresh、存檔等動作
- 可透過 `.Document` 屬性取得對應的後端 `NotesDocument`

---

## 💡 有無「UI」的關鍵差異對照

|後端（Back-end）|前端（Front-end）|差異重點|
|---|---|---|
|`NotesWorkspace`|`NotesUIWorkspace`|是否操作 Client 畫面|
|`NotesView`|`NotesUIView`|是否是使用者正在看的 View|
|`NotesDocument`|`NotesUIDocument`|是否是使用者正在開啟的文件|

> **規律**：`Notes` + `UI` + `物件名` = 前端物件，操作的是「使用者看到的畫面層」；沒有 `UI` 的 = 後端物件，操作的是「資料層」。

---

## 🧭 實際使用情境建議

|情境|建議使用|
|---|---|
|Agent 批次更新大量文件|`NotesSession` → `NotesDatabase` → `NotesDocument`|
|按鈕點擊後即時改變欄位顯示|`NotesUIWorkspace` → `NotesUIDocument`|
|取得目前編輯文件的資料再寫入DB|`NotesUIDocument.Document`（前後端橋接）|
|讀取 View 中的所有文件做報表|`NotesView.GetAllDocumentsByKey()`|

這樣的設計讓開發者能清楚區分「**操作資料**」和「**操作畫面**」，避免在背景 Agent 中誤觸 UI，或在前端事件中做了不必要的 DB 掃描。