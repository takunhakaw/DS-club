# Dream Station（GitHub Pages + Google Apps Script）

本版本不使用 Firebase。前端部署在 GitHub Pages；資料、登入驗證及權限檢查由 Apps Script 管理，資料存放在 Google 試算表。

## 部署步驟
1. 建立 Google Apps Script 專案，將 `Code.gs` 全文貼入。
2. 在 Apps Script 編輯器選擇 `setup` 並執行，授權存取 Google 試算表。執行紀錄會列出資料庫試算表網址。
3. 建立老師帳號：編輯 `createTeacherAccount()` 中的 email、初始密碼及姓名，再執行一次。請勿把初始密碼提交到 GitHub；執行後可立即改成自行管理的安全密碼。也可先用學生帳號註冊，再由編輯器執行 `promoteTeacher('帳號')`。
4. 部署 > 新增部署作業 > 網頁應用程式；執行身分選「我」，存取權選「任何人」。複製 `/exec` 網址。
5. 把 `index.html` 中 `GAS_URL` 的 `請填入_GAS_WEB_APP_EXEC_URL` 改成該 `/exec` 網址。
6. 將 `index.html` 上傳到 GitHub repository 根目錄（或 docs 目錄），Settings > Pages 選擇部署來源並發布。
7. 開啟 GitHub Pages 網址測試註冊、登入、貼文、訂單與老師後台。

## 重要設計
- 公開註冊固定為學生；無法在前端選擇老師。老師角色由 Apps Script 端建立或提升。
- 讀取採 JSONP；寫入採 `no-cors` POST，接著以短效 request id 輪詢結果，避免 GitHub Pages 與 Apps Script 跨網域預檢問題。
- 密碼以每位使用者獨立 salt + SHA-256 雜湊儲存，不保存明文密碼。Session token 存於瀏覽器 localStorage，伺服器端短效快取有效 6 小時。
- 訂單價格與庫存由伺服器重新核對，並以 LockService 避免同時下單超賣。
- 備份是老師登入後下載 JSON；不會自動備份到 Google Drive。試算表本身即為主要資料庫。
- 相簿沿用照片 HTTPS 網址模式，不會把圖片檔案上傳到 Drive。

## 已知限制與正式上線前建議
- Apps Script CacheService session 有效期及快取資料不是永久儲存；使用者可能需重新登入。
- 本專案使用 Apps Script + Spreadsheet，適合中小型社團平台；大量並發時需改用更完整的資料庫。
- SHA-256 是 Apps Script 內建可用的簡化密碼雜湊；若處理大量敏感個資，應採用專業身分驗證服務。不要存放身分證字號、住址等敏感資料。
- 請在 Apps Script 專案設定時確認時區為 Asia/Taipei。
- 由於 GitHub Pages 是公開靜態網站，請勿把老師密碼、管理密鑰或試算表 ID 寫進前端。

## 檔案
- `index.html`：GitHub Pages 前端
- `Code.gs`：Apps Script 後端
