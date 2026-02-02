植生系醫療器材審查代理人系統 (Botanical Agentic Reviewer) 技術規格書 v1.2
文件狀態：正式版
最後更新日期：2025年05月20日
技術堆疊：React 18, TypeScript, Tailwind CSS, Google Gemini SDK
授權：MIT License
1. 專案概述 (Project Overview)
1.1 系統目標
本系統（Botanical Agentic Reviewer）旨在建立一個高度互動、視覺優雅且具備 AI 代理能力的醫療器材查驗登記（Premarket Application）審查輔助平台。系統的核心目標是解決傳統法規文件（Regulatory Affairs）撰寫與審查過程中，資訊繁雜、格式僵化以及重複性工作過高的痛點。
透過整合 Google Gemini AI 模型，本系統不僅提供結構化的資料編輯功能，更引入了「代理人（Agentic）」工作流，能夠即時讀取使用者正在編輯的上下文資料，自動生成摘要、法規評估報告或進行文件除錯。
1.2 設計哲學：植生系美學 (Botanical Aesthetics)
有別於傳統企業軟體冰冷、單調的介面，本系統採用「植生系」設計語言。系統內建多達 20 種以花卉為主題的動態樣式（如玫瑰、向日葵、薰衣草等），透過動態切換 Tailwind CSS 的漸層（Gradients）、強調色（Accents）與卡片背景，為高壓的法規審查工作帶來視覺上的舒緩與愉悅感。
1.3 適用範圍
台灣醫療器材查驗登記 (TW Premarket)：包含新案、變更登記（許可證變更）、展延等場景。
美國 FDA 510(k)：提供智能分析與摘要（目前為預留模組）。
非結構化文件處理：PDF 轉 Markdown 及文本清理。
2. 系統架構 (System Architecture)
2.1 高層次架構圖
本系統採用 單頁應用程式 (Single Page Application, SPA) 架構，完全運行於客戶端瀏覽器中（Client-side only）。這種設計確保了資料隱私性（數據無需經過中介後端伺服器），僅在呼叫 AI 模型時透過安全的 SDK 與 Google Cloud 進行通訊。
code
Mermaid
graph TD
    User[使用者] -->|操作介面| Client[瀏覽器 (React App)]
    Client -->|狀態管理| State[React State / Context]
    Client -->|樣式渲染| Tailwind[Tailwind CSS Engine]
    Client -->|AI 推論請求| SDK[Google GenAI SDK]
    SDK -->|API Call| Gemini[Google Gemini API]
    Client -->|檔案 I/O| LocalFS[本地檔案系統 (JSON/CSV)]
