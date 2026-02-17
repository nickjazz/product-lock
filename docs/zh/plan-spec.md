---
layout: default
title: 計畫規格
nav_order: 6
permalink: /zh/plan-spec
lang: zh
lang_label: 繁體中文
page_id: plan-spec
---

# 產品計畫規格

### 版本 0.1.0 — 2026 年 2 月

> 產品計畫描述如何實作一個產品。它是 product.lock.json 的實作藍圖。
>
> Lock 定義「做什麼」（WHAT）。Plan 定義「怎麼做」（HOW）。
>
> Plan 是給 AI 讀的。人類可以跳過，但當出問題時，人類能看懂。

完整的 lock 規格請參閱 [Product Lock 規格](https://spec.productlock.org)。

---

## 目錄

1. [檔案之間的關係](#1-檔案之間的關係)
2. [設計原則](#2-設計原則)
3. [格式](#3-格式)
4. [必要章節](#4-必要章節)
5. [選用章節](#5-選用章節)
6. [慣例](#6-慣例)
7. [Lock 與 Plan 的比較](#7-lock-與-plan-的比較)

---

## 1. 檔案之間的關係

```
product.lock.json    →  產品是什麼（由人類核准）
product.plan.md      →  如何建構（由 AI 遵循）
codebase             →  實際的產品（由 AI 審查者依據 lock 驗證）
```

| 你擁有的 | 你可以做的 |
|----------|-----------|
| 僅有 Lock | AI 建構產品，自行做出實作決策 |
| Lock + Plan | AI 以相同架構建構出完全相同的產品 |
| Lock + Plan + Codebase | AI 審查者驗證一切是否吻合 |
| 僅有 Plan | AI 可以建構，但沒有產品邊界可供驗證 |

---

## 2. 設計原則

1. **衍生自 Lock** — Plan 必須（MUST）實作 lock 定義的所有內容，不得遺漏、不得衝突
2. **可重現** — 不同的 AI 代理讀取相同的 plan，應該（SHOULD）產出結構上相同的程式碼庫
3. **純文字** — Markdown 格式，不壓縮、不編碼、不加密
4. **完備** — Plan 包含 lock 未涵蓋的所有實作決策；AI 持有此 plan 即可直接撰寫程式碼
5. **針對特定技術棧** — 同一份 lock 可以有多個 plan（TypeScript 版本、Python 版本、Go 版本）

---

## 3. 格式

檔案必須（MUST）命名為 `product.plan.md`，並放置在專案根目錄，與 `product.lock.json` 同層。

```markdown
# Product Plan: {name} v{version}

> Lock: product.lock.json
> Stack: {一行技術棧摘要}
> Generated: {日期}

## Stack

| Layer | Choice |
|-------|--------|
| Language | TypeScript 5.x |
| Runtime | Node.js 20 |
| Framework | Next.js 15 (App Router) |
| Database | PostgreSQL 16 |
| ORM | Prisma 6 |
| Auth | Supabase Auth (magic link) |
| Validation | Zod |
| State | TanStack Query + Zustand |
| Styling | Tailwind CSS 4 + shadcn/ui |
| Deploy | Vercel |

## Schemas

{完整的實體 schema，包含型別、約束、關聯。}

## Endpoints

{每個功能的 API 契約。}

## Implementation Steps

{從零開始建構的有序步驟。}
```

---

## 4. 必要章節

每個 plan 都必須（MUST）包含以下章節。

### 4.1 Stack

產品每一層的技術選型。

```markdown
## Stack

| Layer | Choice |
|-------|--------|
| Language | TypeScript 5.x |
| Runtime | Node.js 20 |
| Framework | Next.js 15 (App Router) |
| Database | PostgreSQL 16 |
| ORM | Prisma 6 |
| Auth | Supabase Auth (magic link) |
| Validation | Zod |
| State | TanStack Query + Zustand |
| Styling | Tailwind CSS 4 + shadcn/ui |
| Deploy | Vercel |
```

### 4.2 Schemas

完整的實體定義，包含型別、約束、關聯與索引。這是 lock 中的 `entities` 在程式碼中的具體呈現。

```markdown
## Schemas

### User

| Field | Type | Constraints |
|-------|------|-------------|
| id | uuid | PK, default gen_random_uuid() |
| email | varchar(255) | UNIQUE, NOT NULL |
| name | varchar(100) | NOT NULL |
| avatarUrl | text | nullable |
| role | enum(Admin,Member,Guest) | NOT NULL, default 'Member' |
| createdAt | timestamptz | NOT NULL, default now() |

Relations: has many Message, has many Conversation (via membership)

### Message

| Field | Type | Constraints |
|-------|------|-------------|
| id | uuid | PK |
| content | text | NOT NULL |
| senderId | uuid | FK → User.id, NOT NULL |
| conversationId | uuid | FK → Conversation.id, NOT NULL |
| createdAt | timestamptz | NOT NULL, default now() |

Relations: belongs to User (sender), belongs to Conversation
Index: (conversationId, createdAt)
```

Plan 必須（MUST）包含所有欄位，而非僅列出 lock 指定的欄位。Lock 有 `["id", "name", "email"]` → plan 加上型別、約束、預設值、關聯、索引。Plan 可以（MAY）新增 lock 未提及的欄位（例如 `createdAt`、`updatedAt`）。

### 4.3 Endpoints

實作每個功能的 API 端點。Lock 中的每個 `features` 都必須（MUST）對應至少一個端點（或背景作業）。

```markdown
## Endpoints

### Messages

#### sendMessage
- `POST /api/messages`
- Auth: Member, Admin
- Body: `{ content: string, conversationId: string }`
- Response: `201 { message: Message }`
- Side effects: broadcast to conversation subscribers
- Validation: content 1-5000 chars, conversationId must exist

#### deleteMessage
- `DELETE /api/messages/:id`
- Auth: Admin only (or message owner)
- Response: `204`
```

每個端點必須（MUST）包含：
- HTTP 方法與路徑
- 驗證要求
- 請求主體 / 查詢參數
- 回應結構與狀態碼
- 驗證規則
- 副作用（如有）

### 4.4 Implementation Steps

從零開始建構的有序步驟。AI 遵循這些步驟即可產出完整的產品。

```markdown
## Implementation Steps

1. Initialize Next.js 15 project with TypeScript
2. Install dependencies: prisma, @supabase/supabase-js, zod, tailwindcss
3. Configure Prisma schema with all entities
4. Run prisma migrate to create database
5. Set up Supabase Auth with magic link
6. Create auth middleware
7. Implement API routes: messages, conversations, groups
8. Implement background jobs: fetchNews, enrichNews
9. Create UI pages: login, dashboard, conversations, settings
10. Configure RLS policies
11. Test all features against lock
12. Generate product.lock.json and verify
```

規則：
- 編號、有序
- 每個步驟應該（SHOULD）可獨立驗證
- 粒度：每個步驟 ≈ AI 的一個工作階段

---

## 5. 選用章節

當產品有需要時，加入這些章節。

### 5.1 Architecture

當使用非簡單架構模式時（微服務、事件溯源、CQRS 等）。

```markdown
## Architecture

REST API with Next.js Route Handlers. Supabase handles auth, database, storage, and real-time subscriptions. Single deployment on Vercel.

- API: Next.js Route Handlers (REST)
- Auth: Supabase middleware, RLS on database
- Real-time: Supabase Realtime subscriptions
- File storage: Supabase Storage
```

### 5.2 File Structure

當專案佈局非標準或需要說明時。

```markdown
## File Structure

├── prisma/schema.prisma
├── src/
│   ├── app/
│   │   ├── (auth)/login/
│   │   ├── (dashboard)/
│   │   └── api/
│   ├── components/
│   │   ├── ui/          ← shadcn primitives
│   │   └── features/    ← domain components
│   ├── lib/
│   │   ├── db.ts
│   │   ├── auth.ts
│   │   └── validations/
│   ├── hooks/
│   └── types/
└── package.json
```

### 5.3 Auth & Permissions

當 lock 包含 `actors` 或 `permissions` 時。

```markdown
## Auth & Permissions

- Auth method: Supabase magic link (email OTP)
- Session: JWT stored in httpOnly cookie
- Middleware: `src/lib/auth.ts` extracts user + role
- Route protection: middleware checks role before handler

| Actor | Implementation |
|-------|---------------|
| Admin | role='admin' in users table, full access |
| Member | role='member', default for new signups |
| Guest | no auth, public routes only |

RLS policies:
- messages: SELECT for conversation members, INSERT for Member/Admin
- conversations: SELECT for members, INSERT for Member/Admin
```

### 5.4 Background Jobs

當 lock 包含系統 `stories`（自主行為）時。

```markdown
## Background Jobs

### fetchNews
- Trigger: cron every 30 minutes
- Process: fetch from RSS feeds → parse → store as NewsArticle
- Error handling: retry 3x, log failures

### enrichNewsWithAi
- Trigger: after new NewsArticle created
- Process: call local LLM → generate summary, impact, signal → update article
- Rate limit: 10 req/min to LLM
```

### 5.5 Third-party Services

當使用外部服務時。

```markdown
## Third-party Services

| Service | Purpose | Config |
|---------|---------|--------|
| Supabase | Auth + DB + Storage + Realtime | SUPABASE_URL, SUPABASE_ANON_KEY |
| OpenAI / Local LLM | AI enrichment | OPENAI_API_KEY or OLLAMA_URL |
| Resend | Transactional email | RESEND_API_KEY |
```

### 5.6 Environment Variables

當需要外部設定時。

```markdown
## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| DATABASE_URL | Yes | PostgreSQL connection string |
| SUPABASE_URL | Yes | Supabase project URL |
| SUPABASE_ANON_KEY | Yes | Supabase anonymous key |
| OPENAI_API_KEY | No | OpenAI API key (if not using local LLM) |
```

### 5.7 Denied Items Implementation

當 lock 包含 `denied` 欄位時。

```markdown
## Denied Items Implementation

- videoCall: no WebRTC, no video-related dependencies, no video UI
- voiceCall: no audio streaming, no voice-related dependencies
- Reaction: no reaction model, no reaction endpoints, no emoji picker on messages
```

---

## 6. 慣例

### 6.1 命名

Plan 使用**目標語言的慣例**，而非 lock 的慣例。

| Lock 中的寫法 | TypeScript plan | Python plan | Go plan |
|-----------|----------------|-------------|---------|
| `sendMessage` (camelCase) | `sendMessage` | `send_message` | `SendMessage` |
| `User` (PascalCase) | `User` | `User` (class) / `users` (table) | `User` |

### 6.2 Schemas

- 包含所有欄位，而非僅列出 lock 指定的欄位
- Lock 有 `["id", "name", "email"]` → plan 加上型別、約束、預設值、關聯、索引
- Plan 可以（MAY）新增 lock 未提及的欄位（例如 `createdAt`、`updatedAt`）— lock 僅約束其所指定的內容

### 6.3 Endpoints

- Lock 中的每個 `features` 都必須（MUST）對應至少一個端點（或背景作業）
- 包含請求/回應結構
- 包含驗證要求
- 包含驗證規則
- 包含副作用

### 6.4 Implementation Steps

- 編號、有序
- 每個步驟應該（SHOULD）可獨立驗證（「完成步驟 5 後，驗證功能應正常運作」）
- 粒度：每個步驟 ≈ AI 的一個工作階段

---

## 7. Lock 與 Plan 的比較

| 面向 | Lock | Plan |
|--------|------|------|
| 對象 | 人類（核准/否決） | AI（實作） |
| 內容 | 產品邊界（WHAT） | 實作藍圖（HOW） |
| 格式 | JSON（確定性的） | Markdown（可讀的） |
| 技術棧 | 未提及 | 完整指定 |
| 實體欄位 | 僅有名稱 | 名稱 + 型別 + 約束 + 關聯 |
| 功能 | 能力名稱 | 完整的 API 契約 |
| 使用者故事 | 自然語言流程 | 對應至端點 + 背景作業 |
| 權限 | 角色-功能矩陣 | 驗證中介層 + RLS 策略 |
| 可攜性 | 語言無關 | 針對特定技術棧 |

---

*本規格為 [Product Lock 規格](https://spec.productlock.org) 的一部分。*
