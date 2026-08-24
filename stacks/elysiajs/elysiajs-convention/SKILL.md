---
name: elysiajs-convention
description: Use when editing Elysia HTTP API code (routes, handlers, plugins, lifecycle hooks, macros) — compose small `Elysia` instances via `.use()`, lean into schema-driven type inference (`body` / `query` / `response`), map errors with `.onError()` and `status()`, and use `macro()` for cross-cutting concerns (auth, request-scoped state) instead of duplicating in handlers.
---

# Elysia conventions

Applies when editing **Elysia HTTP API** code (routes, handlers, plugins,
lifecycle hooks, validation schemas).

## App wiring

- Build the root **`new Elysia()`** in one entry point; compose feature
  modules via **`.use(module)`** rather than registering every route at
  the root.
- Each feature module exports its own `Elysia` instance (or a function
  returning one) so it can be unit-tested and reused.
- Group related routes and a shared prefix with **`.group(prefix, app => ...)`**.
- Keep process-level setup (`Bun.serve`, graceful shutdown, tracing init)
  at the entry point — not inside modules.

## Routing and handlers

- Register routes with **`.get` / `.post` / `.put` / `.patch` / `.delete`**
  directly on the instance; lean into the type inference Elysia provides
  from schemas.
- Keep handlers **thin**: parse input via context, call a service, return
  the result. Do not read raw `Request` unless you must.
- Use **`as const`** on route definitions only when you need the literal
  type for downstream inference.

## Schemas and validation

- Declare per-route schemas via **`body` / `query` / `params` / `response`**
  (TypeBox `t.Object(...)` or any validator Elysia supports). Validation
  failures become 400 automatically — do not validate manually on top.
- Schemas are the source of truth for both validation **and** the inferred
  handler context types. Do not duplicate the same shape in a hand-written
  type.
- Declare response schemas when the contract must be stable (clients,
  OpenAPI export); skip them for trivial internal endpoints.

## Lifecycle hooks

- Use Elysia's ordered hooks for cross-cutting concerns: `onRequest` →
  `onParse` → `onTransform` → `onBeforeHandle` → `onAfterHandle` →
  `onError`.
- Keep hooks **small and composable**; if a hook grows past a few lines,
  move it to a module and call from the hook.
- Do not put business logic in `onBeforeHandle` — that is a guard, not a
  pipeline step.

## Errors

- Map expected failures to HTTP responses with **`status(code, body)`** or
  a global **`.onError()`**. Reserve thrown errors for truly unexpected
  failures (programmer mistakes, invariant violations).
- Reuse a single error shape across the API; do not invent per-route
  `{ error: string }` shapes unless the project already does so.
- Log at the failure site or in `.onError()` — never `console.log` for
  request errors.

## Auth and macros

- Use **`macro()`** for cross-cutting concerns (auth, request-scoped
  state, tenant resolution) instead of duplicating middleware in every
  handler.
- Request-scoped values flow through the **`resolve()`** hook — type them
  explicitly so downstream handlers see the inferred context.
- Apply the macro via `.use(authMacro)` on the module that needs it; do
  not register macros at the root if they only apply to a subset of
  routes.

## Plugins

- Prefer **official `@elysiajs/*` plugins** (`cors`, `jwt`, `swagger`,
  etc.) over hand-rolled equivalents.
- Plugins are applied with **`.use(plugin())`**; order matters for hooks
  and middleware composition.
- When you must write a custom plugin, export it as a function returning
  a fresh `Elysia` instance so it composes cleanly.

## Health and config

- Expose a simple **`GET /health`** that returns liveness without auth.
- Bind address from **`PORT`** / **`HOST`** via env; do not hardcode
  ports or secrets. Load `.env` only for local dev, not as the prod
  config source.
- Database connection (if any) lives on a small app-state object passed
  via `decorate()` or accessed in handlers — do not open a new connection
  per request.

## OpenAPI / docs

- When added, use **`@elysiajs/swagger`** with options kept in a
  dedicated module — not inlined in route definitions.
- Schemas declared on routes are the source of truth for the OpenAPI
  output; keep them aligned with the public contract.