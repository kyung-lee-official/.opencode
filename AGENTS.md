# Coding and documentation conventions

Language-agnostic rules for this monorepo. Stack-specific details live in the
opencode skills **`typescript-convention`**, **`rust-convention`**,
**`frontend-convention`**, and the Axum / NestJS / Excel skills.

## Prepositions: `from` vs `by`

Use **`from`** when a value is **read off** a source already in scope (field,
row, constant, Excel column). Use **`by`** when a value is **found using**
key(s), a query, or an algorithm.

|                      | **`from`**                                                             | **`by`**                                                                                    |
| -------------------- | ---------------------------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| **Meaning**          | Provenance — value is on the object/row/result                         | Lookup — resolve, match, `findFirst`, `Map.get` / `HashMap::get`, prefix rules              |
| **Code names**       | `*From*` / `*_from_*` — `statusFromImportLabel`, `status_from_row`     | `*By*` / `*_by_*` — `findUserByEmail`, `find_user_by_email`                                 |
| **Maps**             | `fooFromBar` / `foo_from_bar` only if values are copied with no lookup | `fooByBar` / `foo_by_bar` for keyed maps                                                    |
| **Prose / comments** | get `orderId` from `record`; set `createdAt` from Excel `Created At`   | get `record` by `orderId`; get `category` by `platform` and `orderId` via `resolveCategory` |

**Heuristic:** lookup / resolve / match → **`by`** (+ key). Field access, ingest
column, "already on `record`" → **`from`**.

**Prose:** use verbs (`get`, `set`, `call`, `read`, `write`, `build`,
`validate`, `return`, `skip`, `append`) — not arrow glyphs (`→`, `←`, `->`) for
flow. Mermaid: `-->` is edge syntax only; mapping tables use **Source** /
**Target**, not arrows in cells.

```typescript
// ✅ resolve BY keys; read field FROM record; Map.get is BY key
function resolveCategoryByPlatformAndOrderId(...) { ... }
const orderId = record.orderId;
const category = categoryBySku.get(sku) ?? "";

// ❌ BAD — name says "from" but body is Map.get
function categoryFromSku(sku: string) {
  return catalog.get(sku)?.category;
}
```

```rust
// ✅ lookup BY key; field FROM row
fn find_user_by_email(email: &str) -> Option<User> { ... }
let order_id = record.order_id;
let category = category_by_sku.get(sku).cloned().unwrap_or_default();

// ❌ BAD — name says "from" but body is a map lookup
fn category_from_sku(sku: &str) -> Option<&str> {
    catalog.get(sku).map(|e| e.category.as_str())
}
```

- Lookup-key parameters: `orderId` / `order_id`, `email` (no `From` / `_from_`
  on the parameter).
- Resolver results: `resolution`, `layers` — not `categoryFromOrderId` /
  `category_from_order_id` when prefix rules were used.

---

## Avoid thin wrappers

Do **not** add a function that only forwards the same arguments with no extra
logic (validation, logging, stable public boundary, or breaking a circular
import). **OK:** DTO shaping, import-cycle break, added behavior.
**Heuristic:** if deleting it leaves only `return otherFn(sameArgs)`, inline
the call. Search for an existing resolver before adding a new export.

| Do                                              | Don't                                        |
| ----------------------------------------------- | -------------------------------------------- |
| Canonical implementation; map args at call site | `fooUnified(x) { return fooCore(x); }`       |
| Adapter at a real boundary                      | Duplicate exports that differ only by prefix |

```typescript
// ❌ BAD — rename-only wrapper; map fields at call site instead
function columnsFromResolution(r: Resolution) {
  return { category: r.cat, layer1: r.l1 };
}
```

```rust
// ❌ BAD — rename-only wrapper
fn columns_from_resolution(r: &Resolution) -> Columns {
    Columns { category: r.cat.clone(), layer1: r.l1.clone() }
}
```

---

## Peer shared code — extract up, name generically

When **two or more peer modules** (siblings, neither owns the other) need the
same snippet, put it under a neutral path and import from each peer. Do **not**
import across a peer boundary. Name exports by **what they do**, not the first
consumer (`resolve_country_from_import_row`, not
`module_a_resolve_country`).

