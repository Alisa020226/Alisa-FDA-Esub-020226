# 技術規格書：FloraAgentic - 醫療器材法規 AI 系統

**版本:** 1.1.0  
**日期:** 2023年10月26日  
**狀態:** 正式版  
**作者:** 資深前端工程團隊

---

## 1. 執行摘要 (Executive Summary)

**FloraAgentic** 是一款專為醫療器材法規事務 (Regulatory Affairs, RA) 專業人員設計的代理人式 AI (Agentic AI) 系統。本應用程式旨在提供一個整合性的工作空間，用於生成、審查及組織複雜的法規文件，特別針對台灣衛生福利部食品藥物管理署 (TFDA) 的查驗登記申請以及美國 FDA 510(k) 上市前通知。

本系統在設計哲學上獨樹一幟，採用了「親生命設計 (Biophilic UI)」理念，整合了 20 種不同的花卉主題美學，旨在減輕法規人員在處理高強度文件審查時的認知負荷與壓力。在技術層面上，這是一個基於 React 19 和 TypeScript 構建的客戶端單頁應用程式 (SPA)，並利用 Google Gemini API (透過 `@google/genai` SDK) 進行智能文本生成與分析。

本文件為 FloraAgentic 系統的權威技術參考，詳細說明了基於當前程式碼庫的系統架構、資料模型、組件規格、Mock 數據機制及整合邏輯。

---

## 2. 系統架構 (System Architecture)

### 2.1. 高階概觀
FloraAgentic 運作為無伺服器 (Serverless) 的純客戶端應用程式。它完全依賴瀏覽器的執行環境來處理所有商業邏輯、狀態管理及 API 編排。外部通訊僅限於與 Google Gemini API 進行推論任務，確保了部署的輕量化與資料的隱私性（數據不經過中間伺服器）。

*   **前端框架:** React 19 (透過 ES Modules/ESM 載入)。
*   **樣式引擎:** Tailwind CSS (Utility-first) 結合動態內聯樣式 (Dynamic Inline Styles) 實現主題切換。
*   **程式語言:** TypeScript (針對法規資料結構進行嚴格型別定義)。
*   **AI 推論核心:** Google Gemini 2.5 Flash (透過 `@google/genai` SDK)。
*   **數據視覺化:** Recharts (用於儀表板的使用記錄分析)。
*   **圖標系統:** Lucide React。

### 2.2. 模組載入策略
應用程式利用定義在 `index.html` 中的 `importmap` 直接從 CDN (`esm.sh`) 載入依賴項。這種架構消除了演示環境中複雜的伺服器端打包步驟，同時利用現代瀏覽器對 ES Modules 的原生支持來實現快速啟動。

**主要依賴:**
*   `react`, `react-dom`: UI 核心庫。
*   `@google/genai`: 用於與 Gemini 模型互動的官方 SDK。
*   `lucide-react`: 輕量級向量圖標。
*   `recharts`: 統計圖表繪製。

### 2.3. 進入點與啟動流程
應用程式進入點定義於 `index.tsx`。它在透過 `ReactDOM.createRoot` 進行 React 應用程式水合 (Hydration) 之前，會對 DOM 根元素進行嚴格的空值檢查 (`null check`)。這確保了如果託管環境配置錯誤，應用程式能夠優雅地失敗並拋出明確錯誤，而非產生未定義的行為。

---

## 3. UI/UX 與主題引擎 (Theming Engine)

FloraAgentic 最複雜的子系統之一是其動態主題引擎，旨在提供類似「老虎機 (Jackpot)」般的視覺驚喜與變化。

### 3.1. FlowerStyle 介面定義
定義於 `types.ts` 中的 `FlowerStyle` 介面控制著應用程式的整體色調。與傳統的 CSS 變數不同，主題是作為包含 Tailwind 工具類字串的 JavaScript 物件注入的，這允許更細粒度的組件級樣式控制。

