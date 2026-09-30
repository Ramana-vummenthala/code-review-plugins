# Core Review Report — Template

Fill this structure in exactly. Section headings under `##` are fixed — you may add extra `###` subsections under `## Review Criteria` if the supplied criteria warrant grouping, and you **must** add one `###` category subsection per category-with-findings under both `## Key Findings` and `## Detailed Findings` (see those sections below) — but do not rename, remove, or reorder the `##` headings themselves, and never change a Key Findings table's columns: `Category | Module Name | File / Location | Type (Component/Function/Module) | Metric | Observation | Severity | Impact | Recommendation | Priority`. **Module Name is the functional module the code belongs to** (Login, Dashboard, Profile, Checkout, ...), never a file name — see the Module Name section of `references/rubrics.md` for how to derive the list and which fallback labels to use when code belongs to no single feature. The file or component path goes in `File / Location`. Finding IDs (`ARCH-001`, etc.) are not their own column — they're embedded in the File / Location cell — but every row still traces to a full write-up in `## Detailed Findings`. `## Code Complexity Summary` is a conditional section, include it only when the supplied criteria ask for complexity or code-smell analysis (see below).

**Writing style**: write the whole report in plain, natural human phrasing. Never use an em dash (`—`) anywhere, use a comma, a period, or a parenthetical instead. Avoid stock AI-report filler and hedging; state findings directly.

````markdown
# Core Review Report

## Executive Summary

This is the report's high-level summary — write it so a reader who reads nothing else still knows what matters. Cover, in 2-5 sentences plus the two count lines below:

- What was reviewed, and the overall health verdict (no hedging filler — state it).
- Count of findings by severity (e.g. "1 Critical, 3 High, 6 Medium, 2 Low").
- Count of findings by category (e.g. "Architecture: 4, Performance: 3, Security: 2, Accessibility: 3").
- The single most important thing to act on first, named specifically (a Finding ID, its functional module, and its file, not "fix the high-priority items").

## Review Scope

- Repository: `<path or repo name>`, commit/branch reviewed if known.
- Functional modules identified in this repository, and how they were derived (routes, feature folders, navigation structure). List each one with what it maps to, e.g. "Login (`src/features/auth/**`), Dashboard (`src/features/dashboard/**`), Profile (`src/features/profile/**`), Checkout (`src/features/checkout/**`), plus Cross-cutting for shared infrastructure". Every Module Name used in Key Findings has to come from this list, so the reader can see the taxonomy the report is grouped by.
- What was in scope (directories/areas actually examined) and what was explicitly out of scope.
- React-specific references used: the `vercel-react-best-practices` skill (cite rule IDs used) and/or the `react-coding-standards-best-practices` (Cognine) skill(s)/check IDs actually used — or, for either source, a note that it was unavailable/not applicable to the supplied criteria and general best-practice judgment was used instead.
- Any limitation on the review itself (e.g. "no access to run the app or test suite; static review only").

## Review Criteria

The criteria supplied for this review, listed so the reader can check coverage:

1. `<criterion 1, verbatim or lightly paraphrased>`
2. `<criterion 2>`
...

If the supplied criteria added a category not in the skill's standard Finding-ID prefix table, note the new prefix here.

### Criteria Not Evaluated

For any supplied criterion that could not be assessed from the repository (state this rather than guessing):

- `<criterion>` — could not be validated because `<concrete reason, e.g. "no CI configuration present in the repo to assess test-in-CI enforcement">`.

Omit this subsection only if every criterion was evaluated.

## Key Findings

Findings are grouped into one `###` subsection per category — not laid out as a single flat table — so a reader can review one discipline (Architecture, Security, Performance, ...) at a time. Use one subsection per category **that has at least one finding**, titled with the category's full name (e.g. `### Architecture`, `### Security`), in this fixed order: Architecture, Code Quality, Security, Performance, Data / API, State Management, Error Handling, Testing, React Best Practices, Accessibility, Dependency Governance — then any minted category (see `references/rubrics.md`) last, in the order it was minted. Skip any category with zero findings entirely (an already-passing criterion is recorded in `## Review Criteria`, not here).

Each category subsection has its own 10-column table with the same columns every time. **Row granularity is one row per affected file/component instance**, not one merged row per root cause: if the same underlying finding recurs in 5 components, that's 5 rows, each carrying the same embedded Finding ID in its File / Location cell but its own Metric/Observation/Impact where those differ between instances (identical text is fine when the instances are truly identical). Several rows sharing one Module Name is expected and correct when those instances sit in the same functional module, the File / Location cell is what makes each row distinct.

### \<Category Name\>