| Stack      | Neutral home                                                           |
| ---------- | ---------------------------------------------------------------------- |
| TypeScript | `helpers/`, `shared/`, `utils/`                                        |
| Rust       | shared crate under `packages/` (or a clear `shared` / `common` module) |

Extract when both peers need it and there is no single owner; keep in one peer
when only one uses it (YAGNI) or B layers on A intentionally.

---

## String normalization — trim before save, trust on read

**Trim (and validate) string fields once at ingest** (Excel, HTTP bodies,
`create` / `update`). Do **not** re-trim or case-fold on read for business
logic or map lookup.

| Stage                    | Rule                                                                         |
| ------------------------ | ---------------------------------------------------------------------------- |
| **Ingest / save**        | Trim here; persist normalized strings only                                   |
| **Read / resolve**       | Use DB strings as stored; map get with exact literals                        |
| **Case-fold / prefixes** | Only where matching requires it (e.g. ASCII id prefixes), not on every label |

```typescript
// ✅ trim on ingest; lookup uses stored literal
sku: row.getCell(col).text.trim(),
return skuMap.get(sku);

// ❌ re-trim values already from DB
return skuMap.get(sku.trim().toLowerCase());
```

```rust
// ✅ trim on ingest
let sku = raw_sku.trim().to_string();
sku_map.get(&sku);

// ❌ re-trim on read for lookup
sku_map.get(sku.trim().to_lowercase().as_str());
```

Use normalize helpers on runtime inputs not guaranteed trimmed (API keys,
mixed catalog keys) — not on fields whose contract is "trimmed at save".
Hardcoded map keys must match post-ingest DB strings. Enum routers: branch on
exact stored literals.

---

## Docs and comments

Write **markdown** (README, guides, skills) and **code comments** in
**English**. Refer to codebase symbols by **actual definitions** — not loose
labels.

- **Literals:** keep non-English text **verbatim** when code/import expects it
  (header enums, Excel columns, UI sentinels). Quote in backticks; do not
  translate in docs.
- **Symbols:** use `` `Model.field` ``, `` `function_name()` ``,
  `` `CONST_NAME` `` — not vague English ("user email in import data" →
  `User.email` / `user::Column::Email`).
- **Name explicitly:** entity/model + field; schema + field; exported function
  - file when helpful; Excel **exact header** + target field.
- **Gloss once:** e.g. "display name (`User.displayName`)"; do not replace
  stored literals with English glosses in mapping tables.
- **Mermaid in `README.md`:** every flowchart block MUST start with
  `theme: neo-dark` (see existing READMEs).

---

## UTF-8 encoding (Windows-safe edits)

Non-ASCII literals in source or docs (localized labels, Excel headers) must
stay **verbatim** and valid UTF-8 on disk. Bad encoding turns them into `???`,
`â€"`, or `Â§`. Prefer root **`.editorconfig`** with `charset = utf-8` when
present.

1. Prefer opencode **`edit` / `write`** — no bulk shell rewrites unless
   encoding is explicit.
2. **Never** use PowerShell `Set-Content`, `Out-File`, or
   `Get-Content | Set-Content` on tracked `.md` / `.ts` / `.tsx` / `.rs`
   without UTF-8.
3. Rewrite scripts: **Bun/Node** with `'utf8'`, or PowerShell
   `[System.IO.File]::ReadAllText` / `WriteAllText` with `UTF8Encoding`.

```typescript
import fs from "node:fs";
const text = fs.readFileSync(path, "utf8");
fs.writeFileSync(path, updated, "utf8");
```

```powershell
# ❌ BAD — default Windows encoding corrupts non-ASCII literals
Get-ChildItem -Recurse *.md | ForEach-Object {
  (Get-Content $_.FullName -Raw) -replace 'old','new' | Set-Content $_.FullName
}
```

| Bulk edits                                                | Avoid                                                                       |
| --------------------------------------------------------- | --------------------------------------------------------------------------- |
| `git mv` + targeted `edit` or one UTF-8-safe script     | PowerShell loops over `*.md` without UTF-8; `sed`/redirects assuming locale |

After editing files with non-ASCII literals, spot-check:
`rg '\?\?\?' <edited-path>`. If `???` appears, restore from git and re-apply
with a UTF-8-safe method.
