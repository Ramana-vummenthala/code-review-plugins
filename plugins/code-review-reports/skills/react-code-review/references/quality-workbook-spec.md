# Code Quality Workbook Spec (`<NAME>_CODE_QUALITY_V1.xlsx`)

This is the default output spec for the skill. The report is one Excel workbook that a developer can open, filter, and work straight from. There is no summary sheet, no narrative writeup sheet, and no Markdown file. Every finding is one row, and every row is self-contained: it names the file, quotes the code, says what is wrong in plain words, and says what to do about it.

The sample rows near the end of this file are the reference for depth and wording. If anything here is ambiguous, match what they do.

Ratings still come from `rubrics.md` (Finding IDs, Module Name derivation, Severity, Remediation Effort, and the Severity x Effort to Priority table). This file defines the sheets, the 12 columns, the wording rules, the colours, and the build script. Where this file and `excel-report-spec.md` disagree, this file wins. `excel-report-spec.md` and `report-template.md` stay on disk for the older 11-column layout and the Markdown mode, and are only used when the user asks for them by name.

## Output file

- Default path: `<repo-path>/<REPO_NAME>_CODE_QUALITY_V1.xlsx`, where `<REPO_NAME>` is the repository or product name in caps with underscores (`ACME_PORTAL_UI`, `ACME_PORTAL`). Keep the `_V1` suffix, it is part of the name.
- `--output <path>` overrides the whole path.
- Never overwrite. If the file is already there, write `<REPO_NAME>_CODE_QUALITY_V1_<YYYY-MM-DD>.xlsx` next to it and say so when you hand it over.

## Workbook structure

One findings sheet per codebase reviewed, then the `Threshold Mapping` sheet last. Nothing else. No `Summary`, no `Detailed Findings`, no leftover `Sheet` tab.

| # | Sheet | Name it | Contents |
|---|---|---|---|
| 1..n | Findings | The codebase, short and readable: `Web UI`, `Orders API`, `Customer Portal UI`, `Customer Portal API` | The 12 column findings table, one row per affected file |
| last | `Threshold Mapping` | Literal name | The two threshold blocks, copied from this spec |

Most runs review one repository, so most workbooks have two sheets. If the user points the run at several repositories, give each its own findings sheet in the order the user listed them, and keep `Threshold Mapping` last.

## The findings sheet

Row 1 is the header. Row 2 onward is data. Exactly these 12 columns, in this order, with this header text:

| Col | Header | What goes in it |
|---|---|---|
| A | `Category` | The top level review area from the supplied criteria, for example `Code Quality`, `Security`, `Performance`. Usually one value for the whole sheet. |
| B | `Sub Category` | The criteria section the finding answers, for example `Code Organization Review`, `Code Complexity & Smells`, `Code Readability & Documentation`. Take the wording from the criteria document, do not invent your own. |
| C | `Module Name` | The functional module the file belongs to (`Orders (Admin)`, `Billing & Autopay`, `Payments`, `Login`) or a fallback label (`Cross-cutting`, `Shared UI`, `Build & Tooling`, `Application Shell`). Never a file name. See `rubrics.md`. |
| D | `File / Location` | Repo relative path in backticks, a line or line range when it helps, then the Finding ID in brackets: `` `src/services/OrdersService.ts` (ARCH-002) `` or `` `src/middlewares/error.ts:11-14` (ARCH-002) ``. |
| E | `Type` | One of `Component`, `Function`, `Module`, `Middleware`, `Route`, `Config`. Describes the code in column D, not the module in column C. |
| F | `Metric` | The number you actually measured. `8 files byte-identical across 3 services, 561 LOC where 187 would do`. Never prose, never a guess. |
| G | `Evidence` | The raw proof. Path, line number, the code line as it really reads. See the Evidence rules below. |
| H | `Observation` | What is wrong and why, in plain words. See the writing rules. |
| I | `Severity` | `Critical`, `High`, `Medium`, or `Low`, per `rubrics.md` and the Threshold Mapping sheet. |
| J | `Impact` | One sentence on what this costs the product or the team. Not a repeat of Observation. |
| K | `Recommendation` | What to do, concretely. Numbered steps when there is more than one. Name the file or folder to move code into, and point at a place in the repo that already does it right. |
| L | `Priority` | `P1`, `P2`, or `P3`, recomputed from Severity x Effort in `rubrics.md`. Never picked by feel. |

