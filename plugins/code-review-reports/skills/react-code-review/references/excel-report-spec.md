# Excel Report Spec (`CORE_REVIEW_REPORT.xlsx`)

The Core Review Report is delivered as an **Excel workbook**, not a Markdown file. This document is the authoritative spec for that workbook: its sheets, the exact Key Findings columns, how each column is written, the formatting to apply, the generation recipe, and the validation gate to run before handing it over.

The rating rules do not change: Finding IDs, Module Name derivation, Type, Metric, the Metric to Severity thresholds, Remediation Effort, and the mechanical Severity x Effort to Priority table all still come from `rubrics.md`. This file only changes the **output format** and adds one column (`Evidence`).

`report-template.md` remains the spec for the optional Markdown output (`--format md`), and its 10-column table stays as it is. When writing Excel, use this file instead.

## Output file

- Default path: `<repo-path>/CORE_REVIEW_REPORT.xlsx`, overridden by `--output <path>`.
- If `--output` names a `.md` file, or the user passes `--format md`, fall back to `report-template.md` and produce Markdown instead.
- Never overwrite an existing report silently. If the target file already exists, write `CORE_REVIEW_REPORT_<YYYY-MM-DD>.xlsx` alongside it and say so when handing over.

## Workbook structure

Four sheets, in this tab order. Sheet names are literal.

| # | Sheet | Required | Contents |
|---|---|---|---|
| 1 | `Summary` | Yes | Executive summary, review scope, supplied criteria, criteria not evaluated, recommendations, conclusion. |
| 2 | `Key Findings` | Yes | The 11-column findings table. One row per affected file instance. This is the sheet that matters most, build it first. |
| 3 | `Detailed Findings` | Yes | One block per Finding ID: the long-form writeup that `report-template.md` puts under `## Detailed Findings`. |
| 4 | `Code Complexity` | Conditional | Only when the supplied Review Criteria ask for complexity or code-smell analysis. Omit the sheet entirely otherwise, do not leave an empty tab. |

### Sheet 1: `Summary`

A two-column layout (`A` = label, `B` = value), not a table with headers. Rows, in order:

1. `Repository` and the path/branch/commit reviewed.
2. `Review date`.
3. `Overall verdict`, 2 to 5 sentences.
4. `Findings by severity`, e.g. `1 Critical, 3 High, 6 Medium, 2 Low`.
5. `Findings by category`, e.g. `Architecture: 4, React Best Practices: 3, Security: 2`.
6. `Act on first`, naming a specific Finding ID, its module and its file.
7. `Functional modules`, the module list from Step 3 with the paths each maps to. Every Module Name used on `Key Findings` has to appear here.
8. `References used`, the Vercel rule IDs and/or Cognine check IDs actually cited, or a note that a source was unavailable.
9. `Review limitations`, e.g. static review only, no test run.
10. `Review criteria`, numbered, one per row.
11. `Criteria not evaluated`, one per row with the concrete reason. Skip this block only if every criterion was evaluated.
12. `Recommendations`, the prioritized cross-category punch list, one action per row, P1 first.
13. `Conclusion`, 2 to 4 sentences.

### Sheet 2: `Key Findings`

Row 1 is the header. Row 2 onward is data. **Exactly these 11 columns, in this order, with this header text:**

| Col | Header | Content |
|---|---|---|
| A | `Category` | Full category name from the `rubrics.md` prefix table, e.g. `React Best Practices`, `Architecture`, `Security`. Not the prefix. |
| B | `Module Name` | The functional module (`Applicant Portal`, `Orders (Admin)`, `Dashboard`, `Login`) or a fallback label (`Cross-cutting`, `Shared UI`, `Build & Tooling`, `Application Shell`). Never a file name. |
| C | `File / Location` | Repository-relative path in backticks, then the Finding ID in parentheses: `` `src/pages/ApplicantDetails/ApplicantDetails.tsx` (RCT-003) ``. |
| D | `Type` | Exactly one of `Component`, `Function`, `Module`. Classifies the code in column C, not the module in column B. |
| E | `Metric` | A real, quantifiable measurement from the repo, e.g. `2 render functions passed as props, 0 useCallback`. Literal `N/A` only when no metric genuinely applies. |
| F | `Evidence` | **New in the Excel format.** The raw proof, see the Evidence column rules below. |
| G | `Observation` | What was found and why the pattern behaves the way it does, in plain language a non-specialist can follow. 2 to 5 sentences. |
| H | `Severity` | `Critical`, `High`, `Medium`, or `Low`. Must meet or exceed the `rubrics.md` threshold for that category. |
| I | `Impact` | One sentence naming the concrete consequence. Not a restatement of Observation. |
| J | `Recommendation` | The actionable fix. Name the pattern, hook, or component to use, and the existing repo utility if one applies. |
| K | `Priority` | `P1`, `P2`, or `P3`, recomputed from the Severity x Effort table in `rubrics.md`. Never assigned by feel. |

