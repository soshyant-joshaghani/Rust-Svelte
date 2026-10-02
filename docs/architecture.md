# Architecture

Rust-Svelte follows the FoxG folder contract. The Svelte site lives in `frontend/`. It is a copy of Fast-Svelte's UI.

```text
rust-svelte/
├── frontend/
├── backend/                 `backend/src/modules/{apps,base,system}`, `backend/src/core`, and `backend/src/bin/{api,worker}.rs`
├── tests/
├── traefik/
├── docs/
├── __plans__/
└── __ctrl__/                Python CLI
```

Request flow: Route (Axum handler) → Service → Repository (trait, SQLx) → PostgreSQL. The crate is one library plus two binaries (`api`, `worker`). Both use the `rust_svelte` library modules; the worker only needs `core`.

Routes speak the [wire contract](../../../../CONTRACT.md): `/api/v1`, `snake_case` JSON, `{"detail": "..."}` errors, form login, JWT HS256. Data is PostgreSQL with the Fast schema. Redis is the cache and the job queue. Both degrade softly: a missing Redis never fails a request.