### Row granularity

One row per affected file. One Finding ID per root cause, repeated in column D on every row that shares that cause, each row carrying its own Metric, Evidence, Observation, and Impact. Do not merge three files into one row, and do not mint `ARCH-002`, `ARCH-003`, `ARCH-004` for the same pattern found in three places.

### Row order

Group by `Category`, then by `Sub Category` in the order the criteria document lists them, then by `Priority` (P1 first), then by `Severity` (Critical, High, Medium, Low). The sheet has an autofilter, so the reader can re-sort. Do not insert blank separator rows between groups.

### Evidence rules

Evidence is what lets a reader check the finding without opening the repo. It is proof, not analysis.

- Start with the path and line: `src/services/OrdersService.ts:7 -> ` then the quoted line.
- Quote the line exactly as it reads. Truncate a long line rather than tidying it up. Never type a line from memory.
- Use real newlines between proof lines (`\n` in the generator string). No `<br>`, no bullet characters, no Markdown lists.
- For a block of code, a short comment header line is fine: `// services/customer-service/src/utils/ApiError.ts - the whole file`.
- Close with the short facts that finish the proof: what is missing (`billingController.ts (400 lines) is never imported here.`), a count (`4 imports from 2 different page folders.`), or a tool message (`tsc reports 14 "Parameter implicitly has an 'any' type" errors in this file.`).
- Keep the reasoning out. Why it matters goes in Observation, what it costs goes in Impact.
- If you cannot quote a line, you have not verified the finding. Drop it.

### Type values

| Type | Use for |
|---|---|
| `Component` | A file or function that returns JSX and renders as an element. |
| `Function` | A hook, controller function, mapper, service method, or utility. Callable code, no JSX. |
| `Module` | A whole file or layer: a service, a store, a types file, a barrel file. |
| `Middleware` | Express or server middleware. |
| `Route` | A route file, router config, or endpoint definition. |
| `Config` | Build, lint, CI, tsconfig, env, or package manifest files. |

## Writing rules

The reader is a developer who joined last week. They know JavaScript, they do not know this codebase, and they have never read an audit report. Write for them.

- **No em dashes anywhere in the workbook.** Use a comma, a full stop, a colon, or brackets. A plain hyphen with spaces around it (` - `) is fine and is what the reference workbook uses.
- Short sentences. One idea per sentence.
- Say what the file is for before you say what is wrong with it. `This file is an API service, its job is to call the backend. But it borrows its types from a screen folder.`
- Explain the rule when you use a term. Not `this violates dependency inversion`, but `so the API layer depends on the screen layer, which is the wrong way round: screens should depend on services, never the reverse`.
- Use the second person for the fix. `Move the types into src/types/.` `Write the parameter type on each callback.`
- Give numbers, not adjectives. Not `this file is very large`, but `this file is 1,086 lines`.
- Point at a working example in the same repo whenever one exists. `The repo already does this for @repo/auth and @repo/db, so copy that setup.` That is the single most useful thing you can put in a Recommendation.
- No filler. Drop `it is important to note that`, `in today's fast paced`, `leverage`, `robust`, `seamless`, `best in class`. No hedging stacks like `it may potentially be possible that`.
- Do not use the word `we` for the codebase authors. Use `this file`, `this service`, `the component`.
- Aim for 2 to 4 sentences in Observation, 1 sentence in Impact, 1 to 4 sentences or numbered steps in Recommendation.

Quick check on tone, both say the same thing:

