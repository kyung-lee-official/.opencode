---
name: seaorm-no-auto-migrations
description: Use when editing SeaORM entities or migration files — populate `up`/`down` from entities, fix `migration/src/lib.rs` registration, run `cargo check -p migration` to verify; never run `migrate up/down/fresh/refresh` or live `schema-sync` against a database unless the user explicitly asks in the same turn.
---

# SeaORM — entity-aligned migrations (no auto-apply)

SeaORM `migrate generate` only stubs files — the agent **does** author
migration bodies from entities when needed.

## Allowed

- Edit SeaORM **entity** / model Rust code, query helpers, and connection
  wiring (`Database::connect`, `ConnectOptions`, `State`)
- Edit the **`migration/`** crate to:
  - Fix `lib.rs` registration (`mod` + `vec![Box::new(...)]`) when the CLI
    leaves it incomplete
  - **Populate** pending migration `up` / `down` from current entities
  - Enable migration crate DB features (`sqlx-postgres`,
    `runtime-tokio-rustls`, etc.) when missing
- Run `cargo check -p migration` to verify the migration crate compiles
- Tell the user to run `sea-orm-cli migrate generate` / `migrate up`
  themselves

## When to populate a migration

Do this when any of the following is true:

- The user asks to populate / fill / write a migration
- The user ran or mentions `sea-orm-cli migrate generate` and a stub
  (`todo!()`) exists
- Entities under `apps/<app>/src/entity/` (or shared entity crates)
  changed and a pending migration stub needs to match

Prefer matching **hand-written `Table::create` / indexes / FKs** to entities
(project chose not to rely on Prisma-like auto-diff). Using
`SchemaBuilder::apply` inside `up` is OK only if the user asks for that
style and entities are importable from the migration crate.

## After populating — mandatory self-review

Before finishing, verify and briefly report:

1. Every relevant **entity table** appears in `up`
2. **Columns / types** match entities (`Uuid`, `string` / `string_len(32)`
   for string enums, `timestamptz`, uniques)
3. **FKs** and **unique indexes** match entity relations and
   `apps/<app>/README.md` (or equivalent docs)
4. **`down`** drops tables in reverse dependency order
5. **`migration/src/lib.rs`** lists the migration in `migrations()`
6. **`cargo check -p migration`** succeeds

End the reply with: entities reviewed, what was added, and that the
**user** should run `sea-orm-cli migrate up` (with `DATABASE_URL` set).

## Forbidden (unless the user explicitly asks in this turn)

- CLI that **applies** or resets schema: `migrate up`, `down`, `fresh`,
  `refresh`, or equivalent `cargo run` in `migration` that mutates the DB
- Runtime: `Migrator::up` / `down` / `fresh` / `refresh` against a live DB
  from app code
- Live **`schema-sync`** / `SchemaBuilder::sync` (or equivalent) against a
  database
- `sea-orm-cli generate entity` / DB pull that overwrites entities
  **unless** the user explicitly requested entity generation

`migrate generate` / `migrate init`: prefer telling the user to run them;
the agent may run **non-applying** generate/init only if the user explicitly
asks the agent to run that CLI.

## After entity-only changes

If entities change and no migration stub exists yet: remind the user to run
`sea-orm-cli migrate generate <name>`, then populate and self-review as
above. Do **not** apply migrations yourself.