# Research: 後台管理系統技術選型

**Feature**: 001-admin-dashboard  
**Date**: 2025-01-27  
**Purpose**: 解決技術上下文中的 NEEDS CLARIFICATION 項目，並確立最佳實務

## 資料庫選擇

### Decision: PostgreSQL

**Rationale**:

- **資料完整性**: PostgreSQL 提供強型別檢查與豐富的約束條件，適合金融相關資料
- **擴展性**: 支援大量資料（10,000+ 客戶，100,000+ 交易記錄）且效能優異
- **JSON 支援**: 原生支援 JSON/JSONB，便於儲存彈性的試算參數與結果
- **開源且成熟**: 廣泛使用於生產環境，社群支援完善
- **Next.js 整合**: 與 Prisma、Drizzle ORM 等工具整合良好
- **交易支援**: ACID 特性確保資料一致性，適合金融資料

**Alternatives considered**:

- **MySQL**: 考慮過但 PostgreSQL 的 JSON 支援與型別系統更適合此專案
- **SQLite**: 適合開發環境，但生產環境需要更強的並發處理能力
- **MongoDB**: NoSQL 不適合需要強一致性的金融資料

### Decision: Prisma ORM

**Rationale**:

- **型別安全**: 自動產生 TypeScript 型別，與 Next.js TypeScript 專案完美整合
- **遷移管理**: 內建遷移工具，便於版本控制與部署
- **開發體驗**: 優秀的開發工具與直觀的查詢 API
- **效能**: 查詢優化與連線池管理
- **Next.js 整合**: 官方推薦的 ORM，與 Next.js 整合良好

**Alternatives considered**:

- **Drizzle ORM**: 較新但生態系統較小
- **TypeORM**: 功能完整但設定較複雜
- **原生 SQL**: 型別安全性較差，開發效率較低

## 第三方登入實作

### Decision: NextAuth.js (Auth.js v5)

**Rationale**:

- **Next.js 原生整合**: 專為 Next.js 設計，與 App Router 完美整合
- **多供應商支援**: 原生支援 Google OAuth，可擴展支援 LINE
- **安全性**: 內建 CSRF 保護、工作階段管理、JWT 處理
- **開發體驗**: 簡單的設定與豐富的文件
- **型別安全**: 完整的 TypeScript 支援

**LINE 登入實作方式**:

- 使用 NextAuth.js 的 Custom Provider 功能
- 實作 LINE Login API 的 OAuth 2.0 流程
- 儲存 LINE 使用者 ID 與基本資訊

**Alternatives considered**:

- **Clerk**: 商業解決方案，功能完整但需要付費
- **Supabase Auth**: 需要 Supabase 基礎設施
- **自建 OAuth**: 開發與維護成本過高

## 狀態管理與 API 串接

### Decision: Redux Toolkit + RTK Query + Axios

**Rationale**:
採用 Redux Toolkit 生態系統進行狀態管理與 API 串接，提供完整的資料流管理：

**Redux Toolkit 用途**:

- **全域狀態管理**: 管理應用程式的全域狀態（如使用者認證狀態、UI 狀態）
- **型別安全**: 完整的 TypeScript 支援，提供型別安全的狀態管理
- **開發工具**: Redux DevTools 提供優秀的除錯體驗
- **可預測性**: 單向資料流，狀態變更可追蹤與預測
- **符合 DDD**: 可與 DDD 架構整合，應用層狀態與展示層分離

**RTK Query 用途**:

- **API 資料快取**: 自動快取 API 回應，減少不必要的請求
- **自動重新驗證**: 支援資料自動重新驗證與背景更新
- **樂觀更新**: 支援樂觀更新，提升使用者體驗
- **請求去重**: 自動合併相同請求，避免重複 API 呼叫
- **型別安全**: 完整的 TypeScript 支援，自動產生型別定義

**Axios 用途**:

- **HTTP 客戶端**: 作為 RTK Query 的基礎 HTTP 客戶端
- **請求攔截器**: 統一處理認證 token、錯誤處理、請求/回應轉換
- **回應攔截器**: 統一處理錯誤回應、狀態碼處理
- **取消請求**: 支援請求取消，避免記憶體洩漏
- **型別安全**: 與 TypeScript 整合良好

**使用策略**:

- **RTK Query**: 用於所有 API 資料取得與快取（客戶列表、客戶詳情、試算結果）
- **Redux Toolkit Slices**: 用於 UI 狀態管理（表單狀態、篩選條件、分頁狀態）
- **Axios 實例**: 建立統一的 Axios 實例，配置基礎 URL、認證、錯誤處理
- **型別生成**: 使用 RTK Query 自動產生 API 端點的型別定義

**實作方式**:

