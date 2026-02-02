Agentic MedReview AI 技術規格書
版本 (Version): 2.0
日期 (Date): 2024-02-02
專案代號 (Project Code): AI-Extend-020226-V2
語言 (Language): Traditional Chinese (繁體中文)
1. 執行摘要 (Executive Summary)
Agentic MedReview AI 是一款專為醫療器材法規事務（Regulatory Affairs）設計的前端智慧化系統。該系統利用 Google Gemini API 構建「代理人（Agentic）」架構，協助使用者進行 TFDA（台灣食藥署）及 FDA 相關文件的填寫、審查與差距分析。
本系統採用現代化 React 19 架構，強調無伺服器（Serverless）的客戶端運算能力。核心功能包含動態表單處理、即時 Markdown 文件生成、AI 智慧審閱、以及基於「花卉主題（Flower-Themed）」的高客製化 UI 體驗。系統內建多組模擬資料集（Mock Datasets），可快速展示不同等級醫材（Class II/III）的申請流程。
2. 系統架構 (System Architecture)
2.1 技術堆疊 (Tech Stack)
核心框架: React 19.2.4 (Client-Side Rendering)
語言: TypeScript (Strict Typing)
樣式系統: Tailwind CSS (支援 Dark Mode 與動態漸層)
圖表視覺化: Recharts 3.7.0
圖標庫: Lucide React 0.563.0
AI 引擎: Google GenAI SDK (@google/genai v1.39.0)
模組載入: ES Modules via esm.sh (無須傳統 Bundler 即可運作於瀏覽器環境)
2.2 應用程式結構 (Application Structure)
系統採用單一進入點（SPA）架構，狀態管理採用 React Context 或提升至頂層元件（Lifted State）的模式。
index.html: 應用程式入口，配置 Import Maps 與全域樣式。
App.tsx: 根元件，持有全域狀態 AppState，負責路由切換（Tabs）與背景主題渲染。
components/: 功能模組化元件。
Dashboard: 數據儀表板。
TWPremarket: 查驗登記表單主邏輯。
TFDAExtension: 展延申請檢查表。
AgentRunner: 通用型 AI 執行器 UI。
Sidebar: 導航與設定控制台。
services/: 封裝外部 API 呼叫邏輯 (geminiService.ts)。
types.ts: 定義全系統的 TypeScript 介面與資料模型。
constants.ts: 靜態資源、多語言字典、預設模擬資料集。
2.3 資料流與狀態管理
系統採用 Centralized State Management (集中式狀態管理) 模式。所有的業務數據、UI 偏好設定、AI 執行歷史紀錄皆儲存於 App.tsx 的 state 物件中，並透過 Props Drilling 或 Setter 傳遞至子元件。
資料持久性: 當前版本為純客戶端運作（In-Memory），刷新瀏覽器會重置狀態。
API 金鑰管理: 支援透過環境變數 (process.env.API_KEY) 自動載入或由使用者在側邊欄手動輸入。
3. 功能模組規格 (Functional Specifications)
3.1 儀表板 (Dashboard)
儀表板負責視覺化呈現使用者與 AI 代理人的互動歷史與資源消耗。
關鍵指標卡 (KPI Cards):
Total Runs: 累計 AI 執行次數。
Total Tokens: 估算的 Token 消耗量（基於字元數/4 的簡易估算）。
Success Rate: AI 回應成功率（排除 API 錯誤）。
Errors: 執行失敗次數。
圖表分析:
Activity by Module (Bar Chart): 依據功能模組（Dashboard, TW Premarket, TFDA Extension 等）統計使用頻率。圖表顏色會根據當前選定的花卉主題動態變化。
Token Usage Timeline (Line Chart): 時間軸呈現 Token 消耗趨勢。
近期活動列表: 顯示最近 5 筆 AI 執行紀錄，包含時間戳記、Agent 名稱、模型版本及執行狀態。
3.2 台灣查驗登記模組 (TW Premarket)
此為系統的核心業務模組，模擬 TFDA Class II/III 醫療器材查驗登記申請書的填寫與生成。
3.2.1 資料輸入與驗證
表單結構: 依據 TwPremarketForm 介面定義，分為五大區塊：
案件基本資料: 文號、申請日、案件類型、產品等級（Class II/III）。
醫材基本資訊: 中英文名稱、適應症、型號規格。
醫療器材商資料: 申請商資訊、聯絡人、QMS/QSD 狀態。
製造廠資訊: 產地、製造廠名稱與地址。
附件摘要: 行政文件、技術性資料（臨床前測試、臨床證據）。
完整度計算 (Completeness Logic): 系統即時監聽 mandatoryFields 陣列中的欄位，動態計算填寫百分比，並以圓環圖（SVG Ring Chart）即時呈現。若未達 100%，系統會提示 "Incomplete"。
3.2.2 模擬資料載入 (Mock Data Injection)
系統內建 DEFAULT_DATASETS（定義於 constants.ts），提供三種典型案例供使用者快速測試：
Case 1: 美國進口 Class II 心導管氣球（CardioX PTCA）。
Case 2: 國產 Class II 骨科骨釘（OrthoFix Bone Screw）。
Case 3: 日本進口 Class III 體外診斷試劑（ViralDetect IVD Kit）。
使用者可透過下拉選單一鍵填入所有表單欄位。
3.2.3 資料匯出 (Data Export)
JSON 匯出: 將當前表單狀態序列化為 JSON 檔案下載。
CSV 匯出: 將表單平面化為 CSV 格式。
技術細節: 自動添加 BOM (\uFEFF) 以確保 Microsoft Excel 開啟時 UTF-8 編碼正確顯示。字串欄位會自動處理引號跳脫（Escape Quotes）。
3.2.4 AI 生成與審閱 (Agentic Workflow)
Markdown 生成: 前端根據表單內容，透過樣板字串（Template Literals）組合成結構化的 Markdown 草稿。
Agent 執行: 整合 AgentRunner 元件，將生成的 Markdown 作為 Context 傳送給 Gemini 模型。
預設 Prompt: "You are a TFDA expert reviewer. Review the provided application markdown for completeness, logic errors, and regulatory compliance."
3.3 TFDA 展延模組 (TFDA Extension)
針對許可證展延（License Extension）設計的檢查清單與差距分析工具。
封包結構 (Packet Structure): 定義了 S1 至 S9 共 9 個標準章節（如原許可證、CFS、LoA、QMS 等）。
適用性切換: 使用者可針對每個章節切換 "Applicable" / "Not Applicable"。
Gap Analysis Agent:
系統會將目前的勾選狀態彙整成摘要文字。
呼叫 AI 進行邏輯檢查（例如：若勾選「國產」，是否不應出現 CFS 文件缺失的警告等）。
3.4 代理人設定 (Agents Config)
提供一個通用的聊天介面，允許使用者自定義 System Prompt，作為通用的法規諮詢助手。
4. AI 整合與模型策略 (AI Integration Strategy)
4.1 模型選擇機制
系統支援多種 Google Gemini 模型，並根據任務複雜度建議不同模型：
Gemini 3 Flash Preview: 用於快速回應、一般對話及基礎檢查。
Gemini 3 Pro Preview: 用於複雜推理、醫療器材臨床評估報告（CER）的深度分析。
Gemini 2.5 Flash Lite: 用於輕量級任務或 Markdown 格式整理。
4.2 API 服務層 (geminiService.ts)
封裝: 使用 @google/genai SDK。
參數控制:
temperature: 預設 0.2，保持回應的穩定性與專業度，減少幻覺（Hallucination）。
systemInstruction: 強制設定 AI 的角色（Persona），例如 "TFDA Expert Reviewer"。
錯誤處理: 包含 API Key 缺失檢查及通用的 Try-Catch 機制，確保前端不會因 API 失敗而崩潰。
5. 使用者介面與體驗設計 (UI/UX Design)
5.1 花卉主題系統 (Flower Theme System)
系統最具特色的視覺設計，旨在降低法規工作的枯燥感。
定義: FlowerStyle 介面包含 name, gradient (CSS Linear Gradient), cardBg (半透明背景色), textColor。
樣式庫: 內建 20 種花卉主題（如 Rose, Lavender, Sunflower 等）。
動態渲染: FlowerBackground 元件透過 React Props 接收當前樣式，並即時套用至全域背景。
5.2 響應式與深色模式 (Responsive & Dark Mode)
Tailwind CSS: 使用 Utility-first CSS 進行排版。
Dark Mode: 支援手動切換。
實作細節: 在 Dark Mode 下，背景會覆蓋為深藍色系 (#1a1a2e) 以確保文字可讀性，並調整表單輸入框的透明度與邊框顏色。
佈局: 採用 "Sidebar + Main Content" 的經典儀表板佈局。
5.3 國際化 (i18n)
支援 English 與 Traditional Chinese (繁體中文) 切換。
使用 LABELS 常數物件進行簡單的鍵值對照（Key-Value Mapping）。
6. 資料模型定義 (Data Models)
以下列出系統核心 TypeScript 介面定義：
6.1 AppState (全域狀態)
code
TypeScript
interface AppState {
  theme: 'light' | 'dark';
  language: 'English' | 'Traditional Chinese';
  currentStyle: FlowerStyle;
  apiKey: string;
  history: AgentRunHistory[];       // AI 執行歷史
  twForm: TwPremarketForm;          // 查驗登記表單資料
  extensionPacket: Record<string, ExtensionSection>; // 展延封包資料
  clipboard: { text: string; source: string; ts: string }; // 內部剪貼簿
}
6.2 TwPremarketForm (查驗登記表單)
包含詳細的法規欄位，如：
doc_no, e_no: 公文號與流水號。
device_category, product_class: 醫材分類與分級。
manu_country, manu_type: 製造廠國別與型態。
md_content: 最終生成的 Markdown 內容。
6.3 AgentRunHistory (執行紀錄)
用於儀表板統計：
code
TypeScript
interface AgentRunHistory {
  id: string;
  tab: string;      // 來源模組
  agent: string;    // Agent 名稱
  model: string;    // 使用模型
  tokens_est: number; // 估計 Token
  status: 'pending' | 'running' | 'done' | 'error';
  ts: string;       // 時間戳記
}
7. 安全性與效能考量 (Security & Performance)
7.1 安全性
API Key 處理:
API Key 僅儲存於瀏覽器記憶體（React State）或環境變數中。
警告: 由於是純前端應用，若佈署至公開網路，不應將 API Key 硬編碼於原始碼中。建議使用者自行輸入 Key，或在受控的內網環境使用環境變數。
資料隱私: 所有資料處理均在客戶端完成，除了發送至 Google Gemini API 的 Prompt 外，資料不會傳送至任何第三方伺服器。
7.2 效能
渲染優化: 使用 React 的虛擬 DOM 機制。
資源載入: 透過 CDN (esm.sh) 載入 React 與相關庫，利用瀏覽器快取機制。
Mock Data: 模擬資料直接編譯於 constants.ts，無須額外的 HTTP 請求，實現零延遲載入。
8. 未來擴充路線 (Future Roadmap)
後端整合: 引入 Supabase 或 Firebase 以實現資料持久化與使用者帳號管理。
RAG (Retrieval-Augmented Generation): 整合向量資料庫，讓 AI 能參考具體的 TFDA 法規條文或過往核准案例進行回答。
PDF 解析: 增加上傳功能，自動解析既有的 PDF 申請書並填入表單。
多代理人協作 (Multi-Agent System): 建立 "Reviewer" Agent 與 "Writer" Agent 的對話迴圈，自動修正表單錯誤。
結語:
Agentic MedReview AI 展示了如何利用現代前端技術結合生成式 AI，將繁瑣的醫療器材法規文件作業流程自動化、智慧化。其模組化的設計與清晰的資料流，為未來的擴充奠定了堅實的基礎。