- Weak: `The service module exhibits an inverted dependency relationship with the presentation layer, violating Clean Architecture boundaries.`
- Good: `This file is an API service, its job is to call the backend. But it takes its TypeScript types and a block of fake demo data from a screen folder (pages/OrderList). So the API layer depends on the screen layer, which is the wrong way round.`

## Formatting

Everything below is what the reference workbook uses. Apply it with `openpyxl`, which is preinstalled. Do not `pip install`.

**Findings sheets**

- Font: Calibri 10 everywhere, including Evidence. No monospace font.
- Header row: Calibri 10 bold, white (`FFFFFF`), fill `1F3864`, wrapped, centered both ways.
- Data cells: Calibri 10 regular, vertical alignment `center` on every cell.
- Wrap text on the long text columns only: `File / Location`, `Metric`, `Evidence`, `Observation`, `Impact`, `Recommendation` (D, F, G, H, J, K), horizontally left aligned.
- The short columns (`Category`, `Sub Category`, `Module Name`, `Type`, `Severity`, `Priority` = A, B, C, E, I, L) are centered horizontally, no wrap.
- Column widths: A 22, B 30, C 26, D 46, E 14, F 34, G 78, H 62, I 11, J 62, K 54, L 10.
- Row height: leave automatic. Do not set one.
- Borders: thin on all four sides of every used cell, header and data.
- Freeze panes at `F2` on the first findings sheet so the reader keeps the first five columns and the header in view while reading Evidence. `A2` is acceptable on the others.
- Autofilter across the full used range, `A1:L<last row>`.
- Severity fill on column I: `Critical` `F4CCCC`, `High` `FCE4D6`, `Medium` `FFF2CC`, `Low` `E2EFDA`. Default font colour, not bold.
- Priority fill on column L: `P1` `F4CCCC`, `P2` `FFF2CC`, `P3` `E2EFDA`. Same palette, so severity and priority read as one colour scale.

**Threshold Mapping sheet**

- Turn gridlines off (`ws.sheet_view.showGridLines = False`).
- Block title row: Calibri 11 bold, font colour `1F3864`.
- Block note row underneath it: Calibri 9, font colour `595959`.
- Table header rows: same style as the findings header (white bold on `1F3864`, wrapped, centered).
- Table body: Calibri 10, wrapped, vertical center. Severity and Priority cells get the same fills as above.
- Column widths: A 32, B 30, C 38, D 26, E 34.
- Two blank rows between Block 1 and Block 2.

## Threshold Mapping sheet content

Copy this in as it reads. Update the count in the Block 1 note to the real number of findings in the workbook, and the sheet names to the sheets you actually built. If the criteria introduce a metric that is not listed, add a row for it in the same style rather than leaving the finding unexplained.

**Row 1 (block title):** `BLOCK 1  |  AUDIT METRIC THRESHOLDS AND TREATMENT`

**Row 2 (note):** `Every threshold below is the rule applied when scoring the <N> findings in the <sheet names> sheets.`

**Row 3 (header):** `Audit Metric` | `Recommended Threshold` | `Audit Treatment` | `Severity` | `Priority`

