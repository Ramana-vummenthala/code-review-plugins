---
name: react-code-review
description: Audit a React/Next.js repository against a supplied set of Review Criteria and produce an evidence-based code quality workbook in Excel - one findings sheet per codebase with 12 columns (Category, Sub Category, Module Name, File / Location, Type, Metric, Evidence, Observation, Severity, Impact, Recommendation, Priority), one row per affected file, plus a Threshold Mapping sheet that states the rule behind every severity. Grounded in React/Vercel performance best practices. Use whenever the user asks for a React code review, code audit, architecture/quality review, or a findings report for a React codebase - invoke as `/react-code-review <criteria> <repo-path>`, or trigger conversationally when the user describes wanting this kind of structured review.
license: MIT
metadata:
  author: tguc
  version: "3.0.0"
---

# React Code Review

Produces a code quality workbook: a structured, evidence-based audit of a React codebase, scored against a specific set of Review Criteria supplied at invocation time, and delivered as an **Excel workbook** (`<REPO_NAME>_CODE_QUALITY_V1.xlsx`). The point of this skill is that every finding is traceable to real code, so never write a finding you haven't verified against the actual repository. The workbook's `Evidence` column exists to make that traceability visible to the reader: it carries the quoted lines, not a description of them.

The workbook layout is defined by `references/quality-workbook-spec.md`: a 12-column findings sheet per codebase, then a `Threshold Mapping` sheet. That file is the default output spec for every run.

Two older output modes stay available, and only when the user asks for them by name:
- `--layout detailed` builds the older 11-column workbook with `Summary` / `Key Findings` / `Detailed Findings` sheets, per `references/excel-report-spec.md`.
- `--format md` (or an `--output` path ending in `.md`) builds the Markdown report, per `references/report-template.md`.

## Why this exists

Generic React advice is cheap and mostly useless to a team that already knows React. What's valuable is a report that says exactly *where* in *this* repository a pattern shows up, quotes the real code, and rates the fix the same way every time it's run. The steps below exist to enforce that discipline - skipping straight to writing findings from general knowledge defeats the purpose.

## Step 1 - Resolve the two required inputs

This skill needs exactly two things: **Review Criteria** and a **Repository** to review. Parse them out of the invocation's arguments:

