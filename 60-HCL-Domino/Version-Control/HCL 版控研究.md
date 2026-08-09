
- 公司下載路徑 : \\f12aoaf01\usrpub\Install\Notes12\R12.0.2_FP7

A running Domino server with the Domino REST API installed.
beginning with verifying that your system meets the required specifications,

- Domino 12 Server 是跑在 **Windows** 還是 **Linux** 環境呢?

- **HCL Domino REST API（前身為 Project KEEP）** 正式支援 **Domino 12.0.2 及以後**的版本。
- 支援平台包含：**Windows、Linux、macOS**。

### 主要功能

|功能|說明|
|---|---|
|**RESTful 標準**|遵循標準 REST 原則，支援 CRUD 操作|
|**身分驗證**|採用 **OAuth2** 認證機制|
|**存取控制**|可依使用者角色設定不同的存取權限|
|**安裝方式**|以 Add-on 方式安裝於 Domino Server 上|
|**多語言整合**|支援各種程式語言串接|

| 比較項目        | **HCL Domino REST API**  | **XPages 作為 API**              |
| ----------- | ------------------------ | ------------------------------ |
| **設計定位**    | 專門為 REST API 設計，現代化架構    | XPages 原本設計給伺服器渲染頁面，非原生 API    |
| **資料格式**    | 原生 JSON                  | 需自行在 XPages 撰寫 JSON 輸出邏輯       |
| **端點管理**    | 圖形化介面（Admin UI）設定 Schema | 每個 API 端點需手動建立 XPage           |
| **通訊協定**    | 標準 REST / HTTP           | HTTP，但需自行模擬 REST 行為            |
| **身分驗證**    | 內建 OAuth2 / JWT          | 依賴 Domino Session Cookie 或自行實作 |
| **CORS 支援** | 內建支援，易於設定                | 需手動在 HTTP Header 中設定           |


```
【XPages 當 API 的典型做法（舊方式）】

SPA (Vue/React)
    ↓ HTTP Request
XPage (手動輸出 JSON)
    ↓ Domino Java API / SSJS
Domino NSF Database

問題：
- XPage 引擎啟動開銷大
- JSON 輸出需手動組裝
- CORS / Token 需自行處理
- 難以維護與測試

```


```
```
【Domino REST API 的做法（現代方式）】

SPA (Vue/React)
    ↓ HTTP REST (JSON)
Domino REST API (KEEP)
    ↓ 原生整合
Domino NSF Database

優點：
- 標準 REST 介面
- 自動 JSON 序列化
- 內建 OAuth2 / Swagger
- Schema 管控資料安全
```


```

## 版控問題

**SVN** 全名是 **Apache Subversion**，是一套**集中式版本控制系統（Centralized VCS）**，比 Git 出現得早（2000年），曾經是業界主流的版控工具。

---

## SVN vs Git 核心差異

| 比較項目           | SVN（集中式）          | Git（分散式）                  |
| -------------- | ----------------- | ------------------------- |
| **架構**         | 所有人連到**同一台中央伺服器** | 每個人本地都有**完整的 Repository** |
| **Commit**     | 必須連線到伺服器才能 commit | 可以**離線 commit**，之後再 push  |
| **速度**         | 較慢（每次操作都要連線）      | 較快（本地操作為主）                |
| **分支（Branch）** | 建立分支成本高、速度慢       | 分支非常輕量、切換快速               |
| **歷史記錄**       | 只有中央伺服器有完整歷史      | 每個人本地都有完整歷史               |
| **學習曲線**       | 較平緩，概念簡單          | 稍陡，概念較多                   |
| **現今普及度**      | 逐漸被取代             | 現今業界主流 ✅                  |

---

## 為什麼說 Designer 對 Git 整合比 SVN 弱？

HCL Domino Designer 最初設計版控整合時，**SVN 才是主流**，所以：

- Designer 內建對 **SVN 有原生 plugin 支援**（基於 Eclipse Subversive / Subclipse）
- 可以直接在 Designer UI 裡做 SVN 的 commit、update、diff 等操作

而對於 **Git**：

- Designer 沒有原生的 Git UI 深度整合
- 通常做法是：**Designer 負責 ODP Sync（NSF ↔ 磁碟）**，然後在磁碟資料夾**另外開終端機或用 Git 工具**（Git CLI、SourceTree、VS Code）去操作
- 兩個工具是**分開**的，不像 SVN 那樣在 Designer 裡一站式完成


## SVN 對 Binary 檔案的版控能力

### SVN 可以追蹤 Binary 嗎？

**可以追蹤，但有限制：**

|能力|SVN|Git|
|---|---|---|
|**儲存 binary 檔案**|✅ 可以|✅ 可以|
|**追蹤版本變更歷史**|✅ 可以|✅ 可以|
|**diff 比較內容**|❌ 看不出差異（binary 就是 binary）|❌ 同左|
|**儲存方式**|每次變更儲存**完整檔案**（delta 壓縮）|同左（Git LFS 可改善）|
|**儲存空間**|binary 大檔會讓 repo 越來越肥|同左，但 Git LFS 可外掛解決|

> ⚠️ 不管 SVN 還是 Git，binary 檔案都**無法做有意義的 diff**，只能知道「這個版本有沒有變」，不能知道「改了什麼」。

