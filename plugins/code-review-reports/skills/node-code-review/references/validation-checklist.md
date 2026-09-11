# Pre-Delivery Validation Checklist

Run through every item below against the drafted report before showing it to the user. This is a gate, not a suggestion: if any item fails, fix the report and re-run the whole checklist — don't patch just the failing item and assume the rest still holds, since a fix can introduce a new inconsistency elsewhere.

## 1. Markdown validation

- [ ] The file is valid Markdown: headings, lists, tables, and code blocks are all correctly structured.
- [ ] Every Markdown table has the same number of columns in every row, including every per-category Key Findings table (13 columns in every row, no exceptions — there is one table per category, not one report-wide table).
- [ ] Every code block that is opened is also closed, and uses a language tag where useful (`ts`, `js`, `json`, `sql`, `dockerfile`, `yaml`, `bash`).
- [ ] No malformed syntax (stray `#`, unclosed `**`, broken link syntax, unescaped `|` inside table cells — check every cell containing `||`, a shell pipe, or a union type).
- [ ] File paths, route paths, env var names, and code identifiers are consistently formatted in backticks throughout.

## 2. Finding validation (for every finding)

- [ ] Finding ID is present and unique within the report.
- [ ] Category is present and matches the prefix table in `rubrics.md`.
- [ ] Key Finding title clearly and specifically describes the issue (not a vague label like "Error handling issue").
- [ ] Example Path exists in the reviewed repository — it was actually opened during the review, not guessed.
- [ ] Problem / Observation is directly supported by something read from the repository.
- [ ] Evidence / Example is specific (real code excerpt, real config line, real metric, or a precise repository observation such as "8 of 11 route modules") — not a generic illustrative snippet that wasn't actually seen in the repo.
- [ ] Why It Is a Problem states a concrete failure scenario with a trigger (which request, input, load condition, or deploy causes it), not a restatement of the observation.
- [ ] Recommended Improvement is actionable — names a concrete change, and reuses an existing repo pattern/utility where one exists rather than inventing a new one.
- [ ] Any cited rule ID exists in `node-best-practices.md` (`NBP-<section>.<item>`), and the section number matches the finding's subject. No invented IDs.
- [ ] Every `SEC` finding cites an OWASP category alongside the NBP ID (or OWASP alone for access-control findings, which NBP does not cover).
- [ ] Benefits of Fix are relevant to the specific recommendation, not boilerplate ("improves maintainability" attached to everything).
- [ ] Severity is exactly one of Critical / High / Medium / Low.
- [ ] Remediation Effort is exactly one of Low / Medium / High.
- [ ] Priority follows the Severity × Effort table in `rubrics.md` — recompute it and check it matches.
- [ ] The finding is mapped to an applicable supplied review criterion where one applies.

## 3. Evidence validation (before including any finding)

- [ ] The referenced file, route, or config key actually exists in the repository at the stated path.
- [ ] The described implementation is actually present in that file (re-open and confirm, don't rely on memory from earlier in the review).
- [ ] **Absence findings were checked globally.** For any finding of the form "X is missing", the app/server bootstrap and middleware/module registration were read, and X is genuinely not applied there either (no global validation pipe, no `express-async-errors`, no framework-level wrapping, no proxy-level rate limiting mentioned in deploy config).
- [ ] **Framework behavior was checked, not assumed.** Claims that depend on framework version behavior (Express 4 vs 5 async error forwarding, Fastify schema defaults, Nest global pipes) were verified against the version in `package.json`.
- [ ] **Security claims were read, not inferred.** A "hardcoded secret" was actually read as a literal value; an "injectable query" was confirmed to be string-interpolated rather than parameterized by the ORM.
- [ ] The recommendation addresses what was actually observed, not a related-but-different concern.
- [ ] No unsupported assumptions were introduced (e.g. claiming a performance impact without naming the mechanism, or asserting "every route"/"never" without checking multiple instances — state the count you actually verified).
- [ ] The finding is not a duplicate — check it doesn't restate another finding's root cause under a different title.

## 4. Consistency validation

- [ ] Severity is consistent with the stated impact (a Critical rating needs a genuinely critical consequence in "Why It Is a Problem," not a Medium-sounding one).
- [ ] Priority is consistent with Severity and Remediation Effort (per the mechanical table — no manual overrides).
- [ ] Remediation Effort is reasonable for the proposed fix's actual scope, and any fix requiring a schema migration, an API contract change, or an auth behavior change is rated at least Medium.
- [ ] Similar issues found in different places are categorized under the same prefix/category, not split inconsistently (e.g. all missing-input-validation findings under one of `API`/`SEC`, not scattered across both).
- [ ] Finding IDs follow the `PREFIX-NNN` convention with no gaps or reused numbers within a prefix.
- [ ] Terminology is used consistently throughout (e.g. don't call the same thing a "handler" in one finding and a "controller" in another; use the repo's own vocabulary).

## 5. Completeness validation

- [ ] Every supplied review criterion appears either in a finding's "Mapped criterion," in the "Criteria Not Evaluated" list, or is implicitly satisfied with no issues found (state that explicitly in Review Criteria or Conclusion — don't just drop it silently).
- [ ] The Review Scope section states the detected stack (framework, module system, language/strictness, data layer, Node version, topology) and, for a monorepo, exactly which workspaces were reviewed.
- [ ] All significant findings surfaced during the review appear in the Key Findings table (nothing was found and then left out of the table).
- [ ] Every row in a category's Key Findings table has a matching `####` subsection under that same `###` category in Detailed Findings, and vice versa — same IDs, same order, same category grouping.
- [ ] Category (`###`) sections appear in the same order in both Key Findings and Detailed Findings — the fixed order from `rubrics.md` (or a minted category appended last, per Review Criteria) — and within each category section, rows/subsections are ordered by Priority (P1 first) then Severity. The report is not laid out as a single flat, report-wide priority-ordered table.
- [ ] Every item in Recommendations traces back to one or more actual findings — nothing invented for the sake of the section.
- [ ] No section is empty or contains placeholder text (e.g. no "TBD," no "N/A" without an explanation).
- [ ] Any low-value, unsupported, or duplicate observations identified during drafting were removed, not left in "just in case."

Only present the report to the user once every box above is checked. If you had to make a fix, say so briefly when presenting the report (e.g. "caught and fixed one Priority value that didn't match its Severity × Effort before finalizing") rather than silently correcting and moving on — it's useful signal that the gate did its job.