- 建立 `src/presentation/store/api/baseApi.ts` - RTK Query base API
- 建立 `src/presentation/store/api/customersApi.ts` - 客戶相關 API endpoints
- 建立 `src/presentation/store/api/retirementApi.ts` - 退休金試算 API endpoints
- 建立 `src/presentation/store/slices/` - Redux slices（UI 狀態）
- 建立 `src/infrastructure/external/http/axiosInstance.ts` - Axios 實例配置
- 在 Next.js 中使用 Redux Provider 包裝應用程式

**與 DDD 架構整合**:

- **Presentation Layer**: 使用 RTK Query hooks 呼叫 API
- **Application Layer**: API endpoints 對應應用層用例
- **Infrastructure Layer**: Axios 實例屬於基礎設施層
- **型別對應**: RTK Query 產生的型別對應應用層 DTO

**Alternatives considered**:

- **React Server Components + Server Actions**: 更適合 SSR，但此專案需要客戶端狀態管理與快取
- **SWR/React Query**: 功能類似 RTK Query，但 Redux Toolkit 提供更完整的狀態管理方案
- **Zustand**: 輕量但缺乏 RTK Query 的 API 快取功能
- **原生 Fetch**: 功能較基礎，需要自行實作快取與錯誤處理

## 資料驗證

### Decision: Zod + Yup (混合使用)

**Rationale**:
採用兩種驗證庫，各自發揮優勢：

**Zod 用途**:

- **API 層驗證**: 用於 Next.js API Routes 和 Server Actions 的輸入驗證
- **領域層驗證**: 用於領域實體與值物件的驗證規則
- **型別安全**: 與 TypeScript 完美整合，自動推斷型別
- **執行時驗證**: 同時提供編譯時與執行時驗證
- **Next.js 整合**: 與 Server Actions 和 API Routes 整合良好
- **輕量高效**: 無額外依賴，效能優異

**Yup 用途**:

- **前端表單驗證**: 用於 React 表單元件的即時驗證
- **React Hook Form 整合**: 與 React Hook Form 完美整合，提供優秀的表單驗證體驗
- **使用者體驗**: 提供即時驗證回饋，提升使用者體驗
- **表單特定功能**: 提供豐富的表單驗證功能（如條件驗證、異步驗證）

**使用策略**:

- **Zod**: 用於後端（API Routes、Server Actions）、領域層驗證、資料傳輸物件驗證
- **Yup**: 用於前端表單驗證、React Hook Form 整合、即時使用者輸入驗證
- **驗證規則同步**: 確保 Zod 與 Yup 的驗證規則保持一致，避免前後端驗證不一致

**實作方式**:

- 在領域層與應用層使用 Zod 定義驗證規則
- 在前端表單元件使用 Yup 定義驗證規則
- 建立驗證規則對照表，確保規則一致性
- 考慮使用工具函數將 Zod Schema 轉換為 Yup Schema（如需要）

**Alternatives considered**:

- **僅使用 Zod**: 但 React Hook Form 與 Yup 整合更成熟，表單驗證體驗更好
- **僅使用 Yup**: 但 Zod 的 TypeScript 支援更強，更適合後端驗證
- **Joi**: 較重且主要用於 Node.js 後端，不適合前端使用
- **自建驗證**: 開發與維護成本過高

## 測試策略

### Decision: Jest + React Testing Library + Playwright

**Rationale**:

- **Jest**: Next.js 官方推薦的測試框架，設定簡單
- **React Testing Library**: 專注於使用者行為測試，符合測試原則
- **Playwright**: 優秀的 E2E 測試工具，支援多瀏覽器測試
- **覆蓋率**: Jest 提供內建覆蓋率報告，符合憲章要求

**測試層級**:

- **單元測試**: 工具函數、驗證邏輯、計算函數
- **元件測試**: UI 元件、表單驗證、互動邏輯
- **整合測試**: API Routes、資料庫操作、認證流程
- **E2E 測試**: 完整使用者流程（登入、查看客戶、試算）

**Alternatives considered**:

- **Vitest**: 較新但 Jest 生態系統更成熟
- **Cypress**: 功能強大但 Playwright 效能更好且支援多瀏覽器

## 效能優化策略

### Decision: Next.js 內建優化 + 資料庫索引

**Rationale**:

- **Next.js 自動優化**: 圖片優化、程式碼分割、SSR/SSG
- **資料庫索引**: 在客戶編號、投資日期等常用查詢欄位建立索引
- **分頁**: 客戶列表使用分頁，避免一次載入過多資料
- **快取策略**: 使用 Next.js 快取機制與資料庫查詢快取

**效能目標達成方式**:

- 客戶列表查詢 < 3 秒：使用分頁（每頁 50 筆）+ 資料庫索引
- API 回應 < 500ms：優化資料庫查詢，使用 Prisma 查詢優化
- 頁面載入 < 2 秒：使用 SSR 與適當的快取策略

## 無障礙性實作

