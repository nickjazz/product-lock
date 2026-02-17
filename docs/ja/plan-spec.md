---
layout: default
title: Plan Spec
nav_order: 6
permalink: /ja/plan-spec
lang: ja
lang_label: 日本語
page_id: plan-spec
---

# Product Plan 仕様書

### バージョン 0.1.0 — 2026年2月

> Product Plan は、プロダクトの実装方法を記述する。product.lock.json の実装ブループリントである。
>
> Lock は WHAT（何を構築するか）を定義する。Plan は HOW（どう構築するか）を定義する。
>
> Plan はAIが読むためのものである。人間はスキップできるが、問題が起きたときに理解できる。

完全なロック仕様については、[Product Lock Specification](https://spec.productlock.org) を参照のこと。

---

## 目次

1. [ファイル間の関係](#1-ファイル間の関係)
2. [設計原則](#2-設計原則)
3. [フォーマット](#3-フォーマット)
4. [必須セクション](#4-必須セクション)
5. [任意セクション](#5-任意セクション)
6. [規約](#6-規約)
7. [Lock と Plan の比較](#7-lock-と-plan-の比較)

---

## 1. ファイル間の関係

```
product.lock.json    →  プロダクトが何であるか（WHAT）（人間が承認する）
product.plan.md      →  どう構築するか（HOW）（AIが従う）
codebase             →  実際のプロダクト（AI Reviewer がロックに照らして検証する）
```

| 持っているもの | できること |
|----------|-----------|
| Lock のみ | AIがプロダクトを構築し、実装に関する判断を自ら行う |
| Lock + Plan | AIが同じアーキテクチャで全く同じプロダクトを構築する |
| Lock + Plan + Codebase | AI Reviewer がすべてが一致しているか検証する |
| Plan のみ | AIは構築できるが、検証すべきプロダクト境界がない |

---

## 2. 設計原則

1. **Lock から派生する** — Plan は Lock が定義するすべてを、省略なく矛盾なく実装しなければならない（MUST）
2. **再現可能** — 同じ Plan を読んだ異なるAIエージェントが、構造的に同一のコードベースを生成すべきである（SHOULD）
3. **プレーンテキスト** — Markdown 形式。圧縮、エンコード、暗号化はしない
4. **完全** — Plan には Lock がカバーしないすべての実装判断が含まれる。この Plan を持つAIは直接コードを書ける
5. **スタック固有** — 同じ Lock に対して複数の Plan を持てる（TypeScript 版、Python 版、Go 版）

---

## 3. フォーマット

ファイル名は `product.plan.md` でなければならず（MUST）、`product.lock.json` と並べてプロジェクトルートに配置する。

```markdown
# Product Plan: {name} v{version}

> Lock: product.lock.json
> Stack: {一行のスタック要約}
> Generated: {日付}

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

{型、制約、リレーションを含む完全なエンティティスキーマ。}

## Endpoints

{すべての機能の API コントラクト。}

## Implementation Steps

{ゼロから構築するための順序付きステップ。}
```

---

## 4. 必須セクション

すべての Plan は以下のセクションを含まなければならない（MUST）。

### 4.1 Stack

プロダクトの各レイヤーの技術選定。

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

型、制約、リレーション、インデックスを含む完全なエンティティ定義。Lock の entities がコードで何になるかを示す。

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

Plan は Lock で指定されたフィールドだけでなく、すべてのフィールドを含まなければならない（MUST）。Lock に `["id", "name", "email"]` がある場合、Plan は型、制約、デフォルト値、リレーション、インデックスを追加する。Plan は Lock に記載されていないフィールド（例：`createdAt`、`updatedAt`）を追加してもよい（MAY）。

### 4.3 Endpoints

各機能を実装する API エンドポイント。Lock 内のすべての機能は少なくとも1つのエンドポイント（またはバックグラウンドジョブ）にマッピングされなければならない（MUST）。

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

各エンドポイントは以下を含まなければならない（MUST）：
- HTTP メソッドとパス
- 認証要件
- リクエストボディ / クエリパラメータ
- レスポンスの形状とステータスコード
- バリデーションルール
- 副作用（ある場合）

### 4.4 Implementation Steps

ゼロから構築するための順序付きステップ。AIがこれらのステップに従うことで完全なプロダクトを生成する。

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

ルール：
- 番号付き、順序付き
- 各ステップは独立して検証可能であるべき（SHOULD）
- 粒度：各ステップ ≒ AIの1作業セッション

---

## 5. 任意セクション

プロダクトが必要とする場合にこれらのセクションを含める。

### 5.1 Architecture

非自明なパターンが使用される場合（マイクロサービス、イベントソーシング、CQRS など）。

```markdown
## Architecture

REST API with Next.js Route Handlers. Supabase handles auth, database, storage, and real-time subscriptions. Single deployment on Vercel.

- API: Next.js Route Handlers (REST)
- Auth: Supabase middleware, RLS on database
- Real-time: Supabase Realtime subscriptions
- File storage: Supabase Storage
```

### 5.2 File Structure

プロジェクトのレイアウトが非標準または説明が必要な場合。

```markdown
## File Structure

├── prisma/schema.prisma
├── src/
│   ├── app/
│   │   ├── (auth)/login/
│   │   ├── (dashboard)/
│   │   └── api/
│   ├── components/
│   │   ├── ui/          ← shadcn プリミティブ
│   │   └── features/    ← ドメインコンポーネント
│   ├── lib/
│   │   ├── db.ts
│   │   ├── auth.ts
│   │   └── validations/
│   ├── hooks/
│   └── types/
└── package.json
```

### 5.3 Auth & Permissions

Lock に `actors` や `permissions` がある場合。

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

Lock にシステムストーリー（自律的な振る舞い）がある場合。

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

外部サービスを使用する場合。

```markdown
## Third-party Services

| Service | Purpose | Config |
|---------|---------|--------|
| Supabase | Auth + DB + Storage + Realtime | SUPABASE_URL, SUPABASE_ANON_KEY |
| OpenAI / Local LLM | AI enrichment | OPENAI_API_KEY or OLLAMA_URL |
| Resend | Transactional email | RESEND_API_KEY |
```

### 5.6 Environment Variables

外部設定が必要な場合。

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

Lock に `denied` フィールドがある場合。

```markdown
## Denied Items Implementation

- videoCall: no WebRTC, no video-related dependencies, no video UI
- voiceCall: no audio streaming, no voice-related dependencies
- Reaction: no reaction model, no reaction endpoints, no emoji picker on messages
```

---

## 6. 規約

### 6.1 命名規則

Plan は Lock の規約ではなく、**ターゲット言語の規約**を使用する。

| Lock の記述 | TypeScript の Plan | Python の Plan | Go の Plan |
|-----------|----------------|-------------|---------|
| `sendMessage` (camelCase) | `sendMessage` | `send_message` | `SendMessage` |
| `User` (PascalCase) | `User` | `User` (class) / `users` (table) | `User` |

### 6.2 Schemas

- Lock で指定されたフィールドだけでなく、すべてのフィールドを含める
- Lock に `["id", "name", "email"]` がある場合、Plan は型、制約、デフォルト値、リレーション、インデックスを追加する
- Plan は Lock に記載されていないフィールド（例：`createdAt`、`updatedAt`）を追加してもよい（MAY）— Lock は指定されたものだけを制約する

### 6.3 Endpoints

- Lock 内のすべての機能は少なくとも1つのエンドポイント（またはバックグラウンドジョブ）にマッピングされなければならない（MUST）
- リクエスト/レスポンスの形状を含める
- 認証要件を含める
- バリデーションルールを含める
- 副作用を含める

### 6.4 Implementation Steps

- 番号付き、順序付き
- 各ステップは独立して検証可能であるべき（SHOULD）（「ステップ5の後、認証が動作するはず」）
- 粒度：各ステップ ≒ AIの1作業セッション

---

## 7. Lock と Plan の比較

| 側面 | Lock | Plan |
|--------|------|------|
| 対象 | 人間（承認/拒否） | AI（実装） |
| 内容 | プロダクト境界（WHAT） | 実装ブループリント（HOW） |
| フォーマット | JSON（決定的） | Markdown（可読） |
| スタック | 言及しない | 完全に指定 |
| エンティティフィールド | 名前のみ | 名前 + 型 + 制約 + リレーション |
| 機能 | 能力の名前 | 完全な API コントラクト |
| ストーリー | 自然言語のフロー | エンドポイント + バックグラウンドジョブへのマッピング |
| パーミッション | ロール-機能マトリクス | 認証ミドルウェア + RLS ポリシー |
| ポータビリティ | 言語非依存 | スタック固有 |

---

*この仕様は [Product Lock Specification](https://spec.productlock.org) の一部です。*
