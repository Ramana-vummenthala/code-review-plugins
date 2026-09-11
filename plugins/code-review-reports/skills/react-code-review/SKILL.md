---
name: react-code-review
description: Audit a React/Next.js repository against a supplied set of Review Criteria and produce an evidence-based "Core Review Report" in Markdown — a structured Key Findings table (severity/effort/priority) plus detailed per-finding writeups, grounded in React/Vercel performance best practices. Use whenever the user asks for a React code review, code audit, architecture/quality review, or a findings report for a React codebase — invoke as `/react-code-review <criteria> <repo-path>`, or trigger conversationally when the user describes wanting this kind of structured review.
license: MIT
metadata:
  author: engineering
  version: "1.0.0"
---

# React Code Review

Produces a "Core Review Report": a structured, evidence-based Markdown audit of a React codebase, scored against a specific set of Review Criteria supplied at invocation time. The point of this skill is that every finding is traceable to real code — never write a finding you haven't verified against the actual repository.

## Why this exists

Generic React advice is cheap and mostly useless to a team that already knows React. What's valuable is a report that says exactly *where* in *this* repository a pattern shows up, quotes the real code, and rates the fix the same way every time it's run. The steps below exist to enforce that discipline — skipping straight to writing findings from general knowledge defeats the purpose.

## Step 1 — Resolve the two required inputs

This skill needs exactly two things: **Review Criteria** and a **Repository** to review. Parse them out of the invocation's arguments:

