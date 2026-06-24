# 🦀 Rust for Hot-Path APIs

Rust is **not the default** — it's the scalpel for the part of the system that's latency- or throughput-critical: gateways, high-RPS APIs, anything where predictable tail latency and memory safety pay for the extra rigor. It lives in the same [monorepo](../architecture/monorepo.md) as the Bun apps.

## Stack
- **tokio** — async runtime.
- **axum** — HTTP framework.
- **sqlx** — compile-time-checked SQL.
- **serde** — (de)serialization.
- **thiserror** — typed errors.

## When to reach for it
| Reach for Rust when | Stay on Bun/Hono when |
|---|---|
| sub-ms latency, high RPS | typical CRUD / dashboards |
| a shared gateway many apps hit | one app's API |
| heavy CPU work (parsing, crypto, ledger math) | I/O-bound glue |
| strict correctness (money, auth, audit) | rapid product iteration |

## File organization — every file ≤300 LOC
```
src/
├── main.rs           # signals, graceful shutdown
├── config/           # schema, validation, secrets, hot reload
├── transport/        # HTTP / framing
├── auth/             # OIDC, JWT, session cache
├── authz/            # grant evaluation, policy decision
├── exec/             # per-DB pools, query exec, timeouts, cancellation
├── audit/            # append-only writes, sinks
├── state/            # state DB queries, migrations
└── errors.rs         # typed error enum
```

## Conventions (production Rust)
- **No `unwrap` outside `main`/tests.** A panic in the hot path kills all in-flight users.
- **Typed errors** (`thiserror`) → stable client codes. Never leak credentials in an error or log line.
- **Newtype over bare primitives:** `RequestId(String)`, not `String`.
- **Borrow by default:** `&str` > `String`, `&[T]` > `Vec<T>`.
- **Async end-to-end.** No `std::sync::Mutex` on the request path — use `tokio::sync`. A slow query on DB A must never block DB B.
- **Cancellation safety:** client disconnect → task dropped → cancel the in-flight DB query → record outcome `cancelled`.
- **Per-(server,database) connection pools** — never one global pool, never share across targets.
- **Tracing spans** on every request with `request_id`, `user`, route.

## axum sketch
```rust
let app = Router::new()
    .route("/healthz", get(|| async { "ok" }))
    .route("/v1/query", post(run_query))
    .with_state(state);

axum::serve(listener, app)
    .with_graceful_shutdown(shutdown_signal())
    .await?;
```

## Build & ship
- `cargo fmt` + `cargo clippy -D warnings` in the lint gate.
- Unit tests for pure logic (no DB); integration tests against a real Postgres CI service container; `proptest` for invariants where the shape matters more than specific values.
- Multi-arch container (amd64 + arm64), pushed to GHCR/DOCR, deployed via [ArgoCD](../infrastructure/kubernetes-gitops.md).

A real-world example of this shape: an SSO-gated, audited DB gateway in Rust — see [../infrastructure/sso-zitadel.md](../infrastructure/sso-zitadel.md) and [../ai-agents/tools-and-mcp.md](../ai-agents/tools-and-mcp.md).
