---
name: typescript-convention
description: Use when editing `*.ts` / `*.tsx` — exhaustive `switch` (with `default: never`) on enums, `z.enum`, and fixed string unions; `Decimal` (Prisma `@db.Decimal(17, 6)` or `decimal.js`) for money in domain logic, persistence, and API DTOs — never raw `number`.
---

# TypeScript conventions

Shared naming/docs rules live in `AGENTS.md` (root) and the
`coding-convention` content there. This file covers TypeScript-only patterns.

## Enum branching

When branching on an **enum**, **`z.enum`**, or a fixed **string union**, use
**`switch`** — not separate **`if`**s on the same discriminant. Use `default`
with `never` for closed unions. Single equality or unrelated conditions: `if`
is fine.

```typescript
function isTerminalStatus(status: OrderStatus): boolean {
  switch (status) {
    case OrderStatus.Pending:
      return false;
    case OrderStatus.Shipped:
    case OrderStatus.Cancelled:
      return true;
    default: {
      const _exhaustive: never = status;
      return _exhaustive;
    }
  }
}
```

| Situation                                | Prefer   |
| ---------------------------------------- | -------- |
| 2+ branches on the **same** discriminant | `switch` |
| Single equality or unrelated conditions  | `if`     |

---

## Financial amounts

Represent financial amounts with a **`Decimal`** type (e.g. Prisma
**`Decimal`**, **`decimal.js`**), not raw **`number`**, for domain logic,
persistence, and API DTOs.

### Persistence

- Prefer Prisma **`Decimal @db.Decimal(17, 6)`** for stored amounts (high
  precision; keep as precise as the product allows).
- Do **not** treat "two decimal places" as the storage scale. Round only when
  a **specific boundary** requires it (display, settlement, FX, export), and
  document that boundary.

### Arithmetic

- Do not use binary floating point for money math.
- Keep full `Decimal` precision through domain calculations; convert/round at
  the edge, not mid-pipeline.

### Excel

- Export: numeric cells + `numFmt` at the export boundary (see
  `exceljs-xlsx-conventions`). Rounding for display does not change the DB
  scale.