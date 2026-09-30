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

## Module Name (functional module)

The Module Name column answers **"which part of the product is this?"**, not "which file is this?". It is a functional/feature module name a product owner would recognise: `Login`, `Registration`, `Dashboard`, `Profile`, `Checkout`, `Orders`, `Reporting`, `Notifications`, `Admin`. The file path lives in the separate `File / Location` column, so never put a file name, a folder path, or a component name in Module Name.

### Deriving the module list

Derive the list from the repository during exploration (Step 3), in this order of preference, and record the result in the report's `## Review Scope` section so the reader can see the taxonomy the findings are grouped by:

1. **Routes** — the router config or the `app/` / `pages/` directory. A route group like `/login`, `/dashboard`, `/profile/:id` usually names the module directly.
2. **Feature folders** — `src/features/*`, `src/modules/*`, `src/pages/*`. Convert the folder name to a readable product name (`user-profile` becomes `Profile`).
3. **Navigation structure** — the primary nav/sidebar items in the shell component, which usually mirror how the business talks about the app.
4. **Domain naming in the code** — store slices, API service groupings, or a documented architecture in the repo's `CLAUDE.md` / README.

Keep the list small and stable: prefer 5 to 15 modules for a typical app. Don't mint a new module per file, and don't re-derive a different list halfway through the report — the same file always maps to the same module in every row it appears in.

### Fallback labels

Plenty of code belongs to no single feature. Use one of these fixed labels rather than forcing a feature name or leaving the cell blank:

| Label | Use for |
|---|---|
| **Cross-cutting** | Code used by most or all feature modules: the http client, auth interceptors, error boundaries, the root store, providers, routing config. |
| **Shared UI** | The shared/design-system component library (`src/components/ui/**`, `src/shared/**`) consumed across modules. |
| **Build & Tooling** | Config, bundler, lint, CI, and dependency-manifest findings (most `DEP` findings land here). |
| **Application Shell** | The app entry point, layout, navigation chrome, and providers that wrap every module. |

If a finding genuinely spans several modules and isn't infrastructure, still write one row per file and give each row the module that file belongs to — that's what makes the table show which parts of the product are affected.

## Type

Type classifies the code named in the row's `File / Location` cell (not the functional module) as exactly one of:

| Type | Definition |
|---|---|
| **Component** | A React component — a file or exported function that returns JSX and is rendered as an element (`<Foo />`). |
| **Function** | A hook, controller function, mapper function, or plain utility — callable code with no JSX, e.g. `usePaymentIframe`, a `*Controller.ts` function, a `*Mapper.ts` function. |
| **Module** | A whole file/store/service/pattern that isn't one function or component — e.g. a Zustand `*Store.ts`, a router config file, an axios interceptor chain, or a pattern cited across a feature folder as a unit. |

Note the two senses of the word in this report: the **Module Name** column is the functional module (a product area), while a **Type** of `Module` is a code-shape classification for the file in `File / Location`. A row reading `Module Name: Checkout` with `Type: Component` is normal, not a contradiction.

## Metric

Every Key Findings row's Metric column is a real, quantifiable measurement pulled from the repository — a count, a percentage, a size delta, a compliance level — never a restatement of the Observation in prose. Use the literal text `N/A` only when no metric genuinely applies to that finding (e.g. a naming-convention inconsistency with nothing to count); it must never stand in for a metric you simply didn't measure.

Representative metrics by category:

| Category | Example Metric values |
|---|---|
| Architecture (`ARCH`) | "8 inbound cross-feature imports", "3 responsibilities in one file" |
| Code Quality (`CQ`) | "480-line file", "cyclomatic complexity ~15" |
| Security (`SEC`) | "1 endpoint missing input validation", "`SEC-004` failed" |
| Performance (`PERF`/`RCT`) | "4 re-renders/keystroke", "bundle +180KB", "3-request waterfall" |
| Data / API (`DATA`) | "3 duplicate fetch implementations for the same endpoint" |
| State Management (`STATE`) | "2 competing sources of truth for the same value" |
| Error Handling (`ERR`) | "0% of catch blocks logged or surfaced to the user" |
| Testing (`TEST`) | "12% line coverage", "0 tests" |
| Accessibility (`A11Y`) | "WCAG 2.2 AA — 3 violations" |
| Dependency Governance (`DEP`) | "1 unapproved dependency", "18 months since last release" |

### Metric → Severity thresholds

Severity must meet or exceed the threshold implied by the Metric value below — these are a floor, not a ceiling. A reviewer may rate higher given genuine production risk, but never lower, and any deviation from the table must be justified in the finding's Impact text.

| Category | High (or above) | Medium | Low |
|---|---|---|---|
| Performance (`PERF`/`RCT`) | ≥3 avoidable re-renders/interaction on a hot path; bundle delta ≥300KB; a documented waterfall of sequential requests that could parallelize | 1-2 avoidable re-renders; bundle delta 100-299KB | Bundle delta <100KB; a re-render off the critical path |
| Architecture (`ARCH`) | ≥8 inbound cross-feature imports; violates a **Mandatory** principle (Critical if it also breaks a production flow) | 4-7 inbound cross-feature imports; violates a **Recommended** principle | ≤3 inbound cross-feature imports |
| Accessibility (`A11Y`) | Any WCAG 2.2 **A**-level failure | Any WCAG 2.2 **AA**-level failure | **AAA**-level or best-practice-only gap |
| Security (`SEC`) | Failed check with a real, reachable exploit path (Critical if user data/production is directly exposed) | Failed check that needs deliberate but plausible conditions to exploit | Hardening/defense-in-depth gap with no direct exploit path |
| Testing (`TEST`) | 0% coverage on a payment- or auth-critical module | <40% coverage on a non-critical module | 40-70% coverage |
| Dependency Governance (`DEP`) | Prohibited, unmaintained (>18 months since last release), or known-CVE dependency in active use | An approved alternative exists but isn't adopted yet | Approved dependency, minor version drift only |

For categories without a natural numeric scale (`CQ`, `DATA`, `STATE`, `ERR`), keep the qualitative Severity definitions below, but the Metric column must still carry a concrete count (e.g. "3 duplicate fetch implementations") that the stated Severity is visibly consistent with.

## Severity

Rate by actual or plausible **user-facing or production impact**, not by how much the code offends you. For categories with a threshold table above, start from that threshold and adjust only upward with justification.

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
