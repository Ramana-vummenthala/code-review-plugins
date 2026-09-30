---
name: node-code-review
description: Audit a Node.js/TypeScript backend repository (Express, Fastify, NestJS, Koa, Hapi, or plain Node services) against a supplied set of Review Criteria and produce an evidence-based "Core Review Report" as an Excel workbook (.xlsx) — one findings sheet per reviewed service with 12 columns (Category/Sub Category/Module Name/File/Type/Metric/Evidence/Observation/Severity/Impact/Recommendation/Priority, where Module Name is the functional module such as Authentication or Orders, one row per affected file), plus a Threshold Mapping sheet, grounded in the Node.js Best Practices list (NBP) and OWASP. Use whenever the user asks for a Node/backend/API code review, code audit, architecture/quality review, or a findings report for a Node.js codebase — invoke as `/node-code-review <criteria> <repo-path>`, or trigger conversationally when the user describes wanting this kind of structured review.
license: MIT
metadata:
  author: tguc
  version: "2.0.0"
---

# Node.js Code Review

Produces a "Core Review Report": a structured, evidence-based audit of a Node.js backend codebase, scored against a specific set of Review Criteria supplied at invocation time and delivered as an Excel workbook. The point of this skill is that every finding is traceable to real code — never write a finding you haven't verified against the actual repository.

This is the backend counterpart to the `react-code-review` skill: same Module Name / File / Location split, same rubrics structure, same evidence discipline — different subject matter, different grounding references, and an `.xlsx` deliverable instead of a Markdown one.

## Why this exists

Generic Node advice ("use async/await", "validate input") is cheap and useless to a team that already ships Node services. What's valuable is a report that says exactly *where* in *this* repository a pattern shows up, quotes the real code, cites the specific practice it violates, and rates the fix the same way every time it's run. The steps below exist to enforce that discipline — skipping straight to writing findings from general knowledge defeats the purpose.

Backend reviews also carry a risk a frontend review doesn't: a missed finding here is a data breach, silent data corruption, or a 3am page, not a slow render. Weight severity accordingly (see `references/rubrics.md`).

## Step 1 — Resolve the two required inputs

This skill needs exactly two things: **Review Criteria** and a **Repository** to review. Parse them out of the invocation's arguments:

- Accept explicit flags: `--criteria <path-or-text>` and `--repo <path>`.
- Or two bare positional tokens: `<criteria> <repo-path>` — the token that resolves to an existing directory is the repo; the other is the criteria (a path to an existing file, or inline text if it isn't a path).
- An `--output <path>` flag may override where the workbook gets written (default is described in Step 6). The output is always an `.xlsx` file; if the user passes a `.md` path, tell them the deliverable is a workbook now and write the `.xlsx` alongside it.

If criteria was given as a file path, read the file — don't paraphrase it from the filename. If the repo path doesn't exist, or either input is genuinely missing, **stop and ask the user** rather than substituting "general Node best practices" as a stand-in for criteria, or defaulting to the current directory as the repo. The whole value of this report is that it's criteria-driven, not a generic scan; guessing the criteria defeats that.

Sanity-check that the target actually looks like a Node backend: read its `package.json` and establish the runtime shape before reviewing anything.

- Confirm it is server-side JavaScript/TypeScript — a `main`/`exports` entry, `engines.node`, a start/dev script running `node`/`tsx`/`ts-node`/`nest`, or a server framework dependency. If `react`/`next` is the dominant dependency and there is no server layer, say so and confirm before proceeding — that is a job for `react-code-review`, not this skill.
- Record, because nearly every later judgment depends on them: **framework** (Express / Fastify / NestJS / Koa / Hapi / raw `node:http` / serverless handlers), **module system** (`"type": "module"` ESM vs CommonJS), **language** (TypeScript vs JavaScript, and whether `strict` is on in `tsconfig.json`), **data layer** (Prisma / TypeORM / Sequelize / Mongoose / Knex / raw driver), **Node version** from `engines` and CI, and **runtime topology** (single process, `pm2`/cluster, Docker, Lambda).
- **Monorepos:** if the root `package.json` has `workspaces` (or there is a `turbo.json` / `pnpm-workspace.yaml` / `nx.json`), the root is usually not the review target on its own. Enumerate the workspace packages and services, then either review all of them (one findings sheet each, and say so in the handover message) or ask the user which service is in scope. Never report on one service as if it were the whole repo.

State the detected stack back to the user in one line before you start exploring — a wrong framework guess poisons every finding that follows.

## Step 2 — Load the Node-specific references

This skill grounds its findings in a bundled reference plus whatever standards are actually installed in the environment. Load them before writing findings, not from general Node intuition.

**1. Node.js Best Practices — `references/node-best-practices.md` (bundled, always available).** A citable index of the community Node.js Best Practices list (`goldbergyoni/nodebestpractices`, the de-facto canonical Node list), organized into its 8 sections with stable numbered IDs. Cite the specific ID in a finding — `NBP-2.10`, `NBP-6.10`, `NBP-5.15` — exactly the way `react-code-review` findings cite a Vercel rule ID. Never invent an ID that is not in that file; if no rule fits, say so and ground the finding in the repository's own evidence instead.

**2. Security — OWASP mapping.** `references/node-best-practices.md` maps its §6 items to the relevant OWASP Top 10 categories. For `SEC` findings, cite both the NBP ID and the OWASP category (e.g. "`NBP-6.4` / OWASP A03:2021 Injection") — a security finding that names the vulnerability class is actionable; one that says "this is insecure" is not. Broken access control (OWASP A01) has no NBP ID and is the defect backend reviews miss most often; cite the OWASP category alone there.

**3. Framework-specific practice.** Once Step 1 has identified the framework and data layer, hold findings to that stack's documented guidance rather than a generic one: Express (middleware ordering, async error propagation, centralized error middleware, `helmet`, `express-rate-limit`), Fastify (schema-first validation, plugin encapsulation), NestJS (DI, modules, Guards/Interceptors/Pipes, DTO + `class-validator`, exception filters), Prisma (connection pooling, `$transaction`, N+1 via missing `include`/`select`, raw-query injection risk), Mongoose/TypeORM equivalents. The per-framework review cues are at the end of `references/node-best-practices.md`. Don't rate an Express app against NestJS conventions, or vice versa.

**4. Installed org standards, if any.** Check whether a Node/backend standards plugin is installed for this environment before falling back to the bundled reference alone: list `~/.claude/plugins/cache/` and `~/.claude/skills/` and look for a Node/backend/API standards skill (the React equivalent in this environment is `cognine-plugin-marketplace/react-coding-standards-best-practices/<version>/`; a Node sibling may exist or appear later — don't hardcode a version number). If one is present, load only the topic docs relevant to the supplied criteria and cite its numbered check IDs alongside the NBP IDs. If none is present, that is expected, not an error — say in the handover message that findings are grounded in NBP/OWASP plus repository evidence.

If a reference you expected is missing, note the gap explicitly in the handover message and fall back to well-established, citable practice for that area — never fabricate a rule or check ID.

## Step 3 — Explore the repository against the criteria

If the target repo has a root `CLAUDE.md`, read it first — it may already document the architecture and save you from re-deriving it. Read `.env.example`, `docker-compose*.yml`, `Dockerfile`, CI config, and `tsconfig.json` early too: on a backend, the deployment and config surface *is* part of the code under review, and several whole categories (`OPS`, `DOCK`, `SEC`) live there.

Launch up to 3 parallel `Explore` agents to cover the repo's surface area, tailored to what the supplied criteria actually ask about (don't run all three if the criteria only concern one area):

1. **Architecture, structure & config** — layering (route → controller/handler → service → data access), component boundaries, whether business logic leaks into route handlers or ORM models, config/secrets loading and environment handling, module boundaries in a monorepo.
2. **API, data & async correctness** — route/controller definitions, request validation (schema/DTO) and authorization on each route, the data-access layer (queries, transactions, N+1, pooling, migrations), async patterns (floating promises, unawaited returns, missing `try/catch`, `Promise.all` misuse, event-emitter `error` handling), and external-service calls (timeouts, retries, circuit breaking).
3. **Security, operations & testing** — authn/authz implementation, injection and input-handling surface, secrets handling, logging/observability (structured logger, correlation IDs, what gets logged), graceful shutdown and process-level handlers, rate limiting and payload limits, Dockerfile/deployment hygiene, and the test suite's shape and coverage.

Alongside the criteria work, establish the service's **functional module map** (Authentication, Orders, Payments, Notifications, ...) early, because every row on a findings sheet has to name one. Derive it from route prefixes, business-component folders, and the domain model per the Module Name section of `references/rubrics.md`, decide which fallback label (Cross-cutting, Shared Libraries, Build & Deployment, Application Bootstrap) covers the infrastructure code, and state the finished list in the handover message so the reader can see the taxonomy the Module Name column uses. The architecture agent is the natural place to ask for this: have it return a mapping of module name to the route prefixes and paths it covers, not just a folder listing.

Give each agent the actual list of Review Criteria (not a summary) and ask it to report back **findings already tied to file paths and code**, mapped to whichever criterion each observation relates to — not generic prose about what it saw. An agent that returns "error handling could be improved" without a file reference is not useful input to this report; push back and re-run narrower if that happens.

Pay attention to what is *absent*, which is where backend reviews earn their keep and file-grep exploration is weakest — a route with no auth middleware, a handler with no input validation, a `process.on('unhandledRejection')` that was never registered, a service with no timeout on its outbound HTTP call. An absence is a legitimate, reportable finding as long as you can point to the specific file or route where the thing should have been and confirm it is not there — and is not handled globally somewhere else (check the app bootstrap before claiming a gap).

## Step 4 — Synthesize findings

For each distinct issue:

- Assign a Finding ID using the prefix table in `references/rubrics.md` (`ARCH-001`, `SEC-002`, `ASYNC-003`, etc.) — one ID per root cause, not per affected file.
- If the same root cause recurs across multiple files, don't merge it into one row: give it one Finding ID, but produce one findings-sheet row per affected file (each embedding that same ID in its File / Location cell), with its own Metric/Observation/Impact where those differ between instances — don't create a *new* Finding ID for the same underlying pattern repeated across routes, and don't collapse the instances into a single row either. Rows can and often will repeat the same Module Name when those files sit in the same functional module.
- For a pattern spread across more files than is readable as rows (say 15 unvalidated routes), still list one row per file for the most significant instances and use a single summarizing row for the remainder, with the Metric carrying the real count ("23 of 31 route handlers"). Say in the detailed write-up which files the summarizing row covers.
- Assign each row its **Module Name**: the functional module the file belongs to, taken from the module map built in Step 3, never the file name. The file path goes in the separate File / Location column, with the Finding ID after it. If the file belongs to no single feature, use a fallback label (Cross-cutting, Shared Libraries, Build & Deployment, Application Bootstrap) per `references/rubrics.md`.
- Set each row's **Category** and **Sub Category** from the supplied criteria, per the Category and Sub Category section of `references/excel-report.md`: Category is the top-level review area the criterion sits under, Sub Category is the criterion itself. These are how the workbook records which criterion a finding answers, so they replace the "mapped criterion" line the Markdown report used to carry. A row's Category does not have to match its Finding ID prefix.
- Classify each row's Type (Route / Middleware / Function / Module / Config) and Metric per `references/rubrics.md` — Type describes the code in File / Location, not the functional module.
- Capture the **Evidence** for every row while the file is open: the real lines from that file, 3 to 15 of them, with a `// path:line` comment on the first line. The workbook has no Detailed Findings section, so the row carries its own proof. Copy the lines, never retype them from memory of an exploration summary.
- Rate Severity (checked against the Metric → Severity threshold table in `references/rubrics.md`), Remediation Effort, and Priority — Priority is derived mechanically from Severity × Effort there, not chosen independently. Effort gets no column in the workbook, but record it on the row anyway: the build script uses it to verify the Priority you assigned.

Track two things explicitly as you go, because the spec requires stating them rather than letting them go unmentioned:
- Which supplied criteria produced **no findings** (note in the report that they were evaluated and passed, don't just omit them).
- Which supplied criteria **could not be evaluated** at all from what is in the repository (e.g. a criterion about production log aggregation when infrastructure lives in a separate repo, or about load behavior when the review is static-only) — these go in a "Criteria Not Evaluated" subsection with the concrete reason, never silently dropped or guessed at.

## Step 5 — Verify every finding before it goes in the report

Before a finding is written into the draft, re-open the cited file and confirm the pattern is really there — quote or closely paraphrase the actual lines, don't reconstruct them from memory of the exploration summary. Confirm the recommendation actually addresses what is shown, not an adjacent concern.

Two verification traps are specific to Node backends, and both produce embarrassing false positives — check for each before keeping a finding:

- **Global handling you didn't look for.** A missing `try/catch` in a handler is not a finding if the framework wraps it (Fastify, NestJS, Express 5, or an `asyncHandler` / `express-async-errors` registered in the bootstrap). Missing per-route validation is not a finding if a global pipe or schema validator is registered. Always read the app/server bootstrap and middleware registration order before reporting an absence.
- **Configuration you assumed.** Don't claim a secret is hardcoded without reading the actual value (it may be a local-dev default, or read from `process.env` with a fallback); don't claim a query is injectable without checking whether the ORM parameterizes it.

This mirrors how this environment's own `code-review` skill treats findings: generate candidates, then adversarially verify each one against the real code before it is allowed to survive into the output. Drop or downgrade anything that doesn't hold up — a report with fewer, verified findings is more valuable than one padded with plausible-sounding guesses.

## Step 6 — Build the workbook

The deliverable is an Excel workbook, not a Markdown document. Follow `references/excel-report.md` exactly: the 12 columns in their fixed order, one findings sheet per reviewed service, a `Threshold Mapping` sheet last, one row per affected file (not per root cause), Module Name holding the functional module and File / Location holding the path plus the Finding ID.

Do not hand-write openpyxl code. Write the findings to a JSON file in the scratchpad directory and run the bundled script, which owns the column widths, fills, borders, row heights, freeze panes, auto filter and Excel's own limits:

```bash
python "${CLAUDE_PLUGIN_ROOT}/skills/node-code-review/scripts/build_report_xlsx.py" findings.json
```

`${CLAUDE_PLUGIN_ROOT}` is set for you when this skill runs from the installed plugin. If it is
unset, the skill has been vendored into a repository instead, so run the copy that sits next to
this file: `python .claude/skills/node-code-review/scripts/build_report_xlsx.py findings.json`.
The script finds its own `threshold_mapping.json` either way, so nothing else changes.

The script validates before it writes: an empty required cell, an unknown Type / Severity / Priority value, or a Priority that contradicts the row's Severity and Effort all stop the build and name the offending sheet and row. Fix the JSON and run it again rather than working around the check.

Sort rows within each sheet by Category (rubric prefix-table order), then Sub Category, then Priority (P1 first), then Severity, then file path. A sheet has no headings, so the sort is the only grouping a reader gets.

The Threshold Mapping sheet is built per review, not shipped whole. `scripts/threshold_mapping.json` is a catalog of one threshold block per review area (Architecture, Code Quality, API Design, Data, Async, Error Handling, Security, Performance, Testing, Operations, Containerization, Dependencies). Set `threshold_areas` in the findings JSON from the **supplied criteria**, not from the findings you happened to produce, so a criterion that was evaluated and passed still shows the rule it was measured against. An entry matches on the area key, the Finding ID prefix, the label, or any of the area's `match` words, and it tolerates a criteria heading wrapped around the word ("Security and OWASP Review" resolves to Security). Leave `threshold_areas` out only as a fallback, in which case the script infers the areas from the Finding ID prefixes in your rows.

If the script warns that an area name didn't match, the criteria asked about something the catalog has no rules for. Write those rows yourself and pass them as `extra_thresholds` and `extra_severity_bands` in the same 5-column shape. Never score a finding against a threshold that is not on that sheet, and never leave a block on the sheet that no criterion asked for.

Two things the Markdown report carried in prose have no home in the workbook, so **say them in the handover message instead**: which supplied criteria were evaluated and passed with no findings, and which could not be evaluated at all, with the concrete reason for each. Also state the functional module list you derived, so the reader can see the taxonomy the Module Name column uses.

If the supplied Review Criteria ask for complexity or code-smell analysis, that is a Sub Category on the findings sheet with the complexity metric in the Metric column, not a separate summary section.

**Writing style**: write every cell in plain, natural human phrasing, in simple English, the way a developer explains a problem to a teammate who joined last week. Never use an em dash (`—`) anywhere in the workbook; use a comma, a period, or a parenthetical instead. Short sentences, one idea each. Name the file and the function instead of saying "the code". Say what to do, not what to consider. Avoid stock AI-report filler ("it is important to note that," "in today's fast-paced development environment," "leverage," "robust," excessive hedging) — just state what was found. The per-column writing rules in `references/excel-report.md` are the full version of this.

Write the file to `<repo-path>/CORE_REVIEW_REPORT.xlsx` unless the user passed `--output <path>`. If a workbook already exists at that path, don't silently overwrite it — say so and confirm, or write alongside it with a version suffix. Tell the user the full path you wrote to.

## Step 7 — Run the pre-delivery validation gate

Before showing anything to the user, run two gates against the built workbook. First the Excel gate at the end of `references/excel-report.md`, which covers structure, per-row completeness, writing style and criteria coverage. Then the evidence, rating and consistency sections of `references/validation-checklist.md`, which still apply in full (its section 1, Markdown validation, is superseded by the structure checks in the Excel gate).

Reopen the saved `.xlsx` with openpyxl as part of the gate and confirm it loads and the row counts match the JSON. A workbook that Excel refuses to open is the one failure mode that makes the whole report worthless.

If any item fails, fix it and **re-run both gates from the top** — a fix in one place can break consistency elsewhere (e.g. correcting a Priority value changes where its row sorts). Only present the workbook once every item passes. If you had to fix something, mention it briefly when handing the file over — it is a useful signal the gate actually did something, not just theater.

## Reference files

- `references/excel-report.md` — the output format: the 12 columns and what goes in each, the Category / Sub Category rules, how to write a cell in simple English, row ordering, the fixed formatting spec, the Threshold Mapping sheet, the build script's JSON input, and the Excel validation gate from Step 7.
- `references/node-best-practices.md` — the citable NBP rule index (8 sections, numbered IDs), the OWASP mapping for the security section, and framework-specific review cues for Express / Fastify / NestJS / Prisma / TypeORM / Mongoose.
- `references/rubrics.md` — Finding-ID prefixes, how to derive the functional Module Name list and its fallback labels, Type and Metric definitions (including the Metric → Severity thresholds), and the Severity / Remediation Effort / Priority definitions (Priority is derived from the other two — always recompute, never assign by feel).
- `references/report-template.md` — the previous Markdown deliverable. No longer the output format, but still the best reference for how a good Observation, Impact and Recommendation is worded, including a fully-written worked example.
- `references/validation-checklist.md` — the evidence, rating, consistency and completeness gate from Step 7. Its Markdown-structure section is superseded by the Excel gate in `excel-report.md`.

## Scripts

- `scripts/build_report_xlsx.py` — builds the workbook from a findings JSON file. Requires `openpyxl`. Run it with `--out <path>` to override the output path in the JSON.
- `scripts/threshold_mapping.json` — the threshold catalog, one block of thresholds and severity bands per review area. The script selects from it per review via `threshold_areas`. Edit this file only to add an area that recurs across reviews; tailor a single review with `threshold_areas` and `extra_thresholds` in the findings JSON.
