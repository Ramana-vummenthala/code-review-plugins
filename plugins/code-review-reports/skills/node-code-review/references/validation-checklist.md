# Pre-Delivery Validation Checklist

Run through every item below against the drafted report before showing it to the user. This is a gate, not a suggestion: if any item fails, fix the report and re-run the whole checklist — don't patch just the failing item and assume the rest still holds, since a fix can introduce a new inconsistency elsewhere.

## 1. Markdown validation

- [ ] The file is valid Markdown: headings, lists, tables, and code blocks are all correctly structured.
- [ ] Every Markdown table has the same number of columns in every row, including every per-category Key Findings table (10 columns — Category, Module Name, File / Location, Type, Metric, Observation, Severity, Impact, Recommendation, Priority — in every row, no exceptions; there is one table per category, not one report-wide table).
- [ ] Every code block that is opened is also closed, and uses a language tag where useful (`ts`, `js`, `json`, `dockerfile`, `yaml`, `sql`, `bash`).
- [ ] No malformed syntax (stray `#`, unclosed `**`, broken link syntax, unescaped `|` inside table cells). Check `||` in evidence cells specifically — `process.env.X || 'default'` breaks a table row unless escaped as `\|\|`.
- [ ] File paths and code identifiers are consistently formatted in backticks throughout.
- [ ] No em dash (`—`) appears anywhere in the report body; the writing reads as plain, natural human phrasing, not stock AI-report filler.

## 2. Finding validation (for every row)