```typescript
export interface FlowerStyle {
  name: string;     // 顯示名稱 (例如: "Rose")
  gradient: string; // 主背景漸層 (例如: "bg-gradient-to-br...")
  sidebar: string;  // 側邊欄背景與透明度設定
  card: string;     // 卡片/面板背景與玻璃擬態效果
  accent: string;   // 主要互動顏色 (文字/按鈕)
  text: string;     // 主要排版顏色
}
```

### 3.2. 主題應用邏輯
`App.tsx` 組件作為樣式的上下文提供者 (Context Provider)。
1.  **狀態管理:** `settings.styleName` 追蹤當前選定的花卉。
2.  **記憶化 (Memoization):** 使用 `useMemo` 衍生 `currentStyle` 物件，防止在無關狀態更新（如表單輸入）期間重新計算主題物件，優化渲染效能。
3.  **樣式瀑布流:** `currentStyle` 物件作為 Props 向下傳遞給所有容器組件 (`Sidebar`, `Dashboard`, `TwPremarket` 等)，確保主題切換時的瞬間同步。
4.  **深色模式 (Dark Mode):** 透過在 `App.tsx` 的根 `div` 上應用 CSS 濾鏡反轉策略實現：
    `settings.theme === 'Dark' ? 'brightness-90 invert-[0.05]' : ''`
    這提供了一種偽深色模式，無需為 20 種花卉主題分別定義深色專用色票，大幅降低了維護成本。

### 3.3. "Jackpot" 機制
位於 `Sidebar.tsx` 的 `handleJackpot` 函式利用 `Math.random()` 從 `constants.ts` 定義的 20 種 `FLOWER_STYLES` 中隨機選取一種。此遊戲化元素旨在打破法規工作的單調性。

---

## 4. 資料模型與型別定義 (Data Models)

系統依賴嚴格的 TypeScript 定義來確保資料完整性，特別是針對醫療器材申請表單。

### 4.1. TwAppData (台灣查驗登記申請書)
這是應用程式中最龐大的資料結構，代表了數位化的 TFDA 第二/三等級醫療器材申請表。它被邏輯性地劃分為五個區塊：

1.  **基本資料 (Basic Data):** 關於提交的元數據 (公文號、電子流水號、申請日期、器材類別、產品分級)。
2.  **器材資訊 (Device Info):** 核心產品識別 (中英文名稱、適應症、規格、分類分級代碼)。
3.  **申請者資料 (Applicant Data):** 本地許可證持有者的企業詳情 (統一編號、地址、聯絡資訊、RAPS/AHWP 認證)。
4.  **製造廠資訊 (Manufacturer Info):** 製造廠詳情，包含國家與製造方式 (OEM/Odm)。
5.  **附件/詳細資料 (Attachments/Details):** 用於 QMS/QSD、技術檔案、臨床證據及臨床前測試摘要的複雜文本欄位。

### 4.2. 代理配置 (Agent Configuration)
`AgentConfig` 介面定義了 AI 助理的行為模式：
*   `systemPrompt`: 代理的「大腦」，定義其角色設定 (例如：「你是一位 TFDA 上市前審查員」)。
*   `maxTokens`: 輸出限制，用於管理延遲與成本。
*   `category`: UI 分組依據 (TFDA, FDA, 生產力工具)。

### 4.3. 日誌記錄 (Logging)
`LogEntry` 介面追蹤系統的使用情況：
*   `tokensEst`: 啟發式計算 (`(input + output) / 4`)，用於儀表板估算 Token 消耗量，因為簡化的客戶端回應中並不總是暴露精確的 Token 計數。

---

## 5. 功能組件規格詳解

