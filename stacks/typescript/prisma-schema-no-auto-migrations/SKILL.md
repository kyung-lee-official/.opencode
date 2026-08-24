---
name: prisma-schema-no-auto-migrations
description: Use when editing `prisma/schema.prisma` or running Prisma CLI — agents may edit the schema and run `prisma generate` only, and must never run `migrate *` / `db push` / `db pull` / `migrate diff` or author anything under `prisma/migrations/`.
---

# Prisma — schema and generate only (no migrations)

Run Prisma CLI from the **directory that contains `schema.prisma`**. Nothing
else under Prisma may be executed or authored by the agent.

**Allowed:** edit `schema.prisma`; run `prisma generate` when the client must
match the schema.

**Forbidden (no exceptions — including if the user asks in the same task):**

- `migrate *`, `db push`, `db pull`, `migrate diff`, `migrate status` (for
  migration workflow), or any command that applies, creates, or records
  migrations
- Create, edit, or delete under `prisma/migrations/`; migration or drift-fix
  SQL for the user to apply
- Config changes solely to enable agent migrations (`shadowDatabaseUrl`,
  migration paths, automation scripts)

**After `schema.prisma` changes:** run `prisma generate` if TypeScript needs
it, then **stop**. Tell the user to migrate manually (e.g.
`prisma migrate dev --name describe_your_change`).