| Audit Metric | Recommended Threshold | Audit Treatment | Severity | Priority |
|---|---|---|---|---|
| Cyclomatic Complexity - Function / Method | >10 per function | Flag for review - extract named helper functions | Medium | P2 |
| Cyclomatic Complexity - Function / Method | >20 per function | High-priority finding - decomposition required before further change | High | P2 |
| Cyclomatic Complexity - Function / Method | >40 per function | Critical - the function cannot be meaningfully unit tested; decompose first | Critical | P1 |
| Cyclomatic Complexity - Module / File | >50 decision points | Flag for review - the module carries more than one responsibility | Medium | P2 |
| Cyclomatic Complexity - Module / File | >100 decision points | Critical - god module; split by responsibility before adding features | Critical | P2 |
| Nested Condition Depth | >= 3 levels | Flag for review | Medium | P3 |
| Nested Condition Depth | >= 4 levels | High-complexity finding - apply guard clauses and early returns | High | P2 |
| Nested Condition Depth | > 5 levels | Critical - extract sub-functions before any further change | Critical | P2 |
| Function Length | >50 LOC | Review for excessive responsibility | Medium | P3 |
| Function Length | >100 LOC | High-priority finding - the function owns several unrelated concerns | High | P2 |
| Component / Module Length | >300 LOC | Review for decomposition | Medium | P2 |
| Component / Module Length | >500 LOC | High maintainability concern | High | P2 |
| Component / Module Length | >1,000 LOC | Critical - blocks review, testing and parallel work on the same file | Critical | P2 |
| Function Parameter Count | >5 parameters | Flag for review - introduce a parameter object | Medium | P2 |
| Function Parameter Count | >8 parameters | High - positional call sites are silently error-prone | High | P2 |
| Component State Density (useState) | >10 per component | Flag for review - consolidate related state into useReducer | Medium | P2 |
| Component State Density (useState) | >20, or mutually exclusive boolean flags | High - illegal UI states are representable | High | P2 |
| Code Duplication | >20 identical lines across files | Flag for review - extract a shared helper | Medium | P2 |
| Code Duplication | >100 identical lines, or >60% file similarity | High - the copies will diverge, and in this codebase already have | High | P2 |
| Code Duplication | A business rule or defect duplicated across copies | Critical - a fix lands in one copy only and the others keep the defect | Critical | P1 |
| Dead Code (0 inbound references) | >50 LOC | Flag for deletion | Medium | P2 |
| Dead Code (0 inbound references) | >250 LOC | High - misleads readers and inflates the review surface | High | P2 |
| Commented-Out Code | >20 consecutive lines | Flag for deletion - version control already holds the history | Low | P3 |
| Commented-Out Code | A file with 0 live code lines | High - indistinguishable from live code at a glance | High | P2 |
| Type Safety (any / strict errors) | >20 `any` annotations in one file | Flag for review - type the exported surface first | Medium | P2 |
| Type Safety (any / strict errors) | >= 1 error under `strict` | High - the compiler is reporting a real defect that nothing gates | High | P2 |
| Type Safety (any / strict errors) | `strict` disabled, or an error on a live write path | Critical - incorrect data can reach the database | Critical | P1 |
| Lint Violations | >10 errors in one file | Flag for review | Medium | P2 |
| Lint Violations | Any `react-hooks/exhaustive-deps` violation | High - stale closures render values that are silently out of date | High | P2 |
| Lint Violations | Any `react-hooks/rules-of-hooks` violation | Critical - conditional hook order corrupts component state at runtime | Critical | P1 |
| Layering & Coupling | >= 3 cross-module imports | Flag for review - identify what is genuinely shared | Medium | P2 |
| Layering & Coupling | A deep import into another module's internals | High - internal file layout becomes an unversioned public contract | High | P2 |
| Layering & Coupling | An inverted dependency (shared -> feature, service -> page, util -> page) | Critical - the layer model is broken; the shared layer cannot be reused or tested alone | Critical | P1 |
| Quality Gate Coverage | 1 gate (lint / type-check / test / validation) missing from CI | Flag for review | Medium | P2 |
| Quality Gate Coverage | 0 gates run, or a gate present but bypassed | Critical - every other threshold on this sheet is unenforced | Critical | P1 |
| Documentation Coverage | <30% of the public surface documented | Flag for review | Medium | P2 |
| Documentation Coverage | Documentation contradicts the running system | Critical - onboarding and operations are actively misled | Critical | P1 |
| Naming / Convention Consistency | >1 convention in the same folder or layer | Hygiene finding - align on one documented convention | Low | P3 |

**Block 2 title row:** `BLOCK 2  |  SEVERITY BAND TABLE`

**Block 2 note row:** `The measured value alone places a finding in a band. Where a row reports several metrics, the worst band wins.`