### Decision: 語義化 HTML + ARIA + 鍵盤導航

**Rationale**:

- **Tailwind CSS**: 提供無障礙工具類別（如 `sr-only`）
- **Next.js**: 預設使用語義化 HTML
- **React**: 支援 ARIA 屬性
- **測試**: 使用 jest-axe 進行無障礙性測試

**實作重點**:

- 所有互動元素支援鍵盤導航
- 表單標籤與錯誤訊息正確關聯
- 色彩對比度符合 WCAG AA 標準
- 螢幕閱讀器支援（使用語義化標籤與 ARIA）

## 架構設計

### Decision: Domain-Driven Design (DDD) 分層架構

**Rationale**:

- **業務邏輯分離**: 將核心業務邏輯集中在領域層，與技術細節分離，提升可維護性
- **可測試性**: 領域層不依賴外部框架，易於進行單元測試，符合憲章測試標準
- **可擴展性**: 清晰的層級劃分使系統易於擴展，新功能可獨立開發
- **團隊協作**: 領域模型作為通用語言，促進技術與業務團隊的溝通
- **技術無關性**: 領域層不依賴特定技術，未來可輕鬆更換技術棧
- **符合憲章**: DDD 架構強調模組化設計與程式品質，完全符合專案憲章要求

**架構層級**:

1. **Domain Layer (領域層)**

   - **職責**: 核心業務邏輯、領域實體、值物件、領域服務
   - **特點**: 不依賴任何外部框架或技術
   - **實作**: TypeScript 類別與介面，純業務邏輯
   - **測試**: 單元測試，無需模擬外部依賴

2. **Application Layer (應用層)**

   - **職責**: 用例實作、協調領域物件、DTO 轉換
   - **特點**: 依賴領域層，實作具體業務用例
   - **實作**: 用例類別，使用依賴注入
   - **測試**: 整合測試，模擬儲存庫與外部服務

3. **Infrastructure Layer (基礎設施層)**

   - **職責**: 資料持久化、外部服務整合、框架實作
   - **特點**: 實作領域層定義的介面
   - **實作**: Prisma 儲存庫、NextAuth.js 適配器、匯出服務
   - **測試**: 整合測試，使用測試資料庫

4. **Presentation Layer (展示層)**
   - **職責**: 使用者介面、API Routes、請求處理
   - **特點**: 依賴應用層，處理 HTTP 請求與回應
   - **實作**: Next.js App Router、React 元件
   - **測試**: E2E 測試、元件測試

**依賴方向**: Presentation → Application → Domain ← Infrastructure

**實作模式**:

- **Repository Pattern**: 領域層定義介面，基礎設施層實作
- **Use Case Pattern**: 應用層實作具體業務用例
- **DTO Pattern**: 應用層定義 DTO，用於層級間資料傳輸
- **Mapper Pattern**: 基礎設施層實作領域模型與資料模型映射

**Alternatives considered**:

- **MVC 架構**: 較簡單但業務邏輯容易分散，不符合複雜業務需求
- **Clean Architecture**: 與 DDD 類似，但 DDD 更強調領域模型與通用語言
- **三層架構**: 傳統分層，但缺乏領域驅動的設計理念

### DDD 實作細節

**領域模型設計**:

- **實體 (Entities)**: 具有唯一識別碼的領域物件（如 Customer、Administrator）
- **值物件 (Value Objects)**: 無識別碼的不可變物件（如 Money、CustomerNumber）
- **領域服務 (Domain Services)**: 不屬於特定實體的業務邏輯（如 RetirementCalculationService）
- **儲存庫介面 (Repository Interfaces)**: 定義在領域層，實作在基礎設施層

**應用層用例**:

- 每個用例對應一個功能需求（如 GetCustomerList、CalculateRetirement）
- 用例類別接收依賴（儲存庫、服務）透過建構函數注入
- 用例返回 DTO，而非領域實體（保持層級隔離）

**依賴注入**:

- 使用 TypeScript 介面定義依賴
- 在 Next.js API Routes 中手動建立依賴實例（或使用輕量 DI 容器）
- 測試時使用模擬物件注入

**領域事件 (如需要)**:

- 定義領域事件介面在領域層
- 實作事件處理在應用層或基礎設施層
- 用於解耦領域物件間的互動

## 總結

所有技術選型均基於 Next.js 生態系統最佳實務，並採用 DDD 架構確保：

- ✅ 開發效率與維護性（清晰的層級劃分）
- ✅ 型別安全與錯誤預防（TypeScript + Zod）
- ✅ 效能與擴展性（Next.js 優化 + 資料庫索引）
- ✅ 符合專案憲章要求（模組化設計、可測試性）
- ✅ 與現有技術棧（Next.js + Tailwind CSS）完美整合
- ✅ 業務邏輯集中管理（領域層）
- ✅ 技術細節隔離（基礎設施層）