### SVN 對 Binary 的小優勢

SVN 用的是**集中式 delta 儲存**，對於大型 binary 檔案，每個 client 不需要下載完整歷史，只需要 checkout 當前版本，**磁碟佔用比 Git 少**（Git 每個 clone 都有完整歷史）。

---

## 開源可自架的 SVN 方案

### 核心引擎（必裝）

|工具|說明|
|---|---|
|**Apache Subversion**|SVN 本體，完全開源免費 (Apache License)|

`# Ubuntu/Debian sudo apt install subversion  # CentOS/RHEL sudo yum install subversion`

---

### Web UI 自架方案（開源）

|工具|特色|推薦度|
|---|---|---|
|**VisualSVN Server** (Linux 版免費有限制)|簡單易裝，有 Web UI|⭐⭐⭐|
|**SVNAdmin**|純 Web UI 管理，PHP 寫的|⭐⭐⭐|
|**Redmine + SVN**|專案管理 + SVN 整合，功能完整|⭐⭐⭐⭐|
|**Apache httpd + mod_dav_svn**|官方標準做法，最穩定|⭐⭐⭐⭐⭐|
|**Gitea（不支援 SVN）**|❌ 僅 Git|-|

---

### 最推薦：Apache httpd + mod_dav_svn

完全開源、免費、穩定，架設方式：

`┌─────────────────────────────────┐ │         Apache httpd            │ │       + mod_dav_svn             │  ← Web 存取 SVN │       + mod_authz_svn           │  ← 權限控管 └────────────┬────────────────────┘              │ ┌────────────▼────────────────────┐ │      SVN Repository             │  ← 放在伺服器磁碟 │  /srv/svn/domino-odp/           │ └─────────────────────────────────┘`

`# 安裝 sudo apt install apache2 subversion libapache2-mod-svn  # 建立 repo sudo svnadmin create /srv/svn/domino-odp  # Apache 設定 <Location /svn/domino-odp>   DAV svn   SVNPath /srv/svn/domino-odp   AuthType Basic   AuthName "SVN Repository"   AuthUserFile /etc/svn/passwd   Require valid-user </Location>`


## Git LFS（Large File Storage）

---

### 一句話說明

> **Git LFS 是 Git 的擴充套件，專門用來處理大型 binary 檔案，讓 Git repo 不會因為 binary 檔案而爆肥。**

---

### 為什麼需要 Git LFS？

先理解 Git 的原始設計問題：

`一般 Git 的行為：  第1次 commit：  app.nsf (50MB)  → repo 累積 50MB 第2次 commit：  app.nsf (51MB)  → repo 累積 101MB 第3次 commit：  app.nsf (52MB)  → repo 累積 153MB ... 第20次 commit：                 → repo 累積 1GB+ 😱  每個人 git clone 都要下載這 1GB！`

---

### Git LFS 怎麼解決這個問題？

`Git LFS 的行為：  Git repo 裡只存一個「指標檔案」(pointer)： ┌─────────────────────────────┐ │ version https://git-lfs...  │  ← 只有幾十 Byte │ oid sha256:a3b4c5d6...      │  ← 指向真正的檔案 │ size 52428800               │ └─────────────────────────────┘          │          │ 實際的大檔存在這裡          ▼ ┌─────────────────────────────┐ │     LFS Storage Server      │  ← 獨立的 binary 儲存空間 │     app.nsf (真正的檔案)     │ └─────────────────────────────┘`

---

### 使用方式很簡單

`# 1. 安裝 Git LFS git lfs install  # 2. 指定哪些檔案用 LFS 追蹤 git lfs track "*.nsf" git lfs track "*.jar" git lfs track "*.ntf"  # 3. 把設定檔 commit 進去 git add .gitattributes git commit -m "Add LFS tracking"  # 之後 git add / commit / push 跟平常完全一樣！ git add app.nsf git commit -m "Update NSF" git push   ← 自動把大檔送到 LFS Server`

---

### Git LFS 架構圖

`開發者 A                    Git Server          LFS Server    │                            │                   │    │── git push ──────────────► │                   │    │   ├── 程式碼 (文字)  ──────►│ (存在 Git repo)   │    │   └── app.nsf (binary) ───────────────────────►│ (存在 LFS)    │                            │                   │ 開發者 B                        │                   │    │── git clone ─────────────► │                   │    │   ├── 拿到程式碼 ◄──────── │                   │    │   └── 拿到 pointer ◄────── │                   │    │── 需要 NSF 時 ─────────────────────────────────►│    │   └── 下載真正的 NSF ◄──────────────────────── │`

---

### Git LFS 開源自架方案

| 方案                  | 說明                    | 推薦度   |
| ------------------- | --------------------- | ----- |
| **Gitea**           | 內建支援 LFS，最容易自架        | ⭐⭐⭐⭐⭐ |
| **Forgejo**         | Gitea 的 fork，同樣內建 LFS | ⭐⭐⭐⭐⭐ |
| **GitLab CE**       | 完整功能，內建 LFS，但較重       | ⭐⭐⭐⭐  |
| **Gogs**            | 輕量，但 LFS 支援較弱         | ⭐⭐⭐   |
| **lfs-test-server** | 官方測試用，不建議 production  | ❌     |
