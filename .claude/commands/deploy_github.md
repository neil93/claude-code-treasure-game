Deploy this project to GitHub Pages.

## 前置準備

1. 確認目前在 git repo 內，且已設定 GitHub remote：
   ```
   git remote get-url origin
   ```
   若尚未初始化或連結 GitHub，請先停止並告知使用者。

2. 從 remote URL 取得 repo 名稱（例如 `origin` 為 `https://github.com/USER/REPO.git` 則 base 為 `/REPO/`）。
   若是 GitHub 個人/組織站台（repo 名稱為 `<username>.github.io`），base 則為 `/`。

## 部署步驟

### Step 1 — 安裝 gh-pages（若尚未安裝）
```
npm list gh-pages || npm install --save-dev gh-pages
```

### Step 2 — 設定 Vite base
在執行 build 前，先確認 `vite.config.ts` 的 `base` 是否正確對應 repo 名稱。
- 若目前 `vite.config.ts` 沒有設定 `base`（或 base 為 `'/'`），且 repo 名稱**不是** `<username>.github.io`，需要暫時傳入 `--base` 參數給 build 指令。

### Step 3 — 建置
```
npm run build -- --base=/REPO_NAME/
```
（將 `REPO_NAME` 替換為實際 repo 名稱；若為個人站台則用 `npm run build`）

建置輸出目錄為 `./build`（由 vite.config.ts 的 `outDir: 'build'` 決定）。

### Step 4 — 部署至 gh-pages 分支
```
npx gh-pages -d build
```
這會將 `build/` 目錄的內容推送到 `gh-pages` 分支。

### Step 5 — 確認部署結果
1. 回報部署完成。
2. 提示使用者至 GitHub repo 的 **Settings → Pages**，確認 Source 設為 `Deploy from a branch`，Branch 選 `gh-pages`，資料夾選 `/ (root)`。
3. 部署完成後，站台 URL 通常為：
   `https://<username>.github.io/<repo-name>/`

## 注意事項

- 若 `npm run build -- --base=...` 語法不被支援，改用：
  `npx vite build --base=/REPO_NAME/`
- 若使用者希望永久固定 base，建議將 `base` 寫入 `vite.config.ts`，而非每次 build 時傳入。
- `gh-pages` 套件會自動處理 `.nojekyll` 檔案（避免 Jekyll 忽略底線開頭的資料夾）。
- 每次部署都會強制覆蓋 `gh-pages` 分支，**不會**影響主分支的程式碼。
