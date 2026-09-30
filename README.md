# Code Review Plugins

A Claude Code plugin marketplace for engineering teams.

## Add the marketplace

```
/plugin marketplace add <github-org>/code-review-plugins
```

## Plugins

| Plugin | Version | Description |
|---|---|---|
| [`code-review-reports`](./plugins/code-review-reports) | 2.0.0 | Criteria-driven **code quality workbook** (`.xlsx`) skills for React/Next.js and Node.js/TypeScript repos |

```
/plugin install code-review-reports@code-review-marketplace
```

See the [plugin README](./plugins/code-review-reports/README.md) for prerequisites and usage.

## Repository layout

```
.claude-plugin/
└── marketplace.json                 # marketplace manifest — lists the plugins below
plugins/
└── code-review-reports/
    ├── .claude-plugin/plugin.json   # plugin manifest
    ├── README.md
    └── skills/
        ├── node-code-review/         # SKILL.md + references/ + scripts/
        └── react-code-review/        # SKILL.md + references/
```

## Adding a plugin

1. Create `plugins/<name>/` with a `.claude-plugin/plugin.json` manifest.
2. Add `skills/`, `commands/`, `agents/`, or `hooks/` subdirectories as needed.
   Inside a skill, reference bundled files by `${CLAUDE_PLUGIN_ROOT}/skills/<name>/...`,
   never by a repo-relative `.claude/skills/...` path.
3. Register it in the `plugins` array of `.claude-plugin/marketplace.json`.
4. Bump the `version` in **both** the plugin manifest and the marketplace entry —
   they must match, or installs resolve to a stale cached version.