- [ ] Finding ID is embedded in the File / Location cell (e.g. `` `src/routes/order.route.ts` (SEC-001) ``), present and unique per root cause (shared across rows only when those rows are genuinely the same root cause in different files).
- [ ] Category is present and matches the prefix table in `rubrics.md`.
- [ ] Module Name is a **functional module** (Authentication, Orders, Payments, Notifications, ...) or one of the fallback labels in `rubrics.md` (Cross-cutting, Shared Libraries, Build & Deployment, Application Bootstrap) — never a file name, folder path, or class name.
- [ ] Every Module Name used anywhere in the report appears in the functional module list recorded in `## Review Scope`, and the same file is always mapped to the same module across every row it appears in.
- [ ] File / Location exists in the reviewed repository at the stated path — it was actually opened during the review, not guessed. Any line number cited points at the code being described.
- [ ] Type is exactly one of Route / Middleware / Function / Module / Config, classifies the code in File / Location (not the functional module), and is correct per the definitions in `rubrics.md`.
- [ ] Metric is a real, quantifiable measurement pulled from the repo (see `rubrics.md`'s Metric section), or the literal `N/A` only when no metric genuinely applies — never `N/A` as a stand-in for one not measured. Where the finding is about coverage, the metric is a count out of a total, not an anecdote.
- [ ] Observation is directly supported by something read from the repository (real code excerpt, real metric, or a precise repository observation) — not a generic illustrative snippet that wasn't actually seen in the repo.
- [ ] Impact states a meaningful, concrete technical or operational consequence, not a restatement of the Observation.
- [ ] Recommendation is actionable — names a concrete change, reuses an existing repo pattern/utility where one exists rather than inventing a new one, and cites the applicable NBP ID (plus the OWASP category for `SEC`) or says plainly that no rule ID applies.
- [ ] Every cited rule ID actually exists in `node-best-practices.md`; no invented IDs, and no OWASP category cited that doesn't match the vulnerability class described.
- [ ] Severity is exactly one of Critical / High / Medium / Low, and meets or exceeds the Metric → Severity threshold for that category in `rubrics.md`.
- [ ] Remediation Effort (tracked internally, shown only in Detailed Findings) is exactly one of Low / Medium / High.
- [ ] Priority follows the Severity × Effort table in `rubrics.md` — recompute it and check it matches.
- [ ] The finding is mapped to an applicable supplied review criterion where one applies.

## 3. Evidence validation (before including any finding)

- [ ] The referenced file or route actually exists in the repository at the stated path.
- [ ] The described implementation is actually present in that file (re-open and confirm, don't rely on memory from earlier in the review).
- [ ] **For every finding that reports an absence** (no validation, no `try/catch`, no auth check, no timeout, no error handler): the app/server bootstrap and middleware registration were read, and the thing is confirmed not handled globally there or by the framework. A gap already covered by a global pipe, a framework-level wrapper, or `express-async-errors` is a false positive, not a finding.
- [ ] Findings are rated against the framework actually detected in Step 1, not a different one (no NestJS conventions applied to an Express app, and vice versa).
- [ ] A claimed hardcoded secret was read in place and is genuinely a secret, not a local-dev default or a `process.env` read; a claimed injection point was checked against whether the ORM parameterizes it.
- [ ] The recommendation addresses what was actually observed, not a related-but-different concern.
- [ ] No unsupported assumptions were introduced (e.g. claiming a load-related failure without a plausible mechanism, or asserting "all routes"/"never" without checking every instance the metric claims).
- [ ] The finding is not a duplicate — check it doesn't restate another finding's root cause under a different title or a different category prefix.

## 4. Consistency validation

- [ ] Severity is consistent with the stated Impact (a Critical rating needs a genuinely critical consequence in Impact, not a Medium-sounding one).
- [ ] Severity matches (or is justifiably higher than) the Metric → Severity threshold table in `rubrics.md` for that row's category — not just "consistent with impact" in the abstract.
- [ ] Priority is consistent with Severity and Remediation Effort (per the mechanical table — no manual overrides).
- [ ] Remediation Effort is reasonable for the proposed fix's actual scope; anything requiring a schema change, an API contract change, or an auth behavior change is at least Medium.
- [ ] Similar issues found in different places are categorized under the same prefix/category, not split inconsistently (a missing timeout is `ASYNC` everywhere, not `ASYNC` once and `PERF` the next time).
- [ ] Finding IDs follow the `PREFIX-NNN` convention with no gaps or reused numbers within a prefix.
- [ ] Terminology is used consistently throughout (e.g. don't call the same thing a "controller" in one finding and a "handler" in another; use the framework's own vocabulary).

## 5. Completeness validation

- [ ] Every supplied review criterion appears either in a finding's "Mapped criterion," in the "Criteria Not Evaluated" list, or is implicitly satisfied with no issues found (state that explicitly in Review Criteria or Conclusion — don't just drop it silently).
- [ ] All significant findings surfaced during the review appear in the Key Findings table (nothing was found and then left out of the table).
- [ ] Every row in a category's Key Findings table maps to a `####` subsection (by embedded Finding ID) under that same `###` category in Detailed Findings, and every Detailed Findings subsection's "Files affected" list matches every row that ID has in the table, in the same order — same IDs, same category grouping.
- [ ] Each Detailed Findings subsection's "Functional module(s)" line lists exactly the distinct Module Names that ID carries in the Key Findings table, in the same order.
- [ ] Where a summarizing row stands in for a widespread pattern, the detailed write-up says which files it covers and the Metric's count matches that set.
- [ ] Category (`###`) sections appear in the same order in both Key Findings and Detailed Findings — the fixed order from `rubrics.md` (or a minted category appended last, per Review Criteria) — and within each category section, rows/subsections are ordered by Priority (P1 first) then Severity. The report is not laid out as a single flat, report-wide priority-ordered table.
- [ ] Review Scope states the detected stack (framework, module system, language, data layer, Node version, topology) and, for a monorepo, exactly which workspace packages were and weren't reviewed.
- [ ] Every item in Recommendations traces back to one or more actual findings — nothing invented for the sake of the section.
- [ ] No section is empty or contains placeholder text (e.g. no "TBD," no unexplained "N/A" — except the Metric column, where a bare `N/A` is allowed only when no metric genuinely applies to that finding).
- [ ] Any low-value, unsupported, or duplicate observations identified during drafting were removed, not left in "just in case."
- [ ] If the supplied criteria asked for complexity or code-smell analysis, `## Code Complexity Summary` is present, its Module Name column also uses functional modules (6 columns: Module Name, File / Location, Type, Complexity Metric, Classification, Finding ID), and every non-"Acceptable" row has a matching Finding ID that also appears in Key Findings and Detailed Findings; if the criteria didn't ask for it, the section is correctly omitted (not left in as an empty shell).

Only present the report to the user once every box above is checked. If you had to make a fix, say so briefly when presenting the report (e.g. "caught and fixed one malformed table row before finalizing") rather than silently correcting and moving on — it's useful signal that the gate did its job.