### 5.1. 側邊欄 (`components/Sidebar.tsx`)
*   **角色:** 全域導航與配置控制器。
*   **功能:**
    *   **主題/語言切換:** 更新全域 `Settings` 狀態。
    *   **風格選擇器:** 特定花卉選擇下拉選單 + Jackpot 按鈕。
    *   **API 金鑰管理:** 安全的 Google Gemini API 金鑰輸入欄位。若偵測到 `process.env.API_KEY` (透過 `isEnvKey` prop)，則會條件式渲染為禁用狀態並顯示已載入，防止使用者覆蓋環境變數注入的金鑰。

### 5.2. 儀表板 (`components/Dashboard.tsx`)
*   **角色:** 使用者活動分析中心。
*   **視覺化:**
    *   **統計卡片:** 顯示總執行次數、估計 Token 消耗量及使用的唯一分頁數。
    *   **長條圖:** 使用 `recharts` 繪製每個代理分頁的活動頻率。
    *   **日誌表格:** 最後 5 次互動的倒序列表，顯示時間戳、代理名稱及使用的模型。
*   **資料來源:** 消耗由 `App.tsx` 傳遞的 `logs` 陣列。

### 5.3. 台灣查驗登記模組 (`components/TwPremarket.tsx`)
這是系統的旗艦功能模組，整合了表單填寫、資料載入、檔案導出與 AI 審查。

*   **狀態管理:** 管理對應於 `TwAppData` 的巨大 `formData` 物件。
*   **Mock 資料載入機制:**
    *   利用 `useEffect` Hook 在組件掛載時從 `/defaultdataset.json` 獲取預設資料集。
    *   將資料存儲於 `mockCases` 狀態中。
    *   提供下拉選單與「Load」按鈕，允許使用者將特定的 Mock Case 注入 `formData`，覆蓋當前表單內容。
*   **資料導出功能:**
    *   **JSON 導出:** 將 `formData` 序列化並創建 Blob URL 供下載。
    *   **CSV 導出:** 處理欄位內的特殊字符（如逗號），並添加 UTF-8 BOM (`\ufeff`) 以確保 Microsoft Excel 開啟時中文顯示正常。
*   **檢視模式:**
    1.  **Form View:** 響應式網格佈局 (使用 Tailwind `grid-cols-1 md:grid-cols-6`) 進行資料輸入。包含可折疊的「附件/詳細資料」區塊以管理垂直螢幕空間。
    2.  **Draft View:** 顯示生成的 Markdown 草稿。
    3.  **AI Report View:** 顯示來自 Gemini 的分析結果。
*   **邏輯 - Markdown 生成:** `generateMarkdown` 函式確定性地將 `formData` 狀態映射為格式化的 Markdown 字串，適用於複製貼上至外部文檔工具 (如 Notion, Obsidian)。
*   **邏輯 - AI 預審 (Screen Review):** `handleScreenReview` 函式構建兩部分提示詞：
    1.  **系統指令:** "You are a TFDA premarket screen reviewer..." (你是一位 TFDA 上市前審查員...)。
    2.  **使用者內容:** `formData` 的 JSON 字串化 + 檢查完整性與不一致性的具體指令。

### 5.4. 通用代理 (`components/GenericAgent.tsx`)
*   **角色:** 用於簡單文字輸入/輸出的 AI 任務的可重用包裝器。
*   **應用:** 用於 "510(k) Analyst", "PDF to Markdown", 及 "Note Organizer"。
*   **屬性:** 接收 `systemPrompt`, `defaultInput`, 及 `placeholder` 以自定義代理行為，無需重複 UI 程式碼。
*   **佈局:** 分割窗格設計 (左/上輸入，右/下輸出)，優化對照與編輯體驗。

### 5.5. 代理配置 (`components/AgentsConfig.tsx`)
*   **角色:** 透明度檢視器。
*   **功能:** 顯示定義在 `constants.ts` 中的硬編碼 `DEFAULT_AGENTS`。它允許使用者查看 AI 是 *如何* 被指示的，建立對系統「推理」過程的信任。

---

## 6. 整合服務：Gemini API

