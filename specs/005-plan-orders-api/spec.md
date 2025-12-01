# Feature Specification: 投資計劃訂單 API

**Feature Branch**: `005-plan-orders-api`  
**Created**: 2025-01-27  
**Status**: Draft  
**Input**: User description: "幫我建立一組 api" - 包含申購、贖回、補單等投資計劃訂單相關 API

## API 端點清單

本功能包含以下 API 端點：

### 申購相關 API

1. **申購估算 - SinoPac**

   - **URL**: `POST /alpha/v1/PlanOrders/LSI/SinoPac/Plan/{plan_id}/Subscription/Estimate`
   - **說明**: 計算 SinoPac 系統投資計劃的申購估算資訊

2. **申購執行 - SinoPac**

   - **URL**: `POST /alpha/v1/PlanOrders/LSI/SinoPac/Plan/{plan_id}/Subscription`
   - **說明**: 執行 SinoPac 系統投資計劃的申購操作

3. **申購估算 - FRS**
   - **URL**: `POST /alpha/v1/PlanOrders/LSI/FRS/Plan/{plan_id}/Subscription/Estimate`
   - **說明**: 計算 FRS 系統投資計劃的申購估算資訊

### 贖回相關 API

4. **贖回估算 - SinoPac**

   - **URL**: `POST /alpha/v1/PlanOrders/LSI/SinoPac/Plan/{plan_id}/Redemption/Estimate`
   - **說明**: 計算 SinoPac 系統投資計劃的贖回估算資訊

5. **贖回執行 - SinoPac**
   - **URL**: `POST /alpha/v1/PlanOrders/LSI/SinoPac/Plan/{plan_id}/Redemption`
   - **說明**: 執行 SinoPac 系統投資計劃的贖回操作

### 補單相關 API

6. **補單估算 - SinoPac**

   - **URL**: `GET /alpha/v1/PlanOrders/LSI/SinoPac/Plan/{plan_id}/Order/{order_id}/Replenish/Estimate`
   - **說明**: 計算 SinoPac 系統投資計劃訂單的補單估算資訊

7. **補單執行 - SinoPac**
   - **URL**: `GET /alpha/v1/PlanOrders/LSI/SinoPac/Plan/{plan_id}/Order/{order_id}/Replenish`
   - **說明**: 執行 SinoPac 系統投資計劃訂單的補單操作

## User Scenarios & Testing _(mandatory)_

### User Story 1 - 申購估算 (Priority: P1)

使用者需要在下單前先估算申購金額、手續費等資訊，以便了解實際需要支付的費用與預期結果。

**API 端點**:

- `POST /alpha/v1/PlanOrders/LSI/SinoPac/Plan/{plan_id}/Subscription/Estimate` (SinoPac 系統)
- `POST /alpha/v1/PlanOrders/LSI/FRS/Plan/{plan_id}/Subscription/Estimate` (FRS 系統)

**Why this priority**: 申購估算是使用者決策的重要依據，必須在實際下單前提供，讓使用者能夠評估投資成本與預期收益。這是投資流程的第一步，必須先完成才能進行實際申購。

**Independent Test**: 可以透過以下方式獨立測試：使用者提供計劃 ID 與申購參數，系統計算並回傳申購估算結果，包括金額、手續費、預期收益等資訊。此功能可獨立運作並提供價值，即使實際申購功能尚未完成。

**Acceptance Scenarios**:

1. **Given** 使用者需要申購投資計劃, **When** 呼叫申購估算 API (`POST /alpha/v1/PlanOrders/LSI/SinoPac/Plan/{plan_id}/Subscription/Estimate`) 並提供計劃 ID 與申購參數, **Then** 系統計算並回傳申購金額、手續費、預期收益等估算資訊
2. **Given** 使用者查詢 SinoPac 系統的計劃, **When** 呼叫 SinoPac 申購估算 API (`POST /alpha/v1/PlanOrders/LSI/SinoPac/Plan/{plan_id}/Subscription/Estimate`), **Then** 系統與 SinoPac 系統連動，取得並回傳估算結果
3. **Given** 使用者查詢 FRS 系統的計劃, **When** 呼叫 FRS 申購估算 API (`POST /alpha/v1/PlanOrders/LSI/FRS/Plan/{plan_id}/Subscription/Estimate`), **Then** 系統與 FRS 系統連動，取得並回傳估算結果
4. **Given** 使用者提供的計劃 ID 不存在, **When** 呼叫申購估算 API, **Then** 系統回傳適當的錯誤訊息，說明計劃不存在
5. **Given** 使用者提供的申購參數不合理, **When** 呼叫申購估算 API, **Then** 系統驗證參數並回傳驗證錯誤訊息

