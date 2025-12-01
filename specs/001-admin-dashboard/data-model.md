# Data Model: 後台管理系統

**Feature**: 001-admin-dashboard  
**Date**: 2025-01-27  
**Database**: PostgreSQL  
**ORM**: Prisma

## Entity Relationship Diagram

```
Administrator (管理員)
  ├── id (PK)
  ├── providerId (第三方帳號 ID)
  ├── provider (LINE/GOOGLE)
  ├── email
  ├── name
  ├── createdAt
  └── updatedAt

Customer (客戶)
  ├── id (PK)
  ├── customerNumber (唯一編號)
  ├── name
  ├── email
  ├── phone
  ├── createdAt
  └── updatedAt

InvestmentPortfolio (投資組合)
  ├── id (PK)
  ├── customerId (FK → Customer)
  ├── totalAmount (總投資金額)
  ├── lastUpdatedAt
  └── investments (1:N → InvestmentRecord)

InvestmentRecord (投資記錄)
  ├── id (PK)
  ├── portfolioId (FK → InvestmentPortfolio)
  ├── symbol (投資標的代碼)
  ├── symbolName (投資標的名稱)
  ├── amount (投資金額)
  ├── purchaseDate (購買日期)
  ├── currentValue (目前價值)
  ├── performance (績效百分比)
  └── createdAt

RetirementCalculation (退休金試算)
  ├── id (PK)
  ├── customerId (FK → Customer)
  ├── administratorId (FK → Administrator)
  ├── retirementAge (退休年齡)
  ├── currentAge (目前年齡)
  ├── monthlyContribution (每月投資金額)
  ├── expectedReturnRate (預期報酬率)
  ├── currentSavings (目前儲蓄)
  ├── calculatedAmount (計算結果)
  ├── calculationDate (試算日期)
  └── createdAt

AuditLog (稽核記錄)
  ├── id (PK)
  ├── administratorId (FK → Administrator)
  ├── action (操作類型: LOGIN/LOGOUT/VIEW_CUSTOMER/EXPORT_DATA/CALCULATE_RETIREMENT)
  ├── resourceType (資源類型)
  ├── resourceId (資源 ID)
  ├── details (JSON: 操作詳情)
  └── createdAt
```

## Entity Definitions

### Administrator (管理員)

**Purpose**: 儲存管理員帳號資訊，透過第三方登入進行身份驗證

**Fields**:
- `id` (UUID, PK): 系統內部唯一識別碼
- `providerId` (String, Unique): 第三方帳號的唯一識別碼（LINE 或 Google 的使用者 ID）
- `provider` (Enum: LINE | GOOGLE): 第三方登入供應商
- `email` (String, Optional): 電子郵件地址（從第三方帳號取得）
- `name` (String): 管理員姓名（從第三方帳號取得）
- `createdAt` (DateTime): 帳號建立時間
- `updatedAt` (DateTime): 最後更新時間

**Validation Rules**:
- `providerId` 與 `provider` 組合必須唯一（同一第三方帳號只能有一個管理員帳號）
- `name` 必填，長度 1-100 字元
- `email` 格式驗證（如果提供）

**State Transitions**:
- 首次登入：自動建立新帳號
- 後續登入：更新 `updatedAt` 時間戳記

**Indexes**:
- `providerId` + `provider` (複合唯一索引)
- `email` (如果提供)

### Customer (客戶)

**Purpose**: 儲存客戶基本資訊

**Fields**:
- `id` (UUID, PK): 系統內部唯一識別碼
- `customerNumber` (String, Unique): 客戶編號（業務識別碼，如 "CUST-2025-001"）
- `name` (String): 客戶姓名
- `email` (String, Optional): 電子郵件地址
- `phone` (String, Optional): 電話號碼
- `createdAt` (DateTime): 客戶建立時間
- `updatedAt` (DateTime): 最後更新時間

**Validation Rules**:
- `customerNumber` 必填且唯一，格式：CUST-YYYY-NNNN
- `name` 必填，長度 1-100 字元
- `email` 格式驗證（如果提供）
- `phone` 格式驗證（如果提供）

**Relationships**:
- 1:N → InvestmentPortfolio (一個客戶可以有多個投資組合，但通常為 1:1)
- 1:N → RetirementCalculation (一個客戶可以有多筆試算記錄)

**Indexes**:
- `customerNumber` (唯一索引)
- `name` (用於搜尋)
- `email` (如果提供，用於搜尋)

### InvestmentPortfolio (投資組合)

**Purpose**: 儲存客戶的整體投資組合資訊

**Fields**:
- `id` (UUID, PK): 系統內部唯一識別碼
- `customerId` (UUID, FK → Customer.id): 所屬客戶
- `totalAmount` (Decimal): 總投資金額
- `lastUpdatedAt` (DateTime): 最後更新時間（投資記錄變更時更新）

**Validation Rules**:
- `customerId` 必填
- `totalAmount` ≥ 0
- `lastUpdatedAt` 自動更新（當投資記錄變更時）

**Relationships**:
- N:1 → Customer (多個投資組合屬於一個客戶)
- 1:N → InvestmentRecord (一個投資組合包含多筆投資記錄)

**Indexes**:
- `customerId` (用於快速查詢客戶的投資組合)

### InvestmentRecord (投資記錄)

**Purpose**: 儲存單筆投資交易的詳細資訊

**Fields**:
- `id` (UUID, PK): 系統內部唯一識別碼
- `portfolioId` (UUID, FK → InvestmentPortfolio.id): 所屬投資組合
- `symbol` (String): 投資標的代碼（如股票代碼、基金代碼）
- `symbolName` (String): 投資標的名稱
- `amount` (Decimal): 投資金額
- `purchaseDate` (Date): 購買日期
- `currentValue` (Decimal): 目前價值（計算欄位，可定期更新）
- `performance` (Decimal): 績效百分比（計算欄位：(currentValue - amount) / amount * 100）
- `createdAt` (DateTime): 記錄建立時間