`services/geminiService.ts` 模組作為 React UI 與 Google GenAI SDK 之間的抽象層。

### 6.1. 模型解析策略
為了確保穩健性並符合特定模型版本控制 (如提示指南所規定)，服務實作了 `MODEL_MAPPING` 字典：
```typescript
const MODEL_MAPPING: Record<string, string> = {
  "gemini-2.5-flash": "gemini-2.5-flash",
  "gemini-2.5-flash-lite": "gemini-2.5-flash-lite-latest",
  // 針對模擬的舊版/競爭對手模型的後備方案
  "gpt-4o-mini": "gemini-2.5-flash", 
  "gpt-4.1-mini": "gemini-2.5-flash",
  "claude-3-5-sonnet-2024-10": "gemini-2.5-flash",
};
```
這確保了即使 UI 理論上允許選擇競爭對手模型 (用於比較場景)，後端邏輯也能安全地將請求路由至支援的 Google Gemini 模型 (`gemini-2.5-flash`)。

### 6.2. 內容生成流程
1.  **驗證:** 檢查 API 金鑰是否存在。
2.  **實例化:** 為每個請求建立新的 `GoogleGenAI` 客戶端實例。這是無狀態的，且對於金鑰可能變更的客戶端環境是安全的。
3.  **請求構建:** 呼叫 `ai.models.generateContent`，參數包含：
    *   `model`: 解析後的模型字串。
    *   `contents`: 使用者的輸入。
    *   `config`: 包含 `systemInstruction` (系統指令) 與 `temperature` (溫度參數)。
4.  **錯誤處理:** 使用 try/catch 區塊包裝異步呼叫，返回使用者友好的錯誤字串而非導致 UI 崩潰。

---

## 7. 功能規格與工作流程

### 7.1. 應用程式生命週期
1.  **初始化:**
    *   App 掛載。
    *   檢查 `process.env.API_KEY`。
    *   設定初始狀態 (語言: English, 主題: Rose)。
    *   **資料載入:** `TwPremarket` 透過 Fetch API 載入 `defaultdataset.json`。
2.  **配置:**
    *   使用者在側邊欄輸入 API 金鑰 (若環境變數中未設定)。
    *   使用者選擇花卉風格 (或點擊 Jackpot)。
3.  **任務執行 (範例：台灣查驗登記):**
    *   使用者導航至 "TW Premarket" 分頁。
    *   使用者可選擇載入預設案例 (Mock Case) 或手動填寫 5 大區塊表單。
    *   使用者點擊 "Generate Draft MD" -> 系統產出格式化文本。
    *   使用者點擊 "Run Screen Review Agent" -> 系統發送 JSON 負載至 Gemini -> 系統顯示法規分析報告。
    *   使用者可點擊 JSON 或 CSV 圖標下載當前申請資料。
4.  **日誌記錄:**
    *   代理成功完成後，呼叫 `addLog`。
    *   儀表板更新即時圖表與計數器。

### 7.2. 國際化 (i18n)
系統支援英文與繁體中文。
*   **實作:** `constants.ts` 中的 `LABELS` 物件將鍵值 (如 `TwPremarket`) 映射至其翻譯。
*   **使用:** 組件使用 `t(key)` 輔助函式根據 `settings.language` 檢索字串。
*   **範圍:** 本地化覆蓋 UI 標籤、導航及靜態標題。注意：AI 生成的 *內容* 取決於提示詞；目前提示詞為硬編碼的英文 ("You are a TFDA reviewer...")，這意味著除非在使用者輸入中明確指示，否則 AI 輸出可能預設為英文。

---

## 8. 數據處理規範 (Data Handling Specifications)