- Accept explicit flags: `--criteria <path-or-text>` and `--repo <path>`.
- Or two bare positional tokens: `<criteria> <repo-path>` - the token that resolves to an existing directory is the repo; the other is the criteria (a path to an existing file, or inline text if it isn't a path).
- An `--output <path>` flag may override where the report gets written (default is described in Step 6).
- A `--format <xlsx|md>` flag may override the output format. The default is `xlsx`. An `--output` path ending in `.md` implies `--format md`.
- A `--layout <quality|detailed>` flag may override the workbook layout. The default is `quality` (`references/quality-workbook-spec.md`). Use `detailed` only when the user asks for the older Summary / Key Findings / Detailed Findings workbook.

If criteria was given as a file path, read the file - don't paraphrase it from the filename. If the repo path doesn't exist, or either input is genuinely missing, **stop and ask the user** rather than substituting "general React best practices" as a stand-in for criteria, or defaulting to the current directory as the repo. The whole value of this report is that it's criteria-driven, not a generic scan; guessing the criteria defeats that.

Sanity-check that the target actually looks like a React project: read its `package.json` and confirm `react` is a dependency. If it isn't, tell the user what you found and confirm before proceeding - don't silently produce a "React" report for a non-React repo.

## Step 2 - Load the React-specific references

This skill grounds its findings in two complementary reference sources. Load both - the second selectively, per the table below - rather than writing findings from general React intuition.

**1. Performance rules - `vercel-react-best-practices`.** Read `~/.claude/skills/vercel-react-best-practices/SKILL.md` (installed globally, Vercel Engineering's 70-rule React/Next.js performance guide across 8 categories: waterfalls, bundle size, server-side performance, client-side data fetching, re-renders, rendering, JS performance, advanced patterns). When a finding matches one of its rules, pull the specific rule file from its `rules/` directory (e.g. `rules/rerender-no-inline-components.md`) and cite the rule ID in the finding - this is what `RCT-*` findings should be grounded in, not vague performance intuition.

**2. Coding standards - `react-coding-standards-best-practices` (Cognine plugin).** Installed as a plugin under a version-pinned cache directory: list `~/.claude/plugins/cache/cognine-plugin-marketplace/react-coding-standards-best-practices/` and use whichever version subdirectory is present (don't hardcode a version number - it will bump over time). It bundles 9 topic skills (`skills/<topic>/SKILL.md`) and 3 audit-command files (`commands/*.md`) that define numbered check IDs and a severity taxonomy for 4 of those topics. Load **only** the topic skill(s) relevant to the supplied Review Criteria (the same tailoring principle Step 3 uses for its Explore agents) - not all 9 unconditionally:

| Prefix | Cognine skill | Numbered checks (if any) |
|---|---|---|
| `ARCH` | `skills/react-architecture/SKILL.md` | `commands/react-coding-architecture-audit-command.md` - BLOCK SOL/DI/SOC/CA/DDD/MOD, classified Mandatory vs Recommended |
| `CQ` | `skills/react-engineering-standards/SKILL.md` | - |
| `SEC` | `skills/react-security/SKILL.md` | `commands/react-secure-coding-practices-command.md` - `SEC-001`..`SEC-010` |
| `PERF` | `skills/react-performance/SKILL.md` | same command - `PERF-001`..`PERF-011` |
| `DATA` | `skills/react-api-standards/SKILL.md` | - |
| `STATE` | `skills/react-state-mgmt/SKILL.md` | same command - `STATE-001`..`STATE-009` |
| `ERR` | `skills/react-error-handling/SKILL.md` | same command - `ERR-001`..`ERR-010` |
| `A11Y` | `skills/react-accessibility/SKILL.md` | `commands/react-coding-accessibility-audit-a11y.md` - P-xx/O-xx/U-xx/R-xx tagged with a WCAG 2.2 success-criterion number |
| `DEP` | `skills/react-dependency-governance/SKILL.md` | - |

When a finding matches a topic with numbered checks, cite the specific check ID (e.g. `SEC-004`, `PERF-002`, `WCAG O-03`) the same way `RCT-*` findings cite a Vercel rule ID - this is what makes the finding standards-grounded rather than generic advice. For `ARCH` findings specifically, also state whether the violated principle is Mandatory or Recommended per the Cognine Architecture Principles Matrix, and let that inform Severity: a Mandatory violation should not be rated below Medium.

**Do not invoke the 3 Cognine audit-commands as sub-processes.** They are separate slash-commands with their own report files (`architecture-audit-report.md`, `coding-practices-review-report.md`) and would fragment output across multiple reports; use only their skill docs and numbered-check-ID content as citation material inside this skill's single code quality workbook.

If either reference is not present in this environment (check paths first), do not fail silently: say so plainly in the handover message to the user and fall back to well-established, citable practices for that area instead of fabricating rule or check IDs.

## Step 3 - Explore the repository against the criteria

If the target repo has a root `CLAUDE.md`, read it first - it may already document the architecture and save you from re-deriving it.

Launch up to 3 parallel `Explore` agents to cover the repo's surface area, tailored to what the supplied criteria actually ask about (don't run all three if the criteria only concern one area):

1. **Architecture & structure** - routing, project layout, module boundaries, how the app is composed.
2. **State & data** - state management approach, data fetching, API/service layer, hooks.
3. **Component design & quality** - component patterns, rendering behavior, styling approach, error handling, and testing conventions.

Alongside the criteria work, establish the repository's **functional module map** (Login, Dashboard, Profile, Checkout, ...) early, because every findings row has to name one. Derive it from routes, feature folders, and the primary navigation structure per the Module Name section of `references/rubrics.md`, decide which fallback label (Cross-cutting, Shared UI, Build & Tooling, Application Shell) covers the infrastructure code, and keep that list stable for the whole run, so the same file always maps to the same module. The architecture agent is the natural place to ask for this: have it return a mapping of module name to the paths it covers, not just a folder listing.

Give each agent the actual list of Review Criteria (not a summary) and ask it to report back **findings already tied to file paths and code**, mapped to whichever criterion each observation relates to - not generic prose about what it saw. An agent that returns "state management could be improved" without a file reference is not useful input to this report; push back and re-run narrower if that happens.

## Step 4 - Synthesize findings

For each distinct issue:

- Assign a Finding ID using the prefix table in `references/rubrics.md` (`ARCH-001`, `PERF-002`, etc.) - one ID per root cause, not per affected file.
- If the same root cause recurs across multiple files, don't merge it into one row: give it one Finding ID, but produce one findings row per affected file (each embedding that same ID in its File / Location cell), with its own Metric/Observation/Impact where those differ between instances - don't create a *new* Finding ID for the same underlying pattern repeated across files, and don't collapse the instances into a single row either. Rows can and often will repeat the same Module Name when those files sit in the same functional module.
- Assign each row its **Module Name**: the functional module the file belongs to, taken from the module map built in Step 3, never the file name. The file path goes in the separate File / Location column, with the Finding ID after it. If the file belongs to no single feature, use a fallback label (Cross-cutting, Shared UI, Build & Tooling, Application Shell) per `references/rubrics.md`.
- Give each row its **Sub Category**: the section heading of the supplied criteria document that the finding answers (`Code Organization Review`, `Code Complexity & Smells`, `Code Readability & Documentation`, and so on). Use the criteria document's own wording, don't invent a label. Column `Category` stays the top-level review area (`Code Quality`, `Security`, `Performance`).
- Classify each row's Type and Metric per `references/rubrics.md` - Type describes the code in File / Location, not the functional module. The workbook accepts six Type values: `Component`, `Function`, `Module`, `Middleware`, `Route`, `Config` (the last three extend the three in `rubrics.md`, for backend and config files).
- Capture the row's **Evidence** while the file is open, not afterwards: the file path, then one line per proof point prefixed with its line number and the code quoted as it actually reads, then the short factual statements that close the proof (where the value is consumed, what is absent, an ESLint or tsc message). Evidence is proof only, the reasoning goes in Observation and the consequence in Impact. See the Evidence rules in `references/quality-workbook-spec.md`. A finding with no quotable evidence is a finding that hasn't been verified yet, so don't carry it forward.
- Rate Severity (checked against the Metric → Severity threshold table in `references/rubrics.md`), Remediation Effort, and Priority - Priority is derived mechanically from Severity × Effort there, not chosen independently.
- Map the finding to the specific supplied criterion it addresses.

Track two things explicitly as you go. The quality workbook has no summary sheet, so these are stated in the handover message when you give the user the file, never silently dropped:
- Which supplied criteria produced **no findings**, so the user can see they were evaluated and passed rather than skipped.
- Which supplied criteria **could not be evaluated** at all from what's in the repository (e.g. a criterion about CI-enforced test coverage when there's no CI config), each with the concrete reason. Never guess at one of these.

## Step 5 - Verify every finding before it goes in the report

Before a finding is written into the draft, re-open the cited file and confirm the pattern is really there - quote or closely paraphrase the actual lines, don't reconstruct them from memory of the exploration summary. Confirm the recommendation actually addresses what's shown, not an adjacent concern. This mirrors how this environment's own `code-review` skill treats findings: generate candidates, then adversarially verify each one against the real code before it's allowed to survive into the output. Drop or downgrade anything that doesn't hold up under this check - a report with fewer, verified findings is more valuable than one padded with plausible-sounding guesses.

## Step 6 - Build the workbook

Follow `references/quality-workbook-spec.md` exactly: one findings sheet per codebase named after that codebase, then a `Threshold Mapping` sheet last, the literal 12-column header (Category, Sub Category, Module Name, File / Location, Type, Metric, Evidence, Observation, Severity, Impact, Recommendation, Priority), the Calibri formatting and fill colours, and the generation recipe. That spec also carries three worked sample rows - match their depth, especially in the Evidence, Observation and Recommendation cells.

Build it in two passes, as the spec's recipe describes: write the findings to a JSON file in the scratchpad, then render the workbook from that JSON with `openpyxl` (preinstalled, don't `pip install`). Keeping the data separate from the formatting is what stops a multi-line cell of quoted code from being mangled by shell escaping, and it lets you regenerate the workbook after a validation fix without retyping the findings. Keep each finding's Remediation Effort in the JSON as `_effort` so Priority stays recomputable; it is not a column.

