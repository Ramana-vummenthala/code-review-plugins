# Rubrics: Module Name, Finding IDs, Type, Metric, Severity, Remediation Effort, Priority

These exist so two runs of the skill (or two reviewers) rate the same kind of issue the same way. Don't eyeball a rating — match it against the definition below and be ready to justify it from the definition, not from gut feel.

## Finding ID prefixes

One prefix per category, a 3-digit zero-padded sequence number per prefix, starting at 001 (e.g. `ARCH-001`, `ARCH-002`, `SEC-001`).

**The order of this table is the fixed category order used throughout the report** — in `## Key Findings`, in `## Detailed Findings`, and in the validation checklist.

| # | Prefix | Category | Grounded in |
|---|---|---|---|
| 1 | `ARCH` | Architecture | NBP §1 — component structure, 3-tier layering, config, framework fit, TypeScript discipline |
| 2 | `CQ` | Code Quality | NBP §3 — style, naming, module patterns, lint setup, dead code, duplication |
| 3 | `API` | API Design & Contracts | NBP §2.5, §6.10, §6.14 — route/controller design, request & response schemas, status codes, versioning, pagination, documentation |
| 4 | `DATA` | Data & Persistence | NBP §6.4 plus the Prisma/TypeORM/Mongoose cues — queries, transactions, migrations, schema/indexing, connection lifecycle, data integrity |
| 5 | `ASYNC` | Async & Concurrency | NBP §2.1, §2.10, §2.12, §2.13, §3.11 — floating promises, unawaited returns, event-emitter errors, race conditions, concurrency limits, timeouts |
| 6 | `ERR` | Error Handling | NBP §2 — error types, central handling, operational vs programmer errors, logging of failures |
| 7 | `SEC` | Security | NBP §6 plus OWASP Top 10 — cite both IDs (`NBP-6.4` / OWASP A03:2021); access-control findings cite OWASP A01 even with no NBP ID |
| 8 | `PERF` | Performance | NBP §7 plus evidence-grounded backend performance (N+1, event-loop blocking, unbounded reads, missing caching/indexes) |
| 9 | `TEST` | Testing | NBP §4 — test shape, coverage, isolation, error-path and e2e coverage, CI enforcement |
| 10 | `OPS` | Operations & Observability | NBP §5 — logging, monitoring, health checks, graceful shutdown, config/secrets at deploy time, statelessness, Node version |
| 11 | `DOCK` | Containerization & Deployment | NBP §8 — Dockerfile hygiene, image size and provenance, signal handling, build caching, CI/CD deploy path |
| 12 | `DEP` | Dependency Governance | NBP §5.4, §5.13, §5.19, §6.7, §6.26, §6.27 — lockfiles, vulnerable/outdated/unmaintained packages, licence risk, dependency sprawl |

Cite the specific rule ID from `node-best-practices.md` in the finding's Recommended Improvement (and the OWASP category for `SEC`). A finding with no applicable ID is still valid — ground it in repository evidence and say which practice it violates in words. Never invent an ID.

If the supplied Review Criteria name a category not covered above, mint a short, uppercase, single-word prefix consistent with this style, add it to the end of the order, and document it in the delivered report's Review Criteria section so the ID scheme is self-documenting. Never reuse a prefix for two different categories within one report.

## Module Name (functional module)

The Module Name column answers **"which part of the product is this?"**, not "which file is this?". It is a functional/domain module name a product owner would recognise: `Authentication`, `Registration`, `Users`, `Orders`, `Payments`, `Merchants`, `Catalog`, `Notifications`, `Reporting`, `Webhooks`, `Admin`. The file path lives in the separate `File / Location` column, so never put a file name, a folder path, or a class name in Module Name.

### Deriving the module list

Derive the list from the repository during exploration (Step 3), in this order of preference, and record the result in the report's `## Review Scope` section so the reader can see the taxonomy the findings are grouped by:

