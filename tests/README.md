# Tests

```bat
__ctrl__\rust-svelte-ctrl.bat test all
__ctrl__\rust-svelte-ctrl.bat test backend
__ctrl__\rust-svelte-ctrl.bat test frontend
```

`test backend` runs `cargo test` in `backend/`. Integration tests live in `tests/backend/*.rs`. `backend/Cargo.toml` points its `[[test]]` targets there, so the kit keeps the shared `tests/backend` folder.

The backend tests use in-memory fakes for the repositories, the cache, and the job queue, so they need no database. `tests/frontend` holds the Vitest suite shared with Fast-Svelte.
