# Core Review Report, Excel output

This file replaces `report-template.md` as the delivery format. The review work does not
change: same exploration, same rubrics, same evidence rules. Only the output changes. You
hand over one `.xlsx` workbook instead of a Markdown document.

`report-template.md`, `rubrics.md`, `node-best-practices.md` and `validation-checklist.md`
are still in use. `rubrics.md` is what you rate every row against. `report-template.md` is
now only a reference for the wording of a good Observation, Impact and Recommendation, not
for the file you produce.

## Why a workbook and not a document

The people who act on this report do not read it top to bottom. A tech lead filters to
Severity = High, a module owner filters to their own module, a manager counts P1 rows. A
Markdown document cannot be filtered or sorted, so every one of those readers has to scroll
through everything. A sheet gives each of them their own view of the same data, and it also
forces one row per affected file, which is what makes the counts honest.

The trade is that there is no place to hide a vague finding. Every row must stand on its
own, because nobody reads the row above it first.

## What the workbook looks like

- One sheet per reviewed service or app. The sheet name is what the team calls that service,
  for example `Orders API`, `Customer Portal Backend`, `Admin Service`. If you reviewed a
  single service, the workbook has one findings sheet.
- One final sheet named `Threshold Mapping`, always last. It explains the numbers behind the
  ratings, so a reader can check why a row was rated High and not Medium.
- No summary sheet, no charts, no pivot tables. The findings sheets are the report.

Sheet name rules: 31 characters or fewer, and no `[ ] : * ? / \`. The build script cleans
and de-duplicates names for you, but pick short ones so it does not have to.

## The 12 columns

Every findings sheet has these columns, in this order, with these exact headers. Do not add,
remove, rename or reorder them.

| # | Column | What goes in it |
|---|---|---|
| A | Category | The top-level review area from the supplied criteria, for example `Code Quality`, `Security`, `Performance`. Same text on every row that belongs to that area. |
| B | Sub Category | The specific criterion inside that area, for example `Code Organization Review`, `Access Control Review`, `Test Coverage Review`. Taken from the supplied criteria list, not invented per row. |
| C | Module Name | The functional module, for example `Orders`, `Authentication`, `Cross-cutting`. See the Module Name section of `rubrics.md`. Never a file name. |
| D | File / Location | The repo-relative path in backticks, then the Finding ID in brackets: `` `src/routes/order.route.ts:22` (SEC-001) ``. Add the line number when it pins the issue. |
| E | Type | Exactly one of `Route`, `Middleware`, `Function`, `Module`, `Config`. Describes the file in column D, not the module in column C. |
| F | Metric | The measured number, usually a count out of a total: `3 of 5 routes`, `640-line file`, `0 of 12 outbound calls`. |
| G | Evidence | The real code or config that proves the finding. Copied from the file, never retyped from memory. |
| H | Observation | What is actually there, in plain English, and why it is a problem. |
| I | Severity | Exactly one of `Critical`, `High`, `Medium`, `Low`. Coloured automatically. |
| J | Impact | The concrete consequence if nobody fixes it. |
| K | Recommendation | The specific change to make, with the rule ID it satisfies. |
| L | Priority | Exactly one of `P1`, `P2`, `P3`. Derived from Severity and Effort, never picked by feel. Coloured automatically. |

Two of these are new compared to the Markdown template: **Sub Category** (column B) and
**Evidence** (column G). Evidence is the important one. In the Markdown report the code
sample lived in the Detailed Findings section. Here there is no Detailed Findings section,
so the code goes in the row. That is what lets a reader check the finding without opening
the repo.

Remediation Effort is still rated for every finding, because Priority is derived from it,
but it does not get a column. Pass it to the build script as `effort` and the script checks
that the Priority you wrote matches the Severity and Effort you rated.

### Category and Sub Category, worked out

The supplied criteria decide these two columns, not the Finding ID prefix. If the criteria
arrive as a grouped list, the group heading is the Category and the item under it is the Sub
Category:

```
Code Quality                        -> Category    = "Code Quality"
  1. Code organization review       -> Sub Category = "Code Organization Review"
  2. Code complexity and smells     -> Sub Category = "Code Complexity & Smells"
  3. Readability and documentation  -> Sub Category = "Code Readability & Documentation"