Row order: group by Category (the fixed category order from `report-template.md`), then within a category by Priority (P1 first), then Severity. Because Excel gives the reader an autofilter, the Category column carries the grouping instead of separate per-category tables, so this is a **single sheet with one table**, not one sheet per category.

Row granularity is unchanged from the Markdown format: **one row per affected file**, sharing one Finding ID when it is the same root cause, with its own Metric, Evidence, Observation, and Impact where those differ between instances.

#### Evidence column rules

Evidence is the cell that makes a finding checkable without opening the repo. It holds the literal proof, not a description of it.

- Start with the file path and, where useful, the file size: `src/pages/ApplicantDetails/ApplicantDetails.tsx` or `src/pages/.../IncomeAndDTI.tsx (1,086 lines)`.
- Then one line per piece of proof, each prefixed with its line number: `` L848 `const renderPersonalField = (key, label, value) => {` ``.
- Quote the code as it really reads in the file. Truncate a long line rather than rewriting it, and never reconstruct a line from memory.
- Close with the short factual statements that complete the proof: where the value is consumed, what is absent (`Neither is wrapped in useCallback.`), or a tool output (`ESLint also flags L132: "React.useMemo has a missing dependency: getFrozenColWidth".`).
- Use real newlines inside the cell (`\n` in the generator string). Do not use `<br>`, bullets, or Markdown lists.
- No prose analysis here. Why it matters belongs in `Observation`, the consequence belongs in `Impact`.

### Sheet 3: `Detailed Findings`

One block per **Finding ID**, not per row of `Key Findings`. Blocks appear in the same category order as sheet 2. Each block is a two-column layout with a bold Finding ID heading row, then labelled rows:

`Category`, `Functional module(s)`, `Files affected`, `Mapped criterion`, `Problem / Observation`, `Evidence`, `Why it is a problem`, `Recommended improvement`, `Benefits of fix`, `Severity`, `Remediation Effort`, `Priority`.

Leave one blank row between blocks. `Files affected` lists every row that ID has on sheet 2, in the same order. `Remediation Effort` appears here and only here, it is not a `Key Findings` column.

### Sheet 4: `Code Complexity` (conditional)

Header row, then one row per file actually measured, including the ones that produced no finding:

`Module Name` | `File / Location` | `Type` | `Complexity Metric` | `Classification` | `Finding ID`

`Classification` is `Acceptable` or `Code smell: <short label>`. A non-`Acceptable` row must carry a Finding ID that also appears on sheets 2 and 3. An `Acceptable` row uses `-`.

## Formatting

Apply with `openpyxl`. It is preinstalled, do not `pip install`.

- **Font**: Arial 10 throughout. Header row Arial 10 bold, white, on fill `1F4E78`.
- **Header row**: freeze panes at `A2` and turn on autofilter across the full header range on `Key Findings` and `Code Complexity`.
- **Wrap text** on every data cell, vertical alignment `top`. Without this the Evidence and Observation cells are unreadable.
- **Column widths** on `Key Findings`: A 18, B 20, C 42, D 12, E 26, F 60, G 60, H 10, I 45, J 55, K 8.
- **Row height**: leave it automatic. Do not set an explicit height, Excel sizes wrapped rows itself.
- **Severity fill** on column H: `Critical` `FFC7CE` with font `9C0006`; `High` `FFD9B3` with font `974706`; `Medium` `FFEB9C` with font `9C6500`; `Low` `D9D9D9` with font `3F3F3F`.
- **Priority fill** on column K: `P1` `FFC7CE`, `P2` `FFEB9C`, `P3` `D9D9D9`, same font colours as above, centered.
- **Borders**: thin, colour `BFBFBF`, on all four sides of every used cell in a table.
- Use the fixed `Evidence` font `Consolas 9` so quoted code lines stay aligned, the rest of the row stays Arial.
- No formulas are needed anywhere in this workbook, so no recalculation step is required. If a count cell is added, write it as a real formula (`=COUNTIF(...)`), never a hardcoded number, and recalculate before delivery.

## Generation recipe

Write the findings to a JSON file in the scratchpad first, then render the workbook from it. Building the data separately from the formatting keeps a long cell of quoted code from being mangled by shell escaping, and lets the workbook be regenerated after a validation fix without retyping the findings.

