# Implementation Plan: 後台管理系統

**Branch**: `001-admin-dashboard` | **Date**: 2025-01-27 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/001-admin-dashboard/spec.md`

**Note**: This template is filled in by the `/speckit.plan` command. See `.specify/templates/commands/plan.md` for the execution workflow.

## Summary

建立一個後台管理系統，提供管理員查看客戶投資狀況與退休金試算功能。系統採用 Domain-Driven Design (DDD) 架構，使用 Next.js 框架搭配 Tailwind CSS 進行開發，支援 LINE 與 Google 第三方登入，提供完整的身份驗證、資料查詢與試算功能。採用 DDD 架構確保領域邏輯與業務規則的清晰分離，提升程式碼的可維護性與可測試性。

## Technical Context

**Language/Version**: TypeScript 5.x, JavaScript (ES2022+)  
**Primary Dependencies**: Next.js 14+, React 18+, Tailwind CSS 3+, NextAuth.js (Auth.js)  
**Architecture**: Domain-Driven Design (DDD) - 分層架構  
**Storage**: PostgreSQL 14+ (使用 Prisma ORM)  
**Validation**: Zod (後端/領域層) + Yup (前端表單)  
**State Management**: Redux Toolkit + RTK Query + Axios  
**Testing**: Jest, React Testing Library, Playwright (E2E)  
**Target Platform**: Web (現代瀏覽器，支援 SSR/SSG)  
**Project Type**: web (單一 Next.js 應用程式，包含前端與 API routes)  
**Performance Goals**:

- 頁面載入時間 < 2 秒（首次載入）
- API 回應時間 < 500ms (p95)
- 支援至少 50 位同時在線管理員
- 客戶列表查詢（1000 筆）< 3 秒  
  **Constraints**:
- 必須符合 WCAG 2.1 AA 無障礙標準
- 必須支援響應式設計（桌面與平板）
- 必須實作完整的錯誤處理與使用者回饋
- 必須符合專案憲章的測試覆蓋率要求（核心邏輯 ≥80%，關鍵路徑 =100%）  
  **Scale/Scope**:
- 預期管理員數量：50-100 位
- 預期客戶資料量：10,000+ 筆客戶記錄
- 預期投資記錄：100,000+ 筆交易記錄
- 主要頁面：登入頁、儀表板、客戶列表、客戶詳情、退休金試算

## Constitution Check

_GATE: Must pass before Phase 0 research. Re-check after Phase 1 design._

### I. 程式品質

- ✅ **可讀性優先**: 使用 TypeScript 提供型別安全，採用清晰的命名慣例
- ✅ **模組化設計**: 採用 DDD 分層架構，確保清晰的職責分離與模組化設計
- ✅ **錯誤處理**: 實作統一的錯誤處理機制與使用者友善的錯誤訊息
- ✅ **程式碼審查**: 所有變更將透過 PR 流程進行審查
- ✅ **技術債務管理**: 建立技術債務追蹤機制

### II. 測試標準

- ✅ **測試覆蓋率**: 目標核心業務邏輯 ≥80%，關鍵路徑（登入、資料查詢）=100%
- ✅ **測試層級**: 規劃單元測試（Jest）、整合測試（React Testing Library）、E2E 測試（Playwright）
- ✅ **測試優先開發**: 遵循 TDD 原則進行開發
- ✅ **自動化測試**: 整合至 CI/CD 流程
- ✅ **測試可維護性**: 保持測試程式碼品質與可讀性

### III. 使用者體驗一致性

- ✅ **設計系統遵循**: 使用 Tailwind CSS 建立一致的設計系統
- ✅ **回應式設計**: Tailwind CSS 提供完整的響應式設計支援
- ✅ **無障礙性**: 實作 WCAG 2.1 AA 標準（使用語義化 HTML、ARIA 屬性、鍵盤導航）
- ✅ **錯誤訊息一致性**: 建立統一的錯誤訊息元件與格式
- ✅ **載入與回饋**: 實作載入狀態與進度指示器

### IV. 效能需求

- ✅ **回應時間標準**:
  - 登入流程 < 2 秒（符合 SC-001: 30 秒內完成）
  - 客戶列表查詢 < 3 秒（符合 SC-002）
  - 客戶詳情查詢 < 5 秒（符合 SC-003）
  - 試算計算 < 2 秒（符合 SC-004）
- ✅ **效能監控**: 整合 Next.js 效能監控工具，追蹤 API 回應時間與前端渲染時間
- ✅ **效能測試**: 進行負載測試確保支援 50 位同時在線管理員
- ✅ **資源優化**: 使用 Next.js 自動優化（圖片優化、程式碼分割、SSR/SSG）
- ✅ **擴展性考量**: 設計可擴展的資料庫架構與 API 結構

**憲章合規狀態**: ✅ 通過 - 所有原則均符合要求

## Project Structure

### Documentation (this feature)

```text
specs/001-admin-dashboard/
├── plan.md              # This file (/speckit.plan command output)
├── research.md          # Phase 0 output (/speckit.plan command)
├── data-model.md        # Phase 1 output (/speckit.plan command)
├── quickstart.md        # Phase 1 output (/speckit.plan command)
├── contracts/           # Phase 1 output (/speckit.plan command)
└── tasks.md             # Phase 2 output (/speckit.tasks command - NOT created by /speckit.plan)
```

### Source Code (repository root) - DDD 架構

```text
src/
├── domain/                    # Domain Layer (領域層)
│   ├── administrator/         # 管理員領域
│   │   ├── entities/          # 領域實體
│   │   │   └── Administrator.ts
│   │   ├── value-objects/      # 值物件
│   │   │   └── ProviderId.ts
│   │   ├── services/          # 領域服務
│   │   │   └── AuthenticationService.ts
│   │   └── repositories/      # 儲存庫介面
│   │       └── IAdministratorRepository.ts
│   ├── customer/              # 客戶領域
│   │   ├── entities/
│   │   │   ├── Customer.ts
│   │   │   ├── InvestmentPortfolio.ts
│   │   │   └── InvestmentRecord.ts
│   │   ├── value-objects/
│   │   │   ├── CustomerNumber.ts
│   │   │   └── Money.ts
│   │   ├── services/
│   │   │   └── PortfolioService.ts
│   │   └── repositories/
│   │       ├── ICustomerRepository.ts
│   │       └── IInvestmentRepository.ts
│   └── retirement/            # 退休金試算領域
│       ├── entities/
│       │   └── RetirementCalculation.ts
│       ├── value-objects/
│       │   ├── Age.ts
│       │   ├── ReturnRate.ts
│       │   └── RetirementAmount.ts
│       ├── services/
│       │   └── RetirementCalculationService.ts
│       └── repositories/
│           └── IRetirementCalculationRepository.ts
│
├── application/                # Application Layer (應用層)
│   ├── use-cases/             # 用例實作
│   │   ├── administrator/
│   │   │   ├── AuthenticateAdministrator.ts
│   │   │   └── LogoutAdministrator.ts
│   │   ├── customer/
│   │   │   ├── GetCustomerList.ts
│   │   │   ├── GetCustomerDetail.ts
│   │   │   └── ExportCustomerData.ts
│   │   └── retirement/
│   │       ├── CalculateRetirement.ts
│   │       └── GetCalculationHistory.ts
│   ├── dtos/                   # 資料傳輸物件
│   │   ├── CustomerDto.ts
│   │   ├── InvestmentDto.ts
│   │   └── RetirementCalculationDto.ts
│   └── interfaces/             # 應用層介面
│       └── IAuditLogger.ts
│
├── infrastructure/             # Infrastructure Layer (基礎設施層)
│   ├── persistence/           # 資料持久化
│   │   ├── prisma/            # Prisma 設定
│   │   │   └── prisma.ts
│   │   ├── repositories/      # 儲存庫實作
│   │   │   ├── AdministratorRepository.ts
│   │   │   ├── CustomerRepository.ts
│   │   │   ├── InvestmentRepository.ts
│   │   │   └── RetirementCalculationRepository.ts
│   │   └── mappers/            # 領域模型與資料模型映射
│   │       ├── CustomerMapper.ts
│   │       └── RetirementMapper.ts
│   ├── external/              # 外部服務
│   │   ├── auth/              # 認證服務
│   │   │   ├── NextAuthAdapter.ts
│   │   │   └── LineProvider.ts
│   │   └── export/             # 匯出服務
│   │       ├── CsvExporter.ts
│   │       └── PdfExporter.ts
│   └── logging/                # 日誌服務
│       └── AuditLogger.ts
│
├── presentation/               # Presentation Layer (展示層)
│   ├── app/                    # Next.js App Router
│   │   ├── (auth)/            # 認證相關路由群組
│   │   │   ├── login/
│   │   │   │   └── page.tsx
│   │   │   └── layout.tsx
│   │   ├── (dashboard)/       # 後台主功能路由群組
│   │   │   ├── layout.tsx
│   │   │   ├── page.tsx       # 儀表板首頁
│   │   │   ├── customers/
│   │   │   │   ├── page.tsx
│   │   │   │   └── [id]/
│   │   │   │       └── page.tsx
│   │   │   └── retirement/
│   │   │       ├── page.tsx
│   │   │       └── [customerId]/
│   │   │           └── page.tsx
│   │   ├── api/                # API Routes (使用用例)
│   │   │   ├── auth/
│   │   │   │   └── [...nextauth]/
│   │   │   │       └── route.ts
│   │   │   ├── customers/
│   │   │   │   ├── route.ts
│   │   │   │   └── [id]/
│   │   │   │       └── route.ts
│   │   │   └── retirement/
│   │   │       └── calculate/
│   │   │           └── route.ts
│   │   └── layout.tsx
│   └── components/            # React 元件
│       ├── ui/                # 基礎 UI 元件
│       ├── auth/
│       ├── customers/
│       ├── retirement/
│       └── layout/
│
├── shared/                     # Shared Kernel (共享核心)
│   ├── types/                  # 共享型別
│   ├── errors/                 # 領域錯誤
│   │   ├── DomainError.ts
│   │   └── ValidationError.ts
│   └── events/                 # 領域事件（如需要）
│
└── config/                     # 設定檔
    ├── auth.ts                 # NextAuth.js 設定
    └── database.ts             # 資料庫設定

