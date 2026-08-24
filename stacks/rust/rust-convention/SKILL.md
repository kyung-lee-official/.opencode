---
name: rust-convention
description: Use when editing `*.rs` or `Cargo.toml` — exhaustive `match` (no wildcards) for closed enums and `z.enum`-style discriminants, `rust_decimal::Decimal` or minor units for money instead of `f32`/`f64`, and `Result + ?` over `.unwrap()` for expected failures in app code.
---

# Rust conventions

Shared naming/docs rules live in `AGENTS.md` (root) and the
`coding-convention` content there. This file covers Rust-only patterns.

## Enum branching

When branching on an **enum**, use **`match`** — not a chain of `if` /
`if let` on the same discriminant. Prefer exhaustive `match` (no wildcard)
for closed enums so new variants fail to compile. Single equality or
unrelated conditions: `if` is fine.

```rust
fn is_terminal(status: OrderStatus) -> bool {
    match status {
        OrderStatus::Pending => false,
        OrderStatus::Shipped | OrderStatus::Cancelled => true,
    }
}
```

| Situation                                | Prefer  |
| ---------------------------------------- | ------- |
| 2+ branches on the **same** discriminant | `match` |
| Single equality or unrelated conditions  | `if`    |

---

## Financial amounts

Do **not** use `f32` / `f64` for money. Prefer **`rust_decimal::Decimal`**
(or integer **cents** / minor units) for domain logic and persistence.

- Round only at a **documented boundary** (display, settlement, FX, export).
- Keep full precision through domain calculations; convert at the edge.

---

## Errors in application code

Prefer **`Result` + `?`** over `.unwrap()` / `.expect()` for expected
failures (I/O, env, DB, bind). Reserve panic for truly unreachable
invariants.