- Accept explicit flags: `--criteria <path-or-text>` and `--repo <path>`.
- Or two bare positional tokens: `<criteria> <repo-path>` — the token that resolves to an existing directory is the repo; the other is the criteria (a path to an existing file, or inline text if it isn't a path).
- An `--output <path>` flag may override where the report gets written (default is described in Step 6).

If criteria was given as a file path, read the file — don't paraphrase it from the filename. If the repo path doesn't exist, or either input is genuinely missing, **stop and ask the user** rather than substituting "general React best practices" as a stand-in for criteria, or defaulting to the current directory as the repo. The whole value of this report is that it's criteria-driven, not a generic scan; guessing the criteria defeats that.

Sanity-check that the target actually looks like a React project: read its `package.json` and confirm `react` is a dependency. If it isn't, tell the user what you found and confirm before proceeding — don't silently produce a "React" report for a non-React repo.

## Step 2 — Load the React-specific references

This skill grounds its findings in two complementary reference sources. Load both — the second selectively, per the table below — rather than writing findings from general React intuition.

**1. Performance rules — `vercel-react-best-practices`.** Read `~/.claude/skills/vercel-react-best-practices/SKILL.md` (installed globally, Vercel Engineering's 70-rule React/Next.js performance guide across 8 categories: waterfalls, bundle size, server-side performance, client-side data fetching, re-renders, rendering, JS performance, advanced patterns). When a finding matches one of its rules, pull the specific rule file from its `rules/` directory (e.g. `rules/rerender-no-inline-components.md`) and cite the rule ID in the finding — this is what `RCT-*` findings should be grounded in, not vague performance intuition.

**2. Coding standards — `react-coding-standards-best-practices` (Cognine plugin).** Installed as a plugin under a version-pinned cache directory: list `~/.claude/plugins/cache/cognine-plugin-marketplace/react-coding-standards-best-practices/` and use whichever version subdirectory is present (don't hardcode a version number — it will bump over time). It bundles 9 topic skills (`skills/<topic>/SKILL.md`) and 3 audit-command files (`commands/*.md`) that define numbered check IDs and a severity taxonomy for 4 of those topics. Load **only** the topic skill(s) relevant to the supplied Review Criteria (the same tailoring principle Step 3 uses for its Explore agents) — not all 9 unconditionally:

| Prefix | Cognine skill | Numbered checks (if any) |
|---|---|---|
| `ARCH` | `skills/react-architecture/SKILL.md` | `commands/react-coding-architecture-audit-command.md` — BLOCK SOL/DI/SOC/CA/DDD/MOD, classified Mandatory vs Recommended |
| `CQ` | `skills/react-engineering-standards/SKILL.md` | — |
| `SEC` | `skills/react-security/SKILL.md` | `commands/react-secure-coding-practices-command.md` — `SEC-001`..`SEC-010` |
| `PERF` | `skills/react-performance/SKILL.md` | same command — `PERF-001`..`PERF-011` |
| `DATA` | `skills/react-api-standards/SKILL.md` | — |
| `STATE` | `skills/react-state-mgmt/SKILL.md` | same command — `STATE-001`..`STATE-009` |
| `ERR` | `skills/react-error-handling/SKILL.md` | same command — `ERR-001`..`ERR-010` |
| `A11Y` | `skills/react-accessibility/SKILL.md` | `commands/react-coding-accessibility-audit-a11y.md` — P-xx/O-xx/U-xx/R-xx tagged with a WCAG 2.2 success-criterion number |
| `DEP` | `skills/react-dependency-governance/SKILL.md` | — |

When a finding matches a topic with numbered checks, cite the specific check ID (e.g. `SEC-004`, `PERF-002`, `WCAG O-03`) the same way `RCT-*` findings cite a Vercel rule ID — this is what makes the finding standards-grounded rather than generic advice. For `ARCH` findings specifically, also state whether the violated principle is Mandatory or Recommended per the Cognine Architecture Principles Matrix, and let that inform Severity: a Mandatory violation should not be rated below Medium.

**Do not invoke the 3 Cognine audit-commands as sub-processes.** They are separate slash-commands with their own report files (`architecture-audit-report.md`, `coding-practices-review-report.md`) and would fragment output across multiple reports; use only their skill docs and numbered-check-ID content as citation material inside this skill's single Core Review Report.

If either reference is not present in this environment (check paths first), do not fail silently: note the gap explicitly in the report's Review Scope section and fall back to well-established, citable practices for that area instead of fabricating rule or check IDs.

## Step 3 — Explore the repository against the criteria

If the target repo has a root `CLAUDE.md`, read it first — it may already document the architecture and save you from re-deriving it.

Launch up to 3 parallel `Explore` agents to cover the repo's surface area, tailored to what the supplied criteria actually ask about (don't run all three if the criteria only concern one area):

1. **Architecture & structure** — routing, project layout, module boundaries, how the app is composed.
2. **State & data** — state management approach, data fetching, API/service layer, hooks.
3. **Component design & quality** — component patterns, rendering behavior, styling approach, error handling, and testing conventions.

Give each agent the actual list of Review Criteria (not a summary) and ask it to report back **findings already tied to file paths and code**, mapped to whichever criterion each observation relates to — not generic prose about what it saw. An agent that returns "state management could be improved" without a file reference is not useful input to this report; push back and re-run narrower if that happens.

## Step 4 — Synthesize findings

For each distinct issue:

- Assign a Finding ID using the prefix table in `references/rubrics.md` (`ARCH-001`, `PERF-002`, etc.).
- If multiple observations share one root cause, merge them into a single finding and list every affected file under "Files Affected" — don't create near-duplicate findings for the same underlying pattern repeated across files.
- Rate Severity, Remediation Effort, and Priority using `references/rubrics.md` — Priority is derived mechanically from Severity × Effort there, not chosen independently.
- Map the finding to the specific supplied criterion it addresses.

Track two things explicitly as you go, because the spec requires stating them rather than letting them go unmentioned:
- Which supplied criteria produced **no findings** (note in the report that they were evaluated and passed, don't just omit them).
- Which supplied criteria **could not be evaluated** at all from what's in the repository (e.g. a criterion about CI-enforced test coverage when there's no CI config) — these go in a "Criteria Not Evaluated" subsection with the concrete reason, never silently dropped or guessed at.

## Step 5 — Verify every finding before it goes in the report

Before a finding is written into the draft, re-open the cited file and confirm the pattern is really there — quote or closely paraphrase the actual lines, don't reconstruct them from memory of the exploration summary. Confirm the recommendation actually addresses what's shown, not an adjacent concern. This mirrors how this environment's own `code-review` skill treats findings: generate candidates, then adversarially verify each one against the real code before it's allowed to survive into the output. Drop or downgrade anything that doesn't hold up under this check — a report with fewer, verified findings is more valuable than one padded with plausible-sounding guesses.

## Step 6 — Write the report

Follow `references/report-template.md` exactly: heading structure, the category-grouped 13-column Key Findings tables, and the per-finding detailed-writeup shape. Findings are grouped into `###` sections by category (rubric prefix-table order), not laid out as one flat priority-ordered table — within each category section, order rows by Priority (P1 first), then Severity. The `## Recommendations` section remains the intentional place for a cross-category, priority-first view.

Write the file to `<repo-path>/CORE_REVIEW_REPORT.md` unless the user passed `--output <path>`. Tell the user the path you wrote to.

## Step 7 — Run the pre-delivery validation gate

Before showing anything to the user, go through `references/validation-checklist.md` item by item against the drafted report. If any item fails, fix the report and **re-run the full checklist again** — a fix in one place can break consistency elsewhere (e.g. correcting a Priority value needs re-checking the row is still sorted correctly). Only present the report once every item passes. If you had to fix something, mention it briefly when handing the report over — it's a useful signal the gate actually did something, not just theater.

## Reference files

- `references/rubrics.md` — Finding-ID prefixes (including which are grounded in the Cognine `react-coding-standards-best-practices` plugin), and the Severity / Remediation Effort / Priority definitions (Priority is derived from the other two — always recompute, never assign by feel).
- `references/report-template.md` — the literal report structure to fill in, including a worked example of one fully-written finding, category-grouped.
- `references/validation-checklist.md` — the mandatory pre-delivery gate from Step 7, transcribed as pass/fail items.