---

### User Story 2 - 申購執行 (Priority: P1)

使用者完成申購估算後，需要執行實際的申購操作，建立投資計劃訂單。

**API 端點**:

- `POST /alpha/v1/PlanOrders/LSI/SinoPac/Plan/{plan_id}/Subscription` (SinoPac 系統)

**Why this priority**: 申購執行是投資流程的核心功能，使用者必須能夠實際下單才能完成投資。這是整個投資計劃訂單系統的核心價值。

**Independent Test**: 可以透過以下方式獨立測試：使用者提供計劃 ID 與申購參數，系統建立申購訂單並回傳訂單資訊。此功能可獨立運作，前提是申購估算功能已完成。

**Acceptance Scenarios**:

1. **Given** 使用者已完成申購估算, **When** 呼叫申購執行 API (`POST /alpha/v1/PlanOrders/LSI/SinoPac/Plan/{plan_id}/Subscription`) 並提供計劃 ID 與申購參數, **Then** 系統建立申購訂單並回傳訂單編號與訂單狀態
2. **Given** 系統建立申購訂單, **When** 訂單建立成功, **Then** 系統記錄訂單資訊，並與外部系統（SinoPac）同步訂單狀態
3. **Given** 使用者提供的申購參數與估算時不同, **When** 呼叫申購執行 API, **Then** 系統驗證參數並允許執行，或回傳警告訊息
4. **Given** 申購執行過程中發生錯誤, **When** 系統偵測到錯誤, **Then** 系統回傳錯誤訊息，並確保不會建立部分完成的訂單
5. **Given** 使用者查詢已建立的申購訂單, **When** 使用訂單編號查詢, **Then** 系統回傳訂單的詳細資訊與當前狀態

---

### User Story 3 - 贖回估算 (Priority: P2)

使用者需要贖回投資時，需要先估算贖回金額、手續費等資訊，以便了解實際可取得的金額。

**API 端點**:

- `POST /alpha/v1/PlanOrders/LSI/SinoPac/Plan/{plan_id}/Redemption/Estimate` (SinoPac 系統)

**Why this priority**: 贖回估算是使用者決策的重要依據，讓使用者能夠評估贖回成本與可取得金額。雖然重要，但可以在申購功能完成後再實作。

**Independent Test**: 可以透過以下方式獨立測試：使用者提供計劃 ID 與贖回參數，系統計算並回傳贖回估算結果。此功能可獨立測試，前提是申購功能已完成。

**Acceptance Scenarios**:

1. **Given** 使用者需要贖回投資計劃, **When** 呼叫贖回估算 API (`POST /alpha/v1/PlanOrders/LSI/SinoPac/Plan/{plan_id}/Redemption/Estimate`) 並提供計劃 ID 與贖回參數, **Then** 系統計算並回傳贖回金額、手續費、可取得金額等估算資訊
2. **Given** 使用者查詢 SinoPac 系統的計劃贖回, **When** 呼叫 SinoPac 贖回估算 API (`POST /alpha/v1/PlanOrders/LSI/SinoPac/Plan/{plan_id}/Redemption/Estimate`), **Then** 系統與 SinoPac 系統連動，取得並回傳估算結果
3. **Given** 使用者提供的贖回金額超過持有金額, **When** 呼叫贖回估算 API, **Then** 系統驗證並回傳錯誤訊息，說明贖回金額超過可贖回範圍
4. **Given** 使用者提供的計劃 ID 不存在或無持有記錄, **When** 呼叫贖回估算 API, **Then** 系統回傳適當的錯誤訊息

---

### User Story 4 - 贖回執行 (Priority: P2)

使用者完成贖回估算後，需要執行實際的贖回操作，完成投資計劃的贖回。

**API 端點**:

- `POST /alpha/v1/PlanOrders/LSI/SinoPac/Plan/{plan_id}/Redemption` (SinoPac 系統)

**Why this priority**: 贖回執行是投資流程的重要功能，讓使用者能夠實際贖回投資。雖然重要，但可以在申購功能完成後再實作。