2.2 核心技術選型
核心框架：React 18.2+。利用 Hooks (useState, useEffect, useCallback) 進行高效的狀態管理與副作用處理。
語言：TypeScript。透過強型別定義（如 TWApplication, FlowerStyle）確保大型表單資料結構的穩定性與開發時的除錯效率。
樣式系統：Tailwind CSS。利用 Utility-first 的特性配合動態字串模板（Template Literals），實現 20 種花卉主題的即時切換與深色模式（Dark Mode）支援。
AI 整合：@google/genai SDK。直接在前端整合 Gemini 2.5/3.0 模型，支援長上下文（Long Context）與系統提示（System Instruction）。
資料視覺化：Recharts。用於儀表板（Dashboard）的統計圖表繪製。
3. 資料模型與結構 (Data Models)
資料結構是本系統的核心，定義於 types.ts 中。所有的編輯器邏輯、AI 上下文注入以及檔案匯出匯入皆基於此結構。
3.1 醫療器材申請書結構 (TWApplication)
此介面模擬了真實世界的法規申請書結構，包含五大核心區塊：
基礎資料 (Basic Data)：
包含欄位：填表日期、產品名稱（中/英）、風險等級（Class I/II/III）、產地、變更理由等。
特性：扁平化物件，易於綁定至 <input> 元件。
產品分類 (Classifications)：
結構：陣列物件。
用途：對應醫材分類分級代碼（如 A.1345）。
變更對照表 (Change Items)：
結構：Array<{ item: string, original: string, requested: string }>。
用途：這是「許可證變更」案件中最關鍵的資料，AI 代理人會重點分析此區塊的 original 與 requested 差異。
申請商資訊 (Applicant)：
包含欄位：統一編號、公司名稱、地址、聯絡人資訊等。
檢核表與文件 (Checklists & Documents)：
ChecklistItem：定義法規要求的章節（如 EP/STED, QSD），具備狀態 Applicable / Not Applicable。
DocumentFile：定義附錄文件，具備狀態 Active / Void / Pending，並支援描述欄位。
3.2 外觀樣式模型 (FlowerStyle)
為了實現動態主題引擎，系統定義了標準化的樣式介面：
code
TypeScript
export interface FlowerStyle {
  name: string;      // 主題名稱 (如 "Rose")
  gradient: string;  // Tailwind 背景漸層類別 (如 "from-red-100 to-rose-300")
  accent: string;    // 文字強調色 (如 "text-rose-600")
  cardBg: string;    // 卡片半透明背景色 (如 "bg-rose-50/50")
}
透過在 constants.ts 中維護 FLOWER_STYLES 陣列，系統可以輕易擴充新的主題而無需修改核心渲染邏輯。
4. 核心模組詳解 (Component Specifications)
4.1 全域控制器 (App.tsx)
職責：
維護全域狀態：theme (亮/暗色), language (語系), currentStyle (花卉主題), activeTab (當前分頁)。
處理 API Log：handleLog 函數負責收集各個子組件的 AI 使用紀錄，並傳遞給 Dashboard。
佈局管理：採用 Flexbox 佈局，左側為 Sidebar，右側為動態內容區（Content Area）。
背景特效：實作了一個跟隨主題變色的全域模糊光暈（Global Blur Orb），利用 mix-blend-multiply 創造玻璃擬態（Glassmorphism）的深邃感。
4.2 導航與設定側欄 (Sidebar.tsx)
功能：
提供全域設定的切換按鈕。
Jackpot 機制：實作 handleJackpot 函數，隨機從 FLOWER_STYLES 陣列中選取一組樣式，增加使用者互動的趣味性。
響應式設計：在行動裝置上會自動調整寬度與顯示模式（雖然目前的 Codebase 主要針對 Desktop 優化，但保留了 Tailwind 的響應式前綴）。
4.3 台灣查驗登記編輯器 (TWPremarket.tsx)
這是系統最複雜的業務組件，整合了表單編輯、資料載入、檔案 I/O 與 AI 整合。
4.3.1 狀態管理策略
使用單一且深層的 State 物件 appData 來儲存整份申請書：
code
TypeScript
const [appData, setAppData] = useState<TWApplication>(DEFAULT_DATASETS["case-001"]);
為了避免 React 渲染效能問題與狀態更新的複雜度，實作了多個 Helper Functions 進行不可變（Immutable）更新：
updateBasic: 使用 Spread Operator 更新第一層物件。
updateChangeItem: 使用 map 或陣列索引更新深層陣列中的特定物件，確保 React 能偵測到 Reference 改變而觸發重繪。
toggleChecklist / cycleDocStatus: 實作狀態機邏輯（例如文件狀態在 Active -> Void -> Pending 之間循環）。
4.3.2 動態增刪邏輯 (CRUD)
針對「變更對照表」與「附錄文件」，實作了完整的 CRUD 功能：
新增 (Create)：透過 setAppData 將新的空物件（預設值）推入陣列末端。
讀取 (Read)：透過 map 函數渲染表格列（Rows）或卡片（Cards）。
更新 (Update)：將 <input> 或 <textarea> 的 onChange 事件綁定至上述的 Helper Functions。
刪除 (Delete)：利用 filter 函數，依據陣列索引（Index）移除特定項目。
4.3.3 檔案匯出與匯入 (File I/O)
JSON Export：利用 Data URI Scheme 將 appData 序列化為 JSON 字串，動態建立 <a> 標籤觸發瀏覽器下載行為。檔名包含許可證字號以利識別。
JSON Import：使用 FileReader API 讀取使用者上傳的 .json 檔案，並進行基礎的結構驗證（檢查是否存在 basicData 等關鍵欄位），防止載入錯誤格式導致崩潰。
CSV Export：
實作了資料扁平化（Flattening）邏輯，將巢狀的 JSON 結構轉換為二維的 CSV 表格。
BOM 處理：在字串開頭加入 \uFEFF，確保 Microsoft Excel 能正確識別 UTF-8 編碼的繁體中文內容，避免亂碼。
跳脫字元處理：針對內容中可能出現的逗號或換行符，使用雙引號包覆欄位，並處理內嵌雙引號的 Escape ("")。
4.3.4 案例切換 (Case Selection)
整合 defaultDatasets.ts，提供下拉選單讓使用者快速切換不同的預設案例（如 Class II 血糖機、Class III 骨科植入物、軟體醫材）。切換邏輯會直接覆寫 appData 狀態，達成即時載入效果。
4.4 AI 代理人執行器 (AgentRunner.tsx)
此組件是 "Agentic" 的核心實作，負責連接使用者介面與 LLM 服務。
即時上下文同步 (Live Context Sync)：
利用 useEffect 監聽父層傳入的 defaultInput（即 appData 的 JSON 字串）。當使用者在 TWPremarket 編輯任何欄位時，AgentRunner 內的 input 狀態會自動更新。這確保了當使用者按下 "Generate Summary" 時，AI 讀取到的是最新版本的資料。
模型參數控制：提供 UI 讓使用者選擇模型（如 gemini-3-flash-preview）、設定 maxTokens。
狀態回饋：維護 status 狀態（idle, running, done, error），並透過不同顏色的指示燈（Indicator）與按鈕狀態給予使用者視覺回饋。
4.5 儀表板 (Dashboard.tsx)
利用 Recharts 函式庫，接收 App.tsx 傳來的 logs 陣列。
統計各個模型的呼叫次數（BarChart）、總 Token 消耗量以及活躍代理人數量。
支援深色模式適配，圖表的座標軸顏色與 Tooltip 背景色會隨全域主題自動切換。
5. AI 整合策略 (AI Integration Strategy)
5.1 服務層 (services/geminiService.ts)
封裝了 Google Gemini SDK 的呼叫邏輯，提供統一的介面 callLLM。
參數化設計：接受 model, systemPrompt, userPrompt, temperature 等參數，使前端能針對不同任務微調模型行為。
錯誤處理：使用 try-catch 區塊捕獲 API 錯誤，並將錯誤訊息回傳至 UI 層顯示，避免應用程式崩潰。
安全性：API Key 透過 process.env.API_KEY 注入。在建構 GoogleGenAI 實例時，直接使用此環境變數，確保金鑰不會硬編碼在原始碼中。
5.2 提示工程 (Prompt Engineering)
系統針對「醫療器材審查」場景內建了優化過的 System Prompt（詳見 TWPremarket.tsx 的 AgentRunner 區塊）：
角色設定 (Persona)：定義 AI 為 "Expert regulatory affairs specialist"（資深法規事務專員）。
任務拆解 (Task Breakdown)：要求產出包含 Executive Summary, Change Analysis, Regulatory Readiness 等五大區塊的結構化 Markdown 報告。
格式限制：明確要求輸出乾淨的 Markdown 格式，便於後續複製或渲染。
6. 使用者介面與互動設計 (UI/UX)
6.1 色彩與光影系統
系統利用 Tailwind CSS 的強大功能建立了一套基於「自然」的色彩系統：
玻璃擬態 (Glassmorphism)：大量使用 backdrop-blur-md、bg-white/60 (或 bg-slate-800/60) 以及細微的邊框 (border-white/40)，創造出懸浮於背景之上的層次感。
漸層運用：每個花卉主題都定義了一組獨特的漸層（Gradient），應用於按鈕、標題強調文字以及進度條背景。
6.2 響應式與可存取性 (Accessibility)
滾動條優化：自定義 CSS Scrollbar (::-webkit-scrollbar)，使其在不同作業系統下風格一致且不突兀。
深色模式：全系統支援 Dark Mode。文字顏色在深色模式下會自動轉為 slate-200 或 slate-400，降低對比度帶來的視覺疲勞；背景則轉為深藍灰色系 (slate-900)。
輸入體驗：表單輸入框去除了預設的瀏覽器樣式（outline），改為自定義的 Focus Ring (focus:ring-2)，提升操作手冊感。
7. 部署與環境配置 (Deployment)
7.1 環境變數
系統依賴單一關鍵環境變數運作：
API_KEY: Google Gemini API 金鑰。
在開發環境中，通常透過 .env 檔案或 Bundler (如 Vite/Webpack) 的 DefinePlugin 注入。
7.2 建置流程
由於是純前端專案，建置流程如下：
依賴安裝：npm install (安裝 React, Tailwind, Gemini SDK 等)。
類型檢查：tsc (TypeScript Compiler) 驗證所有型別定義無誤。
打包：使用 Bundler 將 TypeScript 與 CSS 打包為靜態的 JS/CSS 檔案。
發布：產出的靜態檔案可部署於任何靜態網站託管服務（如 Vercel, Netlify, GitHub Pages, Firebase Hosting）。
8. 未來擴充規劃 (Future Roadmap)
RAG (Retrieval-Augmented Generation) 整合：目前 AI 僅依賴使用者輸入的 JSON 進行分析。未來可加入向量資料庫，讓 AI 能參考台灣 TFDA 的法規資料庫（如醫材管理法、分類分級標準），提供更精準的合規性建議。
多模態輸入 (Multimodal Input)：利用 Gemini 的視覺能力，允許使用者直接上傳產品包裝圖或標籤圖檔（Image），由 AI 自動比對圖片內容與申請書文字是否一致。
PDF 解析增強：目前的 PDF 模組僅為模擬。未來可整合 OCR 或 PDF 解析庫（如 PDF.js），將真實的歷史文件轉換為結構化資料匯入系統。
協作功能：引入 WebSocket 或 Firebase Realtime Database，實現多人同時編輯同一份申請書的功能。
9. 結論
Botanical Agentic Reviewer 不僅僅是一個資料輸入工具，它展示了現代 Web 技術（React + Tailwind）與生成式 AI（Gemini）結合的可能性。透過嚴謹的 TypeScript 型別系統確保了資料的正確性，透過精心設計的 UI/UX 提升了使用者的工作體驗，並透過 Agentic Workflow 大幅降低了法規文件撰寫的門檻。本技術規格書所描述的架構，具備高度的可維護性與擴充性，足以作為下一代法規科技（RegTech）產品的基石。