**Validation Rules**:
- `portfolioId` 必填
- `symbol` 必填，長度 1-50 字元
- `symbolName` 必填，長度 1-200 字元
- `amount` > 0
- `purchaseDate` 不能是未來日期
- `currentValue` ≥ 0
- `performance` 自動計算

**Relationships**:
- N:1 → InvestmentPortfolio (多筆投資記錄屬於一個投資組合)

**Indexes**:
- `portfolioId` (用於快速查詢投資組合的記錄)
- `purchaseDate` (用於時間篩選)
- `symbol` (用於標的搜尋)

### RetirementCalculation (退休金試算)

**Purpose**: 儲存退休金試算的參數與結果

**Fields**:
- `id` (UUID, PK): 系統內部唯一識別碼
- `customerId` (UUID, FK → Customer.id): 試算的客戶
- `administratorId` (UUID, FK → Administrator.id): 執行試算的管理員
- `retirementAge` (Integer): 目標退休年齡
- `currentAge` (Integer): 目前年齡
- `monthlyContribution` (Decimal): 每月投資金額
- `expectedReturnRate` (Decimal): 預期年化報酬率（百分比，如 5.5 表示 5.5%）
- `currentSavings` (Decimal): 目前儲蓄金額
- `calculatedAmount` (Decimal): 計算出的預估退休金金額
- `calculationDate` (DateTime): 試算執行時間
- `createdAt` (DateTime): 記錄建立時間

**Validation Rules**:
- `customerId` 必填
- `administratorId` 必填
- `retirementAge` > `currentAge` 且 ≤ 100
- `currentAge` ≥ 18 且 < `retirementAge`
- `monthlyContribution` ≥ 0
- `expectedReturnRate` ≥ 0 且 ≤ 50（合理範圍）
- `currentSavings` ≥ 0
- `calculatedAmount` 自動計算（使用複利公式）

**Relationships**:
- N:1 → Customer (多筆試算記錄屬於一個客戶)
- N:1 → Administrator (多筆試算記錄由一個管理員執行)

**Indexes**:
- `customerId` (用於快速查詢客戶的試算記錄)
- `administratorId` (用於稽核追蹤)
- `calculationDate` (用於時間排序)

**Calculation Formula**:
```
yearsToRetirement = retirementAge - currentAge
monthsToRetirement = yearsToRetirement * 12
monthlyRate = expectedReturnRate / 100 / 12

futureValueOfSavings = currentSavings * (1 + monthlyRate) ^ monthsToRetirement
futureValueOfContributions = monthlyContribution * (((1 + monthlyRate) ^ monthsToRetirement - 1) / monthlyRate)

calculatedAmount = futureValueOfSavings + futureValueOfContributions
```

### AuditLog (稽核記錄)

**Purpose**: 記錄所有管理員操作，供稽核與安全性追蹤

**Fields**:
- `id` (UUID, PK): 系統內部唯一識別碼
- `administratorId` (UUID, FK → Administrator.id): 執行操作的管理員
- `action` (Enum): 操作類型
  - `LOGIN`: 登入
  - `LOGOUT`: 登出
  - `VIEW_CUSTOMER`: 查看客戶資料
  - `EXPORT_DATA`: 匯出資料
  - `CALCULATE_RETIREMENT`: 執行退休金試算
- `resourceType` (String, Optional): 資源類型（如 "Customer", "InvestmentPortfolio"）
- `resourceId` (UUID, Optional): 資源 ID
- `details` (JSON, Optional): 操作詳情（JSON 格式，包含額外資訊）
- `createdAt` (DateTime): 記錄建立時間

**Validation Rules**:
- `administratorId` 必填
- `action` 必填
- `details` 必須是有效的 JSON（如果提供）

**Relationships**:
- N:1 → Administrator (多筆稽核記錄屬於一個管理員)

**Indexes**:
- `administratorId` + `createdAt` (用於快速查詢管理員的操作歷史)
- `action` + `createdAt` (用於操作類型分析)
- `resourceType` + `resourceId` (用於資源追蹤)

## Database Constraints

### Unique Constraints
- `Administrator.providerId + provider` (複合唯一)
- `Customer.customerNumber` (唯一)
- `InvestmentPortfolio.customerId` (假設一個客戶只有一個投資組合，可選唯一約束)

### Foreign Key Constraints
- `InvestmentPortfolio.customerId` → `Customer.id` (CASCADE DELETE)
- `InvestmentRecord.portfolioId` → `InvestmentPortfolio.id` (CASCADE DELETE)
- `RetirementCalculation.customerId` → `Customer.id` (RESTRICT DELETE - 保留歷史記錄)
- `RetirementCalculation.administratorId` → `Administrator.id` (RESTRICT DELETE)
- `AuditLog.administratorId` → `Administrator.id` (RESTRICT DELETE - 保留稽核記錄)

### Check Constraints
- `InvestmentRecord.amount > 0`
- `InvestmentRecord.currentValue >= 0`
- `RetirementCalculation.retirementAge > currentAge`
- `RetirementCalculation.currentAge >= 18`
- `RetirementCalculation.expectedReturnRate >= 0 AND expectedReturnRate <= 50`

## Data Migration Strategy

### Initial Migration
1. 建立所有表格與索引
2. 建立外鍵約束
3. 建立檢查約束
4. 插入初始資料（如果需要）

### Future Migrations
- 使用 Prisma Migrate 管理資料庫結構變更
- 所有遷移必須可回滾
- 生產環境遷移前必須在測試環境驗證

