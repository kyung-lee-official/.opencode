---
name: axum-convention
description: Use when editing Axum HTTP API code (routes, handlers, extractors, app state, `Router`) — build Router in one place, share deps via `State<T>` + `.with_state(...)` (not per-handler DB connections), typed extractors, `IntoResponse` returns, `AppError` for domain→HTTP mapping, `tracing` instead of `println!` for request errors.
---

# Axum conventions

Applies when editing **Axum HTTP API** code (routes, handlers, extractors,
app state). Peer of **`nestjs-convention`** for the Nest stack. Shared Rust
rules live in **`rust-convention`**.

## App wiring

- Build the **`Router`** in one place (e.g. `main` or `app` module); keep
  handlers thin.
- Prefer **`Router::nest` / `merge`** for feature modules over one giant
  route list as the API grows.
- Share dependencies via **`.with_state(...)`** — especially **SeaORM**
  `DatabaseConnection` (or a small `AppState` struct). Do **not** open a new
  DB connection per request.
- Keep **graceful shutdown** (`with_graceful_shutdown`) and **tracing** init
  at the process boundary.

## Handlers and extractors

- Prefer typed extractors: `State<T>`, `Path`, `Query`, `Json<T>`,
  `Extension` only when State is insufficient.
- Handler signatures should make inputs obvious; avoid reading raw `Request`
  unless necessary.
- Return types that implement **`IntoResponse`** (status + body, or a project
  error type).

```rust
// ✅ State + Json extractor
async fn create_widget(
    State(db): State<DatabaseConnection>,
    Json(body): Json<CreateWidget>,
) -> Result<impl IntoResponse, AppError> {
    // ...
}

// ❌ Connect to DB inside the handler
async fn create_widget(Json(body): Json<CreateWidget>) { /* Database::connect every time */ }
```

## Request / response types

- Use **`serde`** structs (`Deserialize` / `Serialize`) for JSON bodies and
  query params — parallel to Nest Zod DTOs as the boundary types.
- Validate at the edge: serde types + explicit checks, or a validator crate
  if the project already uses one. Reject bad input with **4xx**, not panic.
- Keep API types close to the feature (e.g. `routes/widgets.rs` or `dto`
  module); name by resource (`CreateWidget`, `WidgetResponse`).

## Errors

- Map domain/DB failures to HTTP in **one** error type (`AppError` /
  `IntoResponse`) — do not `.unwrap()` in handlers for expected failures.
- Use **`Result<T, E>`** on handlers; reserve `.expect` for install-time
  impossibilities (e.g. signal handler setup) sparingly.
- Log with **`tracing`** at the failure site or in the error mapper; do not
  `println!` for request errors.

## Health and config

- Expose a simple **`GET /health`** (liveness) that does not require auth.
- Bind address from **`HOST` / `PORT`** (and DB from **`DATABASE_URL`**) via
  env; do not hardcode secrets. Load `.env` only for local dev (`dotenvy`),
  not as the prod config source.

## OpenAPI (when added)

- Keep OpenAPI/schema generation **out of handler bodies** — dedicated
  module or utoipa annotations colocated and kept in sync with serde types
  (same spirit as Nest `*.swagger.ts`).
- When touching a route's contract, update the OpenAPI definition in the
  **same** change.