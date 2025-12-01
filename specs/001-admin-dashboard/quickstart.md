# Quick Start Guide: 後台管理系統

**Feature**: 001-admin-dashboard  
**Date**: 2025-01-27  
**Purpose**: 快速開始開發與測試指南

## 前置需求

### 系統需求

- Node.js 18+
- npm 或 yarn 或 pnpm
- PostgreSQL 14+ (開發環境可使用 Docker)

### 開發工具

- Git
- 程式碼編輯器（推薦 VS Code）
- 瀏覽器（Chrome、Firefox、Safari、Edge）

## 專案初始化

### 1. 建立 Next.js 專案

```bash
npx create-next-app@latest admin-dashboard --typescript --tailwind --app --no-src-dir
cd admin-dashboard
```

### 2. 安裝核心依賴

```bash
# 後端與領域層驗證
npm install next-auth@beta prisma @prisma/client zod

# 前端表單驗證
npm install yup react-hook-form @hookform/resolvers

# 狀態管理與 API 串接
npm install @reduxjs/toolkit react-redux axios

# TypeScript 型別定義
npm install -D @types/node @types/react @types/react-dom
```

### 3. 安裝開發工具

```bash
npm install -D jest @testing-library/react @testing-library/jest-dom
npm install -D @playwright/test
npm install -D eslint-config-next
```

## 環境設定

### 1. 建立環境變數檔案

建立 `.env.local` 檔案：

```env
# Database
DATABASE_URL="postgresql://user:password@localhost:5432/admin_dashboard?schema=public"

# NextAuth.js
NEXTAUTH_URL="http://localhost:3000"
NEXTAUTH_SECRET="your-secret-key-here" # 使用 openssl rand -base64 32 產生

# Google OAuth
GOOGLE_CLIENT_ID="your-google-client-id"
GOOGLE_CLIENT_SECRET="your-google-client-secret"

# LINE OAuth
LINE_CHANNEL_ID="your-line-channel-id"
LINE_CHANNEL_SECRET="your-line-channel-secret"
LINE_CALLBACK_URL="http://localhost:3000/api/auth/callback/line"
```

### 2. 設定資料庫

```bash
# 初始化 Prisma
npx prisma init

# 編輯 prisma/schema.prisma（參考 data-model.md）

# 執行遷移
npx prisma migrate dev --name init

# 產生 Prisma Client
npx prisma generate
```

## 專案結構設定 (DDD 架構)

### 1. 建立 DDD 分層目錄結構

```bash
# Domain Layer (領域層)
mkdir -p src/domain/{administrator,customer,retirement}/{entities,value-objects,services,repositories}

# Application Layer (應用層)
mkdir -p src/application/use-cases/{administrator,customer,retirement}
mkdir -p src/application/dtos
mkdir -p src/application/interfaces

# Infrastructure Layer (基礎設施層)
mkdir -p src/infrastructure/persistence/{repositories,mappers}
mkdir -p src/infrastructure/external/{auth,export}
mkdir -p src/infrastructure/logging

# Presentation Layer (展示層)
mkdir -p src/presentation/app/{api,auth,dashboard}
mkdir -p src/presentation/components/{ui,auth,customers,retirement,layout}

# Shared Kernel (共享核心)
mkdir -p src/shared/{types,errors,events}

# Config
mkdir -p src/config

# Tests
mkdir -p tests/{unit,integration,e2e}
mkdir -p tests/unit/{domain,application,infrastructure}
```

### 2. 設定 Tailwind CSS

確認 `tailwind.config.ts` 包含（符合 DDD 架構路徑）：

```typescript
import type { Config } from "tailwindcss";

const config: Config = {
  content: [
    "./src/presentation/**/*.{js,ts,jsx,tsx,mdx}",
    "./src/presentation/app/**/*.{js,ts,jsx,tsx,mdx}",
    "./src/presentation/components/**/*.{js,ts,jsx,tsx,mdx}",
  ],
  theme: {
    extend: {},
  },
  plugins: [],
};
export default config;
```

## 開發流程

### 1. 啟動開發伺服器

```bash
npm run dev
```

開啟瀏覽器訪問 `http://localhost:3000`

### 2. 資料庫管理

