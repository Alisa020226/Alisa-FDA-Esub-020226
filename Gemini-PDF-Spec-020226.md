Aegis - 智慧文件探索與代理人情報系統技術規格書
文件版本: 1.0
日期: 2023年10月
狀態: 草案 (Draft)
1. 執行摘要 (Executive Summary)
1.1 專案背景
在當前的醫療與法規監管環境中（如 FDA 審查、臨床試驗管理），專業人員面臨著處理海量非結構化文件（PDF、協議書、研究報告）的巨大挑戰。傳統的關鍵字搜尋已無法滿足對於語意理解、風險評估及深度推理的需求。「Aegis - 智慧文件探索與代理人情報系統」（以下簡稱 Aegis）旨在彌合非結構化文件與可執行情報之間的鴻溝。
1.2 系統目標
Aegis 是一個基於瀏覽器的單頁應用程式（SPA），利用最先進的多大型語言模型（Multi-LLM）編排技術，專注於醫療與法規文件的自動化分析。本系統的核心目標包括：
自動化風險發現：在文件上傳時即時進行合規性風險掃描。
代理人化（Agentic）互動：透過切換不同的專家角色（如法規專員、臨床數據分析師），提供針對性的深度問答。
零信任架構體驗：強調前端處理與安全性，確保敏感數據的隱私（本版本為展示架構）。
極致的使用者體驗：採用 React 18 與 Tailwind CSS 打造現代化、響應式且具備深色模式的專業介面。
2. 系統架構設計 (System Architecture)
2.1 整體架構圖
Aegis 採用現代化的前端單體架構（Frontend Monolith），並透過 API 與外部 AI 推論引擎整合。
呈現層 (Presentation Layer): React 18 (TypeScript), Tailwind CSS, Framer Motion (Animation logic).
應用邏輯層 (Application Layer): Context API (State Management), React Router (Routing).
服務層 (Service Layer): Gemini API Integration (@google/genai SDK).
數據層 (Data Layer): In-Memory Storage (本次實作), Browser LocalStorage (未來擴充).
2.2 技術堆疊 (Technology Stack)
領域	技術選型	版本/說明
前端框架	React	v19.2.4 (Latest Stable)
開發語言	TypeScript	強型別以確保醫療數據處理的準確性
樣式系統	Tailwind CSS	Utility-first，支援 Dark Mode 與 RWD
建置工具	ESM (ECMAScript Modules)	透過 importmap 進行瀏覽器端模組加載
AI 模型	Google Gemini	Pro (Reasoning) & Flash (Fast Extraction)
圖表視覺化	Recharts	用於風險評估與數據分析呈現
圖標庫	Heroicons	統一的 UI 視覺語言
2.3 目錄結構 (Directory Structure)
系統遵循模組化的功能分組原則：
contexts/: 全域狀態管理（文件、對話紀錄、設定）。
components/: UI 元件（聊天視窗、側邊欄、儀表板）。
services/: 封裝外部 API 呼叫邏輯，實現關注點分離。
types/: 定義全域 TypeScript 介面，確保資料流的一致性。
3. 功能模組規格 (Functional Specifications)
3.1 智慧文件攝取 (Intelligent Document Ingestion)
功能描述: 使用者透過拖放（Drag & Drop）介面上傳文件。
處理流程:
檔案驗收: 支援 PDF, TXT, JSON, MD 格式。
模擬解析: 針對非文字檔（如 PDF），系統目前採用模擬提取（Mock Extraction）策略，預載入 FDA 臨床試驗協議文本作為範例內容，以展示後續分析能力。
即時風險掃描 (On-upload Analysis):
文件上傳後，系統立即觸發背景任務。
呼叫 gemini-3-flash-preview 模型。
利用 JSON Schema Mode 強制模型輸出結構化的 riskScore (0-100) 與 category (文件分類)。
UI 交互: 上傳區域具備動態視覺回饋（旋轉動畫、拖曳狀態變色）。
3.2 代理人聊天編排 (Agentic Chat Orchestration)
本系統的核心差異化功能，允許使用者與「不同人格」的 AI 進行對話。
多重角色 (Multi-Persona):
Regulatory Specialist (法規專員):
系統提示詞 (System Prompt): 專注於 FDA 合規性、引用條款、風險識別，語氣權威且正式。
適用模型: gemini-3-pro-preview (具備更強的邏輯推理與 Thinking Budget)。
Clinical Data Analyst (臨床數據分析師):
系統提示詞: 專注於病患結果、統計顯著性 (P-value)、不良事件 (AEs)。
適用模型: 依據複雜度動態選擇。
Executive Summarizer (行政摘要員):
系統提示詞: 專注於決策點、底線摘要 (Bottom line)，語氣簡潔。
適用模型: gemini-3-flash-preview (速度優先)。
上下文注入 (Context Injection):
聊天引擎會自動將當前選定文件（Active Document）的全文內容（經截斷處理以符合 Context Window）注入到 Prompt 中，實現 RAG (Retrieval-Augmented Generation) 的基礎形式。
思維鏈配置 (Thinking Config):
針對 gemini-3-pro-preview 模型，系統啟用了 thinkingConfig，設定預算為 1024 tokens，並預留 4096 tokens 用於輸出。這使得 AI 在回答複雜法規問題前能進行「內部獨白」與推演，大幅提升準確度。
3.3 儀表板與視覺化 (Dashboard & Visualization)
右側資訊面板 (InfoPanel):
風險計量表 (Risk Meter): 使用 Recharts 繪製環形圖 (Pie Chart)，直觀顯示合規風險分數。
邏輯：若分數 > 50，顯示紅色警示；否則顯示綠色安全。
摘要卡片: 顯示由 AI 生成的簡短摘要。
元數據網格: 顯示檔案名稱、大小、分類標籤。
3.4 導航與狀態管理 (Navigation & State)
側邊欄 (Sidebar):
顯示歷史文件列表。
支援在不同文件間快速切換，切換時即時更新聊天上下文與儀表板數據。
響應式設計：在行動裝置上自動收折，並透過漢堡選單 (Hamburger Menu) 觸發。
4. 詳細技術實作 (Technical Implementation Details)
4.1 前端核心 (React & TypeScript)
AppProvider (contexts/AppContext.tsx):
作為全域 Store，管理 documents (文件陣列), activeDocumentId (當前焦點), messages (聊天記錄), agentConfig (當前代理人設定)。
使用 useContext Hook 避免 Prop Drilling。
型別定義 (types.ts): 嚴格定義 DocumentMetadata 介面，確保狀態操作的安全性。
4.2 AI 服務層 (services/geminiService.ts)
此模組是與 Google Gemini API 溝通的橋樑，包含兩個主要函數：
analyzeDocumentRisk(text: string):
目的: 快速結構化分析。
模型: gemini-3-flash-preview。
技術特點: 使用 responseSchema 與 Type.OBJECT 定義嚴格的 JSON 輸出格式。
Schema 定義:
code
TypeScript
{
    type: Type.OBJECT,
    properties: {
        riskScore: { type: Type.INTEGER },
        category: { type: Type.STRING }
    },
    propertyOrdering: ["riskScore", "category"]
}
優勢: 確保前端能直接 JSON.parse 回傳結果，無需進行正則表達式解析，提升系統穩定性。
generateAgentResponse(prompt, context, config):
目的: 對話式問答。
模型: 動態選擇 (config.model)。
Prompt Engineering:
動態組裝：System Instruction + Context Document + User Query。
錯誤處理: 包含 try-catch 區塊，若 API key 遺失或網路錯誤，回傳友善的錯誤訊息。
4.3 使用者介面與體驗 (UI/UX)
深色模式 (Dark Mode): 預設為 class="dark"，使用 slate-950 作為背景基底，slate-900 為元件背景，slate-800 為邊框，營造專業、低視覺疲勞的環境。
動畫效果:
animate-pulse: 用於 Loading 狀態。
animate-spin: 用於文件上傳中。
transition-all: 確保按鈕、側邊欄切換滑順。
響應式佈局:
利用 Tailwind 的斷點 (lg:, xl:)。
側邊欄在 lg 以下解析度變為 fixed 定位，並由 MainLayout 控制 translate-x 進行滑入滑出。
5. 資料模型 (Data Models)
5.1 DocumentMetadata
描述單一文件的核心屬性。
code
TypeScript
interface DocumentMetadata {
  id: string;           // UUID
  name: string;         // 原始檔名
  type: string;         // MIME type
  size: number;         // Bytes
  uploadDate: number;   // Timestamp
  status: 'processing' | 'ready' | 'error';
  summary?: string;     // AI 生成摘要
  riskScore?: number;   // 0-100 合規風險
  category?: 'Clinical Trial' | 'Regulatory' | 'Research' | 'Unknown';
}
5.2 AgentConfig
描述當前 AI 代理人的行為模式。
code
TypeScript
interface AgentConfig {
  persona: AgentPersona; // 列舉值：REGULATORY, CLINICAL, SUMMARIZER
  model: string;         // 模型 ID (e.g., gemini-3-pro-preview)
  temperature: number;   // 隨機性控制
}
6. 介面與互動設計規範 (Interface Guidelines)
6.1 色彩系統
Primary (主要色): Blue-600 (#2563eb) - 代表科技、信任。
Accent (強調色): Emerald-500 (#10b981) - 代表醫療安全、合規通過。
Danger (危險色): Red-500 (#ef4444) - 代表高風險、合規失敗。
Neutral (中性色): Slate-900 系列 - 提供深色背景的高對比度與層次感。
6.2 字體系統
Sans-serif: Inter - 用於一般介面文字，提供極佳的螢幕閱讀性。
Monospace: JetBrains Mono - 用於代碼片段、ID 顯示、系統日誌訊息。
6.3 互動反饋
Loading 狀態: 必須在所有非同步操作（上傳、AI 生成）期間顯示明確的視覺指標（骨架屏或 Spinner）。
錯誤提示: 若 API 呼叫失敗，需在對話視窗中以 System Message 形式呈現，而非僅在 Console 報錯。
7. 安全性與限制 (Security & Limitations)
7.1 API Key 管理
目前實作依賴 process.env.API_KEY。在生產環境中，建議透過後端 Proxy 轉發請求，避免將 API Key 暴露於前端代碼中。
7.2 隱私聲明
系統介面上明確標示「Zero-Trust Analysis」與「Do not input PII (Personally Identifiable Information)」，提醒使用者勿上傳真實病患個資。
7.3 瀏覽器限制
由於採用純前端實作，大型 PDF 的文本提取目前使用 Mock Data 替代。實際部署需整合 pdf.js 或 Server-side OCR 解決方案以處理真實二進位檔案。
Context Window 限制：程式碼中主動截斷文本至 30,000 字元，以確保回應速度與 Token 消耗控制。
8. 未來擴充藍圖 (Future Roadmap)
真實 PDF 解析整合: 引入 WebAssembly 版本的 PDF 解析庫，在瀏覽器端實現本地 OCR 與文字提取。
向量資料庫 (Vector Database) 整合: 針對超大型文件（數百頁），改用 RAG 架構，將文本向量化存儲（如 ChromaDB 或 Web-local vector store），實現精準檢索而非全文注入。
多模態分析 (Multimodal Analysis): 利用 Gemini 的原生多模態能力，直接分析文件中的圖表（如臨床數據統計圖）、醫學影像截圖。
對話匯出與報告生成: 支援將 AI 分析結果與對話過程匯出為 PDF 報告，供法規提交使用。
9. 結論 (Conclusion)
Aegis 系統展示了 React 前端架構與 Google GenAI 強大能力的完美結合。透過精心的 Prompt Engineering 與結構化數據輸出（JSON Schema），即使是純前端應用也能執行複雜的法規風險評估任務。其模組化的程式碼結構（Contexts, Services, Components 分離）確保了系統的可維護性與未來的擴充彈性，為醫療法規領域的數位轉型提供了一個強大的技術原型。
