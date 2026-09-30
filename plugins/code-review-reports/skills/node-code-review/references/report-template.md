# Core Review Report — Template (Node.js)

Fill this structure in exactly. Section headings under `##` are fixed — you may add extra `###` subsections under `## Review Criteria` if the supplied criteria warrant grouping, and you **must** add one `###` category subsection per category-with-findings under both `## Key Findings` and `## Detailed Findings` (see those sections below) — but do not rename, remove, or reorder the `##` headings themselves, and never change a Key Findings table's columns: `Category | Module Name | File / Location | Type (Route/Middleware/Function/Module/Config) | Metric | Observation | Severity | Impact | Recommendation | Priority`. **Module Name is the functional module the code belongs to** (Authentication, Orders, Payments, Notifications, ...), never a file name — see the Module Name section of `references/rubrics.md` for how to derive the list and which fallback labels to use when code belongs to no single feature. The file path goes in `File / Location`. Finding IDs (`SEC-001`, etc.) are not their own column — they're embedded in the File / Location cell — but every row still traces to a full write-up in `## Detailed Findings`. `## Code Complexity Summary` is a conditional section, include it only when the supplied criteria ask for complexity or code-smell analysis (see below).

**Writing style**: write the whole report in plain, natural human phrasing. Never use an em dash (`—`) anywhere, use a comma, a period, or a parenthetical instead. Avoid stock AI-report filler and hedging; state findings directly.

````markdown
# Core Review Report

## Executive Summary

This is the report's high-level summary — write it so a reader who reads nothing else still knows what matters. Cover, in 2-5 sentences plus the two count lines below:

- What was reviewed (service name and detected stack), and the overall health verdict (no hedging filler — state it).
- Count of findings by severity (e.g. "1 Critical, 3 High, 6 Medium, 2 Low").
- Count of findings by category (e.g. "Security: 4, Data & Persistence: 3, Async & Concurrency: 2, Operations: 3").
- The single most important thing to act on first, named specifically (a Finding ID, its functional module, and its file, not "fix the high-priority items").

## Review Scope

- Repository / service: `<path or repo name>`, commit/branch reviewed if known. For a monorepo, name the exact workspace packages reviewed and those left out.
- Detected stack: framework and version, module system (ESM/CJS), language (TS/JS, `strict` on or off), data layer, Node version from `engines`/CI/Dockerfile, runtime topology (single process / cluster / Docker / serverless).
- Functional modules identified in this service, and how they were derived (route prefixes, business-component folders, domain model, background jobs). List each one with what it maps to, e.g. "Authentication (`/auth/*`, `src/modules/auth/**`), Orders (`/api/orders/*`, `src/modules/orders/**`), Payments (`/api/payments/*`, `src/modules/payments/**`), plus Cross-cutting for shared middleware and Build & Deployment for the Docker/CI surface". Every Module Name used in Key Findings has to come from this list, so the reader can see the taxonomy the report is grouped by.
- What was in scope (directories/areas actually examined) and what was explicitly out of scope.
- References used: `node-best-practices.md` (cite the NBP IDs actually applied), OWASP categories cited, framework-specific guidance applied, and any installed org standards plugin used — or a note that no Node standards plugin was installed and findings are grounded in NBP/OWASP plus repository evidence.
- Any limitation on the review itself (e.g. "static review only, the service was not run, no database was available, no load testing performed; infrastructure and CI config live in a separate repository").

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

Findings are grouped into one `###` subsection per category — not laid out as a single flat table — so a reader can review one discipline (Architecture, Security, Data, ...) at a time. Use one subsection per category **that has at least one finding**, titled with the category's full name (e.g. `### Security`, `### Data & Persistence`), in the fixed order from `references/rubrics.md`: Architecture, Code Quality, API Design & Contracts, Data & Persistence, Async & Concurrency, Error Handling, Security, Performance, Testing, Operations & Observability, Containerization & Deployment, Dependency Governance — then any minted category last, in the order it was minted. Skip any category with zero findings entirely (an already-passing criterion is recorded in `## Review Criteria`, not here).

Each category subsection has its own 10-column table with the same columns every time. **Row granularity is one row per affected file**, not one merged row per root cause: if the same underlying finding recurs in 5 route modules, that's 5 rows, each carrying the same embedded Finding ID in its File / Location cell but its own Metric/Observation/Impact where those differ between instances (identical text is fine when the instances are truly identical). Several rows sharing one Module Name is expected and correct when those files sit in the same functional module, the File / Location cell is what makes each row distinct. For a pattern spread across more files than is readable as rows, list the most significant instances individually and add one summarizing row whose Metric carries the real count ("23 of 31 route handlers"), then say in the detailed write-up which files that row covers.

### \<Category Name\>