**Block 2 header:** `Metric` | `Low / Acceptable` | `Medium` | `High` | `Critical / Very High`

| Metric | Low / Acceptable | Medium | High | Critical / Very High |
|---|---|---|---|---|
| Cyclomatic Complexity - Function / Method | 1-5 | 6-10 | 11-20 | >20 |
| Cyclomatic Complexity - Module / File (decision points) | <= 20 | 21-50 | 51-100 | >100 |
| Nested Condition Depth | 0-2 levels | 3 levels | 4-5 levels | >5 levels |
| Function Length | <= 30 LOC | 31-50 LOC | 51-100 LOC | >100 LOC |
| Component / Module Length | <= 200 LOC | 201-300 LOC | 301-500 LOC | >500 LOC |
| Function Parameter Count | <= 3 | 4-5 | 6-8 | >8 |
| Component State Density (useState per component) | <= 5 | 6-10 | 11-20 | >20, or mutually exclusive boolean flags |
| Code Duplication | <20 identical lines | 20-100 identical lines | >100 identical lines, or >60% file similarity | A business rule or defect duplicated across copies |
| Dead Code (0 inbound references) | <= 50 LOC | 51-250 LOC | >250 LOC | A dead near-duplicate of a live module |
| Commented-Out Code | <20 lines | 20-150 lines | >150 consecutive lines, or a file with 0 live code lines | Disabled logic on a live path (auth, payment, validation) |
| Type Safety (any / strict-mode errors) | 0 `any`, 0 errors | 1-20 `any`, 0 errors | >20 `any`, or >= 1 error under `strict` | `strict` disabled repo-wide, or a type error on a live write path |
| Lint Violations | 0 | 1-10 warnings | >10 errors, or any `exhaustive-deps` | `rules-of-hooks` or another correctness rule; or a linter configured to report nothing |
| Layering & Coupling | 0 cross-boundary imports | 1-2 imports within the same layer | >= 3 cross-module imports, or any deep import into another module's internals | An inverted dependency (shared -> feature, service -> page, util -> page) |
| Quality Gate Coverage | All gates enforced in CI | 1 gate missing | >1 gate missing | 0 gates run, or a gate present but bypassed |
| Documentation Coverage | >= 60% of the public surface | 30-59% | <30% | Documentation contradicts the running system |

## Sample rows

Match this depth. These are real rows from the reference workbook.

### Sample row 1

- **Category**: `Code Quality`
- **Sub Category**: `Code Organization Review`
- **Module Name**: `Orders (Merchant)`
- **File / Location**: `` `src/services/OrdersService.ts` (ARCH-002) ``
- **Type**: `Module`
- **Metric**: `2 imports from a page folder`
- **Evidence**:
  ```
  src/services/OrdersService.ts:7 -> `} from "@/pages/OrderList/types";`
  src/services/OrdersService.ts:8 -> `import { orderListMockData } from "@/pages/OrderList/data";` both imports still present (file is 259 lines).
  ```
- **Observation**: `This file is an API service - its job is to call the backend. But it borrows its TypeScript types and a block of fake demo data from a screen folder (pages/OrderList). So the API layer depends on the screen layer, which is the wrong way round: screens should depend on services, never the reverse.`
- **Severity**: `High`
- **Impact**: `Mock fixtures are pulled into the production bundle through the service, and the page cannot be moved or renamed without breaking the API layer.`
- **Recommendation**: `1) Move the request/response types into src/types/. 2) Move orderListMockData into src/services/mocks/ (that folder already exists). 3) Change this service and the OrderList page to both import from the new locations.`
- **Priority**: `P1`

### Sample row 2

