

> 原始影片：[HCL Domino REST API](https://www.youtube.com/watch?v=gj3JfQu5VWk)
> 發布者：HCL Digital Solutions Taiwan
> 片長：1:01:31
> 講者：劉小平
> 整理依據：以 Whisper Small 產生含時間碼逐字稿，再以 720p 影片畫面人工交叉校對。影片沒有內建字幕或說明文字。

> [!note] 原影片畫面未收錄
> 「HCL Domino REST API 基礎入門」畫面因可能含講者肖像、介面資料或憑證範例，未納入公開版。

## 先讀這段

這是一支約三年前、以 **Domino REST API 1.0** 為基準的教學影片。本文忠實整理影片中實際出現的流程，不代表目前最新版 HCL Domino REST API 的安裝參數、端點與 UI。正式環境操作前，應再與目前版本的 HCL 官方文件核對。

畫面裡過小、語音辨識不確定，或講者沒有完整展開的 request body／欄位值，本文不自行補造，而以「待確認」標示。

## 一分鐘摘要

Domino REST API 是獨立於舊有 Domino Access Services（DAS）的新 REST 介面。它讓支援 HTTP／REST 的前端、伺服器程式、低程式碼平台與第三方工具存取 Domino 資料，同時仍受 Domino ACL 與安全模型限制。

影片的核心工作流是：

1. 在 Domino Server 或 Notes Client 安裝 Domino REST API。
2. 選擇一個 NSF 資料庫。
3. 建立 **Schema**，決定可公開的表單、視圖、欄位、Agent，以及讀寫權限。
4. 建立並啟用 **Scope**，讓外部程式以指定名稱呼叫該 Schema。
5. 先認證並取得 Token，再透過 Base、Admin、PIM 或 Setup API 操作。
6. 用 Postman 驗證，再整合到實際程式或 Node-RED 等平台。

> [!note] 原影片畫面未收錄
> 「從安裝到開發的整體流程」畫面因可能含講者肖像、介面資料或憑證範例，未納入公開版。

## 章節索引

| 時間 | 內容 | 證據類型 |
|---|---|---|
| [00:00](https://www.youtube.com/watch?v=gj3JfQu5VWk&t=0s) | Domino REST API 1.0 開場 | 概念 |
| [01:41](https://www.youtube.com/watch?v=gj3JfQu5VWk&t=101s) | REST API 與 Domino 安全模型 | 概念 |
| [04:50](https://www.youtube.com/watch?v=gj3JfQu5VWk&t=290s) | 支援的應用類型、OData 與低程式碼平台 | 概念 |
| [07:30](https://www.youtube.com/watch?v=gj3JfQu5VWk&t=450s) | Schema／Scope 的使用流程 | 概念 |
| [11:37](https://www.youtube.com/watch?v=gj3JfQu5VWk&t=697s) | Server、Client 與 Docker 部署選項 | 說明 |
| [12:17](https://www.youtube.com/watch?v=gj3JfQu5VWk&t=737s) | 下載與安裝 | 示範／投影片 |
| [16:14](https://www.youtube.com/watch?v=gj3JfQu5VWk&t=974s) | 啟動與確認安裝 | 示範 |
| [17:59](https://www.youtube.com/watch?v=gj3JfQu5VWk&t=1079s) | 建立 Schema 與 Scope | 示範 |
| [21:50](https://www.youtube.com/watch?v=gj3JfQu5VWk&t=1310s) | 管理埠與功能帳號 | 示範 |
| [29:30](https://www.youtube.com/watch?v=gj3JfQu5VWk&t=1770s) | 四組 OpenAPI | 示範 |
| [32:54](https://www.youtube.com/watch?v=gj3JfQu5VWk&t=1974s) | 官方快速入門與三個案例 | 說明 |
| [35:14](https://www.youtube.com/watch?v=gj3JfQu5VWk&t=2114s) | 案例一：既有 Todo 資料庫 | 示範 |
| [42:29](https://www.youtube.com/watch?v=gj3JfQu5VWk&t=2549s) | 案例二：從零建立資料庫 | 示範 |
| [50:30](https://www.youtube.com/watch?v=gj3JfQu5VWk&t=3030s) | 案例三：Node-RED／Collabsphere | 示範 |
| [57:20](https://www.youtube.com/watch?v=gj3JfQu5VWk&t=3440s) | 除錯、重新登入與最終地圖 | 示範 |

## 核心概念

### Domino REST API 與 DAS 不相同

講者表示 Domino REST API 隨 Domino 12.0.2 推出，是一套全新的 API，而不是在舊 Domino Access Services 上直接修改。它提供較完整的設定與安全控制。

### Schema：決定公開什麼

Schema 用來描述 NSF 中哪些設計元素與資料可以經由 REST API 對外提供，例如：

- 表單（Form）
- 視圖／清單（View／List）
- 欄位（Field）
- Agent
- 欄位型別
- 各欄位的讀取與寫入權限
- DQL、OData 等模式

它不是把整個資料庫無條件公開，而是建立一層可控的公開模型。

### Scope：讓外部程式取得使用範圍

外部程式呼叫 Schema 時使用 Scope 名稱。Scope 必須啟用（Active）後，才算正式發布供外部程式使用。

### 認證與 Token

呼叫資料 API 前，先以 authentication endpoint 登入取得 Token。後續 request 攜帶該 Token。影片的 Postman 範例把 Token 儲存為 Collection 變數，避免每個 request 手動貼上。

### 四組 API

> [!note] 原影片畫面未收錄
> 「OpenAPI 中的 API 分組」畫面因可能含講者肖像、介面資料或憑證範例，未納入公開版。

| API | 影片中的用途 |
|---|---|
| Base | 文件與資料的 CRUD、DQL、OData、認證等基礎操作 |
| Admin | ACL、資料庫權限、使用者與群組等管理操作 |
| PIM | 郵件、行事曆、待辦事項、聯絡人等個人資訊管理 |
| Setup | 建立資料庫、表單、視圖與其他設計元素 |

## 安裝與初始設定

### 前置條件

- 可用的 HCL Domino Server；開發測試也可裝在 Notes Client。
- 對 Domino 程式、Data 目錄與 `notes.ini` 的存取權。
- Java 執行環境。
- Domino REST API 安裝包。
- 後續實驗需要 Postman；第三個實驗另需 Docker 與 Node-RED。

影片也提到 Docker 版下載選項，但沒有在本片實際完成 Docker 版 Domino REST API 部署。

### 步驟 1：執行安裝器

來源：[12:40–14:48](https://www.youtube.com/watch?v=gj3JfQu5VWk&t=760s)
證據：投影片說明；影片未從空白環境完整重跑一次安裝。

> [!note] 原影片畫面未收錄
> 「Windows 安裝命令與參數」畫面因可能含講者肖像、介面資料或憑證範例，未納入公開版。

影片的 Windows 範例：

```bat
java -jar restapiInstall.jar ^
  -d="C:\Program Files\HCL\Domino\Data" ^
  -i="C:\Program Files\HCL\Domino\notes.ini" ^
  -p="C:\Program Files\HCL\Domino" ^
  -k="C:\Program Files\HCL\Keep" ^
  -a
```

也可以把參數放入 response file：

```bat
java -jar restapiInstaller.jar @responses.txt
```

投影片中的 response file 範例使用：

```text
--dataDir=C:\HCL\Domino\data
--ini=C:\HCL\Domino\notes.ini
--restapiDir=C:\HCL\restapi
--programDir=C:\HCL\Domino
--accept
```

> 注意：投影片兩處檔名分別顯示 `restapiInstall.jar` 與 `restapiInstaller.jar`。這是不一致的來源證據；請以實際下載包的檔名及當前官方文件為準。

參數概念：

- `dataDir`：Domino／Notes Data 目錄。
- `ini`：`notes.ini`。
- `programDir`：Domino／Notes 程式目錄。
- `restapiDir`／`-k`：Domino REST API 安裝目錄；舊名稱 Keep 仍會出現在檔名與目錄中。
- `accept`／`-a`：接受條款。

### 步驟 2：確認安裝器做了哪些變更

來源：[14:58–16:14](https://www.youtube.com/watch?v=gj3JfQu5VWk&t=898s)

> [!note] 原影片畫面未收錄
> 「安裝完成後的檔案與設定變更」畫面因可能含講者肖像、介面資料或憑證範例，未納入公開版。

投影片列出：

1. 建立 Domino REST API 目錄。
2. 複製所需的二進位檔。
3. Windows 將 `nrestapi.exe` 複製到 Domino 目錄；Linux 檔名在影片語音中不夠清楚，需查當前文件。
4. 更新 `notes.ini` 的 `ServerTasks`，加入 `restapi`。
5. 新增 `KeepInstallDir`，指向 Domino REST API 目錄。
6. 在 Domino Data 目錄新增 `keepconfig.d` 與 `keepweb.d`。

講者另外提到會加入兩個 Keep 資料庫，但本段沒有列出精確檔名。

### 步驟 3：啟動並開啟設定首頁

來源：[15:26–18:12](https://www.youtube.com/watch?v=gj3JfQu5VWk&t=926s)
證據：示範。

1. 如果尚未由 `ServerTasks` 自動啟動，可在 Domino console 執行：

   ```text
   load restapi
   ```

2. 以 console 狀態／資訊命令確認 REST API task 已啟動。影片畫面中的完整命令字串不夠清楚。
3. 開啟：

   ```text
   http://localhost:8880/
   ```

4. 進入 Configuration，以 Domino 管理者帳號登入。安裝程式會把 Domino 管理者設為 REST API 管理者。

### 步驟 4：建立 Schema

來源：[18:31–20:34](https://www.youtube.com/watch?v=gj3JfQu5VWk&t=1111s)
證據：示範。

> [!note] 原影片畫面未收錄
> 「新增 Schema 並選擇資料庫」畫面因可能含講者肖像、介面資料或憑證範例，未納入公開版。

1. 在 Schema Management 選擇 **Add New Schema**。
2. 輸入名稱；影片範例為 `classdemo`。
3. 選擇目標資料庫；範例使用 Todo 資料庫。
4. 加入要公開的 Form。
5. 選擇要 expose 的欄位。
6. 為欄位設定讀、寫權限；型別會從 Form 定義讀取。
7. 視需要加入 DQL／OData mode。
8. 儲存。

### 步驟 5：建立並啟用 Scope

來源：[20:34–21:31](https://www.youtube.com/watch?v=gj3JfQu5VWk&t=1234s)
證據：示範。

> [!note] 原影片畫面未收錄
> 「新增並啟用 Scope」畫面因可能含講者肖像、介面資料或憑證範例，未納入公開版。

1. 建立 Scope。
2. 指向剛建立的 Schema。
3. 輸入外部程式使用的 Scope 名稱；範例同樣使用 `classdemo`。
4. 將 Scope 設為 Active。
5. 儲存並確認它出現在可用 Scope 清單。

預期結果：外部程式可以 Scope 名稱呼叫該 Schema 所公開的功能。

## 功能帳號與管理介面

來源：[21:50–29:12](https://www.youtube.com/watch?v=gj3JfQu5VWk&t=1310s)

影片區分：

- `8880`：Domino 開發／設定介面，與 Domino Directory 整合。
- `8889`：管理介面，用於日誌、狀態與設定等；以功能帳號登入。
- 另外還展示 health check、Notes Client 與 metrics 等服務埠，但精確部署仍應依當前設定檔確認。

功能帳號透過放在 `keepconfig.d` 的 JSON 檔定義。管理介面可產生 salted password：

> [!note] 原影片畫面未收錄
> 「管理介面產生 salted password」畫面因可能含講者肖像、介面資料或憑證範例，未納入公開版。

影片的重要提醒：

- Salted password 是單向用途；相同密碼多次產生的字串可能不同。
- 可以驗證某個明文是否符合 hash，但不能從 hash 反向還原密碼。
- JSON 變更後需要重新啟動 Domino REST API。
- 不要照抄影片中的示例密碼或 hash 到正式環境。

## Postman 共通設定

來源：[34:01–39:37](https://www.youtube.com/watch?v=gj3JfQu5VWk&t=2041s)

> [!note] 原影片畫面未收錄
> 「影片建議先用 Postman 完成官方實驗」畫面因可能含講者肖像、介面資料或憑證範例，未納入公開版。

1. 安裝 Postman。
2. 每個實驗建立一個 Collection。
3. 在 Collection 中建立 requests。
4. 建立 Collection 變數。影片第一個實驗包括：

   - `host`：Base API 主機／路徑。
   - `adminhost`：Admin API 主機／路徑。
   - `bearer`：認證後的 Token。
   - `unid`：建立文件後得到的文件 ID。

> [!note] 原影片畫面未收錄
> 「Postman Collection 與變數」畫面因可能含講者肖像、介面資料或憑證範例，未納入公開版。

5. 建立 authentication request，Body 放入登入資訊。
6. 在 Postman Tests 中，把 response 取得的 Token 寫入 `bearer` Collection 變數。
7. 後續 request 使用該 Token。

> 影片沒有把所有 endpoint 與 Tests script 放大到可可靠抄錄的程度。應直接從該版本官方 tutorial 匯入或依 OpenAPI 建立 request，不要依本片截圖手抄。

## 實驗一：操作既有 Todo 資料庫

來源：[35:14–42:17](https://www.youtube.com/watch?v=gj3JfQu5VWk&t=2114s)
證據：主要步驟均有示範。

### 準備

1. 從影片指定的 OpenNTF／XPages Extension Library 資源取得 Todo 範例資料庫。
2. 將 Todo NSF 放到 Domino Server。
3. 確認資料庫包含一個 Todo Form 與數個 Views。
4. 依前述流程建立對應 Schema 與 Scope。

影片裡顯示的精確套件版本與下載網址過小，本文不推測。

### 執行順序

1. 認證並把 Token 存入 `bearer`。
2. 使用 Admin API 讀取 Todo 資料庫 ACL，確認回傳包含 default 權限、角色等資訊。
3. 使用 Base API 的 document request，傳入 JSON 建立 Todo 文件。
4. 保存回傳的 UNID。
5. 回 Domino Client／Administrator 開啟資料庫，確認文件存在且狀態為 incomplete。
6. 以 API 更新同一文件，把 complete 狀態改為 true。
7. 重新整理 Domino 視圖，確認文件狀態已變為 complete。

> [!note] 原影片畫面未收錄
> 「由 REST API 建立的 Todo 文件出現在 Domino 中」畫面因可能含講者肖像、介面資料或憑證範例，未納入公開版。

8. 測試讀取 View 全部文件與特定 Todo。
9. 測試 DQL 查詢，例如查詢 Form 為 Todo 且狀態為完成的文件。

注意：Schema 必須存在 `dql` mode 才能執行 DQL；要使用 OData，也需要對應的 `odata` mode。這些 mode 名稱由系統定義。

## 實驗二：從零建立 Domino 資料庫

來源：[42:29–50:20](https://www.youtube.com/watch?v=gj3JfQu5VWk&t=2549s)
證據：示範，但官方 tutorial 的自動 Schema 內容在影片版本有錯誤。

### 執行順序

1. 認證並保存 Token。
2. 使用 Setup API 建立 `customer` 資料庫。
3. 在 Domino Administrator 重新整理並確認新 NSF 已出現；剛建立時約 300 KB 且沒有內容。

> [!note] 原影片畫面未收錄
> 「Setup API 建立的新資料庫」畫面因可能含講者肖像、介面資料或憑證範例，未納入公開版。

4. 在建立 Form／View 前先建立 Schema；Schema 會存於該資料庫的設計資源中。
5. 取得 Schema，確認欄位與要公開的元素。
6. 建立並啟用 Scope。
7. 使用 JSON 描述欄位名稱與型別，透過 Setup API 建立 `Customer`、`Contact` 等 Form。
8. 建立 List／View。
9. 啟用 View。
10. 建立文件並驗證結果。

### 影片版本的已知陷阱

講者實際示範：tutorial 自動產生的 default／DQL Schema 中有錯誤欄位名稱，建立文件時會回報欄位不存在。

> [!note] 原影片畫面未收錄
> 「手動修正 Schema 欄位與權限」畫面因可能含講者肖像、介面資料或憑證範例，未納入公開版。

影片採用的處理方式：

1. 開啟自動產生的 default Schema。
2. 刪除錯誤欄位。
3. 依實際 Form 重新加入 `Name`、`Category`、地址等欄位。
4. 在 DQL mode 刪除錯誤欄位，只加入真正需要查詢的欄位。
5. 將只需查詢的欄位設為 read-only。
6. 儲存後重試 request。

這是本片最值得保留的故障排除資訊之一。

## 實驗三：Docker、Node-RED 與 Collabsphere

來源：[50:30–1:00:55](https://www.youtube.com/watch?v=gj3JfQu5VWk&t=3030s)
證據：示範；中途曾失敗，重新登入後才成功。

### 建立 Node-RED 容器

影片顯示的 tutorial：

```text
https://opensource.hcltechsw.com/domino-keep-tutorials/pages/collabsphere21/nodered_contacts/index
```

下載並解壓 Node-RED Docker ZIP 後，投影片使用：

```bash
docker build -t node-red-restapi-collabsphere:1.0.0 .

docker run -it -p 1880:1880 \
  -e "AUTHENTICATION_HOST=http://MY.HOST.COM:8880" \
  --name nrCSphere21 \
  node-red-restapi-collabsphere:1.0.0
```

> [!note] 原影片畫面未收錄
> 「Node-RED Docker 檔案與命令」畫面因可能含講者肖像、介面資料或憑證範例，未納入公開版。

把 `MY.HOST.COM` 換成自己的 Domino REST API 主機。影片中未示範 TLS、憑證或正式環境的 secret 管理。

### 建立資料與設計

1. 認證並把 Token 存入變數。
2. 使用 Setup API 建立 Collabsphere／Contacts 資料庫。
3. 建立 Scope 與 Schema。
4. 建立 Contact Form。
5. 建立 `ByName` 與 `ByState` 兩個 Views。
6. 更新 Schema。
7. 使用 mock data 批次建立 1,000 筆聯絡人；影片中約十秒完成。

> [!note] 原影片畫面未收錄
> 「批次建立後，Domino 顯示 1,000 筆文件」畫面因可能含講者肖像、介面資料或憑證範例，未納入公開版。

8. 以 DQL 篩選特定州，例如 `CA`。

Mock data 網站名稱在語音中不夠清楚，請以 tutorial 提供的資源為準。

### 在 Node-RED 串接

1. 開啟 Node-RED：

   ```text
   http://localhost:1880/
   ```

2. 依 tutorial 加入 authentication、function、HTTP request 與 debug 等 nodes。
3. 把 nodes 串成認證、呼叫 API、處理結果與顯示資料的流程。

> [!note] 原影片畫面未收錄
> 「Node-RED 中串接的 REST API 流程」畫面因可能含講者肖像、介面資料或憑證範例，未納入公開版。

4. 切換 Debug 面板，觸發流程並確認 HTTP status。
5. Deploy 後開啟地圖。

### 影片現場的真實除錯過程

本次示範第一次執行沒有成功，講者看到錯誤後：

1. 懷疑先前命令或 Schema 更新尚未完成。
2. 再次更新 Schema。
3. 發現登入資訊未取得。
4. 重新執行 authentication。
5. 再次觸發流程，回到 HTTP 200。
6. 地圖成功顯示 1,000 筆聯絡人依州分布；影片指出 California 有 107 筆。

> [!note] 原影片畫面未收錄
> 「Node-RED 最終顯示聯絡人州別地圖」畫面因可能含講者肖像、介面資料或憑證範例，未納入公開版。

這段表示 Node-RED 流程至少依賴有效 Token 與更新完成的 Schema；看到 400 類錯誤時，先檢查認證狀態及 Schema 發布狀態。

## 完成檢查表

- [ ] 已確認 Domino REST API 版本與 Domino 版本相容。
- [ ] 已備份 `notes.ini` 與相關設定。
- [ ] `restapi` task 可以啟動。
- [ ] `http://host:8880/` 可以開啟並登入。
- [ ] 已為目標 NSF 建立 Schema。
- [ ] Schema 只公開必要的 Form、View 與 Fields。
- [ ] 欄位讀寫權限符合最小權限原則。
- [ ] 已建立並啟用 Scope。
- [ ] Authentication request 能取得 Token。
- [ ] 後續 request 正確使用 Token。
- [ ] Domino ACL 仍能限制資料存取。
- [ ] Create／Read／Update／Query 都已在測試資料庫驗證。
- [ ] DQL／OData mode 已依需求設定。
- [ ] 沒有把影片示例密碼、Token 或 hash 用於正式環境。
- [ ] Node-RED 流程能重新認證，並能處理 Token 失效。
- [ ] 正式環境已另行處理 TLS、secret 管理、日誌與監控。

## 限制與待確認事項

- 影片基於 Domino REST API 1.0；當前產品名稱、安裝器、路徑、端點與 UI 可能已變更。
- 影片頁面沒有字幕，逐字稿由本機模型產生；產品名與流程已人工校正，但無法保證每句口語逐字完全正確。
- 多個 Postman request 的完整 URL、Body 與 Tests script 在畫面中太小，本文沒有臆造。
- 影片列出的管理／健康檢查／metrics 埠應以實際版本設定為準。
- 第二個案例的官方 tutorial 在影片版本出現 Schema 欄位錯誤；新版 tutorial 是否已修正，本文未驗證。
- 第三個案例曾因認證或 Schema 狀態失敗，不能只把最終成功畫面視為可重現保證。
- YouTube 留言有人詢問跨伺服器存取設定；影片頁面沒有提供可驗證的設定方式，因此仍列為待確認議題。

## 影片中出現的參考入口

- [HCL Domino REST API 影片](https://www.youtube.com/watch?v=gj3JfQu5VWk)
- [Domino REST API 安裝設定文件入口（影片投影片）](https://opensource.hcltechsw.com/Domino-rest-api/installconfig/index.html)
- [Domino REST API Tutorials 入口（影片投影片）](https://opensource.hcltechsw.com/domino-keep-tutorials/index.html)
- [Postman](https://www.postman.com/)