```python
# scratchpad/build_report.py
import json
from openpyxl import Workbook
from openpyxl.styles import Font, PatternFill, Alignment, Border, Side

COLUMNS = ["Category", "Module Name", "File / Location", "Type", "Metric",
           "Evidence", "Observation", "Severity", "Impact", "Recommendation", "Priority"]
WIDTHS  = [18, 20, 42, 12, 26, 60, 60, 10, 45, 55, 8]
SEV = {"Critical": ("FFC7CE", "9C0006"), "High": ("FFD9B3", "974706"),
       "Medium": ("FFEB9C", "9C6500"), "Low": ("D9D9D9", "3F3F3F")}
PRI = {"P1": ("FFC7CE", "9C0006"), "P2": ("FFEB9C", "9C6500"), "P3": ("D9D9D9", "3F3F3F")}

findings = json.load(open("findings.json", encoding="utf-8"))

wb = Workbook()
ws = wb.active
ws.title = "Key Findings"          # rename, do not create a second default sheet
ws.append(COLUMNS)
for f in findings:
    ws.append([f[c] for c in COLUMNS])

thin = Side(style="thin", color="BFBFBF")
border = Border(left=thin, right=thin, top=thin, bottom=thin)
for i, w in enumerate(WIDTHS, start=1):
    ws.column_dimensions[ws.cell(row=1, column=i).column_letter].width = w
for cell in ws[1]:
    cell.font = Font(name="Arial", size=10, bold=True, color="FFFFFF")
    cell.fill = PatternFill("solid", fgColor="1F4E78")
    cell.alignment = Alignment(wrap_text=True, vertical="center", horizontal="center")
    cell.border = border
for row in ws.iter_rows(min_row=2):
    for cell in row:
        cell.font = Font(name="Arial", size=10)
        cell.alignment = Alignment(wrap_text=True, vertical="top")
        cell.border = border
    row[5].font = Font(name="Consolas", size=9)             # Evidence
    fill, colour = SEV[row[7].value]                        # Severity
    row[7].fill = PatternFill("solid", fgColor=fill)
    row[7].font = Font(name="Arial", size=10, bold=True, color=colour)
    row[7].alignment = Alignment(wrap_text=True, vertical="top", horizontal="center")
    fill, colour = PRI[row[10].value]                       # Priority
    row[10].fill = PatternFill("solid", fgColor=fill)
    row[10].font = Font(name="Arial", size=10, bold=True, color=colour)
    row[10].alignment = Alignment(vertical="top", horizontal="center")

ws.freeze_panes = "A2"
ws.auto_filter.ref = ws.dimensions
# then build Summary, Detailed Findings, and (if applicable) Code Complexity sheets
wb.save("CORE_REVIEW_REPORT.xlsx")
```

Reopen the saved file with `load_workbook` and read a few cells back before handing it over. A workbook that saved without raising can still have the wrong header text or a shifted column.

## Sample data

Three real rows, shown field by field so the multi-line `Evidence` cell survives intact. Use these as the standard for depth and tone: Evidence quotes code, Observation explains the mechanism in plain language, Impact names the user-visible consequence, Recommendation names the concrete fix.

### Sample row 1

