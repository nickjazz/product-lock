---
layout: default
title: Plan Spec
nav_order: 6
permalink: /en/plan-spec
lang: en
lang_label: English
page_id: plan-spec
---

# Product Plan Specification

### Version 0.1.0 — February 2026

> Product Plan describes how to implement a product. It is the implementation blueprint for product.lock.json.
>
> Lock defines WHAT (what to build). Plan defines HOW (how to build it).
>
> Plan is for AI to read. Humans can skip it, but when something goes wrong, humans can understand it.

For the full lock specification, see the [Product Lock Specification](https://spec.productlock.org).

---

## Table of Contents

1. [Relationship Between Files](#1-relationship-between-files)
2. [Design Principles](#2-design-principles)
3. [Format](#3-format)
4. [Required Sections](#4-required-sections)
5. [Optional Sections](#5-optional-sections)
6. [Conventions](#6-conventions)
7. [Lock vs Plan](#7-lock-vs-plan)

---

## 1. Relationship Between Files

```
product.lock.json    →  WHAT the product is (human approves)
product.plan.md      →  HOW to build it (AI follows)
codebase             →  The actual product (AI Reviewer verifies against lock)
```

| You have | You can do |
|----------|-----------|
| Lock only | AI builds product, makes its own implementation decisions |
| Lock + Plan | AI builds the exact same product with the same architecture |
| Lock + Plan + Codebase | AI Reviewer verifies everything matches |
| Plan only | AI can build but has no product boundary to verify against |

---

## 2. Design Principles

1. **Derived from Lock** — Plan MUST implement everything the lock defines, with no omissions and no conflicts
2. **Reproducible** — Different AI agents reading the same plan SHOULD produce structurally identical codebases
3. **Plain text** — Markdown format, not compressed, not encoded, not encrypted
4. **Complete** — Plan contains all implementation decisions the lock does not cover; an AI with this plan can write code directly
5. **Stack-specific** — The same lock can have multiple plans (TypeScript version, Python version, Go version)

---

## 3. Format

The file MUST be named `product.plan.md` and placed in the project root alongside `product.lock.json`.

```markdown
# Product Plan: {name} v{version}

> Lock: product.lock.json
> Stack: {one-line stack summary}
> Generated: {date}

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

{Full entity schemas with types, constraints, relations.}

## Endpoints

{Every feature's API contract.}

## Implementation Steps

{Ordered steps to build from zero.}
```

---

## 4. Required Sections

Every plan MUST include the following sections.

### 4.1 Stack

Technology choices at every layer of the product.

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

Full entity definitions with types, constraints, relations, and indexes. This is what the lock's entities become in code.

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

The plan MUST include ALL fields, not just lock-specified fields. The lock has `["id", "name", "email"]` → the plan adds types, constraints, defaults, relations, indexes. The plan MAY add fields the lock doesn't mention (e.g., `createdAt`, `updatedAt`).

### 4.3 Endpoints

API endpoints implementing each feature. Every feature in the lock MUST map to at least one endpoint (or background job).

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

Each endpoint MUST include:
- HTTP method and path
- Auth requirements
- Request body / query parameters
- Response shape and status code
- Validation rules
- Side effects (if any)

### 4.4 Implementation Steps

Ordered steps to build from zero. An AI following these steps produces the complete product.

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

Rules:
- Numbered, ordered
- Each step SHOULD be independently verifiable
- Granularity: each step ≈ one work session for AI

---

## 5. Optional Sections

Include these sections when the product requires them.

### 5.1 Architecture

When non-trivial patterns are used (microservices, event sourcing, CQRS, etc.).

```markdown
## Architecture

REST API with Next.js Route Handlers. Supabase handles auth, database, storage, and real-time subscriptions. Single deployment on Vercel.

- API: Next.js Route Handlers (REST)
- Auth: Supabase middleware, RLS on database
- Real-time: Supabase Realtime subscriptions
- File storage: Supabase Storage
```

### 5.2 File Structure

When the project layout is non-standard or needs explanation.

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

When the lock has `actors` or `permissions`.

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

When the lock has system stories (autonomous behaviors).

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

When external services are used.

```markdown
## Third-party Services

| Service | Purpose | Config |
|---------|---------|--------|
| Supabase | Auth + DB + Storage + Realtime | SUPABASE_URL, SUPABASE_ANON_KEY |
| OpenAI / Local LLM | AI enrichment | OPENAI_API_KEY or OLLAMA_URL |
| Resend | Transactional email | RESEND_API_KEY |
```

### 5.6 Environment Variables

When external configuration is needed.

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

When the lock has a `denied` field.

```markdown
## Denied Items Implementation

- videoCall: no WebRTC, no video-related dependencies, no video UI
- voiceCall: no audio streaming, no voice-related dependencies
- Reaction: no reaction model, no reaction endpoints, no emoji picker on messages
```

---

## 6. Conventions

### 6.1 Naming

The plan uses **target language conventions**, not lock conventions.

| Lock says | TypeScript plan | Python plan | Go plan |
|-----------|----------------|-------------|---------|
| `sendMessage` (camelCase) | `sendMessage` | `send_message` | `SendMessage` |
| `User` (PascalCase) | `User` | `User` (class) / `users` (table) | `User` |

### 6.2 Schemas

- Include ALL fields, not just lock-specified fields
- Lock has `["id", "name", "email"]` → plan adds types, constraints, defaults, relations, indexes
- Plan MAY add fields the lock doesn't mention (e.g., `createdAt`, `updatedAt`) — the lock only constrains what it specifies

### 6.3 Endpoints

- Every feature in the lock MUST map to at least one endpoint (or background job)
- Include request/response shapes
- Include auth requirements
- Include validation rules
- Include side effects

### 6.4 Implementation Steps

- Numbered, ordered
- Each step SHOULD be independently verifiable ("after step 5, auth should work")
- Granularity: each step ≈ one work session for AI

---

## 7. Lock vs Plan

| Aspect | Lock | Plan |
|--------|------|------|
| Audience | Human (approve/deny) | AI (implement) |
| Content | Product boundary (WHAT) | Implementation blueprint (HOW) |
| Format | JSON (deterministic) | Markdown (readable) |
| Stack | Not mentioned | Fully specified |
| Entity fields | Names only | Names + types + constraints + relations |
| Features | Capability names | Full API contracts |
| Stories | Natural language flows | Mapped to endpoints + background jobs |
| Permissions | Role-feature matrix | Auth middleware + RLS policies |
| Portability | Language-agnostic | Stack-specific |

---

*This specification is part of the [Product Lock Specification](https://spec.productlock.org).*
