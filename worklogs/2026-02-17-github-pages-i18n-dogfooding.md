# [2026-02-17] Add GitHub Pages, i18n, and dogfooding

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
| added | Entity: Worklog | Worklog format with folder structure |
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

- PLS: 0 → 12 (new)
- Level: Simple
- Breakdown:
  - D: 4 × 2.5 × 0.3 = 3
  - F: 5 × 1.0 = 5
  - I: 6 × 0.5 = 3
  - A: 10 × 0.1 = 1

### Review Findings

No review conducted — lock just created.

### Decisions

1. **Spec-only repo**: Added cliTool, lockEditor, schemaValidator to denied. This repo is the specification, not the tooling. Consumer projects build their own validators.

2. **Lock entity fields**: Listed all 10 top-level fields of product.lock.json as Lock entity fields, making this the most strictly defined entity.

3. **Three actors**: AiWorker, AiReviewer, Human map directly to the three roles defined in the specification itself.

4. **Worklog as folder, not single file**: Worklogs accumulate over time. One file per session in `worklogs/` directory, named `{YYYY-MM-DD}-{kebab-case-title}.md`.

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
| created | README-de.md |
| created | README-ja.md |
| created | README-zh-TW.md |
| created | worklogs/2026-02-16-initial-release.md |
| created | worklogs/2026-02-17-github-pages-i18n-dogfooding.md |
| modified | README.md |

### Commits

| Hash | Message |
|------|---------|
| ea3e0df | Add worklog specification for tracking changes across sessions |
| 7f937a8 | Add i18n README translations (zh-TW, ja, de) with language switcher |
| 09e98d6 | Add GitHub Pages site with Just the Docs theme |
| 16931ed | Create CNAME |
| a8339e7 | Dogfood: add product.lock.json and product.worklog.md for this repo |

### Next

- [ ] Verify GitHub Pages renders correctly at productlock.org
- [ ] Run self-review: verify codebase against product.lock.json
- [ ] Add JSON Schema file for editor validation ($schema support)
- [ ] Consider adding example product.lock.json files for common project types
