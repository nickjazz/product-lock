---
layout: default
title: 工作日誌
nav_order: 9
permalink: /zh/worklog
lang: zh
lang_label: 繁體中文
page_id: worklog
---

# Product Lock 工作日誌規格

### 版本 0.1.0 — 2026 年 2 月

> 一份結構化的開發日誌，追蹤相對於 product.lock.json 的變更。
>
> Lock 定義「是什麼」。Plan 定義「怎麼做」。Score 衡量複雜度。**Worklog 記錄「為什麼」和「何時」。**
>
> 這四個檔案共同重建完整的專案脈絡 — 跨工作階段、跨代理、跨時間。

完整規格請參閱 [Product Lock 規格](https://spec.productlock.org)。

---

## 目錄

1. [目的](#1-目的)
2. [封閉迴圈](#2-封閉迴圈)
3. [檔案格式](#3-檔案格式)
4. [必要區段](#4-必要區段)
5. [選用區段](#5-選用區段)
6. [條目生命週期](#6-條目生命週期)
7. [脈絡重建](#7-脈絡重建)
8. [慣例](#8-慣例)

---

## 1. 目的

AI 工作階段是短暫的。上下文視窗會壓縮、對話會過期、代理會失去記憶。但產品決策是持久的。工作日誌捕捉重要的事項：

- **變更了什麼** — 修改的檔案、新增的實體、實作的功能
- **為何變更** — 決策、觸發因素、限制條件
- **變更代表什麼** — lock 的修改、複雜度差異、範圍影響
- **發現了什麼** — 審查者的發現、修正的違規、強化的邊界

沒有工作日誌，每次 AI 工作階段都從零開始。有了工作日誌，任何代理都可以讀取 lock + worklog 來重建專案的軌跡。

### 工作日誌「不是」什麼

- 不是 git log（git 追蹤檔案變更，worklog 追蹤產品決策）
- 不是 changelog（changelog 是給使用者看的，worklog 是給開發者和 AI 看的）
- 不是日記（沒有意見、沒有評論 — 只有結構化資料）

---

## 2. 封閉迴圈

四個 product lock 檔案構成一個封閉的回饋迴圈：

```
product.lock.json ──→ product.plan.md ──→ 實作 ──→ product.worklog.md
       ↑                                                            │
       └────────────────────────────────────────────────────────────┘
                         （根據 worklog 的發現更新 lock）
```

| 檔案 | 角色 | 回答的問題 |
|------|------|---------|
| `product.lock.json` | 邊界 | 這個產品是什麼？ |
| `product.plan.md` | 藍圖 | 我們如何建構它？ |
| `product-lock-scoring.md` | 衡量 | 它有多複雜？ |
| `product.worklog.md` | 歷史 | 發生了什麼、何時、為什麼？ |

每個開發工作階段：

1. **開始前**：讀取 lock + 最新的 worklog 條目 → 了解目前狀態
2. **進行中**：追蹤變更、決策、lock 修改
3. **結束後**：撰寫 worklog 條目 → 執行審查者 → 記錄發現
4. **收尾**：如有需要則更新 lock → 重新計算分數 → 記錄差異

當每個變更都可追溯時，迴圈就是封閉的：從授權變更的 lock，經過實作的工作階段，到驗證的審查者。

---

## 3. 檔案格式

工作日誌儲存在專案根目錄的 `worklogs/` 目錄中，與 `product.lock.json` 並列。每個工作階段一個檔案。

```
project-root/
├── product.lock.json
├── product.plan.md
└── worklogs/
    ├── 2026-02-16-initial-release.md
    ├── 2026-02-17-add-payment-system.md
    ├── 2026-02-17T1430-fix-payment-webhook.md
    └── ...
```

### 檔案命名

```
{date}-{kebab-case-title}.md
```

| 部分 | 格式 | 範例 |
|------|--------|---------|
| 日期 | `YYYY-MM-DD` | `2026-02-17` |
| 日期（同一天） | `YYYY-MM-DDTHHmm` | `2026-02-17T1430` |
| 標題 | kebab-case，60 字元以內 | `add-payment-system` |

範例：
```
2026-02-16-initial-release.md
2026-02-17-add-payment-system.md
2026-02-17T1430-fix-payment-webhook.md
2026-02-18-refactor-auth-middleware.md
```

### 檔案排列順序

檔案按名稱自然排序（日期前綴確保按時間順序）。要找到最新的條目，按降序排列或查看最後一個檔案。

### 每個檔案就是一個工作階段

一個檔案 = 一個工作階段 = 一段專注的工作區塊。每個檔案遵循以下範本：

```markdown
# [{date}] {title}

### Context
- Branch: `feature/xxx`
- Agent: {執行工作的人}
- Lock version: {此工作階段前的版本}
- PLS before: {score}

### Goal
{一句話：此工作階段要完成什麼。}

### Changes

| Action | Target | Detail |
|--------|--------|--------|
| added | Entity: Payment | 4 fields: id, amount, userId, createdAt |
| modified | Feature: checkout | Added Stripe integration |
| removed | Feature: manualInvoice | Replaced by automated billing |

### Lock Delta
{product.lock.json 中變更了什麼（如有）。}

### Score Delta
- PLS: {before} → {after} ({+/-delta})
- Breakdown: D {before}→{after}, F {before}→{after}, I {before}→{after}, A {before}→{after}

### Review Findings
{此工作階段後的 AI 審查者結果。}

### Decisions
{做出的關鍵決策及原因。}

### Files
| Action | Path |
|--------|------|
| created | src/models/payment.ts |
| modified | src/routes/checkout.ts |
| deleted | src/routes/manual-invoice.ts |

### Next
{下一個工作階段應接續的工作。}
```

---

## 4. 必要區段

每個工作日誌條目必須包含以下區段。

### 4.1 Context（脈絡）

工作階段的後設資料。建立起始狀態。

```markdown
### Context
- Branch: `feature/payment-system`
- Agent: Claude (Opus 4)
- Lock version: 1.2.0
- PLS before: 145
```

| 欄位 | 必要 | 說明 |
|-------|----------|-------------|
| Branch | 是 | 此工作階段的 Git 分支 |
| Agent | 是 | 執行工作的人（AI 模型、人名或兩者皆是） |
| Lock version | 是 | 工作階段開始時 product.lock.json 的版本 |
| PLS before | 是 | 工作階段開始時的 Product Lock Score |

### 4.2 Goal（目標）

一句話描述工作階段的目標。這是契約 — 條目中的一切都應與此目標相關。

```markdown
### Goal
Add payment processing with Stripe so Members can pay for premium features.
```

規則：
- 必須是一句話
- 必須足夠具體，使完成與否可驗證
- 應在適用時引用 lock 概念（entities、features、actors）

### 4.3 Changes（變更）

工作階段中產品層級變更的表格。這不是檔案差異 — 這是產品邊界差異。

```markdown
### Changes

| Action | Target | Detail |
|--------|--------|--------|
| added | Entity: Payment | id, amount, currency, userId, status, createdAt |
| added | Entity: Subscription | id, planId, userId, startDate, endDate |
| added | Feature: createPayment | Stripe checkout session creation |
| added | Feature: cancelSubscription | Self-service cancellation |
| modified | Feature: signup | Now creates free-tier Subscription |
| added | Actor: Subscriber | Paying member with premium access |
| added | Permission: Subscriber | createPayment, cancelSubscription, all Member perms |
| added | Denied: refundPayment | Refunds handled manually by admin |
```

操作類型：
- `added` — 產品邊界中的新項目
- `modified` — 已存在的項目有所變更
- `removed` — 從產品邊界中移除的項目
- `unchanged` — 明確註明某項目未被變更（在相關時使用）

Target 格式：`{type}: {name}`，其中 type 可為 Entity、Feature、Actor、Permission、Denied、Story。

### 4.4 Review Findings（審查發現）

工作階段變更後 AI 審查者的結果。如果未執行審查，請明確說明。

```markdown
### Review Findings

Reviewer: Claude (Sonnet 4.5)
Status: PASS (2 warnings)

- [PASS] Entity Payment — model exists with correct fields
- [PASS] Feature createPayment — POST /api/payments endpoint
- [WARN] Undeclared entity: PaymentLog (exists in code, not in lock)
- [WARN] Undeclared feature: retryPayment (exists in code, not in lock)
```

或：

```markdown
### Review Findings

No review conducted this session.
```

規則：
- 必須指名審查者（AI 模型或人員）
- 必須包含整體狀態：PASS、FAIL 或 NOT RUN
- 必須列出所有違規和警告
- 如果為 FAIL：條目必須在 Next 區段中包含後續處理

### 4.5 Files（檔案）

工作階段中建立、修改或刪除的檔案。

```markdown
### Files

| Action | Path |
|--------|------|
| created | prisma/migrations/003_add_payment.sql |
| created | src/models/payment.ts |
| created | src/models/subscription.ts |
| created | src/routes/payments.ts |
| modified | src/routes/auth.ts |
| modified | src/middleware/permissions.ts |
| modified | prisma/schema.prisma |
| deleted | src/routes/legacy-billing.ts |
```

規則：
- 列出每個接觸過的檔案，而不僅是重要的檔案
- 使用從專案根目錄開始的相對路徑
- 操作類型：`created`、`modified`、`deleted`
- 先按操作排序（created → modified → deleted），再於每組中按字母順序排列

---

## 5. 選用區段

在這些區段能提供有用脈絡時包含它們。

### 5.1 Lock Delta（Lock 差異）

此工作階段中 product.lock.json 的變更。如果 lock 未被修改則可跳過。

```markdown
### Lock Delta

Lock updated: 1.2.0 → 1.3.0

Added:
- Entity: Payment [id, amount, currency, userId, status, createdAt]
- Entity: Subscription [id, planId, userId, startDate, endDate]
- Feature: createPayment
- Feature: cancelSubscription
- Actor: Subscriber
- Denied: refundPayment — "Refunds handled manually by admin"

Modified:
- Feature: signup (now includes subscription creation)
- Permission: Member → added cancelSubscription

Removed:
- (none)
```

規則：
- 必須說明版本變更
- 按 Added / Modified / Removed 分組
- 使用 lock 欄位格式（entity 名稱使用 PascalCase，feature 使用 camelCase 等）

### 5.2 Score Delta（分數差異）

Product Lock Score 的變化。如果 lock 未被修改則可跳過。

```markdown
### Score Delta

- PLS: 145 → 172 (+27)
- Level: Moderate → Complex
- Breakdown:
  - D: 79.2 → 94.8 (+15.6) — 2 new entities
  - F: 34 → 38 (+4) — 2 new features + 2 existing
  - I: 22.5 → 27.5 (+5) — added payment stories
  - A: 9 → 11.7 (+2.7) — Subscriber role added
```

如果分數跨越層級邊界（例如 Moderate → Complex），應明確指出。層級轉換是範圍檢視的信號。

### 5.3 Decisions（決策）

工作階段中做出的關鍵決策及其理由。如果沒有重大決策則可跳過。

```markdown
### Decisions

1. **Stripe over Paddle**: Stripe has better API for subscription management. Paddle bundles tax handling but we don't need it for MVP.

2. **No refunds in v1**: Added to denied list. Manual refund via Stripe dashboard is sufficient. Automated refunds add liability and edge cases (partial refunds, proration).

3. **Subscription as separate entity**: Could have been a field on User, but Subscription has its own lifecycle (start, end, cancel, renew) that justifies a separate entity.
```

規則：
- 為每個決策編號
- 將決策標題加粗
- 包含理由（為什麼）
- 如果決策修改了 denied 列表，請註明

### 5.4 Next（後續）

下一個工作階段應接續的工作。僅在專案已完成時才可跳過。

```markdown
### Next

- [ ] Implement webhook handler for Stripe events (payment_intent.succeeded, subscription.canceled)
- [ ] Add PaymentLog to lock or remove from codebase (reviewer warning)
- [ ] Write stories for payment flow and add to lock
- [ ] Run full review after webhook implementation
```

規則：
- 使用核取方塊格式列出可執行的項目
- 在適用時引用審查者的發現
- 足夠具體，使不同的代理也能接手

### 5.5 Commits（提交）

工作階段中的 Git 提交。如果沒有提交（例如僅規劃的工作階段）則可跳過。

```markdown
### Commits

| Hash | Message |
|------|---------|
| a1b2c3d | feat: add Payment and Subscription models |
| e4f5g6h | feat: implement Stripe checkout endpoint |
| i7j8k9l | fix: permission middleware for Subscriber role |
| m0n1o2p | chore: update product.lock.json to v1.3.0 |
```

---

## 6. 條目生命週期

工作日誌條目在開發工作階段中遵循以下生命週期：

```
工作階段開始
│
├─ 1. 讀取最新的 worklog 條目 → 了解目前狀態
├─ 2. 讀取 lock → 了解產品邊界
├─ 3. 讀取 plan → 了解實作決策
│
├─ 4. 執行工作
│   ├─ 追蹤變更（entities、features、actors）
│   ├─ 追蹤修改的檔案
│   └─ 即時記錄決策
│
├─ 5. 如果產品邊界有變更，更新 lock
├─ 6. 如果 lock 有變更，重新計算分數
├─ 7. 對更新後的 lock 執行審查者
│
├─ 8. 撰寫 worklog 條目
│   ├─ Context（分支、代理、lock 版本、PLS）
│   ├─ Goal（目標是什麼）
│   ├─ Changes（產品層級的變更）
│   ├─ Lock Delta（如果 lock 有修改）
│   ├─ Score Delta（如果分數有變化）
│   ├─ Review Findings（審查者結果）
│   ├─ Decisions（關鍵決策 + 理由）
│   ├─ Files（所有接觸過的檔案）
│   ├─ Commits（git 提交）
│   └─ Next（接下來要做什麼）
│
工作階段結束
```

### 何時建立條目

- 每個開發工作階段一個條目（一個工作階段 = 一段專注的工作區塊）
- 僅規劃而未撰寫程式碼的工作階段仍需要一個條目（不含 Files 區段）
- 僅審查而未變更任何內容的工作階段仍需要一個條目（包含 Review Findings）
- 一次處理中的多個小修正 = 一個條目
- 跨越多次處理的大型功能 = 多個條目

### 條目大小

保持條目簡潔。典型的條目為 30–80 行。如果條目超過 150 行，表示該工作階段過大 — 下次應拆分為多個條目。

---

## 7. 脈絡重建

工作日誌的主要價值：任何代理都可以透過讀取結構化檔案來重建專案脈絡，而不需要重播對話歷史。

### 沒有工作日誌

```
新的 AI 工作階段 → 讀取程式碼庫（數千個檔案）→ 猜測意圖 → 經常猜錯
```

### 有工作日誌

```
新的 AI 工作階段 → 讀取 lock（產品邊界）
              → 讀取最新的 worklog（目前狀態、近期決策、後續步驟）
              → 讀取 plan（實作方法）
              → 帶著完整脈絡開始工作
```

### 每個檔案對重建的貢獻

| 問題 | 由何者回答 |
|----------|-------------|
| 這個產品是什麼？ | `product.lock.json` |
| 確切的邊界是什麼？ | `product.lock.json` (denied) |
| 它有多複雜？ | PLS 分數（在 worklog 中） |
| 它是如何建構的？ | `product.plan.md` |
| 最近發生了什麼？ | `product.worklog.md`（最新條目） |
| 為什麼做出 X 決策？ | `product.worklog.md` (Decisions) |
| 我接下來該做什麼？ | `product.worklog.md` (Next) |
| 審查者發現了什麼？ | `product.worklog.md` (Review Findings) |
| 範圍是否在蔓延？ | `product.worklog.md` (Score Delta 隨時間的變化) |

### Token 效率

一場 2 小時的 AI 對話可能消耗超過 100K token 的上下文。同一工作階段的 worklog 條目約為 50 行結構化 markdown。下一個工作階段讀取 50 行，而非重播 100K token。

```
對話歷史：100,000+ tokens（壓縮的、有損的、短暫的）
Worklog 條目：     ~800 tokens（結構化的、完整的、永久的）
```

### 範圍蔓延偵測

透過追蹤各條目的 PLS，範圍蔓延變得可見：

```
[2026-02-10] v1.0.0 — PLS: 85  (Moderate)
[2026-02-12] v1.1.0 — PLS: 92  (+7, minor additions)
[2026-02-14] v1.2.0 — PLS: 145 (+53, significant jump ⚠️)
[2026-02-17] v1.3.0 — PLS: 172 (+27, crossed into Complex)
```

單一工作階段中跳升 +50 或以上，或跨越層級邊界，應觸發範圍檢視。

---

## 8. 慣例

### 8.1 目錄結構

- 目錄：專案根目錄中的 `worklogs/`
- 每個產品一個目錄（對應每個產品一個 lock）
- 每個檔案是一個工作階段

```
project-root/
├── product.lock.json
└── worklogs/
    ├── 2026-02-16-initial-release.md
    └── 2026-02-17-add-payment-system.md
```

### 8.2 檔案命名

```
{YYYY-MM-DD}-{kebab-case-title}.md
```

- 日期前綴確保按時間順序排列
- 標題使用祈使語氣、kebab-case，60 字元以內
- 如果同一天有多個工作階段，使用 `{YYYY-MM-DD}T{HHmm}-{title}.md`

### 8.3 標題

祈使語氣且簡潔：

| 正確 | 不正確 |
|------|-----|
| `add-payment-system` | `added-the-payment-system-and-also-fixed-some-bugs` |
| `fix-auth-middleware` | `authentication-middleware-was-broken` |
| `remove-legacy-billing` | `cleaned-up-old-billing-code` |

### 8.4 交叉引用

引用其他條目時，使用檔案名稱：

```markdown
Continues from `2026-02-14-setup-stripe-integration.md`.
See `2026-02-12-define-payment-entities.md` for schema decisions.
```

### 8.5 命名慣例

在工作日誌中遵循 lock 的慣例：

| 項目 | 格式 | 範例 |
|------|--------|---------|
| Entity 名稱 | PascalCase | `Entity: Payment` |
| Feature 名稱 | camelCase | `Feature: createPayment` |
| Actor 名稱 | PascalCase | `Actor: Subscriber` |
| 檔案路徑 | 從根目錄的相對路徑 | `src/models/payment.ts` |

### 8.6 閱讀順序

為了重建脈絡，AI 代理應：

1. 讀取**最新的** worklog 檔案（依檔案名稱排序的最後一個）
2. 查看 **Next** 區段的待辦工作
3. 如需更多脈絡，閱讀其他近期檔案
4. 讀取 lock 以了解產品邊界

---

*本規格為 [Product Lock 規格](https://spec.productlock.org) 的一部分。*
