---
layout: default
title: Plan Entry Spec
nav_order: 7
permalink: /en/plan-entry-spec
lang: en
lang_label: English
page_id: plan-entry-spec
---

# Product Plan Entry Specification

### Version 0.1.0 — February 2026

> Plan entries record implementation plans before and during development.
>
> All plans live in the `plans/` directory, replacing the single `product.plan.md` file.
>
> Plan = HOW & WHY (what AI intends to do). Worklog = WHAT & WHEN (what AI actually did).

For the full lock specification, see the [Product Lock Specification](https://spec.productlock.org).

---

## Table of Contents

1. [Relationship to Other Files](#1-relationship-to-other-files)
2. [Design Principles](#2-design-principles)
3. [Directory & Naming](#3-directory--naming)
4. [Format](#4-format)
5. [Required Sections](#5-required-sections)
6. [Optional Sections — General](#6-optional-sections--general)
7. [Optional Sections — Full Blueprint](#7-optional-sections--full-blueprint)
8. [Plan vs Worklog](#8-plan-vs-worklog)

---

## 1. Relationship to Other Files

```
plans/*.md        →  Plans (HOW & WHY — what AI intends to do)
worklogs/*.md     →  Records (WHAT & WHEN — what AI actually did)
product.lock.json →  Boundary (WHAT — what the product is)
```

Plans and worklogs are independent but complementary:
- **Plan = before** — records intent, approach, steps, risks
- **Worklog = after** — records actual changes, decisions, discoveries
- Loosely correlated by date/title, no forced 1:1 mapping

The `plans/` directory replaces the single `product.plan.md` file. All plans — from full product blueprints to lightweight session plans — live in the same directory, distinguished by content and naming.

---

## 2. Design Principles

1. **Actionable** — A plan is a concrete guide that AI can follow to implement
2. **Scoped** — Each plan has a clear goal and boundary
3. **Plain text** — Markdown format, human-readable
4. **Flexible granularity** — From full product blueprints to single-session task plans
5. **Non-blocking** — Plans do not require approval to exist; they document intent

---

## 3. Directory & Naming

Plans MUST be placed in the `plans/` directory at the project root.

**Naming convention:** `{YYYY-MM-DD}-{kebab-title}.md`

Examples:
- `plans/2026-02-17-add-payment-entity.md`
- `plans/2026-02-18-full-product-blueprint.md`
- `plans/2026-02-19-refactor-auth-flow.md`

```
project-root/
├── product.lock.json
├── plans/
│   ├── 2026-02-17-add-payment-entity.md
│   ├── 2026-02-18-full-product-blueprint.md
│   └── 2026-02-19-refactor-auth-flow.md
└── worklogs/
    └── ...
```

---

## 4. Format

```markdown
# Plan: {title}

### Context
- Date: {YYYY-MM-DD}
- Branch: `{branch}`
- Agent: {who created this plan}
- Lock version: {version}
- PLS: {score}
- Scope: {scope description}

### Goal
{What this plan aims to achieve — 1-3 sentences.}

### Approach
{High-level strategy — how the goal will be accomplished.}

### Steps
1. {First step}
2. {Second step}
3. ...
```

---

## 5. Required Sections

Every plan entry MUST include the following sections.

### 5.1 Context

Metadata about when and where this plan was created.

| Field | Required | Description |
|-------|----------|-------------|
| Date | Yes | `YYYY-MM-DD` |
| Branch | Yes | Git branch name |
| Agent | No | Who created the plan (default: "AI") |
| Lock version | No | Current lock version |
| PLS | No | Current PLS score |
| Scope | No | Brief scope description |

### 5.2 Goal

One to three sentences describing what this plan aims to achieve. This is the success criteria — when the goal is met, the plan is done.

Rules:
- MUST be specific and measurable
- SHOULD reference lock concepts (entities, features) when applicable
- One to three sentences maximum

### 5.3 Approach

High-level strategy explaining the HOW and WHY. Mentions alternatives considered if relevant.

### 5.4 Steps

Ordered, numbered list of implementation steps.

Rules:
- Numbered, ordered
- Each step SHOULD be independently verifiable when possible
- Aim for 3–15 steps
- Use imperative form: "Add X", "Update Y", "Configure Z"

---

## 6. Optional Sections — General

Include these sections when relevant to the plan.

### 6.1 Background

Additional context: prior art, related issues, user requests.

### 6.2 Constraints

Known limitations or requirements that shape the approach.

```markdown
### Constraints
- Must not break existing API
- Response time under 200ms
- Must work with Node 18+
```

### 6.3 Risks

Potential problems and their mitigations.

```markdown
### Risks
- Stripe webhook delivery may be delayed — implement idempotency keys
- Database migration on large table — run during off-peak hours
```

### 6.4 Expected Outcome

What the codebase should look like after the plan is executed. Useful for the reviewer to verify.

### 6.5 References

Links to issues, PRs, specs, or external resources.

---

## 7. Optional Sections — Full Blueprint

For comprehensive product plans (replacing the old `product.plan.md`), these additional sections may be included:

| Section | When to include |
|---------|----------------|
| Stack | Technology choices at every layer |
| Schemas | Full entity definitions with types, constraints, relations |
| Endpoints | API contracts for each feature |
| Architecture | Non-trivial system patterns (microservices, CQRS, etc.) |
| Auth & Permissions | Auth method, session management, role mapping, RLS |
| File Structure | Project layout when non-standard |

These sections follow the same format as described in the [Plan Spec](/en/plan-spec).

---

## 8. Plan vs Worklog

| Aspect | Plan | Worklog |
|--------|------|---------|
| Timing | Before/during implementation | After implementation |
| Purpose | Intent and approach (HOW & WHY) | Actual changes (WHAT & WHEN) |
| Content | Goal, approach, steps, risks | Changes, files, decisions |
| Granularity | Flexible (blueprint to task) | Per session |
| Verification | Reviewer checks if code matches plan | Reviewer checks if worklog matches changes |
| Directory | `plans/` | `worklogs/` |
| Naming | `{YYYY-MM-DD}-{kebab-title}.md` | `{YYYY-MM-DD}-{kebab-title}.md` |

### Reviewer integration

When reviewing a product, the AI Reviewer checks plans against the codebase and reports:

| Status | Meaning |
|--------|---------|
| MATCH | Implementation aligns with the plan |
| DEVIATION | Implementation differs significantly from the plan |
| INCOMPLETE | Plan is partially implemented |
| UNPLANNED | Significant code exists with no corresponding plan |

---

*This specification is part of the [Product Lock Specification](https://spec.productlock.org).*
