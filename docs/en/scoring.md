---
layout: default
title: Scoring
nav_order: 8
permalink: /en/scoring
lang: en
lang_label: English
page_id: scoring
---

# Product Lock Scoring

### Version 0.1.0 — February 2026

> A complexity scoring system derived from product.lock.json.
>
> Same lock, same score. The score measures product complexity, not code complexity.

For the full specification, see the [Product Lock Specification](https://spec.productlock.org).

---

## Table of Contents

1. [What This Measures](#1-what-this-measures)
2. [Four Dimensions](#2-four-dimensions)
3. [Formula](#3-formula)
4. [Score Levels](#4-score-levels)
5. [Calculation Rules](#5-calculation-rules)
6. [Benchmark](#6-benchmark)
7. [Use Cases](#7-use-cases)

---

## 1. What This Measures

Product Lock Score (PLS) measures the **product boundary complexity** of a software product. It answers: "How complex is this product from a product perspective?"

What PLS measures:
- How much data the product manages
- How many capabilities it has
- How interconnected its parts are
- How complex its access control is

What PLS does NOT measure:
- Code quality or code size
- Technical difficulty of implementation
- Performance requirements
- Infrastructure complexity

**The score is derived entirely from product.lock.json.** Same lock, same score. Language, framework, and architecture do not affect the score.

---

## 2. Four Dimensions

Product complexity breaks down into four independent dimensions:

### D — Data Complexity

How much the product stores. More entities with more fields = more data complexity.

```
D = entity_count × avg_fields_per_entity × 0.3
```

A product with 45 entities averaging 14 fields (Sentry) has fundamentally more data complexity than one with 7 entities averaging 6 fields (Hacker News).

### F — Functional Complexity

What the product can do. Each feature is an independently identifiable capability.

```
F = feature_count × 1.0
```

Weighted at 1.0 because features are the most direct measure of what a product does. A product with 180 features (GitLab) does more than one with 12 features (Hacker News).

### I — Interaction Complexity

How the parts connect. Stories describe cross-entity flows — the more stories, the more interconnected the product.

```
I = story_count × 0.5
```

Weighted at 0.5 because stories are narrative (one story may cover multiple interactions) and their count is less precise than entities or features.

### A — Access Complexity

Who can do what. More actors with more distinct permission sets = more access control complexity.

```
A = permission_matrix_size × 0.1
```

Permission matrix size = number of meaningful actor-feature combinations. Weighted at 0.1 because access control adds organizational complexity but doesn't change what the product does.

---

## 3. Formula

```
PLS = D + F + I + A

Where:
  D = entity_count × avg_fields_per_entity × 0.3
  F = feature_count × 1.0
  I = story_count × 0.5
  A = permission_matrix_size × 0.1
```

### How to read each field from product.lock.json

| Variable | Source | How to count |
|----------|--------|-------------|
| `entity_count` | `entities` | Number of entity keys (loose or strict mode) |
| `avg_fields_per_entity` | `entities` | Average length of field arrays (strict mode only) |
| `feature_count` | `features` | Length of features array |
| `story_count` | `stories` | Length of stories array |
| `permission_matrix_size` | `permissions` | Sum of all permission arrays across all actors |

### Handling missing fields

| Situation | Rule |
|-----------|------|
| `entities` in loose mode (`string[]`) | Use default: 8 fields per entity |
| `entities` in strict mode with `[]` | Count as 0 fields for that entity |
| `features` omitted | F = 0 |
| `stories` omitted | I = 0 |
| `permissions` omitted | A = 0 |
| `permissions` as string (`"rbac"`) | Use default: actors × features × 0.6 |
| `actors` omitted | Use 1 (single user type) |
| `denied` | Not included in score (boundary maturity, not complexity) |

### Why `denied` is excluded

`denied` measures boundary maturity, not product complexity. A product with 50 denied items is not more complex than one with 0 — it is more constrained. Constraint is the opposite of complexity.

---

## 4. Score Levels

| PLS Range | Level | Description | Examples |
|-----------|-------|-------------|----------|
| < 50 | **Simple** | Single-purpose tool, minimal data model, few features | Hacker News (32), Excalidraw (44) |
| 50–150 | **Moderate** | Focused product with clear domain, moderate data model | Plausible (58), Ghost (145) |
| 150–300 | **Complex** | Multi-feature platform, rich data model, cross-entity interactions | Cal.com (162), Discourse (170), Supabase (250) |
| 300–500 | **Very Complex** | Enterprise platform, extensive data model, deep permission system | Elasticsearch (287), Sentry (325) |
| 500+ | **Massive** | Multi-product platform, hundreds of entities and features | GitLab (1037) |

### What the levels mean in practice

| Level | Human review time | AI generation scope | Typical lock size |
|-------|------------------|--------------------| ------------------|
| Simple | Seconds | Single prompt | < 30 lines |
| Moderate | 1–3 minutes | Single session | 30–80 lines |
| Complex | 3–10 minutes | Multiple sessions | 80–200 lines |
| Very Complex | 10–30 minutes | Phased generation | 200–500 lines |
| Massive | 30+ minutes | Team of AI agents | 500+ lines |

---

## 5. Calculation Rules

### Rule 1: Count product entities only

Only count entities that appear in the `entities` field. Implementation tables (caches, queues, migrations) should not be in the lock and therefore do not affect the score.

### Rule 2: Strict mode gives higher scores

Strict mode locks more fields, producing a higher (and more accurate) D score. This is intentional — a product that specifies exact fields has a more defined (and therefore measurable) data model.

```json
// Loose: entity_count=3, avg_fields=8 (default)
// D = 3 × 8 × 0.3 = 7.2
"entities": ["User", "Post", "Comment"]

// Strict: entity_count=3, avg_fields=10
// D = 3 × 10 × 0.3 = 9.0
"entities": {
  "User": ["avatar", "email", "id", "name", "role"],
  "Post": ["authorId", "content", "createdAt", "id", "slug", "status", "tags", "title"],
  "Comment": ["authorId", "content", "createdAt", "id", "postId", "replyToId", "status"]
}
```

### Rule 3: Permission matrix counts actual entries

When permissions is an object, count the total number of permission entries:

```json
"permissions": {
  "Admin": ["createPost", "deletePost", "manageUsers", "publishPost"],
  "Editor": ["createPost", "publishPost"],
  "Viewer": ["viewPost"]
}
// permission_matrix_size = 4 + 2 + 1 = 7
```

When permissions is a string like `"rbac"`, estimate: `actors × features × 0.6`.

### Rule 4: Score is deterministic

Same lock always produces the same score. No randomness, no external data, no subjective judgment. The formula uses only data present in the lock file.

### Rule 5: Round to integer

Final PLS is rounded to the nearest integer.

---

## 6. Benchmark

Validated against 10 well-known open-source products. All scores match the publicly accepted complexity ranking.

| # | Product | Entities | AvgFields | Features | Stories | PermMatrix | D | F | I | A | **PLS** | Level |
|---|---------|----------|-----------|----------|---------|------------|---|---|---|---|---------|-------|
| 1 | Hacker News | 7 | 6 | 12 | 10 | 26 | 12.6 | 12 | 5 | 2.6 | **32** | Simple |
| 2 | Excalidraw | 8 | 10 | 14 | 10 | 10 | 24 | 14 | 5 | 1 | **44** | Simple |
| 3 | Plausible | 12 | 8 | 18 | 15 | 36 | 28.8 | 18 | 7.5 | 3.6 | **58** | Moderate |
| 4 | Ghost | 22 | 12 | 34 | 45 | 90 | 79.2 | 34 | 22.5 | 9 | **145** | Moderate |
| 5 | Cal.com | 26 | 11 | 38 | 55 | 105 | 85.8 | 38 | 27.5 | 10.5 | **162** | Complex |
| 6 | Discourse | 28 | 10 | 42 | 60 | 135 | 84 | 42 | 30 | 13.5 | **170** | Complex |
| 7 | Supabase | 40 | 11 | 78 | 35 | 220 | 132 | 78 | 17.5 | 22 | **250** | Complex |
| 8 | Elasticsearch | 35 | 12 | 90 | 45 | 480 | 126 | 90 | 22.5 | 48 | **287** | Complex |
| 9 | Sentry | 45 | 14 | 75 | 60 | 310 | 189 | 75 | 30 | 31 | **325** | Very Complex |
| 10 | GitLab | 120 | 18 | 180 | 250 | 840 | 648 | 180 | 125 | 84 | **1037** | Massive |

### Validation

The scoring produces a ranking that matches widely accepted complexity perceptions:

```
Hacker News (32) < Excalidraw (44) < Plausible (58) < Ghost (145) < Cal.com (162)
< Discourse (170) < Supabase (250) < Elasticsearch (287) < Sentry (325) < GitLab (1037)
```

Key observations from the benchmark:

- **Simple products** (< 50): single-purpose, < 10 entities, < 15 features
- **Moderate products** (50–150): focused domain, 10–25 entities, 15–35 features
- **Complex products** (150–300): multi-feature platforms, 25–40 entities, 35–80 features
- **Very Complex products** (300–500): enterprise platforms, 35–50 entities, 75–90 features, deep permission models
- **Massive products** (500+): multi-product platforms, 100+ entities, 150+ features, 200+ stories

---

## 7. Use Cases

### Compare products

> "Is our product more complex than Ghost?" Lock both products. Compare scores.

### Track complexity growth

> Version 1.0: PLS 85 (Moderate)
> Version 2.0: PLS 160 (Complex)
> Version 3.0: PLS 340 (Very Complex)

If the score jumps unexpectedly, it may indicate scope creep. Review the diff between lock versions to find what was added.

### Estimate effort

The score correlates with implementation effort. A Complex product (150–300) requires significantly more work than a Simple one (< 50). Use the benchmark products as reference points:

- "Our product scores 170, similar to Discourse. Expect similar scope."
- "Our product scores 45, similar to Excalidraw. This is a focused tool."

### Scope control

Set a target PLS for a product version:

> "MVP must stay under PLS 100. Anything that pushes us over gets moved to v2."

If a feature addition increases the score beyond the target, it's a signal to reconsider scope.

### Compare lock versions

```
v1.0.0 → v1.1.0: PLS 120 → 128 (+8)     // Minor feature additions
v1.1.0 → v2.0.0: PLS 128 → 195 (+67)     // Major scope expansion
v2.0.0 → v2.1.0: PLS 195 → 210 (+15)     // Moderate additions
v2.1.0 → v3.0.0: PLS 210 → 340 (+130)    // Red flag: scope may be creeping
```

---

*This scoring system is part of the [Product Lock Specification](https://spec.productlock.org).*
