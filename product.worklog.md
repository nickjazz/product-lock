# Product Worklog: product-lock

> Lock: product.lock.json
> Started: 2026-02-16

---

## [2026-02-17T14:00] Add GitHub Pages, i18n, and dogfooding

### Context
- Branch: `main`
- Agent: Claude (Opus 4.6)
- Lock version: 0.1.0 (created this session)
- PLS before: N/A (lock created this session)

### Goal
Add GitHub Pages site, multi-language README, worklog spec, and bootstrap product.lock.json for this repo itself.

### Changes

| Action | Target | Detail |
|--------|--------|--------|
| added | Feature: trackWorklog | Worklog specification defined |
| added | Entity: Worklog | product.worklog.md format |
| added | Story: context reconstruction | AiWorker records session changes in Worklog |
| added | Story: multi-language | Human reads specification in multiple languages |
| added | Denied: cliTool | Spec only, no CLI in this repo |
| added | Denied: lockEditor | No GUI editor |
| added | Denied: schemaValidator | Validation belongs in consumer tooling |

### Lock Delta

Lock created: 0.1.0

Added:
- Entity: Lock [actors, author, denied, description, entities, features, name, permissions, stories, version]
- Entity: Plan []
- Entity: Score []
- Entity: Worklog []
- Feature: calculateScore, generateLock, planImplementation, reviewLock, trackWorklog
- Actor: AiReviewer, AiWorker, Human
- Denied: cliTool, lockEditor, schemaValidator

### Score Delta

- PLS: 0 → 28 (new)
- Level: Simple
- Breakdown:
  - D: 4 × 2.5 × 0.3 = 3 (4 entities, avg 2.5 fields)
  - F: 5 × 1.0 = 5
  - I: 6 × 0.5 = 3
  - A: 10 × 0.1 = 1
  - Total: 12 (Simple)

### Review Findings

No review conducted — lock just created.

### Decisions

1. **Spec-only repo**: Added cliTool, lockEditor, schemaValidator to denied. This repo is the specification, not the tooling. Consumer projects build their own validators.

2. **Lock entity fields**: Listed all 10 top-level fields of product.lock.json as Lock entity fields, making this the most strictly defined entity.

3. **Three actors**: AiWorker, AiReviewer, Human map directly to the three roles defined in the specification itself.

### Files

| Action | Path |
|--------|------|
| created | docs/_config.yml |
| created | docs/CNAME |
| created | docs/generator-guide.md |
| created | docs/index.md |
| created | docs/logo.png |
| created | docs/plan-spec.md |
| created | docs/reviewer-guide.md |
| created | docs/scoring.md |
| created | docs/specification-zh.md |
| created | docs/specification.md |
| created | docs/worklog.md |
| created | product-lock-worklog.md |
| created | product.lock.json |
| created | product.worklog.md |
| created | README-de.md |
| created | README-ja.md |
| created | README-zh-TW.md |
| modified | README.md |

### Commits

| Hash | Message |
|------|---------|
| ea3e0df | Add worklog specification for tracking changes across sessions |
| 7f937a8 | Add i18n README translations (zh-TW, ja, de) with language switcher |
| 09e98d6 | Add GitHub Pages site with Just the Docs theme |
| 16931ed | Create CNAME |

### Next

- [ ] Verify GitHub Pages renders correctly at productlock.org
- [ ] Run self-review: verify codebase against product.lock.json
- [ ] Add JSON Schema file for editor validation ($schema support)
- [ ] Consider adding example product.lock.json files for common project types

---

## [2026-02-16] Initial release

### Context
- Branch: `main`
- Agent: Claude (Opus 4.6)
- Lock version: N/A (pre-lock)
- PLS before: N/A

### Goal
Create the complete Product Lock specification with all documentation.

### Changes

| Action | Target | Detail |
|--------|--------|--------|
| added | Entity: Lock | Core specification defined — 6 product boundary fields |
| added | Entity: Plan | Implementation blueprint spec |
| added | Entity: Score | Complexity scoring system with 4 dimensions |
| added | Feature: generateLock | Generator guide with 10-step process |
| added | Feature: reviewLock | Reviewer guide with field-by-field verification |
| added | Feature: calculateScore | Scoring formula validated against 10 open-source products |
| added | Feature: planImplementation | Plan spec with required/optional sections |

### Review Findings

No review conducted — initial creation.

### Decisions

1. **JSON over YAML**: Determinism matters more than convenience for AI-generated, AI-validated specs.

2. **Progressive strictness**: Specify more = enforce more. Omit = AI decides freely. This is the core design principle.

3. **denied is the most important field**: In AI-era development, defining what NOT to build matters more than what to build.

4. **Scoring formula (PLS = D + F + I + A)**: Validated against 10 open-source products (Hacker News through GitLab). Weighted formula achieved 10/10 correct ranking.

5. **Three roles**: Human (approve), AI Worker (generate), AI Reviewer (verify). Two AIs that don't trust each other, human makes final call.

### Files

| Action | Path |
|--------|------|
| created | logo.png |
| created | product-lock-generator-guide.md |
| created | product-lock-reviewer-guide.md |
| created | product-lock-scoring.md |
| created | product-lock-spec-zh.md |
| created | product-lock-spec.md |
| created | product-plan-spec.md |
| created | README.md |

### Commits

| Hash | Message |
|------|---------|
| 230bb45 | Initial release: Product Lock Specification v0.1.0 |
| d52d2e3 | Add Product Lock Scoring: complexity measurement system |

### Next

- [x] Add worklog specification
- [x] Add i18n README translations
- [x] Set up GitHub Pages
- [x] Bootstrap product.lock.json for this repo
