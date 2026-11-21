# Teldrive on Railway — your own Telegram cloud drive

[Teldrive](https://github.com/tgdrive/teldrive) turns a Telegram account into a personal cloud drive: upload files through a clean web UI, and Teldrive stores them in your own private Telegram storage channels with a Postgres metadata index. This template deploys **Teldrive 2.0.13 + Postgres 18 (SSL)** in one click — zero deploy-form prompts, health-gated readiness, and a guided first-login flow.

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.app/new?github_url=https://github.com/lNamelessl/teldrive-railway-template)

## After deploying — connect Telegram in three steps

Teldrive's landing page IS the login. There are no deploy-form prompts to fill; the one real step is signing in:

1. **Open your Teldrive domain** (service Teldrive → Settings → Networking, or the link Railway prints after deploy).
2. **Sign in with your Telegram account** — enter your phone number, then the code Telegram sends you (in your Telegram app, not SMS). Face-ID style QR login is also built in if you prefer scanning. Optional: Teldrive ships public Telegram API credentials by default — per upstream, "no api_id or api_hash is required". Power users can set their own later via the `TELDRIVE_TELEGRAM_APP_ID` / `TELDRIVE_TELEGRAM_APP_HASH` service variables (get them at https://my.telegram.org under API development tools).
3. **Create a folder and upload a file** — Teldrive auto-creates the private Telegram channels that store your data ("teldrive"-prefixed). Download it again to verify the round-trip.

Sessions are issued as short-lived access tokens (15 min) with 30-day refresh, and the Telegram session itself is encrypted at rest in Postgres with the per-deploy `TELDRIVE_SECURITY_DATA_KEY`.

## Variables (all automatic — nothing to fill in at deploy time)

| Setting | Value | Where it comes from |
|---|---|---|
| `TELDRIVE_HTTP_ADDRESS` | `0.0.0.0:8080` | Baked into the service start command as `--http-address` (Teldrive 2.x otherwise binds to loopback only) |
| `TELDRIVE_DATABASE_CONNECT_TIMEOUT` | `5m` | Baked into the start command as `--database-connect-timeout` — lets first boot wait out Postgres initialization instead of racing it |
| `TELDRIVE_DATABASE_URL` | Postgres internal URI + `sslmode=require` | Wired to the Postgres service automatically |
| `TELDRIVE_SECURITY_SIGNING_KEY` | 43 random chars per deploy | Generated at deploy time (JWT signing — rotating logs everyone out) |
| `TELDRIVE_SECURITY_DATA_KEY` | 43 random chars per deploy | Generated at deploy time (encrypts stored Telegram sessions — **NEVER rotate or clear it: data written under an old key cannot be decrypted with a replacement**) |
| `POSTGRES_PASSWORD` | 32 random chars per deploy | Generated at deploy time |
| `PGDATA` | volume path + `/pgdata` | Derived from the Postgres volume automatically |
| `DATABASE_URL` | composed from the above | Lives on the Postgres service |

Every secret is distinct per deployment. Read them anytime in each service's Variables tab.

## Honest limits (measured on Railway, not marketing)

| Limit | Reality |
|---|---|
| Idle RAM | Teldrive 2.0.13: 14–17 MB; Postgres: ~70 MB — the pair fits comfortably on trial/hobby plans |
| Telegram ToS / ban risk | Upstream's own warning, quoted: "Follow Telegram's terms and API limits. Accounts and stored files can become unavailable; keep independent backups. Back up database credentials, signing keys, and encryption keys before upgrading." Storage abuse (multi-TB warehouses, sharing) is exactly what gets accounts limited — personal-drive usage is the intended shape |
| Availability | Your files live in Telegram's cloud, gated by your account. If the account is banned or Telegram blocks the API, files become unreachable — keep independent backups of anything that matters |
| Rate limits | Teldrive rate-limits Telegram API calls per account (50 ms spacing, burst 10 by default) — heavy parallel uploads are deliberately throttled to protect the account |
| Data recovery | Losing `TELDRIVE_SECURITY_DATA_KEY` makes stored Telegram sessions undecryptable; losing the Postgres volume loses file metadata. Back both up (Postgres volume + the Variables tab) |
| Public access | Files are private by default; the `security.public-file-content` switch is OFF — do not turn it on on a shared domain |

## Why this template and not the community one

The older "Teldrive" marketplace listing pins **1.8.3**, which crash-loops on Railway (its config loader demands `$HOME`, which Railway's runtime does not provide for that image — deployments die at boot), asks two deploy-form prompts for Telegram API credentials that upstream 2.0.x explicitly does not need, and ships no healthcheck. This template pins **2.0.13** by digest, asks for nothing at deploy time, health-gates both services on Teldrive's readiness probe, and documents the login step instead of leaving you to discover it.

# Deploy and Host

## About Hosting

Two services are provisioned:

- **Teldrive** — the official `ghcr.io/tgdrive/teldrive` image, pinned by digest to release **2.0.13** (deployed amd64 manifest `sha256:507482449da4cae005183a885f8492cf3da45459501e9501dc8e57a4626fa0dd`, child of the release index `sha256:f1f221bd8c635b732257564798189520c5d5fd2331aa1bf86fdc81c856d41948`, commit `c9c366aa`). It serves the web UI and API on port 8080 behind a public Railway domain and answers `/health/ready` (checks PostgreSQL connectivity, migrations, and runtime state) plus `/health/live` without authentication.
- **Postgres** — `ghcr.io/railwayapp-templates/postgres-ssl` release 18, pinned by digest (`sha256:4fc833b43d2aaf6ecc9b18615f5533602b723a32d0e2efe2dae8ca0f7217c863`), with a persistent volume at `/var/lib/postgresql/data` and `PGDATA` pointed at a `pgdata` subdirectory. It is reachable only over Railway's private network — no public domain.

Teldrive runs its own schema migrations on boot inside the `teldrive` schema, so a fresh deploy needs no manual migration step. Redeploys are safe: the Postgres volume keeps all file metadata and encrypted Telegram sessions, and migrations are non-destructive across restarts.

## Why Deploy

- **One click, zero prompts.** Rival deployments of Teldrive on Railway demand API credentials at deploy time and then crash-loop anyway. This template provisions a working pair (app + database) with every secret generated per deploy and the listen address, database URI, and volume pre-wired.
- **Health-gated deploys.** Railway checks Teldrive's `/health/ready` before calling a deployment SUCCESS — you see real readiness (database connected, migrations done), not just a started container.
- **Current release.** Pinned to upstream 2.0.13 (the 2.0 line rewrote the backend, web UI, and release workflow), with the digest recorded so you can always verify what runs.
- **Guided first run.** The listing, the deploy logs, and the app's landing page all point at the same one step: sign in with Telegram, then use the drive.

## Common Use Cases

- A personal drive in the browser: folders, uploads, downloads, streaming-friendly file serving, shareable links.
- A second copy of files that is independent from your laptop (with the ToS caveats above).
- A lightweight file-drop for a small team or family: anyone you let sign in with their Telegram account gets storage; optional `security.allowed-users` allowlist limits who can log in.
- Self-hosted automation target: Teldrive ships a documented REST API (`/v1/files`, uploads, shares) you can script against once your session is set up.

## Dependencies for

- A **Telegram account** (yours) — it is the storage backend. Sign in once from the web UI; Teldrive creates its private storage channels automatically on first use.
- A **Railway account** with usage-based billing — the two services idle at well under 100 MB combined RAM.
- Nothing else: no S3 keys, no SMTP, no API tokens to paste at deploy time. Optional post-deploy knobs: `TELDRIVE_TELEGRAM_APP_ID` / `TELDRIVE_TELEGRAM_APP_HASH` (https://my.telegram.org), `security.allowed-users` via `TELDRIVE_SECURITY_ALLOWED_USERS`, and a download disk cache via `TELDRIVE_CACHE_DISK_DIRECTORY`.

### Deployment Dependencies

- `ghcr.io/tgdrive/teldrive` 2.0.13 (digest-pinned; upstream repo https://github.com/tgdrive/teldrive — see NOTICE.md for the 2.0.13 license situation).
- `ghcr.io/railwayapp-templates/postgres-ssl` 18 (digest-pinned; Railway's maintained Postgres image with TLS forced).
- Railway private networking between the two services (DATABASE_URL resolves to the internal domain; public exposure of Postgres is deliberately not provisioned).
- Telegram's MTProto API reachability from Railway (outbound HTTPS/TCP is allowed by default; Teldrive talks to Telegram's data centers directly).