```bash
# 查看資料庫（Prisma Studio）
npx prisma studio

# 執行遷移
npx prisma migrate dev

# 重置資料庫（開發環境）
npx prisma migrate reset
```

### 3. 測試

```bash
# 單元測試
npm run test

# 整合測試
npm run test:integration

# E2E 測試
npm run test:e2e
```

## 實作順序建議 (DDD 架構)

### Phase 1: 認證系統（User Story 1 - P1）

#### 1.1 Domain Layer - 管理員領域

- 建立 `src/domain/administrator/entities/Administrator.ts` - 管理員實體
- 建立 `src/domain/administrator/value-objects/ProviderId.ts` - 供應商 ID 值物件
- 建立 `src/domain/administrator/repositories/IAdministratorRepository.ts` - 儲存庫介面
- 建立 `src/domain/administrator/services/AuthenticationService.ts` - 認證領域服務

#### 1.2 Application Layer - 認證用例

- 建立 `src/application/use-cases/administrator/AuthenticateAdministrator.ts` - 認證用例
- 建立 `src/application/use-cases/administrator/LogoutAdministrator.ts` - 登出用例

#### 1.3 Infrastructure Layer - 認證實作

- 建立 `src/infrastructure/persistence/repositories/AdministratorRepository.ts` - 儲存庫實作
- 建立 `src/infrastructure/external/auth/NextAuthAdapter.ts` - NextAuth.js 適配器
- 建立 `src/infrastructure/external/auth/LineProvider.ts` - LINE Provider 實作
- 設定 `src/config/auth.ts` - NextAuth.js 設定

#### 1.4 Presentation Layer - 認證 UI

- 建立 `src/presentation/app/api/auth/[...nextauth]/route.ts` - NextAuth.js API Route
- 建立 `src/presentation/app/auth/login/page.tsx` - 登入頁面
- 建立 `src/presentation/components/auth/LoginForm.tsx` - 登入表單元件

#### 1.5 測試

- 單元測試：領域實體、值物件、領域服務
- 整合測試：認證用例、儲存庫實作
- E2E 測試：完整登入流程

### Phase 2: 客戶投資狀況（User Story 2 - P2）

#### 2.1 Domain Layer - 客戶領域

- 建立 `src/domain/customer/entities/Customer.ts` - 客戶實體
- 建立 `src/domain/customer/entities/InvestmentPortfolio.ts` - 投資組合實體
- 建立 `src/domain/customer/entities/InvestmentRecord.ts` - 投資記錄實體
- 建立 `src/domain/customer/value-objects/CustomerNumber.ts` - 客戶編號值物件
- 建立 `src/domain/customer/value-objects/Money.ts` - 金額值物件
- 建立 `src/domain/customer/repositories/ICustomerRepository.ts` - 客戶儲存庫介面
- 建立 `src/domain/customer/repositories/IInvestmentRepository.ts` - 投資儲存庫介面

#### 2.2 Application Layer - 客戶用例

- 建立 `src/application/use-cases/customer/GetCustomerList.ts` - 取得客戶列表用例
- 建立 `src/application/use-cases/customer/GetCustomerDetail.ts` - 取得客戶詳情用例
- 建立 `src/application/use-cases/customer/ExportCustomerData.ts` - 匯出客戶資料用例
- 建立 `src/application/dtos/CustomerDto.ts` - 客戶 DTO

#### 2.3 Infrastructure Layer - 客戶實作

- 建立 Prisma Schema（參考 data-model.md）
- 執行資料庫遷移：`npx prisma migrate dev`
- 建立 `src/infrastructure/persistence/repositories/CustomerRepository.ts` - 客戶儲存庫實作
- 建立 `src/infrastructure/persistence/repositories/InvestmentRepository.ts` - 投資儲存庫實作
- 建立 `src/infrastructure/persistence/mappers/CustomerMapper.ts` - 領域模型映射
- 建立 `src/infrastructure/external/export/CsvExporter.ts` - CSV 匯出服務
- 建立 `src/infrastructure/external/export/PdfExporter.ts` - PDF 匯出服務

#### 2.4 Presentation Layer - 客戶 UI