Row granularity is unchanged: one row per affected file, not per root cause, with the shared Finding ID embedded in the File / Location cell. Sort rows by Category, then Sub Category (criteria-document order), then Priority (P1 first), then Severity. The `Threshold Mapping` sheet is what makes each Severity checkable, so transcribe both of its blocks from the spec and update the finding count in the Block 1 note.

**Writing style**: this is the part that decides whether the workbook gets used or filed away. Write every cell the way a developer would explain the problem to the teammate sitting next to them, for a reader who joined the team last week: short sentences, plain English, no jargon left unexplained, numbers instead of adjectives, and a concrete next step that names a file or folder. Say what the file is for before saying what is wrong with it, and point at a place in the same repo that already does it the right way whenever one exists. Never use an em dash (Unicode U+2014) anywhere in the workbook; use a comma, a period, a colon, or a plain hyphen with spaces (` - `). Avoid stock AI-report filler ("it is important to note that," "in today's fast-paced development environment," "leverage," "robust," excessive hedging) and just state what was found. The full writing rules, with a weak-versus-good example, are in the Writing rules section of the spec.

Write the file to `<repo-path>/<REPO_NAME>_CODE_QUALITY_V1.xlsx` (repository or product name in caps with underscores, for example `ACME_PORTAL_UI_CODE_QUALITY_V1.xlsx`) unless the user passed `--output <path>`. If that file already exists, write `<REPO_NAME>_CODE_QUALITY_V1_<YYYY-MM-DD>.xlsx` alongside it instead of overwriting. Tell the user the path you wrote to.