- **Category**: `Code Quality`
- **Sub Category**: `Code Organization Review`
- **Module Name**: `Billing & Autopay`
- **File / Location**: `` `src/pages/Billing/Features/PaymentMethodForm.tsx:19` (ARCH-002) ``
- **Type**: `Component`
- **Metric**: `1 direct service import, 0 controller indirection; component 531 lines`
- **Evidence**:
  ```
  PaymentMethodForm.tsx:19
  import { paymentService } from "@/services/paymentService"

  PaymentMethodForm.tsx:386
        await paymentService.processPayment({

  File is 531 lines. src/stores/billingController.ts (400 lines) is never imported here.
  ```
- **Observation**: `This screen calls the payment API straight from the button-click code inside the component. The project already has a "controller" file for Billing & Autopay (billingController.ts) whose job is to hold that kind of logic, but this component skips it and talks to the service itself.`
- **Severity**: `High`
- **Impact**: `Payment orchestration logic that the controller owns gets re-implemented in the component, so the two can drift and a fix applied in one is missed in the other.`
- **Recommendation**: `Move the paymentService.processPayment(...) call out of the component and into billingController.ts. The component should then call one controller function instead. Use the Invoices screen as the example to copy: it calls invoicesController.ts and never imports the service directly.`
- **Priority**: `P1`

### Sample row 3

- **Category**: `Code Quality`
- **Sub Category**: `Code Complexity & Smells`
- **Module Name**: `Payments`
- **File / Location**: `` `src/services/payment.service.ts` (CQ-004) ``
- **Type**: `Module`
- **Metric**: `` 50 of the repo's 155 `: any` annotations sit in this one file; 2 `as any` casts ``
- **Evidence**:
  ```
  src/services/payment.service.ts:2586
    const resolvePaymentAccountForBatch = async (payment: any) => {
  src/services/payment.service.ts:2635
    const processSinglePayment = async (payment: any): Promise<BatchPaymentResult> => {
  src/services/payment.service.ts:1664
    CheckNumber: (validatedPaymentMethod as any).checkNumber,

  Counted across the repository: 155 `: any` markers in total, 50 of them in this file.
  ```
- **Observation**: `TypeScript can catch mistakes for you, but only if you tell it what shape your data is. This file gives up on that almost everywhere: it marks payment rows, gateway replies and request payloads as any, which means "stop checking this". Fifty of the repo's 155 any markers are in this single file, including on functions that move money, such as processSinglePayment(payment: any).`
- **Severity**: `Medium`
- **Impact**: `A field renamed in the Prisma schema or in a gateway response shape compiles cleanly and fails at runtime inside a money-moving path instead of at build time.`
- **Recommendation**: `Add real types on the money paths first. Prisma already generates a type for every table, so use Prisma.PaymentGetPayload instead of any for database rows. For the outside services (the payment gateway, the tokenization provider, the settlement feed), write a small interface describing the reply you actually read. You do not have to fix the whole file at once, start with the functions that charge or refund a customer.`
- **Priority**: `P2`

Note what rows 1 and 2 share: one Finding ID for one root cause, two rows because two files are affected, each with its own Evidence, Observation, and Impact.

## Generation recipe

Two passes. Write the findings to JSON in the scratchpad first, then render the workbook from that JSON. Keeping data and formatting apart stops a multi-line Evidence cell from being mangled by shell escaping, and lets you rebuild the workbook after a validation fix without retyping anything.

**Pass 1**, write `findings.json` in the scratchpad:

```json
[
  {
    "sheet": "Web UI",
    "Category": "Code Quality",
    "Sub Category": "Code Organization Review",
    "Module Name": "Orders (Merchant)",
    "File / Location": "`src/services/OrdersService.ts` (ARCH-002)",
    "Type": "Module",
    "Metric": "2 imports from a page folder",
    "Evidence": "src/services/OrdersService.ts:7 -> `} from \"@/pages/OrderList/types\";`\nsrc/services/OrdersService.ts:8 -> `import { orderListMockData } from \"@/pages/OrderList/data\";` both imports still present (file is 259 lines).",
    "Observation": "This file is an API service - its job is to call the backend. ...",
    "Severity": "High",
    "Impact": "Mock fixtures are pulled into the production bundle through the service, ...",
    "Recommendation": "1) Move the request/response types into src/types/. ...",
    "Priority": "P1",
    "_effort": "Medium"
  }
]
```

`_effort` is not a column. Keep it in the JSON so the validator can recompute Priority from the `rubrics.md` table, and drop it when writing the row.

**Pass 2**, render with this script:

```python
# scratchpad/build_quality_workbook.py
import json
from openpyxl import Workbook
from openpyxl.styles import Font, PatternFill, Alignment, Border, Side

