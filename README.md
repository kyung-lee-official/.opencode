# opencode workspace config

This folder replaces the prior `.cursor/` setup. `.cursor/` is **left in place**
for the Cursor IDE; opencode reads its own config from `.opencode/` and never
looks at `.cursor/`.

opencode does **not** have a "rules" concept the way Cursor does. Instead:

| Cursor (old)              | opencode (new)                                                 |
| ------------------------- | -------------------------------------------------------------- |
| `rules/*.mdc` (always on) | `AGENTS.md` (referenced via `instructions` in `opencode.json`) |
| `rules/*.mdc` (scoped)    | `skills/<name>/SKILL.md` (loaded by description trigger)       |
| `mcp.json`                | `mcp` block in `opencode.json`                                 |
| `.gitignore` for `mcp.json` | not needed — config lives in version-controlled JSON          |

Config is loaded once when opencode starts; **restart opencode** after editing
any file in `.opencode/`.

## Layout

```text
.opencode/
├── opencode.json                     # $schema, instructions, mcp
├── README.md                         # this file
├── AGENTS.md                         # always-on root instructions
└── skills/
    ├── authoring-opencode/
    │   └── SKILL.md                  # how to write skills/agents/instructions
    ├── codegraph/SKILL.md            # CodeGraph MCP workflow
    ├── dev-and-debug/SKILL.md        # Windows dev, bun/cargo/curl.exe
    ├── prisma-schema-no-auto-migrations/SKILL.md
    ├── typescript-convention/SKILL.md
    ├── frontend-convention/SKILL.md
    ├── axum-convention/SKILL.md
    ├── nestjs-convention/SKILL.md
    ├── rust-convention/SKILL.md
    ├── calamine-rust-xlsxwriter-conventions/SKILL.md
    ├── exceljs-xlsx-conventions/SKILL.md
    └── seaorm-no-auto-migrations/SKILL.md
```

## MCP servers

The `opencode.json` ships a **disabled** `codegraph` entry as a template. To
enable it for this machine:

1. Install the CodeGraph CLI (`codegraph`) so it is on `PATH`.
2. Edit `.opencode/opencode.json` and set `mcp.codegraph.enabled: true`. The
   `--path "."` already points at the workspace root — change it if you want a
   different root.
3. Restart opencode.

CodeGraph exposes tools named `mcp__codegraph__*` (search, callers, callees,
trace, impact, node, context, explore, files, status). See the `codegraph`
skill for which tool fits which question.

## Per-developer notes

There is no separate `mcp.json` and no machine-specific gitignore: every
developer's settings live in `.opencode/opencode.json` and are committed. If
you must keep a setting local, prefer environment variables in the
`mcp[name].environment` block (opencode supports `{env:VAR}` interpolation in
header values; shell-style `${VAR}` is **not** substituted).