If the user asked for `--layout detailed`, follow `references/excel-report-spec.md` instead (the 11-column `Key Findings` sheet plus `Summary`, `Detailed Findings`, and a conditional `Code Complexity` sheet). If the run is in Markdown mode (`--format md`), follow `references/report-template.md` instead, unchanged: category-grouped 10-column tables (no Evidence column, the quoted code lives in each finding's Evidence code block) and the `## Code Complexity Summary` section in place of the fourth sheet.

## Step 7 - Run the pre-delivery validation gate

Before showing anything to the user, run the **validation gate** at the end of `references/quality-workbook-spec.md` (structure and content), plus sections 2 to 5 of `references/validation-checklist.md` read as applying to the findings sheet rather than to a Markdown table. Section 1 of that checklist is Markdown-specific and is replaced by the structure half of the workbook gate - apply section 1 only in `--format md` mode, and use the gate in `references/excel-report-spec.md` only for `--layout detailed`. Part of the gate is mechanical: reopen the saved file with `load_workbook`, assert the header row matches the 12 columns exactly, assert no data cell is empty, and assert no cell in any sheet contains an em dash. If any item fails, fix it, regenerate the workbook from the JSON, and **re-run the full gate again** - a fix in one place can break consistency elsewhere (e.g. correcting a Priority value needs re-checking the row is still sorted correctly, and a regeneration can undo a formatting fix). Only present the report once every item passes. If you had to fix something, mention it briefly when handing the report over - it's a useful signal the gate actually did something, not just theater.

## Reference files

- `references/rubrics.md` - Finding-ID prefixes (including which are grounded in the Cognine `react-coding-standards-best-practices` plugin), how to derive the functional Module Name list and its fallback labels, Type and Metric definitions (including the Metric → Severity thresholds), and the Severity / Remediation Effort / Priority definitions (Priority is derived from the other two - always recompute, never assign by feel).
- `references/quality-workbook-spec.md` - **the default output spec**: the findings sheet per codebase plus the `Threshold Mapping` sheet, the literal 12-column header, the Evidence rules, the writing rules that keep every cell plain and developer-readable, the formatting and fill colours, the `openpyxl` generation recipe, three worked sample rows, and the Step 7 gate.
- `references/excel-report-spec.md` - the older 11-column workbook (`Summary` / `Key Findings` / `Detailed Findings` / `Code Complexity`), used only for `--layout detailed`.
- `references/report-template.md` - the Markdown output structure for `--format md`, including a worked example of one fully-written finding, category-grouped.
- `references/validation-checklist.md` - the content half of the Step 7 gate (sections 2 to 5 apply to both formats; section 1 is Markdown-only), transcribed as pass/fail items.