| Category | Module Name | File / Location | Type (Route/Middleware/Function/Module/Config) | Metric | Observation | Severity | Impact | Recommendation | Priority |
|---|---|---|---|---|---|---|---|---|---|
| Security | Merchants | `src/routes/merchant.route.ts` (SEC-001) | Route | 6 of 6 routes take `:id` with no ownership check | \<1-2 sentence observation\> | Critical | \<1 sentence concrete consequence\> | \<1-2 sentence actionable fix, citing the NBP/OWASP ID\> | P1 |
| Security | Orders | `src/routes/order.route.ts` (SEC-001) | Route | 3 of 5 routes take `:id` with no ownership check | \<1-2 sentence observation for this instance\> | High | \<1 sentence concrete consequence\> | \<1-2 sentence actionable fix\> | P1 |

Module Name must be one of the functional modules recorded in `## Review Scope`, written as a plain domain name (Authentication, Orders, Payments, Notifications, Reporting) or one of the fallback labels in `references/rubrics.md` (Cross-cutting, Shared Libraries, Build & Deployment, Application Bootstrap), never a file name, folder path, or class name. File / Location carries the repository-relative path of the file the row is about, with the Finding ID after it; add a line number when it pins the issue precisely (`src/app.ts:42`). Keep each cell to one or two sentences — the full narrative goes in Detailed Findings. Escape any `|` characters inside cell text as `\|` so the table doesn't break (this matters more in Node reports than React ones: `||` appears constantly in `process.env.X || 'default'` evidence). Metric must be a real, quantifiable measurement pulled from the repo (see `references/rubrics.md`'s Metric section) — use the literal text `N/A` only when no metric genuinely applies, never as a placeholder for one you didn't measure. Severity must meet or exceed the Metric → Severity threshold for that category in `references/rubrics.md` — never rate lower than the threshold implies. **Within each category's table**, order rows by Priority (P1 first), then Severity within a priority — priority ordering is a secondary sort inside each category section, not the primary organizing principle of the report (that's category grouping; see `## Recommendations` below for the cross-category priority view).

## Detailed Findings

Mirror `## Key Findings`' category grouping and order exactly: one `###` category subsection per category-with-findings, same categories, same order. Since Key Findings has one row per file but shares one Finding ID across all instances of the same root cause, Detailed Findings has one `####` subsection **per Finding ID** (not per row) — its "Files affected" list must enumerate every file-instance row that ID has in that category's Key Findings table, in the same order. Shape per finding:

### \<Category Name\>

#### SEC-001 — \<short, specific finding title\>

- **Category:** Security
- **Functional module(s):** Merchants, Orders (every distinct Module Name this ID appears under in the Key Findings table, in the same order)
- **Files affected:** `src/routes/merchant.route.ts`, `src/routes/order.route.ts` (list all, matching every row this ID has in the Key Findings table — or "8 of 11 route modules under `src/routes/`" for a widespread pattern, still one Key Findings row per file even when the write-up describes the pattern once)
- **Mapped criterion:** \<which supplied review criterion this addresses\>
- **Grounded in:** `NBP-6.10` / OWASP A01:2021 Broken Access Control \<or the applicable IDs; omit this line only when no rule applies and the finding rests on repository evidence alone\>

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

**Severity:** Critical · **Remediation Effort:** Medium · **Priority:** P1

---

(repeat `####` findings within the category, then move to the next `###` category subsection, matching Key Findings' order throughout)

## Code Complexity Summary

Include this section only when the supplied Review Criteria ask for complexity or code-smell analysis; omit it entirely otherwise.

A scorecard, not a repeat of Detailed Findings: one row per file actually measured for complexity, whether or not it produced a finding, so a reader sees the full complexity landscape, not just the flagged outliers. Module Name is the functional module here too, so a reader can see which parts of the service carry the complexity.

| Module Name | File / Location | Type | Complexity Metric | Classification | Finding ID |
|---|---|---|---|---|---|
| Orders | `src/services/order.service.ts` | Function | cyclomatic complexity ~18, 640 lines | Code smell: God object | CQ-001 |
| Authentication | `src/services/auth.service.ts` | Function | cyclomatic complexity ~5, 90 lines | Acceptable | \- |

Classification is one of `Acceptable` or `Code smell: <short label>` (e.g. "large service," "deep nesting," "duplicate logic," "God object," "fat model"). Every row with a non-"Acceptable" classification must have a Finding ID that also appears in that category's Key Findings table and Detailed Findings; an "Acceptable" row has no Finding ID, use `-`.

## Recommendations

A short, prioritized punch-list (not a repeat of every finding) — group related findings into a handful of concrete next actions a team could put on a roadmap, **in priority order**. This is the intentional cross-category, priority-first view of the report: Key Findings and Detailed Findings are organized by discipline for focused review; this section is where P1s across every category surface together for planning. Where several findings share one root cause (e.g. "introduce a central error handler" resolves four `ERR` findings and two `SEC` leak findings), say so and list the finding IDs each action closes.

## Conclusion

2-4 sentences: overall state of the codebase relative to the supplied criteria, and what re-review (if any) would look for next time.
````
