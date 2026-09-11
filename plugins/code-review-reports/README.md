# Code Review Reports

Two Claude Code skills that produce a **Core Review Report** — a structured,
evidence-based Markdown code audit scored against Review Criteria you supply at
invocation time:

| Skill | Subject | Grounded in |
|---|---|---|
| `react-code-review` | React / Next.js repos | Vercel's 70-rule React performance guide + Cognine React coding standards |
| `node-code-review` | Node.js / TypeScript backends (Express, Fastify, NestJS, Koa, Hapi) | Node.js Best Practices (NBP) + OWASP Top 10 |

Both emit the same report shape: a category-grouped Key Findings table with
Severity / Remediation Effort / derived Priority, plus per-finding writeups where
every finding is traceable to real code in the repo under review.

## Install

```
/plugin marketplace add <github-org>/code-review-plugins
/plugin install code-review-reports@code-review-marketplace
```

Restart Claude Code (or run `/doctor`) and confirm `/react-code-review` and
`/node-code-review` appear. Installed skills work in every project — no
per-project setup.

## Prerequisites

`node-code-review` is self-contained — it bundles its own NBP reference and
works with no extra setup.

`react-code-review` cites two external reference sources. Without them it still
runs, but it degrades to generic advice and records the gap in the report's
Review Scope section. Install both:

**1. Vercel React best practices** (expected at `~/.claude/skills/vercel-react-best-practices/`)

```
/plugin marketplace add anthropics/skills
```
then install the `vercel-react-best-practices` skill from that marketplace.

**2. Cognine React coding standards** (expected at
`~/.claude/plugins/cache/cognine-plugin-marketplace/react-coding-standards-best-practices/<version>/`)

```
/plugin marketplace add https://dev.azure.com/cognine/Cognine-COE/_git/cognine-coe-coding-standards
/plugin install react-coding-standards-best-practices@cognine-plugin-marketplace
```

Requires access to the Cognine Azure DevOps org.

Verify both resolved:

```bash
ls ~/.claude/skills/vercel-react-best-practices
ls ~/.claude/plugins/cache/cognine-plugin-marketplace/react-coding-standards-best-practices
```

## Usage

Both skills take exactly two inputs — **Review Criteria** and a **repository** —
and refuse to guess either one:

```
/react-code-review ./review-criteria.md ../my-react-app
/node-code-review  --criteria ./criteria.md --repo ../my-api  --output ./report.md
```

- Criteria may be a path to a file or inline text. A file is read verbatim.
- Positional form: whichever token resolves to an existing directory is the repo.
- Default output is `<repo-path>/CORE_REVIEW_REPORT.md`.

Criteria are the point — the report is criteria-driven, not a generic scan. Write
them as a list of the specific things your team cares about, e.g.:

```markdown
1. Business logic must not live in route handlers or React components.
2. Every API route validates its request body against a schema.
3. No secrets in source; all config via environment variables.
4. Server state is managed by a query library, not hand-rolled useEffect fetches.
```

## Layout

```
code-review-reports/
├── .claude-plugin/plugin.json
├── README.md
└── skills/
    ├── node-code-review/
    │   ├── SKILL.md
    │   └── references/
    │       ├── node-best-practices.md      # citable NBP rule index + OWASP mapping
    │       ├── rubrics.md                  # finding-ID prefixes, severity/effort/priority
    │       ├── report-template.md          # literal report structure
    │       └── validation-checklist.md     # pre-delivery gate
    └── react-code-review/
        ├── SKILL.md
        └── references/
            ├── rubrics.md
            ├── report-template.md
            └── validation-checklist.md
```

## Contributing

Edit the `SKILL.md` and `references/*.md` files directly — they are prompts, not
code. Three conventions worth preserving:

- **Never introduce a rule or check ID that doesn't exist** in a reference file.
  Both skills instruct Claude to report a gap rather than fabricate a citation.
- **Keep the two skills' report shape identical.** `rubrics.md`,
  `report-template.md`, and `validation-checklist.md` are deliberately parallel
  across both skills so reports are comparable; change one, change both.
- **Bump the version in two places.** `.claude-plugin/plugin.json` and the
  matching entry in the marketplace's `.claude-plugin/marketplace.json`.

Don't commit `.zip` packages of the skill folders — the source directories are
the truth, and archives drift. `.gitignore` already excludes them.
