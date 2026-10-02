# rust-svelte `__ctrl__`

The **control layer** for Rust-Svelte — one predictable CLI for the project lifecycle.

`__ctrl__` is not a loose collection of scripts. It is the official interface for:

- Starting and stopping dev infrastructure and apps
- Selecting runtime profiles (Full / Slim)
- Scaffolding app modules
- Running tests
- Deploying to production via SSH

```text
Rust-Svelte
│
├── Application Layer     (backend + frontend)
├── Infrastructure Layer  (db, redis, workers, proxy)
└── Control Layer         __ctrl__/  ← you are here
```

Prefer `__ctrl__` commands over ad-hoc `docker compose` or manual process management unless you have a specific reason.

This is **not** foxg-ctrl. FoxG platform VMs stay under `foxg-ctrl`; Rust-Svelte kit ops live here.

## Quick start (Windows)

From `rust-svelte/__ctrl__/`:

```bat
rust-svelte-ctrl.bat
```

Interactive prompt, or one-shot:

```bat
rust-svelte-ctrl.bat setup-local
rust-svelte-ctrl.bat dev run all
rust-svelte-ctrl.bat test all
rust-svelte-ctrl.bat list
rust-svelte-ctrl.bat connect
```

Linux/mac:

```bash
chmod +x rust-svelte-ctrl.sh
./rust-svelte-ctrl.sh status
```

## Command map

| Area | Commands |
|------|----------|
| Local tooling | `setup-local [--force]` |
| Dev stack | `dev run\|stop\|down\|purge\|reset {infra,apps,all}` · `--slim` for lightweight runtime |
| App scaffold | `app create <name>` |
| Tests | `test {all,backend,frontend}` |
| Local prod smoke | `prod start\|stop\|reset\|backup-acme\|…` |
| SSH / VM | `setup`, `pubkey`, `clone`, `env`, `start`, `stop`, `update`, `reset`, `backup-acme`, `connect`, … |

On-VM bash/bat scripts (what SSH `start`/`stop` invoke) live in [`remote/`](remote/README.md).

## Layout

| Path | Role |
|------|------|
| `servers.json` | Single VM entry (`rust-svelte`) |
| `safe/` | PEM, address, prod `.env` |
| `static/gpg` | Docker Ubuntu GPG (Iran bootstrap) |
| `remote/` | On-VM / local-prod compose scripts |
| `rust-svelte-ctrl.bat` / `.sh` | CLI entry |

## Typical first deploy (SSH)

```bat
rust-svelte-ctrl.bat setup
rust-svelte-ctrl.bat pubkey
REM add VM pubkey to GitHub
rust-svelte-ctrl.bat clone
rust-svelte-ctrl.bat env
rust-svelte-ctrl.bat start
```

Day-2:

```bat
rust-svelte-ctrl.bat update
rust-svelte-ctrl.bat status
rust-svelte-ctrl.bat backup-acme
```

## Local dev (Docker Desktop / host apps)

```bat
rust-svelte-ctrl.bat setup-local
rust-svelte-ctrl.bat dev run all
rust-svelte-ctrl.bat dev stop all
rust-svelte-ctrl.bat dev down all
rust-svelte-ctrl.bat dev purge infra
rust-svelte-ctrl.bat dev reset all
```

| Action | Infra (compose.dev.yml) | Apps (host) |
|--------|-------------------------|-------------|
| `run` / `start` | `up -d` db, redis (full), proxy, adminer | Rust (Axum) API :8000 (runs SQL migrations), Redis queue worker (full), Vite :5000 |
| `stop` | `compose stop` — containers kept | kill host processes |
| `down` | `compose down` — volumes kept | kill host processes |
| `purge` | `compose down -v` — wipe data, stay down | kill host processes |
| `reset` | wipe then `run` | stop then run |

| Target | Notes |
|--------|-------|
| `infra` | Docker only. SQL migrations run when the API starts |
| `apps` | host processes (needs infra already up) |
| `all` | run: infra→apps · stop/down/purge/reset: apps→infra |

Opens browser tabs for Adminer / Traefik / dashboard / API docs after a successful run.

**Runtime profiles:** `dev run all` (full — includes Redis + worker) · `dev run all --slim` (no Redis/worker). See [docs/runtime-profiles.md](../docs/runtime-profiles.md).

## Tests

```bat
rust-svelte-ctrl.bat test all
rust-svelte-ctrl.bat test backend
rust-svelte-ctrl.bat test frontend
```

Backend tests use in-memory fakes and need neither Postgres nor Redis. `test frontend` runs Vitest and `svelte-check`.

## Local production smoke

```bat
rust-svelte-ctrl.bat prod start
rust-svelte-ctrl.bat prod stop
rust-svelte-ctrl.bat prod reset
rust-svelte-ctrl.bat prod backup-acme
```

Same scripts SSH uses under `remote/`. Prefer SSH `start`/`stop` when operating the real VM from your laptop.

## Setup (ctrl tool itself)

```bat
python -m venv .venv
.venv\Scripts\pip install -r requirements.txt
```

`setup-local` also runs `npm install` for the frontend workspace.

On first `setup-local` / `dev run all`, the ctrl entry installs system **Python 3.10+** (via winget / Homebrew / apt) if missing, then `_setup_local` installs **Node.js LTS + npm** the same way before creating the project `.venv` and running `npm install`.

Iran VMs (`iran_setup: true`) keep provider DNS, rewrite apt to Arvan `apt_mirror`, and use Arvan Docker `registry_mirror`. `clone` routes GitHub SSH via `ssh.github.com:443`.