**Independent Test**: 可以透過以下方式獨立測試：使用者提供計劃 ID 與贖回參數，系統建立贖回訂單並回傳訂單資訊。此功能可獨立測試，前提是贖回估算功能已完成。

**Acceptance Scenarios**:

1. **Given** 使用者已完成贖回估算, **When** 呼叫贖回執行 API (`POST /alpha/v1/PlanOrders/LSI/SinoPac/Plan/{plan_id}/Redemption`) 並提供計劃 ID 與贖回參數, **Then** 系統建立贖回訂單並回傳訂單編號與訂單狀態
2. **Given** 系統建立贖回訂單, **When** 訂單建立成功, **Then** 系統記錄訂單資訊，並與外部系統（SinoPac）同步訂單狀態
3. **Given** 贖回執行過程中發生錯誤, **When** 系統偵測到錯誤, **Then** 系統回傳錯誤訊息，並確保不會建立部分完成的訂單
4. **Given** 使用者查詢已建立的贖回訂單, **When** 使用訂單編號查詢, **Then** 系統回傳訂單的詳細資訊與當前狀態

---

### User Story 5 - 補單估算 (Priority: P3)

使用者需要補單時，需要先估算補單金額、手續費等資訊，以便了解補單的成本與效果。

**API 端點**:

- `GET /alpha/v1/PlanOrders/LSI/SinoPac/Plan/{plan_id}/Order/{order_id}/Replenish/Estimate` (SinoPac 系統)

**Why this priority**: 補單估算是進階功能，提供使用者補單前的評估資訊。雖然有價值，但可以在基本申購與贖回功能完成後再實作。

**Independent Test**: 可以透過以下方式獨立測試：使用者提供計劃 ID 與訂單 ID，系統計算並回傳補單估算結果。此功能可獨立測試，前提是申購功能已完成。

**Acceptance Scenarios**:

1. **Given** 使用者需要補單, **When** 呼叫補單估算 API (`GET /alpha/v1/PlanOrders/LSI/SinoPac/Plan/{plan_id}/Order/{order_id}/Replenish/Estimate`) 並提供計劃 ID 與訂單 ID, **Then** 系統計算並回傳補單金額、手續費等估算資訊
2. **Given** 使用者查詢已存在的訂單, **When** 呼叫補單估算 API, **Then** 系統驗證訂單存在並回傳估算結果
3. **Given** 使用者提供的訂單 ID 不存在, **When** 呼叫補單估算 API, **Then** 系統回傳適當的錯誤訊息，說明訂單不存在

---

### User Story 6 - 補單執行 (Priority: P3)

使用者完成補單估算後，需要執行實際的補單操作，完成投資計劃的補單。

**API 端點**:

- `GET /alpha/v1/PlanOrders/LSI/SinoPac/Plan/{plan_id}/Order/{order_id}/Replenish` (SinoPac 系統)

**Why this priority**: 補單執行是進階功能，讓使用者能夠實際執行補單。雖然有價值，但可以在基本申購與贖回功能完成後再實作。

**Independent Test**: 可以透過以下方式獨立測試：使用者提供計劃 ID 與訂單 ID，系統執行補單並回傳結果。此功能可獨立測試，前提是補單估算功能已完成。

**Acceptance Scenarios**:

1. **Given** 使用者已完成補單估算, **When** 呼叫補單執行 API (`GET /alpha/v1/PlanOrders/LSI/SinoPac/Plan/{plan_id}/Order/{order_id}/Replenish`) 並提供計劃 ID 與訂單 ID, **Then** 系統執行補單並回傳執行結果
2. **Given** 系統執行補單, **When** 補單執行成功, **Then** 系統更新訂單狀態，並與外部系統（SinoPac）同步
3. **Given** 補單執行過程中發生錯誤, **When** 系統偵測到錯誤, **Then** 系統回傳錯誤訊息，並確保不會部分更新訂單狀態

---

### Edge Cases

- 當外部系統（SinoPac、FRS）暫時無法連線時，系統如何處理？
- 當估算結果與實際執行結果不一致時，系統如何處理？
- 當使用者同時對同一計劃進行多個操作時，系統如何處理並發衝突？
- 當訂單執行過程中外部系統回應超時時，系統如何處理？
- 當使用者提供的計劃 ID 或訂單 ID 格式錯誤時，系統如何驗證與回應？
- 當估算參數與執行參數不一致時，系統是否允許執行或需要重新估算？
- 當外部系統回傳的資料格式與預期不符時，系統如何處理？