- **Category**: `React Best Practices`
- **Module Name**: `Applicant Portal`
- **File / Location**: `` `src/pages/ApplicantDetails/ApplicantDetails.tsx` (RCT-003) ``
- **Type**: `Component`
- **Metric**: `2 render functions passed as props, 0 useCallback`
- **Evidence**:
  ```
  src/pages/ApplicantDetails/ApplicantDetails.tsx:
  L848 `const renderPersonalField = (key: "firstName" | "lastName" | "email" | "mobile", label, value) => {`
  L879 `const renderAddressField = (key: "street" | "city" | "state" | "zip", label, value) => {`
  Both are passed to a child at L1106 `renderPersonalField={renderPersonalField}` and L1107 `renderAddressField={renderAddressField}`.
  Neither is wrapped in useCallback.
  ```
- **Observation**: `Two functions are defined inside the component body and passed down as props. Functions defined this way are brand new objects on every render, so the child component sees a "changed" prop every single time and re-renders, even when nothing it displays has actually changed. On a 1,381-line screen with 19 state values, that means these children re-render on every keystroke.`
- **Severity**: `Medium`
- **Impact**: `Both props get a new identity on every keystroke, so the child re-renders on every character typed into the applicant edit form.`
- **Recommendation**: `Wrap both in useCallback with the values they actually use as dependencies. Better still, move each into its own small component that takes the field data as props, so the render-prop pattern is not needed at all.`
- **Priority**: `P2`

### Sample row 2

- **Category**: `React Best Practices`
- **Module Name**: `Orders (Admin)`
- **File / Location**: `` `src/pages/LenderApplicationDetails/Components/IncomeAndDTI.tsx` (RCT-003) ``
- **Type**: `Component`
- **Metric**: `1 render function redefined inside a .map()`
- **Evidence**:
  ```
  src/pages/LenderApplicationDetails/Components/IncomeAndDTI.tsx (1,086 lines):
  L459 `const renderRow = (row: any, key: number) => (` - declared inside a .map() callback
  Invoked at L485 `{detailRows.map((row, i) => renderRow(row, i))}` and L489 `{overallRows.map((row, i) => renderRow(row, i))}`
  ```
- **Observation**: `renderRow is created inside a .map() loop, so a brand-new copy of the function is made for every group on every render. It also takes row: any, meaning TypeScript gives no help about what a row contains. The function never uses anything from the loop it sits in, so there is no reason for it to be there.`
- **Severity**: `Medium`
- **Impact**: `Income and DTI rows re-create their renderer on every render of a 1,087-line underwriting component.`
- **Recommendation**: `Move renderRow out of the map callback, ideally out of the component entirely, into its own small component (<IncomeRow row={row} />). Replace any with a real row type so mistakes are caught at compile time.`
- **Priority**: `P2`

### Sample row 3

- **Category**: `React Best Practices`
- **Module Name**: `Shared UI`
- **File / Location**: `` `src/components/common/SortableTable.tsx` (RCT-003) ``
- **Type**: `Component`
- **Metric**: `16 consumers, 0 memoized row components`
- **Evidence**:
  ```
  src/components/common/SortableTable.tsx (294 lines, 16 importers) - rows are rendered at lines 253-258:
  L253 `table.getRowModel().rows.map((row) => (`
  L254 `<TableRow`  L255 `key={row.id}`  L257 `onClick={() => handleRowClick(row)}`
  No memoized row component exists.
  ESLint also flags L132: "React.useMemo has a missing dependency: getFrozenColWidth".
  ```
- **Observation**: `Every row gets a brand-new onClick function on each render, and there is no memoized row component, so when anything in the table changes, every single row re-renders. This is the most-used component in the app (16 screens), so the cost is paid everywhere, and it is most noticeable on the long application and merchant lists.`
- **Severity**: `Medium`
- **Impact**: `Every table in the product re-renders all rows whenever any parent state changes, which is the single widest re-render surface in the app.`
- **Recommendation**: `Extract the row into its own component wrapped in React.memo, and pass a stable handler (useCallback) plus the row id rather than an inline arrow function. Measure a long list before and after with React DevTools Profiler so the improvement is evidenced. Also fix the missing dependency on line 132.`
- **Priority**: `P2`

Note what these three rows share: one Finding ID (`RCT-003`) for one root cause, three rows because three files are affected, each with its own Evidence, Observation, and Impact. That is the row-granularity rule in practice.

## Excel validation gate

Run this in place of section 1 (Markdown validation) of `validation-checklist.md`. Sections 2 to 5 of that checklist still apply in full to the content of every row, read them as applying to the `Key Findings` sheet rather than to a Markdown table.

### Workbook structure

- [ ] The file opens with `load_workbook` without warnings, and the sheet names and tab order are exactly `Summary`, `Key Findings`, `Detailed Findings`, and `Code Complexity` when that last one applies.
- [ ] No leftover default `Sheet` tab, and no empty sheet.
- [ ] `Key Findings` row 1 reads exactly `Category`, `Module Name`, `File / Location`, `Type`, `Metric`, `Evidence`, `Observation`, `Severity`, `Impact`, `Recommendation`, `Priority`, in that order, with no extra column to the right.
- [ ] Every data row has all 11 cells populated. No blank cell, no `None`, no placeholder text.
- [ ] Freeze panes and the autofilter are set on `Key Findings`.
- [ ] Wrap text and top alignment are set on every data cell, and column widths match the spec.
- [ ] Severity and Priority fills are applied and match the cell value, no row left unfilled.

### Content

- [ ] Every `Evidence` cell names a real file and quotes lines that were actually read from it during the review. Re-open at least one file per Finding ID and confirm the quoted line numbers still match.
- [ ] `Evidence` contains proof, not analysis. Nothing that belongs in `Observation` or `Impact` has leaked into it.
- [ ] Rows are grouped by Category in the fixed order, then by Priority (P1 first), then Severity within a category.
- [ ] Every Finding ID on `Key Findings` has a matching block on `Detailed Findings`, and that block's `Files affected` list matches every row carrying that ID, in the same order.
- [ ] Every Module Name used on `Key Findings` appears in the `Functional modules` row on `Summary`.
- [ ] The severity and category counts on `Summary` match an actual count of the `Key Findings` rows. Recount, do not trust the draft.
- [ ] No em dash appears in any cell. The writing is plain and direct, with no stock AI-report filler.
- [ ] `Priority` recomputes correctly from Severity x Effort per `rubrics.md`, using the effort recorded on `Detailed Findings`.

If any item fails, fix it and re-run the whole gate, structure and content both. A regeneration from the JSON can reintroduce a formatting problem that the previous pass fixed.
