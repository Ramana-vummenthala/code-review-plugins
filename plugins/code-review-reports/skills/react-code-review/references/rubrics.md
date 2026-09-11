# Rubrics: Finding IDs, Severity, Remediation Effort, Priority

These exist so two runs of the skill (or two reviewers) rate the same kind of issue the same way. Don't eyeball a rating — match it against the definition below and be ready to justify it from the definition, not from gut feel.

## Finding ID prefixes

One prefix per category, a 3-digit zero-padded sequence number per prefix, starting at 001 (e.g. `ARCH-001`, `ARCH-002`, `PERF-001`).

| Prefix | Category |
|---|---|
| `ARCH` | Architecture (Cognine `react-architecture` skill: SOLID / Dependency Injection / Separation of Concerns / Modular Architecture / Clean Architecture / DDD, Mandatory vs Recommended) |
| `CQ` | Code Quality (Cognine `react-engineering-standards` skill: naming, folder structure, commenting/docs, modern React patterns) |
| `SEC` | Security (Cognine `react-security` skill, OWASP-grounded, `SEC-*` checks) |
| `PERF` | Performance (Cognine `react-performance` skill, `PERF-*` checks — general/standards-driven performance findings; see `RCT` below for Vercel-rule-driven ones) |
| `DATA` | Data / API / data-fetching layer (Cognine `react-api-standards` skill) |
| `STATE` | State Management (Cognine `react-state-mgmt` skill, `STATE-*` checks) |
| `ERR` | Error Handling (Cognine `react-error-handling` skill, `ERR-*` checks) |
| `TEST` | Testing |
| `RCT` | React Best Practices (Vercel-rule-driven: rendering, re-renders, bundle size, waterfalls — kept distinct from `PERF` above, which is Cognine-standard-driven) |
| `A11Y` | Accessibility (Cognine `react-accessibility` skill, WCAG 2.2 P/O/U/R checks) |
| `DEP` | Dependency Governance (Cognine `react-dependency-governance` skill: approved/prohibited libraries, new-dependency evaluation) |

`ARCH`, `CQ`, `SEC`, `PERF`, `DATA`, `STATE`, `ERR`, `A11Y`, and `DEP` are grounded in the `react-coding-standards-best-practices` (Cognine) plugin loaded per `SKILL.md` Step 2 — cite its numbered check IDs where one exists for the topic, the same way `RCT` findings cite a Vercel rule ID. `TEST` has no dedicated Cognine or Vercel reference; ground it in repository evidence (CI config, coverage config, test-file ratio) as before.

If the supplied Review Criteria name a category not covered above, mint a short, uppercase, single-word prefix consistent with this style, and add it to this table in the delivered report's Review Criteria section so the ID scheme is self-documenting. Never reuse a prefix for two different categories within one report.

## Severity

Rate by actual or plausible **user-facing or production impact**, not by how much the code offends you.

| Severity | Definition |
|---|---|
| **Critical** | Causes (or will imminently cause) a production incident, data loss/corruption, a security vulnerability with real exploit potential, or complete failure of a feature for all users of it. |
| **High** | Significant, measurable degradation — a broken or badly degraded flow for a meaningful subset of users/data, a security weakness that needs deliberate but plausible conditions to exploit, or a performance regression clearly visible to users (e.g. a documented waterfall, a multi-second blocking render). |
| **Medium** | Real but bounded negative impact — unnecessary re-renders or bundle weight in a non-critical path, moderate technical debt that will slow future changes, inconsistent error handling that degrades UX without breaking it. |
| **Low** | Minor stylistic/consistency issues, nice-to-have improvements, or a measurable but negligible impact. Include only if it's still a concrete, evidenced issue — not a taste preference. |

## Remediation Effort

Rate by **blast radius and risk of the fix**, not by how long the finding took to write up.

| Effort | Definition |
|---|---|
| **Low** | Localized to one file or one component; no API/contract/schema changes; safely done in isolation. |
| **Medium** | Spans a handful of files or one feature area; may require updating call sites or tests, but no cross-cutting architectural change. |
| **High** | Cross-cutting refactor touching shared infrastructure (e.g. the http client, a shared store pattern, routing) used by many files, or requires a design decision / migration / phased rollout. |

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