## Requirements _(mandatory)_

### Functional Requirements

- **FR-001**: 系統必須提供申購估算 API，允許使用者查詢申購金額、手續費等估算資訊
- **FR-002**: 系統必須支援 SinoPac 系統的申購估算，與 SinoPac 系統連動取得估算結果
- **FR-003**: 系統必須支援 FRS 系統的申購估算，與 FRS 系統連動取得估算結果
- **FR-004**: 系統必須提供申購執行 API，允許使用者建立實際的申購訂單
- **FR-005**: 系統必須支援 SinoPac 系統的申購執行，與 SinoPac 系統同步訂單狀態
- **FR-006**: 系統必須提供贖回估算 API，允許使用者查詢贖回金額、手續費等估算資訊
- **FR-007**: 系統必須支援 SinoPac 系統的贖回估算，與 SinoPac 系統連動取得估算結果
- **FR-008**: 系統必須提供贖回執行 API，允許使用者建立實際的贖回訂單
- **FR-009**: 系統必須支援 SinoPac 系統的贖回執行，與 SinoPac 系統同步訂單狀態
- **FR-010**: 系統必須提供補單估算 API，允許使用者查詢補單金額、手續費等估算資訊
- **FR-011**: 系統必須支援 SinoPac 系統的補單估算，與 SinoPac 系統連動取得估算結果
- **FR-012**: 系統必須提供補單執行 API，允許使用者執行實際的補單操作
- **FR-013**: 系統必須支援 SinoPac 系統的補單執行，與 SinoPac 系統同步訂單狀態
- **FR-014**: 系統必須驗證所有輸入參數，確保計劃 ID、訂單 ID 等參數的正確性
- **FR-015**: 系統必須在參數驗證失敗時，回傳清晰的錯誤訊息
- **FR-016**: 系統必須處理外部系統連線失敗的情況，提供適當的錯誤訊息
- **FR-017**: 系統必須記錄所有訂單操作，包括估算與執行，供後續查詢與稽核使用
- **FR-018**: 系統必須確保估算與執行的一致性，避免參數不一致導致的問題
- **FR-019**: 系統必須處理並發請求，確保多個操作不會互相干擾
- **FR-020**: 系統必須在訂單建立或更新後，即時同步狀態至外部系統

### Key Entities _(include if feature involves data)_

- **投資計劃 (Investment Plan)**: 使用者選擇的投資計劃，具有計劃 ID、計劃類型、發行機構等屬性
- **申購訂單 (Subscription Order)**: 使用者建立的申購訂單，包含計劃 ID、申購金額、手續費、訂單狀態等
- **贖回訂單 (Redemption Order)**: 使用者建立的贖回訂單，包含計劃 ID、贖回金額、手續費、訂單狀態等
- **補單記錄 (Replenish Record)**: 使用者執行的補單記錄，包含原訂單 ID、補單金額、補單狀態等
- **估算結果 (Estimate Result)**: 系統計算的估算結果，包含金額、手續費、預期收益等資訊
- **外部系統整合記錄 (External System Integration Log)**: 記錄與外部系統（SinoPac、FRS）的互動記錄，包含操作類型、操作時間、操作結果等

## Success Criteria _(mandatory)_

### Measurable Outcomes

- **SC-001**: 系統能夠在 2 秒內完成申購估算計算並回傳結果
- **SC-002**: 系統能夠在 3 秒內完成申購執行並建立訂單
- **SC-003**: 系統能夠在 2 秒內完成贖回估算計算並回傳結果
- **SC-004**: 系統能夠在 3 秒內完成贖回執行並建立訂單
- **SC-005**: 系統能夠在 2 秒內完成補單估算計算並回傳結果
- **SC-006**: 系統能夠在 3 秒內完成補單執行並更新訂單狀態
- **SC-007**: 95% 的 API 請求能夠在 3 秒內完成處理並回傳結果
- **SC-008**: 系統與外部系統（SinoPac、FRS）的狀態同步準確率達到 99% 以上
- **SC-009**: 系統能夠正確驗證並拒絕 100% 的無效輸入參數
- **SC-010**: 當外部系統無法連線時，系統能夠在 2 秒內回傳適當的錯誤訊息
