# opencode workspace config

Config is loaded once when opencode starts; **restart opencode** after editing
any file in `.opencode/`.

## Workflow

This repo is meant to be reused across repositories via **sparse checkout**:
each consumer pulls only the skills it needs. Commands below use
**PowerShell** (Windows). On bash/zsh, replace the trailing `` ` `` with `\`.

### Bootstrap in a consumer repo

`git clone --sparse` creates the `.opencode/` folder itself — no `mkdir` /
`cd` needed:

```powershell
# From the consumer repo root (SSH or HTTPS)
git clone --sparse --filter=blob:none git@github.com:kyung-lee-official/.opencode.git
cd .opencode

# Select the relevant skills (cone mode = directory-level)
git sparse-checkout add `
    methodology/<skill> `
    stacks/<language>/<skill> `
    stacks/<framework>/<skill>
```

Notes:

- **`--filter=blob:none`** skips downloading file blobs for paths you don't
  select; the rest are fetched on demand.
- **Cone mode accepts directories only.** Each entry selects a skill
  directory; the root files (`AGENTS.md`, `opencode.json`, `README.md`) are
  always included automatically, so never list them here.
- The consumer's `.gitignore` should ignore `.opencode/` so the nested repo
  stays isolated.

### Add or remove a skill later

```powershell
cd .opencode

# Add another skill directory
git sparse-checkout add stacks/<language>/<skill>

# Drop one: list only what should remain
git sparse-checkout set stacks/<language>/<skill>

# Materialize the change
git checkout
```

### Pull upstream changes

```powershell
cd .opencode
git pull
```

### Author a new skill and publish back

```powershell
cd .opencode
# create stacks/<language>/<name>/SKILL.md (with `name` + `description` frontmatter)
git add stacks/<language>/<name>
# wait for explicit user approval before committing
git commit -m "..."
git push origin main
```

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
    ├── elysiajs/                                 # complete backend framework (apps/api)
    │   └── elysiajs-convention/SKILL.md
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

On Windows, set `--path` to an **absolute path with doubled backslashes**
inside the JSON string (e.g. `"--path", "C:\\path\\to\\root"`). The default
`"."` works on any OS, so this only matters if you point codegraph at a
different root.

## Per-developer notes

There is no separate `mcp.json` and no machine-specific gitignore: every
developer's settings live in `.opencode/opencode.json` and are committed. If
you must keep a setting local, prefer environment variables in the
`mcp[name].environment` block (opencode supports `{env:VAR}` interpolation in
header values; shell-style `${VAR}` is **not** substituted).

If you previously kept settings under another editor's dot-folder, that
folder is independent from `.opencode/` — opencode reads only its own. Move
MCP entries into `mcp` in `opencode.json` and equivalent always-on guidance
into `AGENTS.md` or a skill.

### Adding more MCP servers

Add another key under `mcp` with the same shape (`type`, `command` as an
array, optional `environment`). Remote servers use `type: "remote"` with
`url` and optional `headers`. Refer to each server's docs for required
arguments.