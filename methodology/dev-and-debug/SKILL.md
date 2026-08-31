---
name: dev-and-debug
description: Use when running scripts, debugging, or hitting local services on Windows — picks the toolchain by stack (bun/moon/cargo), uses `curl.exe --max-time` instead of `Invoke-WebRequest`, `memurai-cli` for local Redis, and read-only DB access via `DATABASE_URL`.
---

# Dev and debug (Windows)

## Pick the toolchain by stack

Use the runner that matches the code you're touching. Do **not** default
everything to Bun in a Rust workspace (or vice versa).

| Area                                           | Prefer                                                                     | Fall back                                                                        |
| ---------------------------------------------- | -------------------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| TypeScript / JS packages, Nest, Next, frontend | **`bun`** / **`bunx`**                                                     | npm / pnpm / yarn only if the project or task requires it, or Bun is unavailable |
| Rust crates / Rust workspaces                  | **`moon`** for workspace tasks; **`cargo`** for crate-level build/test/run | —                                                                                |
| Mixed change                                   | Run the tool for **each** affected stack                                   | —                                                                                |

Examples:

```powershell
# TypeScript
bun install
bun run <script>

# Rust / moon
moon run <workspace-target>:dev
cargo check -p <crate>
cargo test -p <crate>
```

## Shell commands

- Development is on **Windows** with **PowerShell** unless the user specifies
  otherwise.
- Chain commands with **`;`** instead of **`&&`**; avoid bash-only syntax unless
  the user is clearly in WSL/Git Bash.
- Use paths and quoting that work on Windows.
- Ensure **`cargo`** / **`moon`** / **`bun`** are on `PATH` (e.g.
  `$env:USERPROFILE\.cargo\bin`) when invoking them from agent shells.
- **`curl` in PowerShell is `Invoke-WebRequest`** — not real curl. For HTTP
  probes (health checks, quick API tests), use **`curl.exe`** and always set a
  timeout so a dead port does not hang the shell:

```powershell
curl.exe -s --max-time 5 http://localhost:3001/health
```

Do not use bare `curl -s http://localhost:...` in PowerShell agent commands.

## Redis CLI

Use **`memurai-cli`** for local Redis on Windows (compatible with
**`redis-cli`** — same arguments for typical operations). Use **`redis-cli`**
only when docs or the environment explicitly require it.

```powershell
memurai-cli -u redis://localhost:6379
memurai-cli KEYS 'prefix:*'
```

If repo docs or scripts say `redis-cli`, run the same command with
**`memurai-cli`** locally.

## Database access

- **Read-only** DB access is OK for debugging. Load the connection URL from
  project env files (e.g. `.env`, `.env.local`, `DATABASE_URL`). Do **not**
  modify data without explicit permission.
- Load env for scripts:
  - TypeScript: `bun --env-file=.env run scripts/your-script.ts`
  - Rust: rely on process env, or load `.env` via the app (`dotenvy`) / shell
    before `cargo` / `moon`.

## Temporary debug scripts

- Prefer throwaway scripts under **`scripts/`** (repo root when the project
  has one).
- **TypeScript:** **`bun`** + **`pg`** (raw queries) or the project's **ORM
  client**; run with `bun run scripts/your-script.ts` (add `--env-file` when
  DB access is needed).
- **Rust:** a small binary or `examples/` / `scripts/*.rs` via `cargo run` /
  `cargo script` only if already used in the repo; prefer `cargo check` /
  short-lived `main` experiments you delete after.
- Delete scripts when the investigation is complete unless the user asks to
  keep them.
