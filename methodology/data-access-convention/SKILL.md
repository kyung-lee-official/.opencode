---
name: data-access-convention
description: Use when writing or reviewing data access code — which layer owns queries, transactions, and locks, and the layering between application layers, services, and the data access layer (DAL).
---

# Data Access Layering (DAL)

Keep persistence behind a **data access layer (DAL)**. The DAL owns the schema: queries, transactions, keys, and per-resource locking. Callers stay schema-agnostic.

Calls flow one way: an **application layer** (HTTP handler, job, CLI) calls a **service**; the service calls the **DAL**; the DAL talks to the store. The application layer never reaches past the service.

| Layer             | Owns                                                        |
| ----------------- | ----------------------------------------------------------- |
| Application layer | Request handling, input validation, calling a service       |
| Service           | Use-case orchestration, business rules, composing DAL calls |
| DAL               | Queries, transactions, row shapes, keys, per-resource locks |

## Rules

- **Import the DAL in services only.** Application layers call services, not DAL functions. A thin read-only path with no business logic is a rare, deliberate exception — not the default.
- **Transactions and keys live in the DAL.** A multi-statement write (insert a parent, then its children) is one transaction inside the DAL, not a sequence assembled by the caller.
- **Single-writer protection locks inside the DAL.** Keep the lock next to the state it guards so no caller can bypass it; a lock in a service or handler can be skipped by another entry point.