- 建立 `src/presentation/app/api/customers/route.ts` - 客戶列表 API
- 建立 `src/presentation/app/api/customers/[id]/route.ts` - 客戶詳情 API
- 建立 `src/presentation/app/api/customers/[id]/export/route.ts` - 匯出 API
- 建立 `src/presentation/app/dashboard/customers/page.tsx` - 客戶列表頁面
- 建立 `src/presentation/app/dashboard/customers/[id]/page.tsx` - 客戶詳情頁面
- 建立 `src/presentation/components/customers/CustomerList.tsx` - 客戶列表元件
- 建立 `src/presentation/components/customers/CustomerDetail.tsx` - 客戶詳情元件

#### 2.5 測試

- 單元測試：領域實體、值物件、用例
- 整合測試：儲存庫、用例整合
- E2E 測試：客戶列表與詳情查看流程

### Phase 3: 退休金試算（User Story 3 - P3）

#### 3.1 Domain Layer - 退休金領域

- 建立 `src/domain/retirement/entities/RetirementCalculation.ts` - 試算實體
- 建立 `src/domain/retirement/value-objects/Age.ts` - 年齡值物件
- 建立 `src/domain/retirement/value-objects/ReturnRate.ts` - 報酬率值物件
- 建立 `src/domain/retirement/value-objects/RetirementAmount.ts` - 退休金額值物件
- 建立 `src/domain/retirement/services/RetirementCalculationService.ts` - 試算領域服務
- 建立 `src/domain/retirement/repositories/IRetirementCalculationRepository.ts` - 試算儲存庫介面

#### 3.2 Application Layer - 試算用例

- 建立 `src/application/use-cases/retirement/CalculateRetirement.ts` - 執行試算用例
- 建立 `src/application/use-cases/retirement/GetCalculationHistory.ts` - 取得試算歷史用例
- 建立 `src/application/dtos/RetirementCalculationDto.ts` - 試算 DTO

#### 3.3 Infrastructure Layer - 試算實作

- 建立 `src/infrastructure/persistence/repositories/RetirementCalculationRepository.ts` - 試算儲存庫實作
- 建立 `src/infrastructure/persistence/mappers/RetirementMapper.ts` - 試算映射

#### 3.4 Presentation Layer - 試算 UI

- 建立 `src/presentation/app/api/retirement/calculate/route.ts` - 試算 API
- 建立 `src/presentation/app/api/retirement/calculate/[id]/route.ts` - 取得試算記錄 API
- 建立 `src/presentation/app/dashboard/retirement/page.tsx` - 試算頁面
- 建立 `src/presentation/app/dashboard/retirement/[customerId]/page.tsx` - 特定客戶試算頁面
- 建立 `src/presentation/components/retirement/RetirementForm.tsx` - 試算表單元件
- 建立 `src/presentation/components/retirement/RetirementResult.tsx` - 試算結果元件

#### 3.5 測試

- 單元測試：領域服務（試算邏輯）、值物件驗證
- 整合測試：試算用例、儲存庫
- E2E 測試：完整試算流程

## 測試資料

### 建立測試資料腳本

建立 `prisma/seed.ts`：

```typescript
import { PrismaClient } from "@prisma/client";

const prisma = new PrismaClient();

async function main() {
  // 建立測試客戶與投資資料
  // ...
}

main()
  .catch((e) => {
    console.error(e);
    process.exit(1);
  })
  .finally(async () => {
    await prisma.$disconnect();
  });
```

執行種子資料：

```bash
npx prisma db seed
```

## 常見問題

### Q: NextAuth.js 設定問題

A: 確認 `NEXTAUTH_URL` 與 `NEXTAUTH_SECRET` 正確設定，檢查 OAuth 應用程式的回調 URL 設定。

### Q: 資料庫連線失敗

A: 確認 `DATABASE_URL` 格式正確，資料庫服務正在運行，連線權限正確。

### Q: Tailwind CSS 樣式未生效

A: 確認 `tailwind.config.ts` 的 `content` 路徑正確，檢查 CSS 檔案是否正確匯入。

### Q: TypeScript 型別錯誤

A: 執行 `npx prisma generate` 更新 Prisma Client 型別，確認所有依賴已安裝。

### Q: Zod 與 Yup 驗證規則不一致

A: 建立驗證規則對照表，確保前後端驗證規則一致。建議將驗證規則定義在共享位置，或使用工具函數進行轉換。

