# Product Lock Generator Guide

### Version 0.1.0 — February 2026

> You are an AI Worker. Your job is to analyze a codebase and generate a `product.lock.json` that accurately describes the product's boundary.
>
> You generate the lock. A human approves it. An AI Reviewer verifies code against it.

For the full specification, see the [Product Lock Specification](https://spec.productlock.org).

---

## Table of Contents

1. [What is product.lock.json?](#1-what-is-productlockjson)
2. [Generation Process](#2-generation-process)
3. [Field Reference](#3-field-reference)
4. [Naming Conventions](#4-naming-conventions)
5. [Validation Checklist](#5-validation-checklist)
6. [Common Mistakes](#6-common-mistakes)
7. [Full Example](#7-full-example)

---

## 1. What is product.lock.json?

A `product.lock.json` defines **what a software product is and isn't** — at the product level, not the code level.

It answers six questions:

| Field | Question | Format |
|-------|----------|--------|
| `actors` | Who uses this product? | PascalCase nouns |
| `entities` | What data does it store? | PascalCase nouns |
| `features` | What can it do? | camelCase verbs |
| `stories` | How do actors, entities, and features interact? | Natural language sentences |
| `permissions` | Who can do what? | Actor-to-feature matrix |
| `denied` | What must it NOT have? | Explicit exclusions |

**The lock describes product boundary, not implementation.** No routes, no dependencies, no framework choices, no file structure.

---

## 2. Generation Process

### Step 1: Read the codebase

Scan the codebase to understand:

- Database schemas / models / types → entities
- API routes / handlers / controllers → features
- Auth / role definitions → actors, permissions
- UI pages / views → features, stories
- Middleware / background jobs / cron → stories
- Tests → help confirm what features exist

### Step 2: Extract metadata

```json
{
  "name": "<kebab-case product name>",
  "version": "<semver, start with 0.1.0 if new>",
  "description": "<one-line product description>",
  "author": "<who will approve this lock>"
}
```

- `name`: derived from project folder name or package.json, always kebab-case
- `description`: describe the product's purpose in one sentence, from a user's perspective
- `author`: ask the human, or use the git committer name

### Step 3: Identify actors

Actors are **the people who use the product**, not code entities.

Look for:
- Role definitions in auth/RBAC code
- Different permission levels
- Distinct user types in the UI

```json
"actors": ["Admin", "Member", "Guest"]
```

Rules:
- PascalCase
- Sorted alphabetically
- NOT code entities (no `UserService`, `AuthMiddleware`)
- If only one type of user exists, use `["User"]`

### Step 4: Extract entities

Entities are **what the product stores** — the data model.

Look for:
- Database tables / Prisma models / TypeORM entities
- Mongoose schemas / SQLAlchemy models
- TypeScript interfaces with `id` fields
- GraphQL types

**Decision: Loose or Strict mode?**

Use **loose mode** when:
- Quick overview is sufficient
- Fields are standard and predictable
- Human doesn't need to lock specific fields

```json
"entities": ["Conversation", "Message", "User"]
```

Use **strict mode** when:
- Fields are critical to the product (e.g., financial data)
- Human needs to verify exact data model
- Product has complex or unusual field requirements

```json
"entities": {
  "Message": ["content", "conversationId", "createdAt", "id", "senderId"],
  "User": ["avatar", "email", "id", "name"]
}
```

Rules:
- PascalCase for entity names
- camelCase for field names
- Sorted alphabetically (both keys and field arrays)
- `[]` = entity exists, fields not locked
- Fields are names only — **no types** (type is implementation detail)

**Critical judgment: Entity or NOT an entity?**

| Include as entity | Do NOT include |
|-------------------|---------------|
| User, Product, Order — domain data | CachedQuote, TempSession — implementation detail |
| Message, Conversation — user-facing | MigrationLog, AuditTrail — operational |
| StockMetadata — product stores this | RedisKey, QueueJob — infrastructure |

If something looks like an entity but is really implementation (cache tables, temp storage, job queues), **do NOT include it as an entity**. Instead, capture the product requirement it serves as a **story** (see Step 6).

### Step 5: Extract features

Features are **atomic product capabilities** — what the product can do.

Look for:
- API endpoints (each major endpoint = a feature)
- UI actions (buttons, forms, workflows)
- Background job capabilities
- System integrations

```json
"features": ["createGroup", "listConversations", "readReceipts", "sendMessage"]
```

Rules:
- camelCase
- Verb + Noun format: `sendMessage`, `createGroup`, `viewDashboard`
- Sorted alphabetically
- Flat list — no sub-features, no nesting
- Each feature MUST be independently identifiable in the codebase

**Granularity guide:**

| Too broad | Good | Too narrow |
|-----------|------|-----------|
| `manageMessages` | `sendMessage`, `deleteMessage` | `validateMessageLength` |
| `handleAuth` | `login`, `register`, `resetPassword` | `hashPassword` |
| `stockAnalysis` | `analyzeStock`, `viewStockChart` | `calculateMovingAverage` |

Features should map to things a **product manager** would list, not things a **developer** would list.

### Step 6: Write stories

Stories describe **how actors, entities, and features interact**. They are the narrative of the product.

Three types:

**Functional flows** — what happens when someone uses the product:
```
"Member sends Message to Conversation"
"Admin deletes any Message from Conversation"
"When Member reads Message, system creates ReadReceipt"
```

**Experience expectations** — what the user perceives (non-functional requirements):
```
"User views StockQuote with data refreshing in real-time"
"User views Dashboard with market data loading instantly"
```

**System behaviors** — what the system does autonomously:
```
"System fetches NewsArticle from multiple sources on schedule"
"System rate limits API requests per IP address"
"System calibrates model with Bayesian posteriors and detects concept drift"
```

Rules:
- One sentence per story
- Start with actor name (PascalCase, MUST be in `actors`) or `"System"`
- Reference entities by exact PascalCase name
- Present tense, active voice
- Describe WHAT happens, not HOW it's implemented
- Ordered by **narrative flow** (NOT alphabetically — this is the one exception)

**The entity-to-story rule:**

If you find implementation details that serve a product purpose, convert them to stories:

| Implementation found | Story to write |
|---------------------|---------------|
| `CachedQuote` table | `"User views StockQuote with data refreshing in real-time"` |
| `CachedHistory` table | `"User views StockChart with historical data loading instantly"` |
| Rate limiter middleware | `"System rate limits API requests per IP address"` |
| Embedding vector store | `"System searches knowledge base by semantic similarity"` |
| Background cron job | `"System fetches NewsArticle from multiple sources on schedule"` |

### Step 7: Define permissions

Permissions connect **actors to features** — who can do what.

**Choose the right mode:**

If the product has no auth or just basic auth:
- Omit the field entirely (AI decides freely)

If the product uses a known auth model:
```json
"permissions": "rbac"
```

If you need to specify exact role-feature mapping:
```json
"permissions": {
  "Admin": ["createGroup", "deleteMessage", "sendMessage"],
  "Guest": ["listConversations"],
  "Member": ["createGroup", "sendMessage"]
}
```

Rules:
- Keys are PascalCase, MUST match `actors`
- Values are camelCase arrays, SHOULD be subsets of `features`
- Both keys and value arrays sorted alphabetically
- System-only features (like `fetchNews`, `enrichNewsWithAi`) do NOT go in permissions — they are not user-invocable

### Step 8: Determine denied items

**This is the most important step.** Denied items are the guardrail that prevents AI from exceeding product scope.

In traditional development, products are bounded by human time. With AI, there is no natural limit — AI will add features, entities, and behaviors unless explicitly told not to. `denied` is how you draw that line.

This is NOT "everything the product doesn't have yet." It's for **intentional product decisions** where the absence matters.

**Decision framework:**

Ask: "If an AI accidentally added this, would it be a product violation?"
- Yes → include in denied
- No, it's just not built yet → do NOT include

**Loose mode** — flat list:
```json
"denied": ["executeTrade", "Reaction", "voiceCall"]
```

**With reasons** (recommended) — each item with explanation:
```json
"denied": {
  "executeTrade": "Market intelligence only — no brokerage liability",
  "Reaction": "Intentionally excluded to keep messaging simple",
  "voiceCall": "Text-based communication only"
}
```

Reasons help the Reviewer understand WHY something is excluded, and help the Human make better approve/deny decisions.

**Three categories of denied items:**

| Category | Example | Reason |
|----------|---------|--------|
| Product scope | `"voiceCall"` | "Text-based communication only" |
| Liability boundary | `"executeTrade"` | "No brokerage liability" |
| Scope guardrail | `"Reaction"` | "Keep messaging simple" |

### Step 9: Assemble and validate

Assemble the complete lock in this key order:

```
$schema → name → version → description → author → license → keywords → private
→ actors → entities → features → stories → permissions → denied
```

Run through the validation checklist (see [Section 5](#5-validation-checklist)).

### Step 10: Generate Markdown view

After the JSON lock, generate a `product.lock.md` for human review:

```markdown
# {name} v{version}

{description}

**Author:** {author}

---

## Actors

- Actor1
- Actor2

---

## Entities

Entity1, Entity2, Entity3

---

## Features

- feature1
- feature2

---

## Stories

- Story 1
- Story 2

---

## Permissions

### Actor1
feature1, feature2, feature3

### Actor2
feature1

---

## Denied

- item1 — reason1
- item2 — reason2
```

---

## 3. Field Reference

### Metadata Fields

| Field | Required | Format | Description |
|-------|----------|--------|-------------|
| `$schema` | No | string | Schema URL for validation |
| `name` | **Yes** | kebab-case string | Product identifier |
| `version` | **Yes** | semver string | Product version |
| `description` | **Yes** | string | One-line product description |
| `author` | **Yes** | string | Who approves this lock |
| `license` | No | string | License identifier |
| `keywords` | No | lowercase string[] | Tags for discoverability |
| `private` | No | boolean | Not for public sharing |

### Product Boundary Fields

All optional. Omitted = AI decides freely.

| Field | Type | Description |
|-------|------|-------------|
| `actors` | `string[]` | User roles |
| `entities` | `string[]` or `Record<string, string[]>` | Data model |
| `features` | `string[]` | Product capabilities |
| `stories` | `string[]` | Interaction flows and behaviors |
| `permissions` | `string` or `Record<string, string[]>` | Access control |
| `denied` | `string[]` or `Record<string, string>` | Explicit exclusions |

---

## 4. Naming Conventions

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

---

## 5. Validation Checklist

Before submitting the lock, verify all of the following:

| # | Check | Pass? |
|---|-------|-------|
| 1 | `name`, `version`, `description`, `author` present | |
| 2 | At least one product boundary field present | |
| 3 | All entity names PascalCase | |
| 4 | All feature names camelCase (verb + noun) | |
| 5 | All actor names PascalCase | |
| 6 | All arrays sorted alphabetically (except stories) | |
| 7 | All object keys sorted alphabetically | |
| 8 | Stories start with actor name or "System" | |
| 9 | Stories reference entities by exact PascalCase name | |
| 10 | `permissions` keys match `actors` | |
| 11 | `denied` items don't appear in `entities` or `features` | |
| 12 | No implementation entities (cache, temp, queue → stories) | |
| 13 | No implementation details in stories (no Redis, WebSocket, framework names) | |
| 14 | Entity fields are names only, no types | |

---

## 6. Common Mistakes

### 1. Including implementation entities
```
BAD:  "entities": ["CachedQuote", "Message", "User"]
GOOD: "entities": ["Message", "User"]
      "stories": ["User views StockQuote with data refreshing in real-time"]
```

### 2. Implementation details in stories
```
BAD:  "System uses Redis to cache stock quotes"
GOOD: "User views StockQuote with data refreshing in real-time"
```

### 3. Spec language instead of stories
```
BAD:  "Users should be able to send messages"
GOOD: "Member sends Message to Conversation"
```

### 4. Over-denying
```
BAD:  "denied": ["exportData", "multiLanguage", "mobileApp", "paymentBilling"]
      (These are just not built yet, not intentional exclusions)

GOOD: "denied": {
        "executeTrade": "Market intelligence only — no brokerage liability",
        "voiceCall": "Text-based communication only"
      }
      (Deliberate product decisions with reasons)
```

### 5. Under-denying
```
BAD:  "denied": []   or omitting denied entirely
      (AI has no guardrail, can add anything)

GOOD: Think hard about what the product must NOT become.
      denied is the most important field in AI-era development.
```

### 6. Too granular features
```
BAD:  "features": ["validateEmail", "hashPassword", "generateToken", "refreshToken"]
GOOD: "features": ["login", "register", "resetPassword"]
```

### 7. Forgetting system stories
If there are background jobs, crons, or automated processes, they need stories too:
```
"System fetches NewsArticle from multiple sources on schedule"
"System detects SupplyChainDisruption from NewsArticle"
```

### 8. Unsorted arrays
```
BAD:  "actors": ["Member", "Admin", "Guest"]
GOOD: "actors": ["Admin", "Guest", "Member"]
```

---

## 7. Full Example

Given a chat application codebase with:
- Prisma schema: User, Conversation, Message, ReadReceipt models
- API routes: /messages, /conversations, /groups
- Auth: three roles (admin, member, guest) via RBAC
- No voice/video features
- No message editing

Generated lock:

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

---

*This guide is part of the [Product Lock Specification](https://spec.productlock.org).*