| Category | Module Name | File / Location | Type (Component/Function/Module) | Metric | Observation | Severity | Impact | Recommendation | Priority |
|---|---|---|---|---|---|---|---|---|---|
| Architecture | Checkout | `src/checkout/Bar.tsx` (ARCH-001) | Component | 8 inbound cross-feature imports | \<1-2 sentence observation\> | High | \<1 sentence concrete consequence\> | \<1-2 sentence actionable fix\> | P1 |
| Architecture | Profile | `src/profile/Baz.tsx` (ARCH-001) | Component | 6 inbound cross-feature imports | \<1-2 sentence observation for this instance\> | Medium | \<1 sentence concrete consequence\> | \<1-2 sentence actionable fix\> | P2 |

Module Name must be one of the functional modules recorded in `## Review Scope`, written as a plain product-facing name (Login, Dashboard, Profile, Checkout, Reporting) or one of the fallback labels in `references/rubrics.md`, never a file name, folder path, or component name. File / Location carries the repository-relative path of the file the row is about, with the Finding ID after it. Keep each cell to one or two sentences — the full narrative goes in Detailed Findings. Escape any `|` characters inside cell text as `\|` so the table doesn't break. Metric must be a real, quantifiable measurement pulled from the repo (see `references/rubrics.md`'s Metric section) — use the literal text `N/A` only when no metric genuinely applies, never as a placeholder for one you didn't measure. Severity must meet or exceed the Metric → Severity threshold for that category in `references/rubrics.md` — never rate lower than the threshold implies. **Within each category's table**, order rows by Priority (P1 first), then Severity within a priority — priority ordering is a secondary sort inside each category section, not the primary organizing principle of the report (that's category grouping; see `## Recommendations` below for the cross-category priority view).

## Detailed Findings

Mirror `## Key Findings`' category grouping and order exactly: one `###` category subsection per category-with-findings, same categories, same order. Since Key Findings has one row per file instance but shares one Finding ID across all instances of the same root cause, Detailed Findings has one `####` subsection **per Finding ID** (not per row) — its "Files affected" list must enumerate every file-instance row that ID has in that category's Key Findings table, in the same order. Shape per finding:

### \<Category Name\>

#### ARCH-001 — \<short, specific finding title\>

- **Category:** Architecture
- **Functional module(s):** Checkout, Profile (every distinct Module Name this ID appears under in the Key Findings table, in the same order)
- **Files affected:** `src/checkout/Bar.tsx`, `src/profile/Baz.tsx`, `src/profile/Qux.tsx` (list all, matching every row this ID has in the Key Findings table — or "12 files matching `src/features/*/index.tsx`" for a widespread pattern, still one Key Findings row per file/instance even when the write-up describes the pattern once)
- **Mapped criterion:** \<which supplied review criterion this addresses\>

**Problem / Observation**

What was found, in plain terms.

**Evidence**

```tsx
// src/foo/Bar.tsx — representative excerpt, not the whole file
<the actual code observed in the repo, or a metric/measurement>
```

**Why it is a problem**

The concrete technical or operational consequence — cite a specific failure scenario, not a generic "this is bad practice."

**Recommended improvement**

Specific, actionable steps. Name the pattern/utility to use if one already exists in the repo (reuse over reinvention) or, where relevant, cite the Vercel rule ID or Cognine check ID being applied (e.g. `rerender-no-inline-components`, `SEC-004`, `PERF-002`).

**Benefits of fix**

What improves, concretely (e.g. "eliminates a re-render on every keystroke in the payment form" beats "improves performance").

**Severity:** High · **Remediation Effort:** Medium · **Priority:** P1

---

(repeat `####` findings within the category, then move to the next `###` category subsection, matching Key Findings' order throughout)

## Code Complexity Summary

Include this section only when the supplied Review Criteria ask for complexity or code-smell analysis; omit it entirely otherwise.

A scorecard, not a repeat of Detailed Findings: one row per file actually measured for complexity, whether or not it produced a finding, so a reader sees the full complexity landscape, not just the flagged outliers. Module Name is the functional module here too, so a reader can see which parts of the product carry the complexity.

| Module Name | File / Location | Type | Complexity Metric | Classification | Finding ID |
|---|---|---|---|---|---|
| Checkout | `src/checkout/Bar.tsx` | Component | cyclomatic complexity ~15, 480 lines | Code smell: large component | CQ-001 |
| Profile | `src/profile/Baz.ts` | Function | cyclomatic complexity ~4, 60 lines | Acceptable | \- |

Classification is one of `Acceptable` or `Code smell: <short label>` (e.g. "large component," "deep nesting," "duplicate logic," "God object"). Every row with a non-"Acceptable" classification must have a Finding ID that also appears in that category's Key Findings table and Detailed Findings; an "Acceptable" row has no Finding ID, use `-`.

## Recommendations

A short, prioritized punch-list (not a repeat of every finding) — group related findings into a handful of concrete next actions a team could put on a roadmap, **in priority order**. This is the intentional cross-category, priority-first view of the report: Key Findings and Detailed Findings are organized by discipline for focused review; this section is where P1s across every category surface together for planning.

## Conclusion

2-4 sentences: overall state of the codebase relative to the supplied criteria, and what re-review (if any) would look for next time.
````