```

If the criteria arrive as a flat list with no grouping, use the category name from the
prefix table in `rubrics.md` as the Category (Architecture, Security, Data & Persistence and
so on), and write the criterion itself as the Sub Category, titled in 2 to 5 words ending in
`Review`.

A row's Category does not have to match its Finding ID prefix. A duplicated `ApiError.ts`
gets the ID `ARCH-002` because layering is an Architecture concern, but it sits under
Category `Code Quality` if that is the criteria block that asked for it. The ID traces the
root cause. The Category traces the criterion. Keep both.

## How to write each cell

New developers read this sheet. Write for someone who joined last week and has not seen the
file you are describing.

### Rules for all text columns

- Short sentences. One idea per sentence.
- Name the thing. `getOrderById in order.service.ts` beats `the code`.
- Explain a term the first time you use it. "N+1 query, meaning one query for the list and
  then one more per row" is fine to write out. Assume nothing.
- Say what to do, not what to think about. "Add a timeout of 5 seconds" beats "consider
  timeout handling".
- No em dashes anywhere. Use a comma, a full stop, or brackets.
- No filler. Drop "it is important to note that", "in today's fast-paced environment",
  "leverage", "utilize", "facilitate", "robust", "seamless".
- Present tense, active voice. "The route never checks ownership", not "ownership checking
  has not been implemented".
- Plain words over clever ones. Simple English is the point, this sheet gets read by people
  whose first language is not English.

### Column by column

**Metric (F).** A number you actually counted. Prefer a count out of a total, because that
shows coverage instead of one anecdote. Write `23 of 31 route handlers`, not `many routes`.
Use the literal text `N/A` only when nothing is countable, never as a stand-in for a count
you did not take.

**Evidence (G).** Copy the real lines out of the file. Start with a comment giving the path
and line, then the code:

```
// services/order-service/src/routes/order.route.ts:22
router.get('/:id', auth(), orderController.getOrder);
```

Keep it between 3 and 15 lines. Trim the middle with `// ...` if the file is long. If the
finding is that something is missing, show the code where it should have been and say so in
one line above it. Use real line breaks in the cell, which means `\n` in the JSON. Never
paste a snippet you did not read in the repo.

**Observation (H).** 2 to 6 sentences. First say what is there. Then say why it is a
problem. If you measured something, say how, for example "we ran diff on all three copies
and they match character for character". A reader should finish this cell knowing the
problem is real without having to trust you.

**Impact (J).** One or two sentences on what breaks, and for whom. This is not a repeat of
the Observation. "Any logged-in user can read another customer's order by changing the id in
the URL" is an impact. "The ownership check is missing" is the observation again.

**Recommendation (K).** Name the change. If the repo already solves this problem somewhere
else, point at that place and say to copy it, because reusing an existing pattern is always
easier to review than a new one. Cite the rule ID at the end: an NBP ID from
`node-best-practices.md`, plus the OWASP category for a security row. If no rule ID fits, say
so in words instead of inventing one.

## Row order

Excel has no headings, so ordering is the only grouping you get. Sort rows in each sheet:

1. Category, in the fixed order from the prefix table in `rubrics.md`.
2. Sub Category, in the order the criteria list them.
3. Priority, P1 first.
4. Severity, Critical first.
5. File path, alphabetically, so repeat instances of one Finding ID sit together.

One row per affected file, the same rule as the Markdown report. If a root cause hits five
files, that is five rows sharing one Finding ID, each with its own Metric and Evidence. If it
hits more files than is readable, list the significant ones and add one summary row whose
Metric carries the real count, and name the covered files in that row's Observation.