### Q: RTK Query 快取不更新

A: 確認使用 `providesTags` 和 `invalidatesTags` 正確設定快取標籤，使用 `refetch` 或 `invalidateTags` 手動更新快取。

### Q: Redux Store 在 Next.js SSR 中無法使用

A: 使用 `"use client"` 標記使用 Redux 的元件，或使用 Next.js 的 `use client` 指令建立客戶端元件。Store 應在客戶端建立。

### Q: Axios 攔截器無法取得 NextAuth session

A: 在 Next.js 中，需要在客戶端元件中使用 `useSession` hook 取得 session，或在攔截器中使用非同步方式取得 token。

## 狀態管理與 API 串接設定

### 1. 建立 Redux Store

建立 `src/presentation/store/store.ts`:

```typescript
import { configureStore } from "@reduxjs/toolkit";
import { baseApi } from "./api/baseApi";
import { customersSlice } from "./slices/customersSlice";

export const store = configureStore({
  reducer: {
    [baseApi.reducerPath]: baseApi.reducer,
    customers: customersSlice.reducer,
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(baseApi.middleware),
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

### 2. 建立 RTK Query Base API

建立 `src/presentation/store/api/baseApi.ts`:

```typescript
import { createApi, fetchBaseQuery } from "@reduxjs/toolkit/query/react";
import { axiosInstance } from "@/infrastructure/external/http/axiosInstance";

export const baseApi = createApi({
  reducerPath: "api",
  baseQuery: fetchBaseQuery({
    baseUrl: "/api",
    prepareHeaders: (headers, { getState }) => {
      // 從 NextAuth session 取得 token
      const token = getState().auth?.token;
      if (token) {
        headers.set("authorization", `Bearer ${token}`);
      }
      return headers;
    },
  }),
  tagTypes: ["Customer", "RetirementCalculation"],
  endpoints: () => ({}),
});
```

### 3. 建立 Axios 實例

建立 `src/infrastructure/external/http/axiosInstance.ts`:

```typescript
import axios from "axios";

export const axiosInstance = axios.create({
  baseURL: process.env.NEXT_PUBLIC_API_URL || "/api",
  timeout: 10000,
  headers: {
    "Content-Type": "application/json",
  },
});

// 請求攔截器
axiosInstance.interceptors.request.use(
  (config) => {
    // 從 NextAuth session 取得 token
    // const session = await getSession();
    // if (session?.accessToken) {
    //   config.headers.Authorization = `Bearer ${session.accessToken}`;
    // }
    return config;
  },
  (error) => {
    return Promise.reject(error);
  }
);

// 回應攔截器
axiosInstance.interceptors.response.use(
  (response) => response,
  (error) => {
    // 統一錯誤處理
    if (error.response?.status === 401) {
      // 處理未授權錯誤
    }
    return Promise.reject(error);
  }
);
```

### 4. 設定 Redux Provider

在 `src/presentation/app/layout.tsx` 中包裝應用程式:

```typescript
"use client";

import { Provider } from "react-redux";
import { store } from "@/presentation/store/store";

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="zh-TW">
      <body>
        <Provider store={store}>{children}</Provider>
      </body>
    </html>
  );
}
```

### 5. 建立 Typed Hooks

建立 `src/presentation/store/hooks.ts`:

```typescript
import { useDispatch, useSelector } from "react-redux";
import type { RootState, AppDispatch } from "./store";