COLUMNS = ["Category", "Sub Category", "Module Name", "File / Location", "Type", "Metric",
           "Evidence", "Observation", "Severity", "Impact", "Recommendation", "Priority"]
WIDTHS  = [22, 30, 26, 46, 14, 34, 78, 62, 11, 62, 54, 10]
WRAP_COLS = {4, 6, 7, 8, 10, 11}          # 1-based: D, F, G, H, J, K
BAND = {"Critical": "F4CCCC", "High": "FCE4D6", "Medium": "FFF2CC", "Low": "E2EFDA",
        "P1": "F4CCCC", "P2": "FFF2CC", "P3": "E2EFDA"}
HEADER_FILL = PatternFill("solid", fgColor="1F3864")
HEADER_FONT = Font(name="Calibri", size=10, bold=True, color="FFFFFF")
BODY_FONT   = Font(name="Calibri", size=10)
thin = Side(style="thin", color="000000")
BORDER = Border(left=thin, right=thin, top=thin, bottom=thin)

findings = json.load(open("findings.json", encoding="utf-8"))
sheets = []
for f in findings:                                  # keep first-seen order
    if f["sheet"] not in sheets:
        sheets.append(f["sheet"])

wb = Workbook()
wb.remove(wb.active)                                # no leftover default sheet

def style_header(ws, ncols):
    for c in range(1, ncols + 1):
        cell = ws.cell(row=1, column=c)
        cell.font, cell.fill, cell.border = HEADER_FONT, HEADER_FILL, BORDER
        cell.alignment = Alignment(wrap_text=True, vertical="center", horizontal="center")

for name in sheets:
    ws = wb.create_sheet(title=name)
    ws.append(COLUMNS)
    for f in (x for x in findings if x["sheet"] == name):
        ws.append([f[c] for c in COLUMNS])
    for i, w in enumerate(WIDTHS, start=1):
        ws.column_dimensions[ws.cell(row=1, column=i).column_letter].width = w
    style_header(ws, len(COLUMNS))
    for row in ws.iter_rows(min_row=2):
        for idx, cell in enumerate(row, start=1):
            cell.font, cell.border = BODY_FONT, BORDER
            if idx in WRAP_COLS:
                cell.alignment = Alignment(wrap_text=True, vertical="center", horizontal="left")
            else:
                cell.alignment = Alignment(vertical="center", horizontal="center")
        for idx in (9, 12):                          # Severity, Priority
            row[idx - 1].fill = PatternFill("solid", fgColor=BAND[row[idx - 1].value])
    ws.freeze_panes = "F2"
    ws.auto_filter.ref = "A1:L%d" % ws.max_row

# Threshold Mapping sheet. BLOCK1_ROWS and BLOCK2_ROWS are lists of 5-item lists,
# transcribed from the two tables in this spec file, in the same order.
tm = wb.create_sheet(title="Threshold Mapping")
tm.sheet_view.showGridLines = False
for col, w in zip("ABCDE", [32, 30, 38, 26, 34]):
    tm.column_dimensions[col].width = w

