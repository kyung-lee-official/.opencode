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
├── opencode.json                                 # $schema, instructions, mcp
├── README.md                                     # this file
├── AGENTS.md                                     # always-on root instructions
├── methodology/                                  # stack-agnostic theory / workflow
│   ├── authoring-opencode/SKILL.md              # meta: how to write skills/agents/instructions
│   ├── codegraph/SKILL.md                       # CodeGraph MCP workflow
│   └── dev-and-debug/SKILL.md                   # Windows dev, bun/cargo/curl.exe
└── stacks/
    ├── typescript/                               # generic TS + TS-related tool/frontend conventions
    │   ├── typescript-convention/SKILL.md
    │   ├── frontend-convention/SKILL.md         # React/Next.js patterns (RHF/Zod/TanStack Query)
    │   ├── exceljs-xlsx-conventions/SKILL.md    # TS tool
    │   └── prisma-schema-no-auto-migrations/SKILL.md  # TS tool
    ├── rust/                                     # generic Rust + Rust-related tool conventions
    │   ├── rust-convention/SKILL.md
    │   ├── seaorm-no-auto-migrations/SKILL.md   # Rust ORM tool
    │   └── calamine-rust-xlsxwriter-conventions/SKILL.md  # Rust xlsx peer libs
    ├── nestjs/                                   # complete backend framework
    │   └── nestjs-convention/SKILL.md
    ├── elysiajs/                                 # complete backend framework (apps/api) — reserved
    └── axum/                                     # complete backend framework
        └── axum-convention/SKILL.md
```

### Folder rules

- **`methodology/`** — stack-agnostic theory and workflow (no syntax, no
  framework API). Tools like `dev-and-debug` live here because they describe
  *how to develop*, not a specific language.
- **`stacks/<language>/`** — generic language rules plus any tool/frontend
  conventions that are specific to that language (e.g. `prisma` and `exceljs`
  under TS, `seaorm` under Rust). Skills share the language folder regardless
  of whether they target syntax, frontend patterns, or a library.
- **`stacks/<framework>/`** — only **complete backend frameworks** earn a
  top-level slot (NestJS, ElysiaJS, Axum). Frontend frameworks like Next.js
  are not split out — their conventions live under `stacks/typescript/`.
- **`stacks/<framework>/` with no skill yet** — the folder exists on disk but
  is empty. Git won't track an empty directory; the folder is created when
  its first `SKILL.md` lands. Add a `.gitkeep` if you want it reserved
  visibly in version control.

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