1. **Route prefixes** — the router registration in the app bootstrap. `/api/orders`, `/api/payments`, `/auth/*` usually name the module directly, and this is the most reliable signal on a backend because it is the service's actual public surface.
2. **Business-component folders** — `src/components/*`, `src/modules/*`, `src/domains/*`, or NestJS feature modules (`orders.module.ts`). This is what `NBP-1.1` asks a well-structured Node service to have, so a repo that has it is naming its own modules for you.
3. **The domain model** — Prisma models, TypeORM entities, Mongoose schemas, or migration table names, grouped into the aggregates they belong to. Useful when the code is organized by technical layer (`controllers/`, `services/`, `models/`) rather than by feature, which is common and is itself often an `ARCH` finding.
4. **Background work and integrations** — queue consumers, cron/scheduled jobs, and webhook handlers. These are real functional modules even though they have no HTTP route, name them for the business process they serve (`Payout Processing`, `Email Delivery`), not for the transport.

Keep the list small and stable: prefer 5 to 15 modules for a typical service. Don't mint a new module per file, and don't re-derive a different list halfway through the report — the same file always maps to the same module in every row it appears in. In a monorepo, name the module within the reviewed service (`Orders`), not the workspace package (`@acme/orders-api`), and say in Review Scope which service the list belongs to.

### Fallback labels

Plenty of backend code belongs to no single feature. Use one of these fixed labels rather than forcing a domain name or leaving the cell blank:

| Label | Use for |
|---|---|
| **Cross-cutting** | Code every module routes through: the error-handling middleware, the logger, auth middleware/guards, the http client wrapper, the validation pipe, the ORM client singleton, rate limiting. |
| **Shared Libraries** | Internal shared utilities and types consumed across modules (`src/common/**`, `src/utils/**`, or a shared workspace package in a monorepo). |
| **Build & Deployment** | `package.json`, lockfiles, `tsconfig.json`, ESLint config, `Dockerfile`, `docker-compose`, and CI pipeline findings (most `DEP` and `DOCK` findings land here). |
| **Application Bootstrap** | The entry point and server wiring: `app.ts`/`main.ts`, middleware registration order, config/env loading, graceful shutdown, `process.on` handlers, DB connection setup. |

`Application Bootstrap` and `Cross-cutting` overlap in practice. Use **Application Bootstrap** when the finding is about the one-time wiring at startup (registration order, a handler never registered), and **Cross-cutting** when it is about the shared component itself (the error middleware leaks stack traces, the logger is unstructured).

If a finding genuinely spans several modules and isn't infrastructure, still write one row per file and give each row the module that file belongs to — that's what makes the table show which parts of the service are affected.

## Type

Type classifies the code named in the row's `File / Location` cell (not the functional module) as exactly one of:

| Type | Definition |
|---|---|
| **Route** | An HTTP route/endpoint definition or the controller/handler that serves it — a file or exported handler reachable from a request, e.g. `orders.route.ts`, an Express `router.post(...)` handler, a NestJS controller method, a Lambda handler. |
| **Middleware** | Code in the request pipeline that wraps or intercepts handlers rather than terminating a request: Express middleware, a NestJS Guard / Interceptor / Pipe / exception filter, a Fastify hook or plugin. |
| **Function** | A service method, repository/data-access function, mapper, job/consumer function, or plain utility — callable code that is not itself a route or middleware, e.g. `OrderService.create`, a `*.repository.ts` function, a queue consumer's handler function. |
| **Module** | A whole file/class/service/pattern that isn't one function, route, or middleware — e.g. an entire service class, a Prisma schema, a router module cited as a unit, or a pattern cited across a business component. |
| **Config** | A non-executable or declarative artifact: `package.json`, a lockfile, `tsconfig.json`, ESLint config, `Dockerfile`, `docker-compose.yml`, a CI workflow, `.env.example`. |

Note the two senses of the word in this report: the **Module Name** column is the functional module (a product area), while a **Type** of `Module` is a code-shape classification for the file in `File / Location`. A row reading `Module Name: Orders` with `Type: Route` is normal, not a contradiction.

## Metric

Every Key Findings row's Metric column is a real, quantifiable measurement pulled from the repository — a count, a percentage, a size, a version, a compliance level — never a restatement of the Observation in prose. Use the literal text `N/A` only when no metric genuinely applies to that finding (e.g. a naming-convention inconsistency with nothing to count); it must never stand in for a metric you simply didn't measure.