export const useAppDispatch = useDispatch.withTypes<AppDispatch>();
export const useAppSelector = useSelector.withTypes<RootState>();
```

## DDD 架構開發注意事項

### 依賴方向

- **Presentation → Application → Domain**
- **Infrastructure → Domain** (實作領域層介面)
- **Domain 層不依賴任何其他層**

### 實作原則

1. **領域層保持純淨**: 不引入任何外部依賴（如 Prisma、Next.js）
2. **使用介面定義依賴**: 領域層定義介面，基礎設施層實作
3. **DTO 用於層級間傳輸**: 應用層定義 DTO，避免直接暴露領域實體
4. **Mapper 處理模型轉換**: 基礎設施層使用 Mapper 轉換資料模型與領域模型
5. **用例類別保持簡單**: 每個用例只做一件事，協調領域物件完成業務邏輯

### 驗證策略 (Zod + Yup)

**Zod 使用場景**:

- **領域層**: 值物件的驗證規則（如 `CustomerNumber.ts`、`Money.ts`）
- **應用層**: DTO 的驗證規則（如 `CustomerDto.ts`）
- **API Routes**: 請求參數與請求體的驗證
- **Server Actions**: 輸入參數驗證

**Yup 使用場景**:

- **前端表單**: React 表單元件的即時驗證
- **React Hook Form**: 與 `@hookform/resolvers` 整合，提供表單驗證
- **使用者輸入**: 即時驗證使用者輸入，提供即時回饋

**驗證規則同步**:

- 確保 Zod Schema 與 Yup Schema 的驗證規則保持一致
- 考慮建立驗證規則對照表或工具函數
- 前端驗證通過後，後端仍需使用 Zod 進行驗證（安全考量）

**實作範例**:

```typescript
// 領域層使用 Zod (src/domain/customer/value-objects/CustomerNumber.ts)
import { z } from "zod";

const CustomerNumberSchema = z.string().regex(/^CUST-\d{4}-\d{4}$/);

// 前端表單使用 Yup (src/presentation/components/customers/CustomerForm.tsx)
import * as yup from "yup";
import { useForm } from "react-hook-form";
import { yupResolver } from "@hookform/resolvers/yup";

const customerFormSchema = yup.object({
  customerNumber: yup.string().matches(/^CUST-\d{4}-\d{4}$/, "格式錯誤"),
  // ...
});
```

### 測試策略

- **領域層**: 純單元測試，無需模擬
- **應用層**: 整合測試，模擬儲存庫與服務
- **基礎設施層**: 整合測試，使用測試資料庫
- **展示層**: E2E 測試與元件測試
- **Redux Store**: 單元測試 Redux slices 與 RTK Query endpoints
- **API 串接**: 使用 MSW (Mock Service Worker) 模擬 API 回應

### 狀態管理與 API 串接注意事項

**RTK Query 使用原則**:

1. **API Endpoints**: 在 `src/presentation/store/api/` 中定義所有 API endpoints
2. **自動快取**: RTK Query 自動快取 API 回應，使用 `providesTags` 和 `invalidatesTags` 管理快取
3. **型別安全**: 使用 TypeScript 定義請求與回應型別
4. **錯誤處理**: 在元件中使用 `isError` 和 `error` 處理錯誤狀態

**Redux Slices 使用原則**:

1. **UI 狀態**: 僅用於 UI 相關狀態（如分頁、篩選、表單狀態）
2. **API 資料**: 使用 RTK Query 管理，不應在 Redux slices 中重複儲存
3. **型別安全**: 使用 TypeScript 定義 state 型別

**Axios 使用原則**:

1. **統一配置**: 使用單一 Axios 實例，統一處理認證、錯誤、請求/回應轉換
2. **攔截器**: 使用請求與回應攔截器處理通用邏輯
3. **錯誤處理**: 統一錯誤處理邏輯，轉換為應用程式可用的錯誤格式

**與 DDD 架構整合**:

- **Presentation Layer**: 使用 RTK Query hooks 呼叫 API，對應應用層用例
- **Application Layer**: API endpoints 對應應用層用例，RTK Query 型別對應 DTO
- **Infrastructure Layer**: Axios 實例屬於基礎設施層，處理 HTTP 通訊細節

## 下一步

完成快速開始後，參考以下文件繼續開發：

- [plan.md](./plan.md) - 完整實作計畫（包含 DDD 架構說明）
- [research.md](./research.md) - 技術選型與 DDD 架構實作方式
- [data-model.md](./data-model.md) - 資料模型定義
- [contracts/api.yaml](./contracts/api.yaml) - API 規格
- [spec.md](./spec.md) - 功能規格

## 開發檢查清單

- [ ] 專案初始化完成
- [ ] 環境變數設定完成
- [ ] 資料庫連線成功
- [ ] NextAuth.js 設定完成
- [ ] 登入功能測試通過
- [ ] 資料模型建立完成
- [ ] API Routes 實作完成
- [ ] 前端頁面實作完成
- [ ] 測試覆蓋率達到標準
- [ ] 無障礙性檢查通過
