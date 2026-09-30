# Code Review Reports

Two Claude Code skills that audit a repository against **Review Criteria you supply
at invocation time** and deliver an evidence-based **Excel workbook** — every
finding traceable to quoted lines in the real code.

| Skill | Subject | Grounded in | Default output |
|---|---|---|---|
| `react-code-review` | React / Next.js repos | Vercel's 70-rule React performance guide + Cognine React coding standards | `<REPO_NAME>_CODE_QUALITY_V1.xlsx` |
| `node-code-review` | Node.js / TypeScript backends (Express, Fastify, NestJS, Koa, Hapi) | Node.js Best Practices (NBP) + OWASP Top 10 | `CORE_REVIEW_REPORT.xlsx` |

Both emit the same workbook shape: a 12-column findings sheet per codebase
(**Category, Sub Category, Module Name, File / Location, Type, Metric, Evidence,
Observation, Severity, Impact, Recommendation, Priority**), one row per affected
file, followed by a **Threshold Mapping** sheet that states the rule behind every
severity so a reader can check the rating rather than take it on trust.

Both skills refuse to guess their inputs, verify every finding against the file
before it survives into the workbook, and run a pre-delivery validation gate that
reopens the saved `.xlsx` and asserts its structure.

## Install

```
/plugin marketplace add <github-org>/code-review-plugins
/plugin install code-review-reports@code-review-marketplace
```

Restart Claude Code (or run `/doctor`) and confirm `/react-code-review` and
`/node-code-review` appear. Installed skills work in every project — no
per-project setup.

> Upgrading from 1.x? The deliverable changed from Markdown to Excel. Run
> `/plugin marketplace update code-review-marketplace` first, or the install
> resolves to a stale cached 1.0.0.

## Prerequisites

**`openpyxl`** — both skills write `.xlsx`. It is preinstalled in most Claude Code
environments; the skills are told not to `pip install` it. Verify:

```bash
python -c "import openpyxl; print(openpyxl.__version__)"
```

**`node-code-review`** is otherwise self-contained — it bundles its own NBP
reference, the workbook build script, and the threshold catalog.

**`react-code-review`** cites two external reference sources. Without them it
still runs, but it degrades to generic advice and records the gap in the handover
message. Install both:

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

Requires access to the Cognine Azure DevOps org. The version subdirectory is not
hardcoded anywhere — the skill lists the directory and uses whichever version is
present.

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
/node-code-review  --criteria ./criteria.md --repo ../my-api --output ./report.xlsx
```

- Criteria may be a path to a file or inline text. A file is read verbatim.
- Positional form: whichever token resolves to an existing directory is the repo.
- `--output <path>` overrides the default output path.

`react-code-review` keeps two older output modes, used only when asked for by name:

| Flag | Output |
|---|---|
| *(default)* | 12-column quality workbook, per `references/quality-workbook-spec.md` |
| `--layout detailed` | older 11-column workbook with Summary / Key Findings / Detailed Findings sheets |
| `--format md` | Markdown report, per `references/report-template.md` |

`node-code-review` is `.xlsx` only; its `report-template.md` is retained as a
wording reference for Observation / Impact / Recommendation, not as an output
format.

Criteria are the point — the report is criteria-driven, not a generic scan. Write
them as a list of the specific things your team cares about, e.g.:

```markdown
1. Business logic must not live in route handlers or React components.
2. Every API route validates its request body against a schema.
3. No secrets in source; all config via environment variables.
4. Server state is managed by a query library, not hand-rolled useEffect fetches.
```

Criteria headings become the workbook's `Sub Category` column, so write them as
headings you would want to see in a report.

## Layout

```
code-review-reports/
├── .claude-plugin/plugin.json
├── README.md
└── skills/
    ├── node-code-review/
    │   ├── SKILL.md
    │   ├── references/
    │   │   ├── excel-report.md             # THE output spec: 12 columns, formatting, gate
    │   │   ├── node-best-practices.md      # citable NBP rule index + OWASP mapping
    │   │   ├── rubrics.md                  # finding-ID prefixes, severity/effort/priority
    │   │   ├── report-template.md          # retained as a wording reference only
    │   │   └── validation-checklist.md     # evidence/rating/consistency gate
    │   └── scripts/
    │       ├── build_report_xlsx.py        # builds the workbook from a findings JSON
    │       └── threshold_mapping.json      # threshold catalog, one block per review area
    └── react-code-review/
        ├── SKILL.md
        └── references/
            ├── quality-workbook-spec.md    # THE default output spec (12 columns)
            ├── excel-report-spec.md        # older 11-column workbook, `--layout detailed`
            ├── rubrics.md
            ├── report-template.md          # Markdown output, `--format md`
            └── validation-checklist.md
```

`node-code-review` does not hand-write openpyxl code — it writes findings to JSON
and runs `scripts/build_report_xlsx.py`, which validates the data (empty required
cells, unknown Type/Severity/Priority values, a Priority that contradicts its
Severity × Effort) and refuses to build on a failure. The skill invokes it via
`${CLAUDE_PLUGIN_ROOT}`, so it resolves whether the plugin is installed or the
skill folder has been vendored into a repo's `.claude/skills/`.

## Contributing

Edit the `SKILL.md` and `references/*.md` files directly — they are prompts, not
code. `scripts/build_report_xlsx.py` is real code and owns all workbook
formatting; change styling there, never in the prompts.

Four conventions worth preserving:

- **Never introduce a rule or check ID that doesn't exist** in a reference file.
  Both skills instruct Claude to report a gap rather than fabricate a citation.
- **Keep the two skills' workbook shape identical.** The 12 columns, the Threshold
  Mapping sheet, `rubrics.md` and `validation-checklist.md` are deliberately
  parallel across both skills so reports are comparable; change one, change both.
- **Reference bundled files by `${CLAUDE_PLUGIN_ROOT}`**, never by a repo-relative
  `.claude/skills/...` path — that only resolves when the skill is vendored.
- **Bump the version in two places.** `.claude-plugin/plugin.json` and the
  matching entry in the marketplace's `.claude-plugin/marketplace.json`. They must
  match, or installs resolve to a stale cached version.

Don't commit `.zip` packages of the skill folders, or `__pycache__` from running
the build script — the source directories are the truth. `.gitignore` already
excludes both.
