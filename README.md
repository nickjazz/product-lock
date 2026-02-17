<p align="center">
  <a href="https://productlock.org">
    <img src="logo.png" alt="Product Lock" width="60">
  </a>
</p>

<h1 align="center">product.lock.json</h1>

<p align="center">
  A product boundary specification for humans and AI.
</p>

<p align="center">
  <a href="https://productlock.org"><img src="https://img.shields.io/badge/Website-productlock.org-blue?style=flat-square" alt="Website"></a>
  <a href="https://github.com/anthropics/product-lock/blob/main/LICENSE"><img src="https://img.shields.io/badge/License-MIT-green?style=flat-square" alt="License"></a>
  <a href="product-lock-spec.md"><img src="https://img.shields.io/badge/Spec-v0.1.0-orange?style=flat-square" alt="Spec Version"></a>
</p>

<p align="center">
  <strong>English</strong> ·
  <a href="README-zh-TW.md">繁體中文</a> ·
  <a href="README-ja.md">日本語</a> ·
  <a href="README-de.md">Deutsch</a>
</p>

---

**AI can write code now. But who stops it from adding stuff you never asked for?**

product.lock.json defines a software product's boundary — what it **is** and what it **is not**. Doesn't matter what language, framework, or architecture. The boundary is the same.

```
AI Worker writes code  →  generates lock  →  Human approves  →  AI Reviewer verifies
```

---

## Why?

AI can write most of your code now. The problem: **it doesn't know when to stop.**

You ask for a chat app. It helpfully adds emoji reactions, voice calls, file sharing. You didn't ask for any of that. But AI doesn't know, because you never said no.

Traditional development has a natural boundary — human time and effort. AI doesn't. So the boundary has to be written down explicitly.

**product.lock.json is that boundary.**

---

## What does it look like?

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

6 fields. 6 questions:

| Field | Question |
|-------|----------|
| `actors` | Who uses it? |
| `entities` | What data does it store? |
| `features` | What can it do? |
| `stories` | How do things interact? |
| `permissions` | Who can do what? |
| `denied` | **What must it NOT have?** |

That last one is the most important. In the AI era, defining what you **won't** build matters more than what you will.

---

## Key ideas

### WHAT, not HOW

No routes, no frameworks, no dependencies, no file structure in the lock.

Same lock, different stacks. Give it to AI with "build this in Python + FastAPI" or "build this in TypeScript + Next.js". Product boundary stays identical.

### Specify more, enforce more

```json
"entities": ["User"]                              // just checks User exists
"entities": { "User": [] }                        // same — just checks it exists
"entities": { "User": ["id", "name", "email"] }   // checks exact fields, no more no less
```

Omit a field = AI decides freely. You only lock what you care about.

### Three roles

| Role | Job |
|------|-----|
| **Human** | Scans the lock, approves or denies, manages the denied list |
| **AI Worker** | Writes code, generates the lock |
| **AI Reviewer** | Independently checks code against the lock. Does NOT trust the Worker |

The human never reads code. The human reads the lock (rendered as Markdown). Takes minutes, not hours.

### `denied` is the guardrail

```json
"denied": {
  "executeTrade": "Market intelligence only — no brokerage liability",
  "voiceCall": "Text-based communication only"
}
```

If AI accidentally adds something in `denied`, the Reviewer flags it as a violation. This is the most important line of defense.

The test: "If AI added this on its own, would it be a product violation?" Yes → put it in denied. No → that's just backlog, leave it out.

---

## Documentation

| Document | Audience | Purpose |
|----------|----------|---------|
| [Specification](product-lock-spec.md) | Everyone | Full spec (English) |
| [Specification (Chinese)](product-lock-spec-zh.md) | Everyone | Full spec (Chinese) |
| [Generator Guide](product-lock-generator-guide.md) | AI Worker | How to generate a lock from a codebase |
| [Reviewer Guide](product-lock-reviewer-guide.md) | AI Reviewer | How to verify code against a lock |
| [Plan Spec](product-plan-spec.md) | AI Worker | product.plan.md format (the HOW) |
| [Scoring](product-lock-scoring.md) | Everyone | Quantify product complexity from a lock |
| [Worklog](product-lock-worklog.md) | AI + Human | Track changes, decisions, and scope across sessions |

---

## Quick start

### Simplest possible lock

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

4 metadata fields + at least 1 product field. That's it.

### Have AI generate one

Give the [Generator Guide](product-lock-generator-guide.md) to an AI and point it at your codebase:

> "Read this guide, then analyze this codebase and generate a product.lock.json."

### Have AI review against one

Give the [Reviewer Guide](product-lock-reviewer-guide.md) to a *different* AI, along with the lock and codebase:

> "Read this guide, then review this codebase against this product.lock.json."

Two AIs. They don't trust each other. One generates, one verifies. Human makes the final call.

---

## Naming conventions

| Field | Format | Example |
|-------|--------|---------|
| `name` | kebab-case | `"chat-system"` |
| `actors` | PascalCase | `"Admin"`, `"Member"` |
| `entities` | PascalCase | `"User"`, `"ReadReceipt"` |
| entity fields | camelCase | `"senderId"`, `"createdAt"` |
| `features` | camelCase | `"sendMessage"`, `"createGroup"` |
| `denied` | PascalCase or camelCase | `"Reaction"`, `"editMessage"` |

All arrays sorted alphabetically. One exception: `stories` are ordered by narrative flow.

---

## FAQ

**Why JSON and not YAML?**
JSON is deterministic — same data, one way to write it. When AI generates and AI validates, determinism beats convenience. Humans read the rendered Markdown anyway.

**One lock per microservice?**
One lock per product. Microservices are HOW, not WHAT.

**Can I skip `denied`?**
You can. But that's telling AI "add whatever you want." Are you sure?

**Can different languages implement the same lock?**
Yes. That's the whole point.

---

## License

MIT

---

<p align="center">
  <a href="https://productlock.org"><strong>productlock.org</strong></a>
</p>
