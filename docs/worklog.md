---
layout: default
title: Worklog
nav_order: 8
permalink: /worklog
---

# Product Lock Worklog Specification

### Version 0.1.0 — February 2026

> A structured development log that tracks changes relative to product.lock.json.
>
> Lock defines WHAT. Plan defines HOW. Score measures complexity. **Worklog records WHY and WHEN.**
>
> Together, these four files reconstruct full project context — across sessions, across agents, across time.

For the full specification, see the [Product Lock Specification](https://productlock.org).

---

## Table of Contents

1. [Purpose](#1-purpose)
2. [The Closed Loop](#2-the-closed-loop)
3. [File Format](#3-file-format)
4. [Required Sections](#4-required-sections)
5. [Optional Sections](#5-optional-sections)
6. [Entry Lifecycle](#6-entry-lifecycle)
7. [Context Reconstruction](#7-context-reconstruction)
8. [Conventions](#8-conventions)

---

## 1. Purpose

AI sessions are ephemeral. Context windows compress, conversations expire, agents lose memory. But product decisions persist. The worklog captures what matters:

- **What changed** — files modified, entities added, features implemented
- **Why it changed** — the decision, the trigger, the constraint
- **What it means** — lock modifications, complexity delta, scope impact
- **What was caught** — reviewer findings, violations fixed, boundary enforced

Without a worklog, each AI session starts from zero. With a worklog, any agent can read lock + worklog and reconstruct the project's trajectory.

### What worklog is NOT

- Not a git log (git tracks file changes, worklog tracks product decisions)
- Not a changelog (changelog is for users, worklog is for developers and AI)
- Not a journal (no opinions, no commentary — structured data only)

---

## 2. The Closed Loop

The four product lock files form a closed feedback loop:

```
product.lock.json ──→ product.plan.md ──→ Implementation ──→ product.worklog.md
       ↑                                                            │
       └────────────────────────────────────────────────────────────┘
                         (lock updated based on worklog findings)
```

| File | Role | Answers |
|------|------|---------|
| `product.lock.json` | Boundary | What is this product? |
| `product.plan.md` | Blueprint | How do we build it? |
| `product-lock-scoring.md` | Measurement | How complex is it? |
| `product.worklog.md` | History | What happened, when, and why? |

Each development session:

1. **Before**: Read lock + latest worklog entry → understand current state
2. **During**: Track changes, decisions, lock modifications
3. **After**: Write worklog entry → run reviewer → record findings
4. **Close**: Update lock if needed → recalculate score → note delta

The loop is closed when every change is traceable: from the lock that authorized it, through the session that implemented it, to the reviewer that verified it.

---

## 3. File Format

Worklogs are stored in a `worklogs/` directory in the project root, alongside `product.lock.json`. Each session is one file.

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

### File naming

```
{date}-{kebab-case-title}.md
```

| Part | Format | Example |
|------|--------|---------|
| Date | `YYYY-MM-DD` | `2026-02-17` |
| Date (same day) | `YYYY-MM-DDTHHmm` | `2026-02-17T1430` |
| Title | kebab-case, under 60 chars | `add-payment-system` |

Examples:
```
2026-02-16-initial-release.md
2026-02-17-add-payment-system.md
2026-02-17T1430-fix-payment-webhook.md
2026-02-18-refactor-auth-middleware.md
```

### File listing order

Files sort naturally by name (date prefix ensures chronological order). To find the latest entry, sort descending or look at the last file.

### Each file is one session

One file = one session = one focused block of work. Each file follows the template below:

```markdown
# [{date}] {title}

### Context
- Branch: `feature/xxx`
- Agent: {who did the work}
- Lock version: {version before this session}
- PLS before: {score}

### Goal
{One sentence: what this session set out to do.}

### Changes

| Action | Target | Detail |
|--------|--------|--------|
| added | Entity: Payment | 4 fields: id, amount, userId, createdAt |
| modified | Feature: checkout | Added Stripe integration |
| removed | Feature: manualInvoice | Replaced by automated billing |

### Lock Delta
{What changed in product.lock.json, if anything.}

### Score Delta
- PLS: {before} → {after} ({+/-delta})
- Breakdown: D {before}→{after}, F {before}→{after}, I {before}→{after}, A {before}→{after}

### Review Findings
{AI Reviewer results after this session.}

### Decisions
{Key decisions made and why.}

### Files
| Action | Path |
|--------|------|
| created | src/models/payment.ts |
| modified | src/routes/checkout.ts |
| deleted | src/routes/manual-invoice.ts |

### Next
{What the next session should pick up.}
```

---

## 4. Required Sections

Every worklog entry MUST include the following sections.

### 4.1 Context

Session metadata. Establishes the starting state.

```markdown
### Context
- Branch: `feature/payment-system`
- Agent: Claude (Opus 4)
- Lock version: 1.2.0
- PLS before: 145
```

| Field | Required | Description |
|-------|----------|-------------|
| Branch | Yes | Git branch for this session |
| Agent | Yes | Who did the work (AI model, human name, or both) |
| Lock version | Yes | Version of product.lock.json at session start |
| PLS before | Yes | Product Lock Score at session start |

### 4.2 Goal

One sentence describing the session objective. This is the contract — everything in the entry should relate to this goal.

```markdown
### Goal
Add payment processing with Stripe so Members can pay for premium features.
```

Rules:
- MUST be one sentence
- MUST be specific enough that completion is verifiable
- SHOULD reference lock concepts (entities, features, actors) when applicable

### 4.3 Changes

A table of product-level changes made during the session. This is NOT a file diff — it's a product boundary diff.

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

Actions:
- `added` — new item in the product boundary
- `modified` — existing item changed
- `removed` — item removed from the product boundary
- `unchanged` — explicitly noting something was NOT changed (when relevant)

Target format: `{type}: {name}` where type is Entity, Feature, Actor, Permission, Denied, Story.

### 4.4 Review Findings

Results from the AI Reviewer after the session's changes. If no review was run, state that explicitly.

```markdown
### Review Findings

Reviewer: Claude (Sonnet 4.5)
Status: PASS (2 warnings)

- [PASS] Entity Payment — model exists with correct fields
- [PASS] Feature createPayment — POST /api/payments endpoint
- [WARN] Undeclared entity: PaymentLog (exists in code, not in lock)
- [WARN] Undeclared feature: retryPayment (exists in code, not in lock)
```

Or:

```markdown
### Review Findings

No review conducted this session.
```

Rules:
- MUST name the reviewer (AI model or human)
- MUST include overall status: PASS, FAIL, or NOT RUN
- MUST list all violations and warnings
- If FAIL: the entry MUST include a follow-up in the Next section

### 4.5 Files

Files created, modified, or deleted during the session.

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

Rules:
- List every file touched, not just the important ones
- Use relative paths from project root
- Actions: `created`, `modified`, `deleted`
- Sort by action (created → modified → deleted), then alphabetically within each group

---

## 5. Optional Sections

Include these sections when they provide useful context.

### 5.1 Lock Delta

What changed in product.lock.json during this session. Skip if the lock was not modified.

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

Rules:
- MUST state version change
- Group by Added / Modified / Removed
- Use lock field format (entity names PascalCase, features camelCase, etc.)

### 5.2 Score Delta

How the Product Lock Score changed. Skip if the lock was not modified.

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

If the score crosses a level boundary (e.g., Moderate → Complex), this SHOULD be called out explicitly. Level transitions are signals for scope review.

### 5.3 Decisions

Key decisions made during the session and their rationale. Skip if no significant decisions were made.

```markdown
### Decisions

1. **Stripe over Paddle**: Stripe has better API for subscription management. Paddle bundles tax handling but we don't need it for MVP.

2. **No refunds in v1**: Added to denied list. Manual refund via Stripe dashboard is sufficient. Automated refunds add liability and edge cases (partial refunds, proration).

3. **Subscription as separate entity**: Could have been a field on User, but Subscription has its own lifecycle (start, end, cancel, renew) that justifies a separate entity.
```

Rules:
- Number each decision
- Bold the decision title
- Include rationale (the WHY)
- If a decision modifies the denied list, note it

### 5.4 Next

What the next session should pick up. Skip only if the project is complete.

```markdown
### Next

- [ ] Implement webhook handler for Stripe events (payment_intent.succeeded, subscription.canceled)
- [ ] Add PaymentLog to lock or remove from codebase (reviewer warning)
- [ ] Write stories for payment flow and add to lock
- [ ] Run full review after webhook implementation
```

Rules:
- Use checkbox format for actionable items
- Reference reviewer findings when applicable
- Be specific enough that a different agent can pick this up

### 5.5 Commits

Git commits made during the session. Skip if no commits were made (e.g., planning-only session).

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

## 6. Entry Lifecycle

A worklog entry follows this lifecycle within a development session:

```
Session Start
│
├─ 1. Read latest worklog entry → understand current state
├─ 2. Read lock → understand product boundary
├─ 3. Read plan → understand implementation decisions
│
├─ 4. Do the work
│   ├─ Track changes (entities, features, actors)
│   ├─ Track files modified
│   └─ Record decisions as they happen
│
├─ 5. Update lock if product boundary changed
├─ 6. Recalculate score if lock changed
├─ 7. Run reviewer against updated lock
│
├─ 8. Write worklog entry
│   ├─ Context (branch, agent, lock version, PLS)
│   ├─ Goal (what was the objective)
│   ├─ Changes (product-level changes)
│   ├─ Lock Delta (if lock was modified)
│   ├─ Score Delta (if score changed)
│   ├─ Review Findings (reviewer results)
│   ├─ Decisions (key decisions + rationale)
│   ├─ Files (all files touched)
│   ├─ Commits (git commits)
│   └─ Next (what to pick up next)
│
Session End
```

### When to create an entry

- One entry per development session (a session = one focused block of work)
- A session that only plans but writes no code still gets an entry (with no Files section)
- A session that only reviews but changes nothing still gets an entry (with Review Findings)
- Multiple small fixes in one sitting = one entry
- Large feature spanning multiple sittings = multiple entries

### Entry size

Keep entries concise. A typical entry is 30–80 lines. If an entry exceeds 150 lines, the session was too large — split into multiple entries next time.

---

## 7. Context Reconstruction

The primary value of the worklog: any agent can reconstruct project context by reading structured files instead of replaying conversation history.

### Without worklog

```
New AI session → reads codebase (thousands of files) → guesses intent → often wrong
```

### With worklog

```
New AI session → reads lock (product boundary)
              → reads latest worklog (current state, recent decisions, next steps)
              → reads plan (implementation approach)
              → starts working with full context
```

### What each file contributes to reconstruction

| Question | Answered by |
|----------|-------------|
| What is this product? | `product.lock.json` |
| What are the exact boundaries? | `product.lock.json` (denied) |
| How complex is it? | PLS score (in worklog) |
| How is it built? | `product.plan.md` |
| What happened recently? | `product.worklog.md` (latest entries) |
| Why was X decided? | `product.worklog.md` (Decisions) |
| What should I do next? | `product.worklog.md` (Next) |
| What did the reviewer find? | `product.worklog.md` (Review Findings) |
| Has scope been creeping? | `product.worklog.md` (Score Delta over time) |

### Token efficiency

A 2-hour AI conversation might consume 100K+ tokens of context. The worklog entry for that same session is ~50 lines of structured markdown. The next session reads 50 lines instead of replaying 100K tokens.

```
Conversation history: 100,000+ tokens (compressed, lossy, ephemeral)
Worklog entry:             ~800 tokens (structured, complete, permanent)
```

### Scope drift detection

By tracking PLS across entries, scope creep becomes visible:

```
[2026-02-10] v1.0.0 — PLS: 85  (Moderate)
[2026-02-12] v1.1.0 — PLS: 92  (+7, minor additions)
[2026-02-14] v1.2.0 — PLS: 145 (+53, significant jump ⚠️)
[2026-02-17] v1.3.0 — PLS: 172 (+27, crossed into Complex)
```

A jump of +50 or more in a single session, or a level boundary crossing, SHOULD trigger a scope review.

---

## 8. Conventions

### 8.1 Directory structure

- Directory: `worklogs/` in project root
- One directory per product (matches one lock per product)
- Each file is one session

```
project-root/
├── product.lock.json
└── worklogs/
    ├── 2026-02-16-initial-release.md
    └── 2026-02-17-add-payment-system.md
```

### 8.2 File naming

```
{YYYY-MM-DD}-{kebab-case-title}.md
```

- Date prefix ensures chronological sort
- Title is imperative, kebab-case, under 60 characters
- If multiple sessions on same day, use `{YYYY-MM-DD}T{HHmm}-{title}.md`

### 8.3 Titles

Imperative and concise:

| Good | Bad |
|------|-----|
| `add-payment-system` | `added-the-payment-system-and-also-fixed-some-bugs` |
| `fix-auth-middleware` | `authentication-middleware-was-broken` |
| `remove-legacy-billing` | `cleaned-up-old-billing-code` |

### 8.4 Cross-referencing

When referencing other entries, use the filename:

```markdown
Continues from `2026-02-14-setup-stripe-integration.md`.
See `2026-02-12-define-payment-entities.md` for schema decisions.
```

### 8.5 Naming conventions

Follow lock conventions within the worklog:

| Item | Format | Example |
|------|--------|---------|
| Entity names | PascalCase | `Entity: Payment` |
| Feature names | camelCase | `Feature: createPayment` |
| Actor names | PascalCase | `Actor: Subscriber` |
| File paths | relative from root | `src/models/payment.ts` |

### 8.6 Reading order

To reconstruct context, an AI agent SHOULD:

1. Read the **latest** worklog file (last by filename sort)
2. Check the **Next** section for pending work
3. Read additional recent files if more context is needed
4. Read the lock for product boundary

---

*This specification is part of the [Product Lock Specification](https://productlock.org).*