tests/
├── unit/                       # 單元測試
│   ├── domain/                 # 領域層測試
│   │   ├── entities/
│   │   ├── value-objects/
│   │   └── services/
│   ├── application/            # 應用層測試
│   │   └── use-cases/
│   └── infrastructure/         # 基礎設施層測試
│       └── repositories/
├── integration/                # 整合測試
│   ├── application/            # 應用層整合測試
│   └── infrastructure/         # 基礎設施層整合測試
└── e2e/                        # 端對端測試
    ├── auth.spec.ts
    ├── customers.spec.ts
    └── retirement.spec.ts
```

**Structure Decision**: 採用 Domain-Driven Design (DDD) 分層架構，將系統分為四個主要層級：

1. **Domain Layer (領域層)**: 包含核心業務邏輯、領域實體、值物件與領域服務。此層不依賴任何外部框架或技術，確保業務邏輯的可移植性與可測試性。

2. **Application Layer (應用層)**: 包含用例實作、DTO 與應用服務。此層協調領域物件與基礎設施，實作具體的業務用例。

3. **Infrastructure Layer (基礎設施層)**: 包含資料持久化、外部服務整合與框架相關實作。此層實作領域層定義的介面，提供技術實作細節。

4. **Presentation Layer (展示層)**: 包含 Next.js 頁面、API Routes 與 React 元件。此層處理使用者互動與請求/回應。

此架構確保：

- ✅ 領域邏輯與技術細節分離
- ✅ 業務規則集中在領域層，易於測試與維護
- ✅ 依賴方向由外層指向內層（依賴倒置原則）
- ✅ 符合專案憲章的模組化設計要求

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

目前無憲章違規項目需要說明。
