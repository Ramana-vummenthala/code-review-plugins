# Rubrics: Finding IDs, Severity, Remediation Effort, Priority

These exist so two runs of the skill (or two reviewers) rate the same kind of issue the same way. Don't eyeball a rating — match it against the definition below and be ready to justify it from the definition, not from gut feel.

## Finding ID prefixes

One prefix per category, a 3-digit zero-padded sequence number per prefix, starting at 001 (e.g. `ARCH-001`, `ARCH-002`, `SEC-001`).

**The order of this table is the fixed category order used throughout the report** — in `## Key Findings`, in `## Detailed Findings`, and in the validation checklist.

| # | Prefix | Category | Grounded in |
|---|---|---|---|
| 1 | `ARCH` | Architecture | NBP §1 — component structure, 3-tier layering, config, framework fit, TypeScript discipline |
| 2 | `CQ` | Code Quality | NBP §3 — style, naming, module patterns, lint setup, dead code, duplication |
| 3 | `API` | API Design & Contracts | NBP §2.5, §6.10, §6.14 — route/controller design, request & response schemas, status codes, versioning, pagination, documentation |
| 4 | `DATA` | Data & Persistence | NBP §6.4 + the Prisma/TypeORM/Mongoose cues — queries, transactions, migrations, schema/indexing, connection lifecycle, data integrity |
| 5 | `ASYNC` | Async & Concurrency | NBP §2.1, §2.10, §2.12, §2.13, §3.11 — floating promises, unawaited returns, event-emitter errors, race conditions, concurrency limits, timeouts |
| 6 | `ERR` | Error Handling | NBP §2 — error types, central handling, operational vs programmer errors, logging of failures |
| 7 | `SEC` | Security | NBP §6 + OWASP Top 10 — cite both IDs (`NBP-6.4` / OWASP A03:2021); access-control findings cite OWASP A01 even with no NBP ID |
| 8 | `PERF` | Performance | NBP §7 + evidence-grounded backend performance (N+1, event-loop blocking, unbounded reads, missing caching/indexes) |
| 9 | `TEST` | Testing | NBP §4 — test shape, coverage, isolation, error-path and e2e coverage, CI enforcement |
| 10 | `OPS` | Operations & Observability | NBP §5 — logging, monitoring, health checks, graceful shutdown, config/secrets at deploy time, statelessness, Node version |
| 11 | `DOCK` | Containerization & Deployment | NBP §8 — Dockerfile hygiene, image size and provenance, signal handling, build caching, CI/CD deploy path |
| 12 | `DEP` | Dependency Governance | NBP §5.4, §5.13, §5.19, §6.7, §6.26, §6.27 — lockfiles, vulnerable/outdated/unmaintained packages, licence risk, dependency sprawl |

Cite the specific rule ID from `node-best-practices.md` in the finding's Recommended Improvement (and the OWASP category for `SEC`). A finding with no applicable ID is still valid — ground it in repository evidence and say which practice it violates in words. Never invent an ID.

If the supplied Review Criteria name a category not covered above, mint a short, uppercase, single-word prefix consistent with this style, add it to the end of the order, and document it in the delivered report's Review Criteria section so the ID scheme is self-documenting. Never reuse a prefix for two different categories within one report.

## Severity

Rate by actual or plausible **production impact**, not by how much the code offends you. On a backend, impact means data, availability, and confidentiality — not developer taste.

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
