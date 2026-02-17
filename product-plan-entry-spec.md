# Product Plan Entry Specification

### Version 0.1.0 — February 2026

> Plan entries record implementation plans before and during development.
>
> All plans live in the `plans/` directory, replacing the single `product.plan.md` file.
>
> Plan = HOW & WHY (what AI intends to do). Worklog = WHAT & WHEN (what AI actually did).

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

- **Date** (required): `YYYY-MM-DD`
- **Branch** (required): Git branch name
- **Agent** (optional): Who created the plan (default: "AI")
- **Lock version** (optional): Current lock version
- **PLS** (optional): Current PLS score
- **Scope** (optional): Brief scope description

### 5.2 Goal

One to three sentences describing what this plan aims to achieve. This is the success criteria — when the goal is met, the plan is done.

### 5.3 Approach

High-level strategy. Explains the "why" behind the chosen method. Should mention alternatives considered if relevant.

### 5.4 Steps

Ordered, numbered list of implementation steps. Each step should be independently verifiable when possible.

---

## 6. Optional Sections — General

Include these sections when relevant to the plan.

### 6.1 Background

Additional context that helps understand why this plan exists. Prior art, related issues, user requests.

### 6.2 Constraints

Known limitations or requirements that shape the approach.

### 6.3 Risks

Potential problems and their mitigations.

### 6.4 Expected Outcome

What the codebase should look like after the plan is executed. Useful for the reviewer to verify.

### 6.5 References

Links to issues, PRs, specs, or external resources.

---

## 7. Optional Sections — Full Blueprint

For comprehensive product plans (replacing the old `product.plan.md`), these sections may be included:

### 7.1 Stack

Technology choices at every layer, presented as a table.

### 7.2 Schemas

Full entity definitions with types, constraints, relations, and indexes.

### 7.3 Endpoints

API contracts for each feature.

### 7.4 Architecture

System architecture when non-trivial patterns are used.

### 7.5 Auth & Permissions

Auth method, session management, role mapping, RLS policies.

### 7.6 File Structure

Project layout when non-standard or needs explanation.

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

---

*This specification is part of the [Product Lock Specification](https://productlock.org).*
