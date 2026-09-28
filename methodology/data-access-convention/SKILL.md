---
name: data-access-convention
description: Use when writing or reviewing data access code or HTTP endpoints — the application/service/DAL layering, where queries, transactions, and locks live, and why routes stay business-shaped even when a table (and its DAL) is shared.
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

## Endpoints: business-shaped, not table-shaped

A DAL is **one-to-one with a table** and is deliberately shared — several domains may read/write the same table through the same DAL. Endpoints are the opposite: they belong to a domain and speak its language. Never publish a table-shaped route (`GET /log/:id`) just because the table is shared.

| Do (business-shaped)                 | Don't (table-shaped)                 |
| ------------------------------------ | ------------------------------------ |
| `GET /orders/:orderId/log/:logId`    | `GET /log/:logId`                    |
| `GET /users/:userId/settings`        | `GET /settings/:id`                  |
| One route group per consuming domain | One shared route over the shared DAL |

When two domains use the same DAL, each exposes its **own** route(s) under its **own** resource; the shared DAL stays an implementation detail. Scope the path by the domain's identifying ids (the owner), not by the table name, and nest sub-resources under that owner.