On a backend the most useful metric is almost always **a count out of a total**, because it shows coverage rather than an anecdote: "23 of 31 route handlers", "4 of 4 write paths", "0 of 12 outbound calls". Prefer that shape wherever it applies.

Representative metrics by category:

| Category | Example Metric values |
|---|---|
| Architecture (`ARCH`) | "3 responsibilities in one file", "`req`/`res` passed into 6 service methods", "`strict: false` in `tsconfig.json`" |
| Code Quality (`CQ`) | "640-line file", "cyclomatic complexity ~18", "`no-floating-promises` rule not enabled" |
| API Design (`API`) | "9 of 14 routes return an ad hoc error shape", "0 routes versioned", "no response schema on 11 routes" |
| Data & Persistence (`DATA`) | "N+1: 1 + 250 queries per request", "3 writes outside a transaction", "unindexed column on a 2M-row table" |
| Async & Concurrency (`ASYNC`) | "7 floating promises", "0 of 12 outbound calls have a timeout", "`Promise.all` over an unbounded array" |
| Error Handling (`ERR`) | "0 central error handlers registered", "18 catch blocks that swallow the error" |
| Security (`SEC`) | "23 of 31 routes lack input validation", "1 hardcoded JWT secret", "`NBP-6.4` failed on 2 raw queries" |
| Performance (`PERF`) | "`readFileSync` on the request path", "`findMany` with no `take` on an unbounded table", "sequential awaits adding ~600ms" |
| Testing (`TEST`) | "12% line coverage", "0 API/component tests", "0 tests assert a 4xx response" |
| Operations (`OPS`) | "0 structured log statements", "no `SIGTERM` handler", "Node 16 in `engines` (EOL)" |
| Containerization (`DOCK`) | "single-stage image, 1.2GB", "`CMD [\"npm\", \"start\"]`", "no `USER node`" |
| Dependency Governance (`DEP`) | "4 high-severity advisories in `npm audit`", "28 months since last release", "no lockfile committed" |

### Metric → Severity thresholds

Severity must meet or exceed the threshold implied by the Metric value below — these are a floor, not a ceiling. A reviewer may rate higher given genuine production risk, but never lower, and any deviation from the table must be justified in the finding's Impact text.

| Category | High (or above) | Medium | Low |
|---|---|---|---|
| Security (`SEC`) | A reachable exploit path from an unauthenticated or low-privilege request (Critical if user/tenant data or a credential is directly exposed); any missing ownership check on a route that takes an ID | A failed check needing deliberate but plausible conditions, or an authenticated-only exposure | Hardening/defense-in-depth gap with no direct exploit path |
| Data & Persistence (`DATA`) | A multi-write flow with no transaction (Critical if a partial failure corrupts money or ownership data); an N+1 over an unbounded set on a hot path | An N+1 on a low-traffic path; a missing index on a table that will grow; over-fetching large columns | A projection or naming issue with no measurable effect at current scale |
| Async & Concurrency (`ASYNC`) | Any floating promise or unhandled emitter `error` that can crash the process; no timeout on an outbound call on the request path | An unawaited call whose failure is silently lost but cannot crash; unbounded `Promise.all` on a bounded-in-practice set | Sequential awaits that could be concurrent, off the critical path |
| Error Handling (`ERR`) | No central error handler, or errors swallowed on a critical path so failures are invisible in production | Inconsistent error shapes across routes; errors logged without context | Message/wording inconsistencies with correct handling underneath |
| Architecture (`ARCH`) | The web layer crosses into data access or business rules across a whole component (`NBP-1.2`), or config/secrets are read ad hoc throughout | 4-7 files reaching past a module boundary; one layering violation contained to a feature | ≤3 boundary violations, or a naming/structure inconsistency only |
| Performance (`PERF`) | Event-loop blocking on the request path; an unbounded result set on a growing table; a per-request DB client | A measurable regression on a low-traffic path; a missing cache on a hot, stable read | A micro-optimization with negligible measured effect |
| Testing (`TEST`) | 0% coverage on an auth, payment, or data-write critical path | <40% coverage on a non-critical module; no error-path tests | 40-70% coverage; test-naming or structure issues only |
| Operations (`OPS`) | No graceful shutdown on a deploy path that serves in-flight requests; an EOL Node version; nothing logged on a critical failure path | Unstructured logs or no correlation ID; no health endpoint | Log-noise or level inconsistencies |
| Containerization (`DOCK`) | Secrets baked into image layers; running as root; `npm start` as `CMD` on a service that needs graceful shutdown | devDependencies or source shipped in the final image; no `.dockerignore` | Layer-caching and image-size inefficiencies only |
| Dependency Governance (`DEP`) | A known-CVE, unmaintained (>18 months since last release), or prohibited dependency in active use; no lockfile committed | An approved or maintained alternative exists but isn't adopted; `npm install` in CI instead of `npm ci` | Minor version drift only |

