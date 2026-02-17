# [2026-02-17] Custom theme, 4-language i18n, and product repositioning

### Context
- Branch: `main`
- Agent: Claude (Opus 4.6)
- Lock version: 0.1.0
- PLS before: 12

### Goal
Replace default Jekyll theme with custom dark/light theme, add path-based i18n for 4 languages (en/zh/ja/de), and reposition product messaging from "product boundary spec" to "manage AI like you manage teams".

### Changes

| Action | Target | Detail |
|--------|--------|--------|
| added | Feature: customTheme | Dark/light mode toggle with CSS custom properties |
| added | Feature: pathBasedI18n | /en/, /zh/, /ja/, /de/ URL structure with language switcher |
| added | Feature: dataDrivenHomePage | _data/home.yml powers all home page content across 4 languages |
| modified | Story: multi-language | Extended from 2 languages (en/zh) to 4 languages (en/zh/ja/de) |
| added | docs/ja/* | Full Japanese translation of all 6 specification documents |
| added | docs/de/* | Full German translation of all 6 specification documents |
| modified | Hero messaging | Changed from "Define what your product is and is not" to "Manage AI like you manage teams" |
| modified | README headlines | Updated all 4 README files to reflect new positioning |
| modified | Root guide links | Updated productlock.org → spec.productlock.org in root guide files |
| added | Lucide SVG icons | Replaced emoji icons with Lucide SVG icons on home page |

### Lock Delta

No lock changes — these are documentation/site improvements, not product boundary changes.

### Score Delta

- PLS: 12 → 12 (no change)
- No product boundary modifications

### Review Findings

No review conducted — documentation-only changes.

### Decisions

1. **Custom theme over Just the Docs**: Replaced default Just the Docs theme with fully custom dark/light theme for brand consistency and full control over layout.

2. **Path-based i18n over subdomain**: Chose `/en/`, `/zh/`, `/ja/`, `/de/` URL structure over subdomains. Simpler to manage with GitHub Pages, no DNS complexity.

3. **Data-driven home page**: All home page content lives in `_data/home.yml` with language keys. Layout uses `{% assign t = site.data.home[page.lang] %}` — zero hardcoded strings.

4. **Product repositioning**: "Define what your product is and is not" was too generic. New positioning "Manage AI like you manage teams" reflects that Product Lock encodes management experience (scope control, review processes, decision tracking) as a protocol AI can follow.

5. **Lucide SVG icons**: Replaced emoji icons with Lucide SVGs for consistent rendering across platforms. SVGs use `stroke="currentColor"` for theme compatibility.

6. **Domain split**: `spec.productlock.org` for GitHub Pages docs site, `productlock.org` reserved for future main site.

### Files

| Action | Path |
|--------|------|
| created | docs/_data/home.yml |
| created | docs/_includes/theme.html |
| created | docs/_layouts/home.html |
| created | docs/_layouts/default.html |
| created | docs/de/index.html |
| created | docs/de/generator-guide.md |
| created | docs/de/plan-spec.md |
| created | docs/de/reviewer-guide.md |
| created | docs/de/scoring.md |
| created | docs/de/specification.md |
| created | docs/de/worklog.md |
| created | docs/ja/index.html |
| created | docs/ja/generator-guide.md |
| created | docs/ja/plan-spec.md |
| created | docs/ja/reviewer-guide.md |
| created | docs/ja/scoring.md |
| created | docs/ja/specification.md |
| created | docs/ja/worklog.md |
| modified | docs/_config.yml |
| modified | docs/index.html |
| modified | docs/zh/index.html |
| modified | docs/en/*.md (front matter for i18n) |
| modified | docs/zh/*.md (front matter for i18n) |
| modified | README.md |
| modified | README-zh-TW.md |
| modified | README-ja.md |
| modified | README-de.md |
| modified | product-lock-generator-guide.md |
| modified | product-lock-reviewer-guide.md |

### Next

- [ ] Run self-review: verify site renders correctly at spec.productlock.org
- [ ] Add JSON Schema file for editor validation ($schema support)
- [ ] Consider adding interactive examples or playground
- [ ] Add more languages if community demand exists (ko, es, fr)
