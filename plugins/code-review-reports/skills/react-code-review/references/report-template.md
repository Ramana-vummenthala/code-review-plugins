# Core Review Report — Template

Fill this structure in exactly. Section headings under `##` are fixed — you may add extra `###` subsections under `## Review Criteria` if the supplied criteria warrant grouping, and you **must** add one `###` category subsection per category-with-findings under both `## Key Findings` and `## Detailed Findings` (see those sections below) — but do not rename, remove, or reorder the `##` headings themselves, and never change a Key Findings table's columns.

````markdown
# Core Review Report

## Executive Summary

2-5 sentences: what was reviewed, the overall health verdict, the count of findings by severity (e.g. "1 Critical, 3 High, 6 Medium, 2 Low"), and the single most important thing to act on first. No hedging filler — state the verdict.

## Review Scope

- Repository: `<path or repo name>`, commit/branch reviewed if known.
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

Each category subsection has its own 13-column table with the same columns every time:

### \<Category Name\>

| Finding ID | Category | Key Finding | Files Affected | Example Path | Problem / Observation | Evidence / Example | Why It Is a Problem | Recommended Improvement | Benefits of Fix | Severity | Remediation Effort | Priority |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| ARCH-001 | Architecture | \<short title\> | 3 files | `src/foo/Bar.tsx` | \<1-2 sentence observation\> | \<short inline code/metric, escape pipes as `\|`\> | \<1 sentence consequence\> | \<1-2 sentence actionable fix\> | \<1 sentence benefit\> | High | Medium | P1 |

Keep each cell to one or two sentences — the full narrative goes in Detailed Findings. Escape any `|` characters inside cell text as `\|` so the table doesn't break. **Within each category's table**, order rows by Priority (P1 first), then Severity within a priority — priority ordering is a secondary sort inside each category section, not the primary organizing principle of the report (that's category grouping; see `## Recommendations` below for the cross-category priority view).

## Detailed Findings

Mirror `## Key Findings`' category grouping and order exactly: one `###` category subsection per category-with-findings, same categories, same order, each containing one `####` subsection per finding in that category's table, same order as that table. Shape per finding:

### \<Category Name\>

#### ARCH-001 — \<Finding title matching the table's "Key Finding" column\>

- **Category:** Architecture
- **Files affected:** `src/foo/Bar.tsx`, `src/foo/Baz.tsx`, `src/foo/Qux.tsx` (list all, or "12 files matching `src/features/*/index.tsx`" for a widespread pattern — don't enumerate more than ~10 individually, describe the pattern instead)
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

## Recommendations

A short, prioritized punch-list (not a repeat of every finding) — group related findings into a handful of concrete next actions a team could put on a roadmap, **in priority order**. This is the intentional cross-category, priority-first view of the report: Key Findings and Detailed Findings are organized by discipline for focused review; this section is where P1s across every category surface together for planning.

## Conclusion

2-4 sentences: overall state of the codebase relative to the supplied criteria, and what re-review (if any) would look for next time.
````
