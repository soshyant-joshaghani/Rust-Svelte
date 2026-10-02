# safe/ — keys, addresses, prod env (local only)

| Pattern | Purpose |
|---------|---------|
| `*-privatekey.pem` | SSH private key |
| `*-address.txt` | VM IP / hostname (first line) |
| `*-env.env` | Production secrets → uploaded as `~/projects/rust-svelte/.env` |

| Files | Server id |
|-------|-----------|
| `ar-rust-svelte-bamdad-*` | `rust-svelte` |

Copy the `*.example` stubs, drop the `.example` suffix, and fill real values.

`*.pem`, `*.env`, `*-address.txt` are gitignored.

Upload env to VM:

```bat
rust-svelte-ctrl.bat env
```

That copies `safe/ar-rust-svelte-bamdad-env.env` → `~/projects/rust-svelte/.env`.
