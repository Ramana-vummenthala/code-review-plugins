# Pre-Delivery Validation Checklist

Run through every item below against the drafted report before showing it to the user. This is a gate, not a suggestion: if any item fails, fix the report and re-run the whole checklist — don't patch just the failing item and assume the rest still holds, since a fix can introduce a new inconsistency elsewhere.

## 1. Markdown validation

- [ ] The file is valid Markdown: headings, lists, tables, and code blocks are all correctly structured.
- [ ] Every Markdown table has the same number of columns in every row, including every per-category Key Findings table (10 columns — Category, Module Name, File / Location, Type, Metric, Observation, Severity, Impact, Recommendation, Priority — in every row, no exceptions; there is one table per category, not one report-wide table).
- [ ] Every code block that is opened is also closed, and uses a language tag where useful.
- [ ] No malformed syntax (stray `#`, unclosed `**`, broken link syntax, unescaped `|` inside table cells).
- [ ] File paths and code identifiers are consistently formatted in backticks throughout.
- [ ] No em dash (`—`) appears anywhere in the report body; the writing reads as plain, natural human phrasing, not stock AI-report filler.

## 2. Finding validation (for every row)

- [ ] Finding ID is embedded in the File / Location cell (e.g. `` `src/checkout/Bar.tsx` (ARCH-001) ``), present and unique per root cause (shared across rows only when those rows are genuinely the same root cause in different files).
- [ ] Category is present and matches the prefix table in `rubrics.md`.
- [ ] Module Name is a **functional module** (Login, Dashboard, Profile, Checkout, ...) or one of the fallback labels in `rubrics.md` (Cross-cutting, Shared UI, Build & Tooling, Application Shell) — never a file name, folder path, or component name.
- [ ] Every Module Name used anywhere in the report appears in the functional module list recorded in `## Review Scope`, and the same file is always mapped to the same module across every row it appears in.
- [ ] File / Location exists in the reviewed repository at the stated path — it was actually opened during the review, not guessed.
- [ ] Type is exactly one of Component / Function / Module, classifies the code in File / Location (not the functional module), and is correct per the definitions in `rubrics.md`.
- [ ] Metric is a real, quantifiable measurement pulled from the repo (see `rubrics.md`'s Metric section), or the literal `N/A` only when no metric genuinely applies — never `N/A` as a stand-in for one not measured.
- [ ] Observation is directly supported by something read from the repository (real code excerpt, real metric, or a precise repository observation) — not a generic illustrative snippet that wasn't actually seen in the repo.
- [ ] Impact states a meaningful, concrete technical or operational consequence, not a restatement of the Observation.
- [ ] Recommendation is actionable — names a concrete change, and reuses an existing repo pattern/utility where one exists rather than inventing a new one.
- [ ] Severity is exactly one of Critical / High / Medium / Low, and meets or exceeds the Metric → Severity threshold for that category in `rubrics.md`.
- [ ] Remediation Effort (tracked internally, shown only in Detailed Findings) is exactly one of Low / Medium / High.
- [ ] Priority follows the Severity × Effort table in `rubrics.md` — recompute it and check it matches.
- [ ] The finding is mapped to an applicable supplied review criterion where one applies.

## 3. Evidence validation (before including any finding)

- [ ] The referenced file or component actually exists in the repository at the stated path.
- [ ] The described implementation is actually present in that file (re-open and confirm, don't rely on memory from earlier in the review).
- [ ] The recommendation addresses what was actually observed, not a related-but-different concern.
- [ ] No unsupported assumptions were introduced (e.g. claiming a performance impact without a plausible mechanism, or asserting "always"/"never" without checking multiple instances).
- [ ] The finding is not a duplicate — check it doesn't restate another finding's root cause under a different title.

## 4. Consistency validation

- [ ] Severity is consistent with the stated Impact (a Critical rating needs a genuinely critical consequence in Impact, not a Medium-sounding one).
- [ ] Severity matches (or is justifiably higher than) the Metric → Severity threshold table in `rubrics.md` for that row's category — not just "consistent with impact" in the abstract.
- [ ] Priority is consistent with Severity and Remediation Effort (per the mechanical table — no manual overrides).
- [ ] Remediation Effort is reasonable for the proposed fix's actual scope.
- [ ] Similar issues found in different places are categorized under the same prefix/category, not split inconsistently.
- [ ] Finding IDs follow the `PREFIX-NNN` convention with no gaps or reused numbers within a prefix.
- [ ] Terminology is used consistently throughout (e.g. don't call the same thing a "controller" in one finding and a "handler" in another).

## 5. Completeness validation

- [ ] Every supplied review criterion appears either in a finding's "Mapped criterion," in the "Criteria Not Evaluated" list, or is implicitly satisfied with no issues found (state that explicitly in Review Criteria or Conclusion — don't just drop it silently).
- [ ] All significant findings surfaced during the review appear in the Key Findings table (nothing was found and then left out of the table).
- [ ] Every row in a category's Key Findings table maps to a `####` subsection (by embedded Finding ID) under that same `###` category in Detailed Findings, and every Detailed Findings subsection's "Files affected" list matches every row that ID has in the table, in the same order — same IDs, same category grouping.
- [ ] Each Detailed Findings subsection's "Functional module(s)" line lists exactly the distinct Module Names that ID carries in the Key Findings table, in the same order.
- [ ] Category (`###`) sections appear in the same order in both Key Findings and Detailed Findings — the fixed order from `report-template.md` (or a minted category appended last, per Review Criteria) — and within each category section, rows/subsections are ordered by Priority (P1 first) then Severity. The report is not laid out as a single flat, report-wide priority-ordered table.
- [ ] Every item in Recommendations traces back to one or more actual findings — nothing invented for the sake of the section.
- [ ] No section is empty or contains placeholder text (e.g. no "TBD," no unexplained "N/A" — except the Metric column, where a bare `N/A` is allowed only when no metric genuinely applies to that finding).
- [ ] Any low-value, unsupported, or duplicate observations identified during drafting were removed, not left in "just in case."
- [ ] If the supplied criteria asked for complexity or code-smell analysis, `## Code Complexity Summary` is present, its Module Name column also uses functional modules (6 columns: Module Name, File / Location, Type, Complexity Metric, Classification, Finding ID), and every non-"Acceptable" row has a matching Finding ID that also appears in Key Findings and Detailed Findings; if the criteria didn't ask for it, the section is correctly omitted (not left in as an empty shell).

Only present the report to the user once every box above is checked. If you had to make a fix, say so briefly when presenting the report (e.g. "caught and fixed one malformed table row before finalizing") rather than silently correcting and moving on — it's useful signal that the gate did its job.
