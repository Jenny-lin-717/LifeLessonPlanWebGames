# LifeLessonPlanWebGames

這是老師使用的後台。
老師用後台做這些工作：

- 登入
- 管理學生
- 編排關卡流程
- 準備網頁遊戲的素材

## 目錄

- `frontend/` 是老師使用的網頁。
- `backend/` 是 FastAPI 服務。
- MariaDB 存放資料。
- `backend/static/` 存放圖檔。
- `backend/storage/` 存放關卡 JSON。
- `docs/` 存放設計文件與實作文件。

## 啟動

在 `backend/` 目錄執行下列步驟。

1. 複製 `.env.example`。將複本命名為 `.env`。

```powershell
copy .env.example .env
```

2. 在 `.env` 填入資料庫連線設定，包含資料庫名稱 `DB_NAME` 與密碼 `DB_PASSWORD`。

不要提交 `.env`。`.env` 含有資料庫密碼。可提交的範例是 `.env.example`。

3. 先建立與 `.env` 裡 `DB_NAME` 同名的資料庫。再用 `mysql` 匯入 `backend/sql/schema.sql`，此檔只有建表指令，不會建立或選取資料庫。

確認 MariaDB 已啟動、`mysql` 指令可用，且 `.env` 的資料庫帳號有建庫與建表權限。在 `backend/` 目錄將以下整段貼入 PowerShell；提示時輸入 `.env` 對應欄位的值（不含外層引號）。

```powershell
& {
    $dbHost = Read-Host '請輸入 .env 的 DB_HOST'
    $dbPort = Read-Host '請輸入 .env 的 DB_PORT'
    $dbUser = Read-Host '請輸入 .env 的 DB_USER'
    $dbName = Read-Host '請輸入 .env 的 DB_NAME'
    if ([string]::IsNullOrWhiteSpace($dbName)) { throw '請先在 .env 設定 DB_NAME' }

    $dbIdentifier = '`' + $dbName.Replace('`', '``') + '`'
    mysql --host=$dbHost --port=$dbPort --user=$dbUser --password --default-character-set=utf8mb4 --execute="CREATE DATABASE IF NOT EXISTS $dbIdentifier CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"
    if ($LASTEXITCODE -ne 0) { throw '建立資料庫失敗，請確認連線設定與權限' }

    mysql --host=$dbHost --port=$dbPort --user=$dbUser --password --default-character-set=utf8mb4 --database=$dbName --execute="source sql/schema.sql"
    if ($LASTEXITCODE -ne 0) { throw '匯入資料表失敗，請查看 mysql 錯誤訊息' }
}
```

兩次 `mysql` 指令都會提示輸入密碼，請輸入 `.env` 的 `DB_PASSWORD`；不需將密碼寫進指令。因為目前位於 `backend/`，匯入路徑使用 `sql/schema.sql`。匯入成功後再繼續下列步驟。

4. 確認 Python 版本是 3.12 或更新的版本。
5. 執行 `uv sync`。此命令將依賴安裝到專案虛擬環境。

```powershell
uv sync
```

6. 執行 `uv run uvicorn backend.main:app --reload --port 8001`。

```powershell
uv run uvicorn backend.main:app --reload --port 8001
```

## 確認服務

1. 開啟 http://127.0.0.1:8001/ 。
2. 開啟 http://127.0.0.1:8001/api/health 。`status` 是 `ok` 時，服務已啟動。
3. 開啟 http://127.0.0.1:8001/api/db-check 。`connected` 是 `true` 時，資料庫已連線。

## 選用工具

2D 生圖的路徑寫在 `.env`。
本機沒有 2D 生圖工具時，在 `.env` 設定 `TEXT2IMAGE_MOCK=1`。
3D 生模型的路徑寫在 `.env`。
本機沒有 3D 生模型工具時，在 `.env` 設定 `TRELLIS_MOCK=1`。
Unity 探針的金鑰寫在 `.env` 的 `UNITY_API_KEY`。

## 測試

在 `backend/` 目錄執行此命令。

```powershell
uv run pytest
```
