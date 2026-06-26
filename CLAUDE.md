# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 常用指令

```
npm install      # 安裝依賴套件
npm run dev      # 啟動 Vite 開發伺服器，預設網址 http://localhost:3000（會自動開啟瀏覽器）
npm run build    # 建置正式版本，輸出至 ./build
```

目前 `package.json` 並未設定 lint、test 或 typecheck 腳本。請不要假設這些指令存在（例如 `npm run lint` 或 `npm test`），使用前務必先確認。

## 架構說明

這是一個以 React + TypeScript 撰寫、使用 Vite（`@vitejs/plugin-react-swc`）建置的單畫面遊戲。進入點是 `src/main.tsx`，會掛載 `src/App.tsx` —— 目前所有的遊戲邏輯與畫面渲染都集中在這一個檔案中（state、開箱/重置的處理函式、JSX 全部寫在一起，沒有額外的狀態管理或路由層）。

遊戲狀態（`src/App.tsx`）：
- `boxes`：3 個 `Box` 物件（`id`、`isOpen`、`hasTreasure`），每一輪會在 `initializeGame()` 中隨機選一個箱子放入寶藏。
- `score`：打開寶藏箱 +100 分，打開非寶藏箱 -50 分。
- `gameEnded`：當找到寶藏或三個箱子都被打開後變為 true；遊戲結束後點擊會被忽略，直到呼叫 `resetGame()` 重新隨機分配箱子。

素材是以 ES module 方式直接 import，並用其產生的 URL 來引用（並非透過 public 資料夾）：
- `src/assets/*.png` —— 寶箱圖片（關閉／已開啟有寶藏／已開啟為骸骨三種狀態）。
- `src/audios/*.mp3` —— 開箱音效，在 `App.tsx` 中播放。

`src/components/ui/` 是 shadcn/ui 風格的元件庫（以 `class-variance-authority` + Tailwind 包裝 Radix UI 元件）。目前遊戲只用到其中的 `button.tsx`，其餘元件都是初始範本附帶但未被使用 —— 可視為可隨需取用的元件庫，不需要當作正在維護中的程式碼。`src/components/figma/ImageWithFallback.tsx` 是 Figma 匯出用的輔助元件，目前 `App.tsx` 也沒有使用到它。

### 樣式相關的重要陷阱

這個專案**沒有 Tailwind 的編譯流程**（沒有 `tailwindcss`/`postcss` 的設定檔或依賴套件）。`main.tsx` 實際 import 的 `src/index.css` 是一個約 4700 行、**已經編譯好、靜態的 Tailwind v4 CSS 檔案**，直接被提交進版本控制。而看起來像「樣式來源」的 `src/styles/globals.css`（裡面有 `@theme`、顏色/圓角等 CSS 自訂屬性，以及基本的文字排版規則）**實際上沒有被任何地方 import**，對畫面完全沒有影響。

實務上的影響：
- 修改 `src/styles/globals.css` 不會讓瀏覽器畫面有任何變化。
- `.tsx` 檔案中使用的 Tailwind utility class，只有在 `src/index.css` 裡已經存在對應的編譯結果時才會生效。如果新增了一個 `index.css` 裡沒有的 Tailwind class，畫面上不會套用 —— 必須重複使用既有的 utility class，或直接把缺少的編譯規則加進 `index.css`。
- 若需求是調整主題色票（顏色、圓角、字型等），改動必須落在 `src/index.css` 中才會生效（或是另外導入真正的 Tailwind 編譯流程）。

### 路徑別名（path alias）

`vite.config.ts` 定義了 `@` → `./src`，另外還列出一大串「帶版本號」的 import 別名（例如 `'lucide-react@0.487.0' → 'lucide-react'`），目的是讓從 Figma/shadcn 匯出時帶有版本號的 import 寫法，在不需要修改程式碼的情況下也能正確解析。
