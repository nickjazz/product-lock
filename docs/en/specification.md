---
layout: default
title: Specification
nav_order: 2
permalink: /en/specification
lang: en
lang_label: English
page_id: specification
---

# Product Lock Specification

### Version 0.1.0 — February 2026

> A product boundary specification for humans and AI.
>
> product.lock.json describes what a software product **is** and **is not**.
> It does not describe how the product is built.

The key words "MUST", "MUST NOT", "SHOULD", "SHOULD NOT", and "MAY" in this document are to be interpreted as described in [RFC 2119](https://tools.ietf.org/html/rfc2119).

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Quick Start](#2-quick-start)
3. [Design Principles](#3-design-principles)
4. [File Format](#4-file-format)
5. [Metadata Fields](#5-metadata-fields)
6. [Product Boundary Fields](#6-product-boundary-fields)
7. [Progressive Strictness](#7-progressive-strictness)
8. [Conventions](#8-conventions)
9. [Validation Rules](#9-validation-rules)
10. [Roles](#10-roles)
11. [Lifecycle](#11-lifecycle)
12. [FAQ](#12-faq)
13. [References](#13-references)
14. [Future Considerations](#14-future-considerations)

---

## 1. Introduction

### 1.1 Problem

AI can now write most of the code in a software product. But code generation without boundaries creates a new problem: **scope drift**. AI adds features no one asked for, creates entities that shouldn't exist, and builds capabilities that cross product lines.

In traditional development, products are naturally bounded by human time and effort. With AI, that constraint disappears. The product boundary must be explicitly defined.

### 1.2 Solution

`product.lock.json` is a machine-readable, human-reviewable file that defines what a software product contains and what it must not contain. It serves as a contract between three parties:

- **Human** — approves the boundary
- **AI Worker** — builds within the boundary
- **AI Reviewer** — verifies code against the boundary

### 1.3 Scope

Product Lock defines the **product layer** only:

| In scope | Out of scope |
|----------|-------------|
| Who uses the product (actors) | Routes, endpoints |
| What data it stores (entities) | Dependencies, packages |
| What it can do (features) | Framework choices |
| How things interact (stories) | File structure |
| Who can do what (permissions) | Deployment config |
| What it must not have (denied) | Code style, patterns |

### 1.4 Format Strategy

**JSON is the source of truth. Markdown is the rendered view.**

`product.lock.json` is the canonical format — deterministic, schema-validatable, one way to write it.

When presented to a human for approval, the lock is rendered as Markdown for readability. The human never needs to read raw JSON.

```
AI Worker writes JSON  →  AI Reviewer reads JSON  →  Human reads Markdown
```

---

## 2. Quick Start

### 2.1 Minimal Example

The simplest valid lock:

```json
{
  "name": "todo-app",
  "version": "0.1.0",
  "description": "Simple to-do list application",
  "author": "kim",

  "entities": ["Todo", "User"],
  "features": ["completeTodo", "createTodo", "deleteTodo"]
}
```

Four metadata fields and at least one product boundary field. That's it.

### 2.2 Typical Example

A more complete lock with all product boundary fields:

```json
{
  "name": "chat-system",
  "version": "1.0.0",
  "description": "Real-time chat with group conversations and read receipts",
  "author": "kim",

  "actors": ["Admin", "Guest", "Member"],

  "entities": {
    "Conversation": [],
    "Message": ["content", "conversationId", "createdAt", "id", "senderId"],
    "ReadReceipt": [],
    "User": ["avatar", "email", "id", "name"]
  },

  "features": ["createGroup", "listConversations", "readReceipts", "sendMessage"],

  "stories": [
    "Member sends Message to Conversation",
    "Member creates Conversation as Group and invites other Members",
    "When Member reads Message, system creates ReadReceipt",
    "Admin deletes any Message from Conversation",
    "Guest views Conversation but cannot send Message"
  ],

  "permissions": {
    "Admin": ["createGroup", "deleteMessage", "listConversations", "removeMember", "sendMessage"],
    "Guest": ["listConversations"],
    "Member": ["createGroup", "listConversations", "readReceipts", "sendMessage"]
  },

  "denied": {
    "Reaction": "Keep messaging simple, no emoji reactions",
    "deleteAccount": "Account lifecycle managed by admin only",
    "editMessage": "Messages are immutable once sent",
    "videoCall": "Text-based communication only",
    "voiceCall": "Text-based communication only"
  }
}
```

### 2.3 Lock as Spec

A lock is portable. Someone shares their `chat-system` lock; you give it to an AI:

> "Build this product in Python with FastAPI and SQLAlchemy."

Same lock, different stack. Product boundary identical.

---

## 3. Design Principles

### 3.1 Product, Not Code

Lock describes product boundaries, not technical implementation. No routes, no dependencies, no framework choices. Two products built with different stacks can share the same lock.

### 3.2 Progressive Strictness

Specify more, enforce more. Omit a field, and AI decides freely. Write `entities: ["User"]`, and the Reviewer only checks that User exists. Write `entities: { "User": ["id", "name"] }`, and the Reviewer checks exact fields. The lock enforces exactly what you specify, nothing more.

### 3.3 Scannable

A human SHOULD be able to scan the lock's structure and make an approve/deny decision without reading code. Simple products take seconds; complex products take minutes. Either way, scanning a lock is an order of magnitude faster than reviewing code.

### 3.4 Language-Agnostic

The same lock works for TypeScript, Python, Go, Java, or any other language. Lock uses product-level naming conventions (PascalCase entities, camelCase features) that AI Workers adapt to target language conventions during code generation.

### 3.5 Lock as Spec

A lock is a shareable product specification. Receiving someone's lock is equivalent to receiving their product requirements. It can be versioned, diffed, and shared like any other specification.

### 3.6 Boundary Is What You Exclude

In AI-era development, defining what a product must NOT do is more important than defining what it does. AI can add features, entities, and behaviors indefinitely. The only way to maintain product scope is to explicitly declare exclusions. The `denied` field is the guardrail.

---

## 4. File Format

- File MUST be named `product.lock.json`
- File MUST be valid JSON ([RFC 8259](https://datatracker.ietf.org/doc/html/rfc8259))
- File MUST use UTF-8 encoding
- File SHOULD be placed in the project root directory
- An optional `product.lock.md` MAY be generated as a human-readable rendered view

---

## 5. Metadata Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `$schema` | string | No | Schema URL for editor validation and autocomplete |
| `name` | string | **Yes** | Product identifier in kebab-case |
| `version` | string | **Yes** | Product version, SHOULD follow [semver](https://semver.org/) |
| `description` | string | **Yes** | One-line product description |
| `author` | string | **Yes** | Who approved this lock |
| `license` | string | No | License identifier (e.g., `"MIT"`, `"UNLICENSED"`) |
| `keywords` | string[] | No | Tags for discoverability when sharing locks |
| `private` | boolean | No | If `true`, lock is not intended for public sharing |

These fields follow [package.json](https://docs.npmjs.com/cli/v10/configuring-npm/package-json) conventions by design.

**Example:**
```json
{
  "$schema": "https://productlock.org/schema/v1.json",
  "name": "chat-system",
  "version": "1.0.0",
  "description": "Real-time chat with group conversations and read receipts",
  "author": "kim",
  "license": "MIT",
  "keywords": ["chat", "group-messaging", "realtime"],
  "private": false
}
```

---

## 6. Product Boundary Fields

All product boundary fields are **optional**. An omitted field means AI decides freely and the Reviewer does not check it.

Product boundary fields MUST appear in this order when present:

```
actors → entities → features → stories → permissions → denied
```

This follows a conceptual flow: who uses it → what data → what actions → how they interact → who can do what → what's excluded.

---

### 6.1 `actors`

**Purpose:** Define the people who use the product. These are user roles, not code entities.

**Format:** `string[]`

**Example:**
```json
"actors": ["Admin", "Guest", "Member"]
```

**Rules:**
- Entries MUST be PascalCase
- Array MUST be sorted alphabetically
- If omitted, AI decides all user roles

**Counter-example:**
```json
"actors": ["admin", "UserService", "AuthMiddleware"]
```
`admin` is not PascalCase. `UserService` and `AuthMiddleware` are code entities, not user roles.

---

### 6.2 `entities`

**Purpose:** Define the data model — what the product stores.

**Loose mode** — array of names. Reviewer only checks entities exist.

```json
"entities": ["Conversation", "Message", "ReadReceipt", "User"]
```

**Strict mode** — object with field lists. Reviewer checks entities AND their fields.

```json
"entities": {
  "Conversation": [],
  "Message": ["content", "conversationId", "createdAt", "id", "senderId"],
  "ReadReceipt": [],
  "User": ["avatar", "email", "id", "name"]
}
```

**Rules:**
- Entity names MUST be PascalCase
- Field names MUST be camelCase
- Empty array `[]` means entity MUST exist, but fields are not locked
- Non-empty array means these fields MUST exist — no more, no less
- Fields are names only, no types (types are implementation detail)
- Arrays and keys MUST be sorted alphabetically

**Counter-example:**
```json
"entities": {
  "CachedQuote": ["symbol", "price", "cachedAt"],
  "User": ["id", "name"]
}
```
`CachedQuote` is an implementation detail (cache table), not a product entity. The product requirement it serves SHOULD be captured as a story instead:
```json
"stories": ["User views StockQuote with data refreshing in real-time"]
```

**Key rule:** Entities answer "what does the product store?" If something is infrastructure (caches, queues, temp tables, migration logs), it does not belong in entities. Capture the product need it serves as a story.

---

### 6.3 `features`

**Purpose:** Define product capabilities — what the product can do.

**Format:** `string[]`

**Example:**
```json
"features": ["createGroup", "listConversations", "readReceipts", "sendMessage"]
```

**Rules:**
- Features MUST be camelCase
- Features SHOULD follow verb + noun format (`sendMessage`, `createGroup`, `viewDashboard`)
- Array MUST be sorted alphabetically
- No sub-features — keep it flat
- Each feature MUST be independently identifiable in the codebase

**Granularity guide:**

| Too broad | Good | Too narrow |
|-----------|------|-----------|
| `manageMessages` | `sendMessage`, `deleteMessage` | `validateMessageLength` |
| `handleAuth` | `login`, `register` | `hashPassword` |

Features are what a **product manager** would list, not what a **developer** would list.

**Counter-example:**
```json
"features": ["hashPassword", "validateEmail", "generateJwt"]
```
These are implementation details. The product features are `login`, `register`, `resetPassword`.

---

### 6.4 `stories`

**Purpose:** Define interaction flows, experience expectations, and system behaviors. Stories are the narrative of the product — they connect actors, entities, and features into meaningful sequences.

**Format:** `string[]`

**Example:**
```json
"stories": [
  "Member sends Message to Conversation",
  "Member creates Conversation as Group and invites other Members",
  "When Member reads Message, system creates ReadReceipt",
  "Admin deletes any Message from Conversation",
  "Guest views Conversation but cannot send Message"
]
```

**Three types of stories:**

| Type | Purpose | Example |
|------|---------|---------|
| Functional flow | What happens when someone uses the product | `"Member sends Message to Conversation"` |
| Experience expectation | What the user perceives (non-functional requirements) | `"User views Dashboard with data loading instantly"` |
| System behavior | What the system does autonomously | `"System fetches NewsArticle from multiple sources on schedule"` |

**Rules:**
- Each story MUST be one sentence
- Stories MUST start with an actor name (PascalCase, matching `actors`) or `"System"`
- Stories MUST reference entities by exact PascalCase name
- Stories MUST use present tense, active voice
- Stories MUST describe WHAT happens, not HOW it's implemented
- Stories are ordered by **narrative flow**, NOT alphabetically (this is the only exception to the alphabetical sorting rule)

**Counter-examples:**
```
"Messages are sent via WebSocket"          ← implementation detail (WebSocket)
"Use Redis to cache stock quotes"          ← implementation detail (Redis)
"The user should be able to send messages" ← spec language, not a story
"member sends message to conversation"     ← wrong casing
```

**Implementation-to-story conversion:** If something feels like an entity but is really infrastructure, capture the product requirement it serves as a story:

| Implementation detail | Story |
|----------------------|-------|
| CachedQuote table | `"User views StockQuote with data refreshing in real-time"` |
| Rate limiter middleware | `"System rate limits API requests per IP address"` |
| Embedding vector store | `"System searches knowledge base by semantic similarity"` |

---

### 6.5 `permissions`

**Purpose:** Define access control — who can do what.

**Loose mode** — just the model name. AI implements details freely.

```json
"permissions": "rbac"
```

Valid model names: `"rbac"`, `"abac"`, `"acl"`.

**Strict mode** — role-permission matrix. Reviewer checks each role has exactly these permissions.

```json
"permissions": {
  "Admin": ["createGroup", "deleteMessage", "listConversations", "removeMember", "sendMessage"],
  "Guest": ["listConversations"],
  "Member": ["createGroup", "listConversations", "readReceipts", "sendMessage"]
}
```

**Rules:**
- When object: keys MUST match `actors` entries
- When object: values MUST be subsets of `features` (or feature-like actions)
- Keys and value arrays MUST be sorted alphabetically
- System-only features (e.g., `fetchNews`, `enrichData`) do NOT appear in permissions — they are not user-invocable
- If omitted, AI decides all access control

**Counter-example:**
```json
"permissions": {
  "admin": ["send-message"]
}
```
`admin` is not PascalCase (MUST match actors). `send-message` is not camelCase (MUST match features).

---

### 6.6 `denied`

**Purpose:** Define explicit exclusions — what the product must NOT have.

**This is the most important field in AI-era development.**

In traditional development, products are naturally bounded by human time and effort. With AI, there is no natural limit — AI can add features, entities, and behaviors indefinitely. The only way to maintain product scope is to explicitly declare what must NOT exist. Other fields define what the product IS. `denied` defines what the product IS NOT. Without `denied`, AI has no guardrail.

**Loose mode** — flat list.

```json
"denied": ["Reaction", "editMessage", "voiceCall"]
```

**With reasons** (recommended) — items with explanation. Reasons help the Reviewer understand the weight of each exclusion and help the Human make informed decisions.

```json
"denied": {
  "executeTrade": "Market intelligence only — no brokerage liability",
  "Reaction": "Intentionally excluded to keep messaging simple",
  "voiceCall": "Text-based communication only"
}
```

**Rules:**
- Reviewer MUST check that denied items do NOT exist in the codebase
- If found, it is a violation
- Denied keys MUST be PascalCase (if entity-like) or camelCase (if feature-like)
- Keys MUST be sorted alphabetically
- Reasons are informational — they do not change verification behavior

**What belongs in denied:**

| Category | Example | Reason |
|----------|---------|--------|
| Product scope | `"voiceCall"` | "Text-based communication only" |
| Liability boundary | `"executeTrade"` | "No brokerage liability" |
| Scope guardrail | `"Reaction"` | "Keep messaging simple" |

**What does NOT belong in denied:**
- Features that haven't been built yet (that's just the backlog, not a product decision)
- Implementation approaches (`"don't use Redis"` — that's HOW, not WHAT)
- Temporary limitations that will be added in the next version

**Counter-example:**
```json
"denied": ["exportData", "multiLanguage", "mobileApp", "paymentBilling"]
```
These are features that haven't been built yet, not intentional product exclusions. Ask: "If an AI accidentally added this, would it be a product violation?" If no, it doesn't belong in denied.

---

## 7. Progressive Strictness

The lock enforces exactly what you specify. Nothing more.

| What you write | What the Reviewer checks |
|----------------|------------------------|
| Field omitted entirely | Nothing — AI decides freely |
| `"entities": ["User"]` | User entity exists |
| `"entities": { "User": [] }` | User entity exists (same as above) |
| `"entities": { "User": ["id", "name"] }` | User has exactly `id` and `name`, no more, no less |
| `"features": ["sendMessage"]` | sendMessage behavior exists in codebase |
| `"stories": [...]` | Each described flow exists in codebase |
| `"permissions": "rbac"` | Some form of RBAC exists |
| `"permissions": { "Admin": ["delete"] }` | Admin has exactly these permissions |
| `"denied": ["Reaction"]` | Reaction does NOT exist anywhere |
| `"denied": { "Reaction": "reason" }` | Same check, reason documents the decision |

**Rule: what's not locked is free.** AI MAY add fields to unlocked entities, add features not in the list, create entities not mentioned. The Reviewer only checks what's in the lock.

**Exception: `denied`.** Denied items are always checked for absence regardless of other fields. This is the guardrail that prevents AI from exceeding product scope.

---

## 8. Conventions

These conventions ensure every lock looks the same regardless of who — or which AI — generates it.

### 8.1 Naming

| Field | Convention | Example |
|-------|-----------|---------|
| `name` | kebab-case | `"chat-system"` |
| `actors` | PascalCase | `"Admin"`, `"Member"` |
| `entities` | PascalCase | `"User"`, `"ReadReceipt"` |
| entity fields | camelCase | `"senderId"`, `"createdAt"` |
| `features` | camelCase | `"sendMessage"`, `"createGroup"` |
| `permissions` keys | PascalCase | `"Admin"`, `"Guest"` |
| `permissions` values | camelCase | `"deleteMessage"` |
| `denied` keys | PascalCase or camelCase | `"Reaction"`, `"editMessage"` |
| `denied` values | Free-form string | `"Text-based only"` |
| `keywords` | lowercase | `"chat"`, `"realtime"` |

The lock defines **product names**, not code names. When generating code, AI Workers adapt naming to target language conventions (e.g., `sendMessage` in lock becomes `send_message` in Python).

### 8.2 Ordering

All arrays and object keys MUST be sorted alphabetically.

```json
"actors": ["Admin", "Guest", "Member"]

"entities": {
  "Conversation": [],
  "Message": ["content", "conversationId", "createdAt", "id", "senderId"],
  "User": ["avatar", "email", "id", "name"]
}

"features": ["createGroup", "listConversations", "readReceipts", "sendMessage"]
```

**Exception:** `stories` are ordered by narrative flow, not alphabetically. The sequence tells a story.

### 8.3 Key Order

Top-level keys MUST appear in this order:

```
$schema → name → version → description → author → license → keywords → private
→ actors → entities → features → stories → permissions → denied
```

Metadata first, then product boundary fields in conceptual order.

### 8.4 Stories Format

- One sentence per story
- Start with an actor name or `"System"`
- Reference entities by exact PascalCase name
- Present tense, active voice
- Three types: **functional flows**, **experience expectations**, **system behaviors**

---

## 9. Validation Rules

A valid `product.lock.json` MUST satisfy all of the following:

| # | Rule | Severity |
|---|------|----------|
| 1 | `name`, `version`, `description`, `author` MUST be present | Error |
| 2 | At least one product boundary field MUST be present | Error |
| 3 | `entities` MUST be `string[]` or `Record<string, string[]>` | Error |
| 4 | `features` MUST be `string[]` | Error |
| 5 | `actors` MUST be `string[]` | Error |
| 6 | `stories` MUST be `string[]` | Error |
| 7 | `permissions` MUST be `string` or `Record<string, string[]>` | Error |
| 8 | `denied` MUST be `string[]` or `Record<string, string>` | Error |
| 9 | `permissions` keys MUST match `actors` entries (when both defined) | Error |
| 10 | `denied` items MUST NOT appear in `entities` or `features` | Error |
| 11 | All arrays and object keys MUST be sorted alphabetically (except `stories`) | Error |
| 12 | `name` MUST be kebab-case; `entities` MUST be PascalCase; `features` MUST be camelCase; `actors` MUST be PascalCase | Error |
| 13 | Entity/feature/actor names referenced in `stories` SHOULD exist in their respective fields | Warning |
| 14 | `version` SHOULD follow semver format | Warning |

---

## 10. Roles

### 10.1 Human (Boss)

- Scans the lock (rendered as Markdown)
- Approves or denies
- Modifies the `denied` list
- Does NOT read code, does NOT review implementation

### 10.2 AI Worker

1. Writes code
2. Generates `product.lock.json` from code
3. Prepares evidence (internal, not shown to human)
4. Submits lock + evidence to Reviewer

### 10.3 AI Reviewer

1. Receives lock from Worker
2. Independently inspects the codebase (does NOT trust Worker's evidence)
3. Checks:
   - Every locked entity exists (with correct fields if strict mode)
   - Every locked feature exists as identifiable behavior
   - Every story's flow exists in code
   - Every permission is correctly enforced
   - Every denied item does NOT exist
   - No undeclared entities or features exist that should be in the lock
4. Reports: **pass** or **violation list**

---

## 11. Lifecycle

```
1. Human describes intent (natural language)
        ↓
2. AI Worker builds the product
        ↓
3. AI Worker generates product.lock.json from code
        ↓
4. Human scans lock (Markdown view) → approve / modify / deny
        ↓
5. Lock is frozen
        ↓
6. AI Worker continues development within lock boundary
        ↓
7. AI Reviewer periodically checks code against lock
        ↓
8. Violations reported → Human decides action
        ↓
9. Next iteration → version bump → repeat from step 4
```

---

## 12. FAQ

### Why JSON and not YAML?

JSON is deterministic — there is exactly one way to represent the same data. YAML has multiple equivalent representations (indentation styles, flow vs block, quoting rules). For a specification that AI generates and AI validates, determinism matters more than human writability. The human reads the Markdown view anyway.

### When should I use loose mode vs strict mode?

Use **loose mode** when the general structure matters but details don't. Use **strict mode** when specific fields are critical to the product (e.g., financial data models, compliance-sensitive entities). Start loose, tighten when needed.

### What goes in `denied` vs what's just "not built yet"?

Ask: "If an AI accidentally added this, would it be a product violation?" If yes, it belongs in `denied`. If no, it's just the backlog.

### Can the same lock produce different codebases?

Yes. The lock defines WHAT, not HOW. The same lock with different tech stacks produces structurally different code that implements the same product boundary. Pair the lock with a `product.plan.md` to also specify the HOW.

### What about microservices? One lock per service?

One lock per **product**, not per service. If your microservices together form one product, they share one lock. If they are independent products, each gets its own lock.

### Can I extend the lock with custom fields?

Not in v0.1. Future versions may support extension fields with a `x-` prefix pattern. For now, use only the defined fields.

---

## 13. References

- [RFC 8259 — The JavaScript Object Notation (JSON) Data Interchange Format](https://datatracker.ietf.org/doc/html/rfc8259)
- [RFC 2119 — Key words for use in RFCs to Indicate Requirement Levels](https://tools.ietf.org/html/rfc2119)
- [Semantic Versioning 2.0.0](https://semver.org/)
- [package.json — npm documentation](https://docs.npmjs.com/cli/v10/configuring-npm/package-json)

---

## 14. Future Considerations

These features are not part of v0.1 but may be added in future versions:

- `extends` — inherit from another lock and override specific fields
- `milestones` — phase tagging (MVP / v2 / future)
- `contributors` — multiple authors
- `repository` — source code location
- `changelog` — lock version diff history
- Extension fields with `x-` prefix

---

*Product Lock Specification is licensed under [MIT License](https://opensource.org/licenses/MIT).*
*Specification source: [productlock.org](https://spec.productlock.org)*
