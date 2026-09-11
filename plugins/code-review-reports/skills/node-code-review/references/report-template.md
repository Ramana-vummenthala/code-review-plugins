# Core Review Report — Template (Node.js)

Fill this structure in exactly. Section headings under `##` are fixed — you may add extra `###` subsections under `## Review Criteria` if the supplied criteria warrant grouping, and you **must** add one `###` category subsection per category-with-findings under both `## Key Findings` and `## Detailed Findings` (see those sections below) — but do not rename, remove, or reorder the `##` headings themselves, and never change a Key Findings table's columns.

````markdown
# Core Review Report

## Executive Summary

2-5 sentences: what was reviewed (service name and stack), the overall health verdict, the count of findings by severity (e.g. "1 Critical, 3 High, 6 Medium, 2 Low"), and the single most important thing to act on first. No hedging filler — state the verdict.

## Review Scope

- Repository / service: `<path or repo name>`, commit/branch reviewed if known. For a monorepo, name the exact workspace packages reviewed and those left out.
- Detected stack: framework and version, module system (ESM/CJS), language (TS/JS, `strict` on or off), data layer, Node version from `engines`/CI/Dockerfile, runtime topology (single process / cluster / Docker / serverless).
- What was in scope (directories/areas actually examined) and what was explicitly out of scope.
- References used: `node-best-practices.md` (cite the NBP IDs actually applied), OWASP categories cited, framework-specific guidance applied, and any installed org standards plugin used — or a note that no Node standards plugin was installed and findings are grounded in NBP/OWASP plus repository evidence.
- Any limitation on the review itself (e.g. "static review only — the service was not run, no database was available, no load testing performed; infrastructure and CI config live in a separate repository").

## Review Criteria

The criteria supplied for this review, listed so the reader can check coverage:

1. `<criterion 1, verbatim or lightly paraphrased>`
2. `<criterion 2>`
...

If the supplied criteria added a category not in the skill's standard Finding-ID prefix table, note the new prefix here.

### Criteria Not Evaluated

For any supplied criterion that could not be assessed from the repository (state this rather than guessing):

- `<criterion>` — could not be validated because `<concrete reason, e.g. "deployment and alerting configuration is not in this repository, so production monitoring coverage could not be assessed">`.

Omit this subsection only if every criterion was evaluated.

## Key Findings

Findings are grouped into one `###` subsection per category — not laid out as a single flat table — so a reader can review one discipline (Architecture, Security, Data, ...) at a time. Use one subsection per category **that has at least one finding**, titled with the category's full name (e.g. `### Architecture`, `### Security`), in the fixed order from `references/rubrics.md`: Architecture, Code Quality, API Design & Contracts, Data & Persistence, Async & Concurrency, Error Handling, Security, Performance, Testing, Operations & Observability, Containerization & Deployment, Dependency Governance — then any minted category last, in the order it was minted. Skip any category with zero findings entirely (an already-passing criterion is recorded in `## Review Criteria`, not here).

Each category subsection has its own 13-column table with the same columns every time:

### \<Category Name\>

| Finding ID | Category | Key Finding | Files Affected | Example Path | Problem / Observation | Evidence / Example | Why It Is a Problem | Recommended Improvement | Benefits of Fix | Severity | Remediation Effort | Priority |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| SEC-001 | Security | \<short title\> | 8 routes | `src/routes/merchant.route.ts` | \<1-2 sentence observation\> | \<short inline code/metric, escape pipes as `\|`\> | \<1 sentence consequence\> | \<1-2 sentence actionable fix, citing the NBP/OWASP ID\> | \<1 sentence benefit\> | High | Medium | P1 |

Keep each cell to one or two sentences — the full narrative goes in Detailed Findings. Escape any `|` characters inside cell text as `\|` so the table doesn't break (this matters more in Node reports than React ones: `||` appears constantly in `process.env.X || 'default'` evidence). **Within each category's table**, order rows by Priority (P1 first), then Severity within a priority — priority ordering is a secondary sort inside each category section, not the primary organizing principle of the report (that's category grouping; see `## Recommendations` below for the cross-category priority view).

## Detailed Findings

Mirror `## Key Findings`' category grouping and order exactly: one `###` category subsection per category-with-findings, same categories, same order, each containing one `####` subsection per finding in that category's table, same order as that table. Shape per finding:

### \<Category Name\>

#### SEC-001 — \<Finding title matching the table's "Key Finding" column\>

- **Category:** Security
- **Files affected:** `src/routes/merchant.route.ts`, `src/controllers/merchant.controller.ts` (list all, or "8 of 11 route modules under `src/routes/`" for a widespread pattern — don't enumerate more than ~10 individually, describe the pattern instead)
- **Mapped criterion:** \<which supplied review criterion this addresses\>
- **Grounded in:** `NBP-6.10` / OWASP A04:2021 Insecure Design \<or the applicable IDs; omit this line only when no rule applies and the finding rests on repository evidence alone\>

**Problem / Observation**

What was found, in plain terms. Name the concrete route, handler, query, or config — not "the API layer".

**Evidence**

```ts
// src/routes/merchant.route.ts:34 — representative excerpt, not the whole file
<the actual code observed in the repo, or a metric/measurement>
```

**Why it is a problem**

The concrete technical or operational consequence — a specific failure scenario with a trigger: which request, which input, which load condition, which deploy produces the bad outcome. "An attacker with any valid token can read another merchant's payouts by changing the `:id` in the path" beats "this is insecure."

**Recommended improvement**

Specific, actionable steps. Name the pattern/utility that already exists in the repo (reuse over reinvention — if there is already a `validate(schema)` middleware or an `ApiError` class, say to apply it rather than proposing a new one) and cite the rule ID being applied (e.g. `NBP-2.10`, `NBP-6.4` / OWASP A03).

**Benefits of fix**

What improves, concretely (e.g. "closes cross-tenant read access on the 8 merchant endpoints" or "removes a per-request Prisma client, freeing ~40 idle connections at peak" beats "improves security/performance").

**Severity:** High · **Remediation Effort:** Medium · **Priority:** P1

---

(repeat `####` findings within the category, then move to the next `###` category subsection, matching Key Findings' order throughout)

## Recommendations

A short, prioritized punch-list (not a repeat of every finding) — group related findings into a handful of concrete next actions a team could put on a roadmap, **in priority order**. This is the intentional cross-category, priority-first view of the report: Key Findings and Detailed Findings are organized by discipline for focused review; this section is where P1s across every category surface together for planning. Where several findings share one root cause (e.g. "introduce a central error handler" resolves four `ERR` findings and two `SEC` leak findings), say so and list the finding IDs each action closes.

## Conclusion

2-4 sentences: overall state of the codebase relative to the supplied criteria, and what re-review (if any) would look for next time.
````