## Fixed formatting

The build script applies all of this. It is written down so the sheet looks the same every
time the skill runs.

| Thing | Value |
|---|---|
| Font | Calibri 10 everywhere |
| Header row | Bold white text on `1F3864` navy, centred, wrapped, height 33.75 |
| Column widths (A to L) | 32.1, 30, 24.6, 38.1, 18.3, 34.4, 82.4, 75.9, 13.4, 47.6, 81.7, 12.7 |
| Cell alignment | A, B, C, E, I, L centred. D, F, G, H, J, K left. All wrapped, all vertically centred. |
| Borders | Thin box on every cell including the header |
| Freeze panes | `F2`, so the header row and the first five columns stay put while scrolling |
| Auto filter | `A1:L<last row>` |
| Row height | Estimated from the longest wrapped cell, 11.25 points per line, floor 40.5, ceiling 400 |
| Severity fill (I) | Critical `F4CCCC`, High `FCE4D6`, Medium `FFF2CC`, Low `E2EFDA` |
| Priority fill (L) | P1 `F4CCCC`, P2 `FFF2CC`, P3 `E2EFDA` |

Static fills, not conditional formatting, so the colours survive a copy into another
workbook or an export to PDF.

## The Threshold Mapping sheet

Always the last sheet. Two blocks:

- **Block 1, audit metric thresholds and treatment.** Five columns: Audit Metric,
  Recommended Threshold, Audit Treatment, Severity, Priority. This is the rule that was
  applied, so a reader can check a rating instead of arguing with it.
- **Block 2, severity band table.** Five columns: Metric, Low / Acceptable, Medium, High,
  Critical / Very High. The measured value alone places a finding in a band. Where a row
  reports several metrics, the worst band wins.

**This sheet is built per review, not shipped whole.** It must carry the thresholds this
review actually used and nothing else. A sheet listing container hygiene rules on a review
that never looked at a Dockerfile tells the reader the audit covered ground it did not.

`scripts/threshold_mapping.json` is a catalog, not a fixed sheet. It holds one block of
thresholds and one block of severity bands per review area:

| Area key | Prefix | Covers |
|---|---|---|
| `architecture` | ARCH | Layering, business logic in the web layer, responsibilities per file, config loading, TypeScript strictness |
| `code-quality` | CQ | Complexity, nesting, length, parameters, duplication, dead code, commented-out code, type safety, lint, docs, naming |
| `api-design` | API | Response shapes, status codes, versioning, pagination, API documentation |
| `data` | DATA | Transactions, N+1, indexing, over-fetching, result bounds, connection lifecycle, migrations |
| `async` | ASYNC | Floating promises, timeouts, concurrency limits, emitter errors, sequential awaits |
| `error-handling` | ERR | Central handler, swallowed errors, operational vs programmer errors, process handlers, detail leakage |
| `security` | SEC | Input validation, authorization, injection, secrets, rate limiting, headers, CORS, tokens, payload limits |
| `performance` | PERF | Event loop blocking, unbounded reads, caching, payload size, per-request cost |
| `testing` | TEST | Coverage, error-path tests, API tests, isolation, CI quality gates |
| `operations` | OPS | Logging, graceful shutdown, health checks, Node version, boot-time config validation, statelessness |
| `containerization` | DOCK | Image user, signal handling, image contents, base image, build context |
| `dependencies` | DEP | Advisories, lockfile, CI install, unmaintained packages, version drift, licences |

The script picks the areas two ways:

1. **You name them.** Put `threshold_areas` in the findings JSON. An entry matches on the
   area key, the prefix, the label, or any of the area's `match` words, and it tolerates a
   criteria heading around the word, so `"Security and OWASP Review"`, `"security"` and
   `"SEC"` all land on the same block. This is the option to use, because the supplied
   criteria are what the sheet is supposed to reflect.