### 8.1. 預設資料集 (Mock Datasets)
系統包含一個 `defaultdataset.json` 檔案，其中定義了三個完整的範例案例，涵蓋不同情境以測試 AI 的反應能力：
1.  **Case 1 (額溫槍):** 國產、二級、有類似品、適用標準 QMS。這是一個典型的「通過」案例。
2.  **Case 2 (登革熱試劑 - IVD):** 輸入、二級、有類似品、適用替代條款、美國製造。這測試 AI 對 IVD 與進口文件 (CFS/Auth) 的檢查。
3.  **Case 3 (AI 診斷軟體 - SaMD):** 國產、二級、無類似品 (全球首創)、軟體確效。這測試 AI 對新興科技與臨床證據要求的判斷。

### 8.2. CSV 導出規範
為了確保與本地辦公軟體的最大相容性，CSV 生成遵循以下規則：
*   **編碼:** UTF-8 with BOM (Byte Order Mark)。
*   **跳脫字元:** 所有值皆以雙引號包覆 (`"value"`)，且值內部的雙引號會被取代為雙重雙引號 (`""`)，符合 RFC 4180 標準。
*   **格式:** 單一標頭行，隨後為單一資料行 (因為目前表單代表單一申請案)。

---

## 9. 安全性與配置

### 9.1. API 金鑰處理
*   **環境變數優先:** 系統優先使用 `process.env.API_KEY`。若存在，UI 欄位將被鎖定並遮蔽。
*   **使用者輸入:** 若無環境變數，使用者手動輸入金鑰。
*   **儲存:** 金鑰僅儲存於 React State (`settings.apiKey`) 中。在此程式碼庫中，金鑰 **不會** 持久化至 `localStorage` 或 `cookies`，意味著頁面刷新將清除金鑰。這是針對共享工作站的安全特性。
*   **傳輸:** 金鑰透過 SDK 直接發送至 Google API 端點。在此架構中，不經過任何中間代理伺服器。

---

## 10. 效能考量

### 10.1. 渲染優化
*   **Tailwind CSS:** 使用原子化工具類確保 CSS 檔案大小保持小巧且固定，無論有多少獨特的「花卉風格」。
*   **React StrictMode:** 在 `index.tsx` 中啟用，以在開發期間強調潛在的副作用。
*   **Lazy Loading:** 目前未實作 (單一 Bundle)，但模組化結構允許未來對重型組件如 `TwPremarket` 或 `Recharts` 進行代碼分割 (Code-splitting)。

### 10.2. AI 延遲管理
*   **模型選擇:** 系統預設使用 `gemini-2.5-flash`。「Flash」系列專為低延遲與高吞吐量選擇，這對於使用者等待反饋的互動式表單審查至關重要。
*   **Token 管理:** 「筆記整理器」上限為 2000 tokens，而「TFDA 預審」上限為 4000 tokens。這種分段確保了基於任務複雜度的適當資源分配。

---

## 11. 未來擴充路線圖

基於當前架構，以下擴充在技術上是可行的：

1.  **持久化:** 實作 `localStorage` 持久化 `formData`，防止重新整理時資料遺失。
2.  **PDF 解析:** 「PDF 轉 Markdown」代理目前依賴使用者貼上原始文本。整合客戶端 PDF 解析器 (如 `pdf.js`) 可自動化提取步驟。
3.  **提示詞工程:** 當前系統提示詞硬編碼於 `constants.ts` 與組件中。將這些移至使用者可編輯的配置面板將允許「代理調優 (Agent Tuning)」。
4.  **後端整合:** 對於生產環境使用，將 API 金鑰處理移至伺服器端代理將隱藏客戶端瀏覽器的金鑰，並允許更細緻的存取控制與計費管理。

---

## 12. 結論

FloraAgentic 代表了專業垂直領域 AI 應用的現代化方法。透過結合高度特定的領域焦點 (醫療器材 RA)、強大的「代理人式」框架以及以使用者為中心的「親生命」設計，它同時解決了法規合規的功能性需求以及專業人員執行重複性高壓任務時的使用者體驗需求。程式碼庫模組化、強型別化，並建立在業界標準技術之上，為未來的開發提供了堅實的基礎。