For categories without a natural numeric scale (`CQ`, `API`), keep the qualitative Severity definitions below, but the Metric column must still carry a concrete count (e.g. "9 of 14 routes return an ad hoc error shape") that the stated Severity is visibly consistent with.

## Severity

Rate by actual or plausible **production impact**, not by how much the code offends you. On a backend, impact means data, availability, and confidentiality — not developer taste. For categories with a threshold table above, start from that threshold and adjust only upward with justification.

| Severity | Definition |
|---|---|
| **Critical** | Causes (or will imminently cause) a production incident, data loss or corruption, unauthorized access to another tenant's or user's data, credential/secret exposure, or an exploitable vulnerability reachable from an unauthenticated or low-privilege request. Anything that silently writes wrong data belongs here even if nothing has crashed yet. |
| **High** | Significant, measurable degradation — a broken or badly degraded flow for a meaningful subset of users or data, a security weakness that needs deliberate but plausible conditions to exploit (authenticated IDOR, missing rate limiting on auth endpoints), an unhandled rejection that can crash the process, a query pattern that will not survive realistic load, or a failure mode that is invisible in production (errors swallowed, no logging on a critical path). |
| **Medium** | Real but bounded negative impact — inconsistent error responses that degrade client behavior without breaking it, an N+1 on a low-traffic path, missing tests on a non-critical flow, layering violations that will slow future changes, log noise that impedes debugging. |
| **Low** | Minor consistency or hygiene issues with a measurable but negligible impact. Include only if it is still a concrete, evidenced issue — not a taste preference. |

Two calibration rules specific to backend reviews:

- **A security finding's severity follows exploitability, not the fix's size.** A one-line missing ownership check that exposes other customers' records is Critical, even though the fix is trivial.
- **Don't inflate on absence alone.** "No tests for module X" is Medium unless you can say what breaks unnoticed as a result; "no validation on the public payment webhook" is High/Critical because you can.

## Remediation Effort

Rate by **blast radius and risk of the fix**, not by how long the finding took to write up.

| Effort | Definition |
|---|---|
| **Low** | Localized to one file, route, or handler; no API contract, schema, or migration change; safely done in isolation and easily reverted. |
| **Medium** | Spans a handful of files or one feature area; may require updating call sites, adding middleware, or writing tests, but no cross-cutting architectural change and no data migration. |
| **High** | Cross-cutting refactor touching shared infrastructure (the error-handling layer, the data-access layer, auth, the logger, the Docker/deploy pipeline) used by many modules, or requires a schema/data migration, an API contract change affecting clients, or a phased rollout. |

A fix that changes a database schema, an existing API response shape, or authentication behavior is **at least Medium**, and High if clients or stored data must migrate — regardless of how few lines it touches.

## Priority

Priority is **derived mechanically** from Severity × Effort — don't assign it independently. This keeps priority traceable and auditable (a validator should be able to recompute it from the table below).

| Severity \ Effort | Low | Medium | High |
|---|---|---|---|
| **Critical** | P1 | P1 | P1 |
| **High** | P1 | P1 | P2 |
| **Medium** | P2 | P2 | P3 |
| **Low** | P3 | P3 | P3 |

- **P1** — fix before/alongside the current work; urgent.
- **P2** — fix soon, plan it into near-term work.
- **P3** — backlog-worthy; fix opportunistically or when touching that area anyway.

If a finding's assigned priority doesn't match this table for its stated severity/effort, that's a consistency bug in the report — fix the mismatch, not the table.
