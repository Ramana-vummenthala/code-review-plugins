---
name: node-code-review
description: Audit a Node.js/TypeScript backend repository (Express, Fastify, NestJS, Koa, Hapi, or plain Node services) against a supplied set of Review Criteria and produce an evidence-based "Core Review Report" in Markdown — a structured Key Findings table (severity/effort/priority) plus detailed per-finding writeups, grounded in the Node.js Best Practices list (NBP) and OWASP. Use whenever the user asks for a Node/backend/API code review, code audit, architecture/quality review, or a findings report for a Node.js codebase — invoke as `/node-code-review <criteria> <repo-path>`, or trigger conversationally when the user describes wanting this kind of structured review.
license: MIT
metadata:
  author: engineering
  version: "1.0.0"
---

# Node.js Code Review

Produces a "Core Review Report": a structured, evidence-based Markdown audit of a Node.js backend codebase, scored against a specific set of Review Criteria supplied at invocation time. The point of this skill is that every finding is traceable to real code — never write a finding you haven't verified against the actual repository.

This is the backend counterpart to the `react-code-review` skill: same report shape, same rubrics, same validation gate — different subject matter and different grounding references.

## Why this exists

Generic Node advice ("use async/await", "validate input") is cheap and useless to a team that already ships Node services. What's valuable is a report that says exactly *where* in *this* repository a pattern shows up, quotes the real code, cites the specific practice it violates, and rates the fix the same way every time it's run. The steps below exist to enforce that discipline — skipping straight to writing findings from general knowledge defeats the purpose.

Backend reviews also carry a risk a frontend review doesn't: a missed finding here is a data breach, silent data corruption, or a 3am page — not a slow render. Weight severity accordingly (see `references/rubrics.md`).

## Step 1 — Resolve the two required inputs

This skill needs exactly two things: **Review Criteria** and a **Repository** to review. Parse them out of the invocation's arguments:

- Accept explicit flags: `--criteria <path-or-text>` and `--repo <path>`.
- Or two bare positional tokens: `<criteria> <repo-path>` — the token that resolves to an existing directory is the repo; the other is the criteria (a path to an existing file, or inline text if it isn't a path).
- An `--output <path>` flag may override where the report gets written (default is described in Step 6).

If criteria was given as a file path, read the file — don't paraphrase it from the filename. If the repo path doesn't exist, or either input is genuinely missing, **stop and ask the user** rather than substituting "general Node best practices" as a stand-in for criteria, or defaulting to the current directory as the repo. The whole value of this report is that it's criteria-driven, not a generic scan; guessing the criteria defeats that.

Sanity-check that the target actually looks like a Node backend: read its `package.json` and establish the runtime shape before reviewing anything.

- Confirm it is server-side JavaScript/TypeScript — a `main`/`exports` entry, `engines.node`, a start/dev script running `node`/`tsx`/`ts-node`/`nest`, or a server framework dependency. If `react`/`next` is the dominant dependency and there is no server layer, say so and confirm before proceeding — that is a job for `react-code-review`, not this skill.
- Record, because nearly every later judgment depends on them: **framework** (Express / Fastify / NestJS / Koa / Hapi / raw `node:http` / serverless handlers), **module system** (`"type": "module"` ESM vs CommonJS), **language** (TypeScript vs JavaScript, and whether `strict` is on in `tsconfig.json`), **data layer** (Prisma / TypeORM / Sequelize / Mongoose / Knex / raw driver), **Node version** from `engines` and CI, and **runtime topology** (single process, `pm2`/cluster, Docker, Lambda).
- **Monorepos:** if the root `package.json` has `workspaces` (or there is a `turbo.json` / `pnpm-workspace.yaml` / `nx.json`), the root is usually not the review target on its own. Enumerate the workspace packages and services, then either review all of them (stating so in Review Scope) or ask the user which service is in scope. Never report on one service as if it were the whole repo.

State the detected stack back to the user in one line before you start exploring — a wrong framework guess poisons every finding that follows.

## Step 2 — Load the Node-specific references

This skill grounds its findings in a bundled reference plus whatever standards are actually installed in the environment. Load them before writing findings, not from general Node intuition.

**1. Node.js Best Practices — `references/node-best-practices.md` (bundled, always available).** A citable index of the community Node.js Best Practices list (`goldbergyoni/nodebestpractices`, the de-facto canonical Node list), organized into its 8 sections with stable numbered IDs. Cite the specific ID in a finding — `NBP-2.10`, `NBP-6.10`, `NBP-5.15` — exactly the way `react-code-review` findings cite a Vercel rule ID. Never invent an ID that is not in that file; if no rule fits, say so and ground the finding in the repository's own evidence instead.

**2. Security — OWASP mapping.** `references/node-best-practices.md` maps its §6 items to the relevant OWASP Top 10 categories. For `SEC` findings, cite both the NBP ID and the OWASP category (e.g. "`NBP-6.4` / OWASP A03:2021 Injection") — a security finding that names the vulnerability class is actionable; one that says "this is insecure" is not.

**3. Framework-specific practice.** Once Step 1 has identified the framework and data layer, hold findings to that stack's documented guidance rather than a generic one: Express (middleware ordering, async error propagation, centralized error middleware, `helmet`, `express-rate-limit`), Fastify (schema-first validation, plugin encapsulation), NestJS (DI, modules, Guards/Interceptors/Pipes, DTO + `class-validator`, exception filters), Prisma (connection pooling, `$transaction`, N+1 via missing `include`/`select`, raw-query injection risk), Mongoose/TypeORM equivalents. The per-framework review cues are in `references/node-best-practices.md`. Don't rate an Express app against NestJS conventions, or vice versa.

**4. Installed org standards, if any.** Check whether a Node/backend standards plugin is installed for this environment before falling back to the bundled reference alone: list `~/.claude/plugins/cache/` and `~/.claude/skills/` and look for a Node/backend/API standards skill (the React equivalent in this environment is `cognine-plugin-marketplace/react-coding-standards-best-practices/<version>/`; a Node sibling may exist or appear later — don't hardcode a version number). If one is present, load only the topic docs relevant to the supplied criteria and cite its numbered check IDs alongside the NBP IDs. If none is present, that is expected, not an error — note in Review Scope that findings are grounded in NBP/OWASP plus repository evidence.

If a reference you expected is missing, note the gap explicitly in the report's Review Scope section and fall back to well-established, citable practice for that area — never fabricate a rule or check ID.

## Step 3 — Explore the repository against the criteria

If the target repo has a root `CLAUDE.md`, read it first — it may already document the architecture and save you from re-deriving it. Read `.env.example`, `docker-compose*.yml`, `Dockerfile`, CI config, and `tsconfig.json` early too: on a backend, the deployment and config surface *is* part of the code under review, and several whole categories (`OPS`, `DOCK`, `SEC`) live there.

Launch up to 3 parallel `Explore` agents to cover the repo's surface area, tailored to what the supplied criteria actually ask about (don't run all three if the criteria only concern one area):

1. **Architecture, structure & config** — layering (route → controller/handler → service → data access), component boundaries, whether business logic leaks into route handlers or ORM models, config/secrets loading and environment handling, module boundaries in a monorepo.
2. **API, data & async correctness** — route/controller definitions, request validation (schema/DTO) and authorization on each route, the data-access layer (queries, transactions, N+1, pooling, migrations), async patterns (floating promises, unawaited returns, missing `try/catch`, `Promise.all` misuse, event-emitter `error` handling), and external-service calls (timeouts, retries, circuit breaking).
3. **Security, operations & testing** — authn/authz implementation, injection and input-handling surface, secrets handling, logging/observability (structured logger, correlation IDs, what gets logged), graceful shutdown and process-level handlers, rate limiting and payload limits, Dockerfile/deployment hygiene, and the test suite's shape and coverage.

Give each agent the actual list of Review Criteria (not a summary) and ask it to report back **findings already tied to file paths and code**, mapped to whichever criterion each observation relates to — not generic prose about what it saw. An agent that returns "error handling could be improved" without a file reference is not useful input to this report; push back and re-run narrower if that happens.

Pay attention to what is *absent*, which is where backend reviews earn their keep and file-grep exploration is weakest — a route with no auth middleware, a handler with no input validation, a `process.on('unhandledRejection')` that was never registered, a service with no timeout on its outbound HTTP call. An absence is a legitimate, reportable finding as long as you can point to the specific file or route where the thing should have been and confirm it is not there — and is not handled globally somewhere else (check the app bootstrap before claiming a gap).

## Step 4 — Synthesize findings

For each distinct issue:

- Assign a Finding ID using the prefix table in `references/rubrics.md` (`ARCH-001`, `SEC-002`, `ASYNC-003`, etc.).
- If multiple observations share one root cause, merge them into a single finding and list every affected file under "Files Affected" — don't create near-duplicate findings for the same underlying pattern repeated across routes. "23 of 31 route handlers lack input validation" is one finding, not 23.
- Rate Severity, Remediation Effort, and Priority using `references/rubrics.md` — Priority is derived mechanically from Severity × Effort there, not chosen independently.
- Map the finding to the specific supplied criterion it addresses.

Track two things explicitly as you go, because the spec requires stating them rather than letting them go unmentioned:
- Which supplied criteria produced **no findings** (note in the report that they were evaluated and passed, don't just omit them).
- Which supplied criteria **could not be evaluated** at all from what is in the repository (e.g. a criterion about production log aggregation when infrastructure lives in a separate repo, or about load behavior when the review is static-only) — these go in a "Criteria Not Evaluated" subsection with the concrete reason, never silently dropped or guessed at.

## Step 5 — Verify every finding before it goes in the report

Before a finding is written into the draft, re-open the cited file and confirm the pattern is really there — quote or closely paraphrase the actual lines, don't reconstruct them from memory of the exploration summary. Confirm the recommendation actually addresses what is shown, not an adjacent concern.

Two verification traps are specific to Node backends, and both produce embarrassing false positives — check for each before keeping a finding:

- **Global handling you didn't look for.** A missing `try/catch` in a handler is not a finding if the framework wraps it (Fastify, NestJS, Express 5, or an `asyncHandler` / `express-async-errors` registered in the bootstrap). Missing per-route validation is not a finding if a global pipe or schema validator is registered. Always read the app/server bootstrap and middleware registration order before reporting an absence.
- **Configuration you assumed.** Don't claim a secret is hardcoded without reading the actual value (it may be a local-dev default, or read from `process.env` with a fallback); don't claim a query is injectable without checking whether the ORM parameterizes it.

This mirrors how this environment's own `code-review` skill treats findings: generate candidates, then adversarially verify each one against the real code before it is allowed to survive into the output. Drop or downgrade anything that doesn't hold up — a report with fewer, verified findings is more valuable than one padded with plausible-sounding guesses.

## Step 6 — Write the report

Follow `references/report-template.md` exactly: heading structure, the category-grouped 13-column Key Findings tables, and the per-finding detailed-writeup shape. Findings are grouped into `###` sections by category (rubric prefix-table order), not laid out as one flat priority-ordered table — within each category section, order rows by Priority (P1 first), then Severity. The `## Recommendations` section remains the intentional place for a cross-category, priority-first view.

Write the file to `<repo-path>/CORE_REVIEW_REPORT.md` unless the user passed `--output <path>`. If a report already exists at that path, don't silently overwrite it — say so and confirm, or write alongside it. Tell the user the path you wrote to.

## Step 7 — Run the pre-delivery validation gate

Before showing anything to the user, go through `references/validation-checklist.md` item by item against the drafted report. If any item fails, fix the report and **re-run the full checklist again** — a fix in one place can break consistency elsewhere (e.g. correcting a Priority value needs re-checking the row is still sorted correctly). Only present the report once every item passes. If you had to fix something, mention it briefly when handing the report over — it is a useful signal the gate actually did something, not just theater.

## Reference files

- `references/node-best-practices.md` — the citable NBP rule index (8 sections, numbered IDs), the OWASP mapping for the security section, and framework-specific review cues for Express / Fastify / NestJS / Prisma.
- `references/rubrics.md` — Finding-ID prefixes, and the Severity / Remediation Effort / Priority definitions (Priority is derived from the other two — always recompute, never assign by feel).
- `references/report-template.md` — the literal report structure to fill in, including a worked example of one fully-written finding, category-grouped.
- `references/validation-checklist.md` — the mandatory pre-delivery gate from Step 7, transcribed as pass/fail items.
