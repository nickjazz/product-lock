---
layout: default
title: Plan-Spezifikation
nav_order: 6
permalink: /de/plan-spec
lang: de
lang_label: Deutsch
page_id: plan-spec
---

# Product Plan Spezifikation

### Version 0.1.0 — Februar 2026

> Product Plan beschreibt, wie ein Produkt implementiert wird. Er ist die Implementierungsblaupause fuer product.lock.json.
>
> Lock definiert WAS (was gebaut werden soll). Plan definiert WIE (wie es gebaut wird).
>
> Plan ist fuer KI zum Lesen gedacht. Menschen koennen ihn ueberspringen, aber wenn etwas schiefgeht, koennen Menschen ihn verstehen.

Die vollstaendige Lock-Spezifikation findest du in der [Product Lock Spezifikation](https://spec.productlock.org).

---

## Inhaltsverzeichnis

1. [Beziehung zwischen den Dateien](#1-beziehung-zwischen-den-dateien)
2. [Designprinzipien](#2-designprinzipien)
3. [Format](#3-format)
4. [Erforderliche Abschnitte](#4-erforderliche-abschnitte)
5. [Optionale Abschnitte](#5-optionale-abschnitte)
6. [Konventionen](#6-konventionen)
7. [Lock vs Plan](#7-lock-vs-plan)

---

## 1. Beziehung zwischen den Dateien

```
product.lock.json    →  WAS das Produkt ist (Mensch genehmigt)
product.plan.md      →  WIE es gebaut wird (KI folgt)
codebase             →  Das eigentliche Produkt (KI-Reviewer verifiziert gegen Lock)
```

| Du hast | Du kannst |
|----------|-----------|
| Nur Lock | KI baut Produkt, trifft eigene Implementierungsentscheidungen |
| Lock + Plan | KI baut exakt dasselbe Produkt mit derselben Architektur |
| Lock + Plan + Codebasis | KI-Reviewer verifiziert, dass alles uebereinstimmt |
| Nur Plan | KI kann bauen, hat aber keine Produktgrenze zur Verifizierung |

---

## 2. Designprinzipien

1. **Vom Lock abgeleitet** — Plan MUSS alles implementieren, was das Lock definiert, ohne Auslassungen und ohne Konflikte
2. **Reproduzierbar** — Verschiedene KI-Agenten, die denselben Plan lesen, SOLLTEN strukturell identische Codebasen erzeugen
3. **Klartext** — Markdown-Format, nicht komprimiert, nicht kodiert, nicht verschluesselt
4. **Vollstaendig** — Plan enthaelt alle Implementierungsentscheidungen, die das Lock nicht abdeckt; eine KI mit diesem Plan kann direkt Code schreiben
5. **Stack-spezifisch** — Dasselbe Lock kann mehrere Plans haben (TypeScript-Version, Python-Version, Go-Version)

---

## 3. Format

Die Datei MUSS `product.plan.md` heissen und im Projektstammverzeichnis neben `product.lock.json` platziert werden.

```markdown
# Product Plan: {name} v{version}

> Lock: product.lock.json
> Stack: {einzeilige Stack-Zusammenfassung}
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

{Vollstaendige Entitaetsschemata mit Typen, Einschraenkungen, Relationen.}

## Endpoints

{Jeden Feature-API-Vertrag.}

## Implementation Steps

{Geordnete Schritte zum Aufbau von Null.}
```

---

## 4. Erforderliche Abschnitte

Jeder Plan MUSS die folgenden Abschnitte enthalten.

### 4.1 Stack

Technologiewahl auf jeder Ebene des Produkts.

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

Vollstaendige Entitaetsdefinitionen mit Typen, Einschraenkungen, Relationen und Indizes. Dies ist das, was die Entitaeten des Locks im Code werden.

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

Der Plan MUSS ALLE Felder enthalten, nicht nur die im Lock spezifizierten. Das Lock hat `["id", "name", "email"]` → der Plan fuegt Typen, Einschraenkungen, Standardwerte, Relationen, Indizes hinzu. Der Plan KANN Felder hinzufuegen, die das Lock nicht erwaehnt (z.B. `createdAt`, `updatedAt`).

### 4.3 Endpoints

API-Endpunkte, die jedes Feature implementieren. Jedes Feature im Lock MUSS auf mindestens einen Endpunkt (oder Hintergrundjob) abgebildet werden.

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

Jeder Endpunkt MUSS enthalten:
- HTTP-Methode und Pfad
- Auth-Anforderungen
- Request-Body / Query-Parameter
- Response-Form und Statuscode
- Validierungsregeln
- Seiteneffekte (falls vorhanden)

### 4.4 Implementation Steps

Geordnete Schritte zum Aufbau von Null. Eine KI, die diesen Schritten folgt, erzeugt das vollstaendige Produkt.

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

Regeln:
- Nummeriert, geordnet
- Jeder Schritt SOLLTE unabhaengig verifizierbar sein
- Granularitaet: jeder Schritt ≈ eine Arbeitssitzung fuer KI

---

## 5. Optionale Abschnitte

Diese Abschnitte einbeziehen, wenn das Produkt sie erfordert.

### 5.1 Architecture

Wenn nicht-triviale Patterns verwendet werden (Microservices, Event Sourcing, CQRS usw.).

```markdown
## Architecture

REST API with Next.js Route Handlers. Supabase handles auth, database, storage, and real-time subscriptions. Single deployment on Vercel.

- API: Next.js Route Handlers (REST)
- Auth: Supabase middleware, RLS on database
- Real-time: Supabase Realtime subscriptions
- File storage: Supabase Storage
```

### 5.2 File Structure

Wenn das Projektlayout nicht standardmaessig ist oder Erklaerung braucht.

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

Wenn das Lock `actors` oder `permissions` hat.

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

Wenn das Lock System-Stories hat (autonome Verhaltensweisen).

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

Wenn externe Dienste verwendet werden.

```markdown
## Third-party Services

| Service | Purpose | Config |
|---------|---------|--------|
| Supabase | Auth + DB + Storage + Realtime | SUPABASE_URL, SUPABASE_ANON_KEY |
| OpenAI / Local LLM | AI enrichment | OPENAI_API_KEY or OLLAMA_URL |
| Resend | Transactional email | RESEND_API_KEY |
```

### 5.6 Environment Variables

Wenn externe Konfiguration benoetigt wird.

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

Wenn das Lock ein `denied`-Feld hat.

```markdown
## Denied Items Implementation

- videoCall: no WebRTC, no video-related dependencies, no video UI
- voiceCall: no audio streaming, no voice-related dependencies
- Reaction: no reaction model, no reaction endpoints, no emoji picker on messages
```

---

## 6. Konventionen

### 6.1 Benennung

Der Plan verwendet **Konventionen der Zielsprache**, nicht Lock-Konventionen.

| Lock sagt | TypeScript-Plan | Python-Plan | Go-Plan |
|-----------|----------------|-------------|---------|
| `sendMessage` (camelCase) | `sendMessage` | `send_message` | `SendMessage` |
| `User` (PascalCase) | `User` | `User` (Klasse) / `users` (Tabelle) | `User` |

### 6.2 Schemas

- ALLE Felder einschliessen, nicht nur die im Lock spezifizierten
- Lock hat `["id", "name", "email"]` → Plan fuegt Typen, Einschraenkungen, Standardwerte, Relationen, Indizes hinzu
- Plan KANN Felder hinzufuegen, die das Lock nicht erwaehnt (z.B. `createdAt`, `updatedAt`) — das Lock beschraenkt nur, was es spezifiziert

### 6.3 Endpoints

- Jedes Feature im Lock MUSS auf mindestens einen Endpunkt (oder Hintergrundjob) abgebildet werden
- Request-/Response-Formen einschliessen
- Auth-Anforderungen einschliessen
- Validierungsregeln einschliessen
- Seiteneffekte einschliessen

### 6.4 Implementation Steps

- Nummeriert, geordnet
- Jeder Schritt SOLLTE unabhaengig verifizierbar sein ("nach Schritt 5 sollte Auth funktionieren")
- Granularitaet: jeder Schritt ≈ eine Arbeitssitzung fuer KI

---

## 7. Lock vs Plan

| Aspekt | Lock | Plan |
|--------|------|------|
| Zielgruppe | Mensch (genehmigen/ablehnen) | KI (implementieren) |
| Inhalt | Produktgrenze (WAS) | Implementierungsblaupause (WIE) |
| Format | JSON (deterministisch) | Markdown (lesbar) |
| Stack | Nicht erwaehnt | Vollstaendig spezifiziert |
| Entitaetsfelder | Nur Namen | Namen + Typen + Einschraenkungen + Relationen |
| Features | Faehigkeitsnamen | Vollstaendige API-Vertraege |
| Stories | Natuerlichsprachliche Ablaeufe | Auf Endpunkte + Hintergrundjobs abgebildet |
| Permissions | Rollen-Feature-Matrix | Auth-Middleware + RLS-Policies |
| Portabilitaet | Sprachunabhaengig | Stack-spezifisch |

---

*Diese Spezifikation ist Teil der [Product Lock Spezifikation](https://spec.productlock.org).*