def write_block(ws, start, title, note, header, rows):
    ws.cell(row=start, column=1, value=title).font = Font(name="Calibri", size=11, bold=True, color="1F3864")
    ws.cell(row=start + 1, column=1, value=note).font = Font(name="Calibri", size=9, color="595959")
    hr = start + 2
    for c, text in enumerate(header, start=1):
        cell = ws.cell(row=hr, column=c, value=text)
        cell.font, cell.fill, cell.border = HEADER_FONT, HEADER_FILL, BORDER
        cell.alignment = Alignment(wrap_text=True, vertical="center", horizontal="center")
    last = hr
    for r, data in enumerate(rows, start=hr + 1):
        for c, text in enumerate(data, start=1):
            cell = ws.cell(row=r, column=c, value=text)
            cell.font, cell.border = BODY_FONT, BORDER
            cell.alignment = Alignment(wrap_text=True, vertical="center",
                                       horizontal="center" if text in BAND else None)
            if text in BAND:
                cell.fill = PatternFill("solid", fgColor=BAND[text])
        last = r
    return last + 1

n = len(findings)
end = write_block(tm, 1, "BLOCK 1  |  AUDIT METRIC THRESHOLDS AND TREATMENT",
    "Every threshold below is the rule applied when scoring the %d findings in the %s sheets."
        % (n, ", ".join(sheets)),
    ["Audit Metric", "Recommended Threshold", "Audit Treatment", "Severity", "Priority"],
    BLOCK1_ROWS)
write_block(tm, end + 2, "BLOCK 2  |  SEVERITY BAND TABLE",
    "The measured value alone places a finding in a band. Where a row reports several metrics, the worst band wins.",
    ["Metric", "Low / Acceptable", "Medium", "High", "Critical / Very High"],
    BLOCK2_ROWS)

wb.save("ACME_PORTAL_CODE_QUALITY_V1.xlsx")
```

Then reopen the saved file with `load_workbook` and read a few cells back. A workbook that saved without an error can still have a shifted column or a header typo.

## Validation gate

Run all of it before showing the file to the user. Fix, regenerate from the JSON, then run the whole gate again, because a regeneration can undo a formatting fix.

### Structure (mechanical, script it)

- [ ] The file opens with `load_workbook` and no warning.
- [ ] Sheet names and tab order match the plan, `Threshold Mapping` is last, there is no default `Sheet` tab and no empty sheet.
- [ ] Row 1 of every findings sheet reads exactly `Category`, `Sub Category`, `Module Name`, `File / Location`, `Type`, `Metric`, `Evidence`, `Observation`, `Severity`, `Impact`, `Recommendation`, `Priority`, with nothing in column M.
- [ ] Every data cell is populated. No `None`, no empty string, no `TBD`.
- [ ] `Type` is one of the six allowed values in every row, `Severity` is one of four, `Priority` is one of three.
- [ ] Severity and Priority fills are set on every data row and match the cell value.
- [ ] Freeze panes and the autofilter are set on every findings sheet, and the autofilter range ends at the last data row.
- [ ] Column widths match the spec, wrap is on for D, F, G, H, J, K and off for the rest.
- [ ] No cell anywhere in the workbook contains the em dash character. Assert this in code across every sheet, do not eyeball it.

### Content (read it, do not script it)

- [ ] Every Evidence cell names a real file and quotes lines that were actually read. Re-open at least one file per Finding ID and confirm the line numbers still match.
- [ ] Evidence holds proof only. Nothing that belongs in Observation or Impact has leaked into it.
- [ ] Metric holds a real measurement, not prose.
- [ ] Impact is not a restatement of Observation.
- [ ] Recommendation says what to do and where, and names an existing in-repo example when one exists.
- [ ] Rows are grouped by Category, then Sub Category, then Priority, then Severity.
- [ ] A Finding ID that appears on more than one row is the same root cause every time, and no two IDs describe the same pattern.
- [ ] Severity matches the band table on the Threshold Mapping sheet for the measurement in Metric, and Priority recomputes from Severity x `_effort` per `rubrics.md`.
- [ ] Every Sub Category value comes from the supplied criteria document.
- [ ] Read three Observation cells out loud. If a developer who joined last week would need to look something up, rewrite it.
- [ ] No filler phrases, no `leverage`, no `robust`, no hedging stacks.