2. **It infers them.** With no `threshold_areas`, the script reads the Finding ID prefixes
   out of the File / Location cells and keeps the matching areas. A review that produced only
   `SEC` and `ARCH` findings gets exactly those two blocks.

Option 2 is the safety net, not the plan. Name the areas yourself, because a criterion that
was evaluated and passed with no findings still deserves its threshold on the sheet. That is
how a reader tells "we checked and it was fine" apart from "we never looked".

If the script warns that an area name did not match, it is telling you the criteria asked
about something the catalog has no rules for. Do not ignore it. Write the rows yourself and
pass them as `extra_thresholds` and `extra_severity_bands`, using the same 5-column shape.
A criterion with no threshold row is a criterion you cannot score consistently.

Two rules that do not bend: never score a finding against a threshold that is not on this
sheet, and never leave a block on the sheet that nothing was scored against and no criterion
asked for.

Adding a new area to the catalog is worth doing when the same criterion shows up across
reviews. Copy an existing area block in `scripts/threshold_mapping.json`, give it a `key`, a
`prefix` that matches the Finding ID prefix from `rubrics.md`, a `label`, and a `match` list
of the words a criteria heading is likely to use.

## Building the file

Write the findings to a JSON file in the scratchpad directory, then run the script. Do not
hand-write openpyxl code, the script already handles the styling, the row heights, the
colours and the Excel limits.

```bash
python "${CLAUDE_PLUGIN_ROOT}/skills/node-code-review/scripts/build_report_xlsx.py" findings.json
```

`${CLAUDE_PLUGIN_ROOT}` is set for you when this skill runs from the installed plugin. If it is
unset, the skill has been vendored into a repository instead, so run the copy that sits next to
this file: `python .claude/skills/node-code-review/scripts/build_report_xlsx.py findings.json`.
The script finds its own `threshold_mapping.json` either way, so nothing else changes.

The JSON:

```json
{
  "output": "E:/path/to/ACME_PORTAL_CODE_QUALITY_V1.xlsx",
  "evidence_monospace": false,
  "sheets": [
    {
      "name": "Orders API",
      "rows": [
        {
          "category": "Security",
          "sub_category": "Access Control Review",
          "module": "Orders",
          "file": "`services/order-service/src/routes/order.route.ts:22` (SEC-001)",
          "type": "Route",
          "metric": "3 of 5 routes take :id with no ownership check",
          "evidence": "// services/order-service/src/routes/order.route.ts:22\nrouter.get('/:id', auth(), orderController.getOrder);",
          "observation": "The route checks that the caller is logged in but never checks that the order belongs to them. auth() only verifies the token.",
          "severity": "Critical",
          "impact": "Any logged-in user can read another customer's order by changing the id in the URL.",
          "recommendation": "Load the order first and compare its customerId with req.user.id before returning it. The same check already exists in invoice.service.ts, copy that. OWASP A01:2021.",
          "priority": "P1",
          "effort": "Low"
        }
      ]
    }
  ]
}
```

Optional keys:

- `evidence_monospace`: `true` renders column G in Consolas 9. Default is `false`, which
  matches the reference workbook.
- `threshold_areas`: the review areas the Threshold Mapping sheet should cover, for example
  `["security", "performance", "Code Organization Review"]`. Set this from the supplied
  criteria. Left out, the script infers the areas from the Finding ID prefixes in the rows.
- `extra_thresholds`, `extra_severity_bands`: extra rows appended after the catalog rows, for
  a criterion the catalog does not cover. Same 5-column shape as the catalog:
  `[metric, threshold, treatment, severity, priority]` and
  `[metric, low, medium, high, critical]`.
- `thresholds`, `severity_bands`: replace the catalog outright. Only for a review whose
  criteria share nothing with the catalog. Prefer `threshold_areas` plus `extra_thresholds`.
- `threshold_intro`, `band_intro`: replace the sentence above each block. The default intro
  already gets the covered area names appended to it.
- `include_threshold_sheet`: `false` drops the Threshold Mapping sheet. Leave it on.
- `effort` on a row: not written to the sheet. The script uses it to check that Priority
  matches the Severity and Effort matrix in `rubrics.md`.

The script refuses to write anything if a row has an empty required cell, an unknown Type,
Severity or Priority value, or a Priority that contradicts the Severity and Effort. Read the
errors it prints, fix the JSON, run it again.

Two Excel details the script already handles, worth knowing about: a cell whose text starts
with `=` is written as plain text and not as a formula, and a cell over 32767 characters is
trimmed with a warning. If you see that warning, the Evidence cell is too long, cut it down
by hand instead of leaving the trim in.

## Where to write the file

Default output is `<repo-path>/CORE_REVIEW_REPORT.xlsx`, or the path given by `--output` at
invocation. If a file is already there, do not overwrite it silently. Say so and confirm, or
write next to it with a version suffix. Tell the user the full path when you are done.

## Validation gate for the Excel output

Run this after the workbook is built and before you hand it over. The evidence and rating
items in `validation-checklist.md` still apply in full. These are the extra ones the file
format brings, plus the ones that replace its Markdown checks.

**Structure**

- [ ] Every findings sheet has the 12 columns in the documented order, with the exact header text.
- [ ] Header row is frozen with the first five columns (`F2`) and auto filter covers `A1:L<last row>` on every findings sheet.
- [ ] `Threshold Mapping` is present and is the last sheet.
- [ ] No sheet name is over 31 characters or contains `[ ] : * ? / \`.
- [ ] Reopen the saved file with openpyxl after writing it. If it loads and the row counts match the JSON, it is not corrupt.

**Every row**

- [ ] All 12 cells have content. No blanks, no `TBD`, no `-`.
- [ ] Category and Sub Category come from the supplied criteria and are spelled identically on every row that shares them. Sort column B and check for near-duplicates such as `Code Organisation Review` next to `Code Organization Review`.
- [ ] Module Name is a functional module from the list you derived, never a file name, and the same file always maps to the same module.
- [ ] File / Location holds a path that exists in the repo, in backticks, with the Finding ID in brackets.
- [ ] Type is one of the five allowed values and describes the file, not the module.
- [ ] Metric is a real measurement, or `N/A` only where nothing is countable.
- [ ] Evidence is real code from that file, and the path in its first-line comment matches column D.
- [ ] Severity meets or exceeds the threshold its Metric implies on the Threshold Mapping sheet.
- [ ] Priority matches Severity and Effort per `rubrics.md`. The script checks this when you pass `effort`, so pass it.
- [ ] Severity and Priority cells are colour filled. An uncoloured cell means the value was misspelled.

**Writing**

- [ ] No em dash anywhere in the workbook, including the Threshold Mapping sheet.
- [ ] No filler phrases, no hedging, no AI-report voice.
- [ ] Impact says something different from Observation on every row.
- [ ] Recommendation names a concrete change and cites a rule ID that really exists in `node-best-practices.md`, or says plainly that no rule ID applies.
- [ ] Read three rows at random out loud. If a sentence needs a second pass to parse, rewrite it shorter.

**Coverage**

- [ ] Every supplied criterion is either present as a Sub Category, listed as passing, or listed as not evaluated with a reason. Report the last two to the user in the handover message, since the workbook has no prose section for them.
- [ ] The Threshold Mapping sheet covers the areas the supplied criteria asked about, and no others. Read the "Areas covered" line under the block 1 title and check it against the criteria list.
- [ ] Every finding was scored against a row that is on that sheet, and no block sits there that neither a finding nor a criterion needed.
- [ ] Any area name the script warned about was handled with `extra_thresholds` rather than left out.
- [ ] Rows are sorted by Category, Sub Category, Priority, Severity, then file path.

If an item fails, fix it and run the whole gate again. A fix in one place moves rows and can
break the sort somewhere else.
