# NOTICE

## Upstream project

This repository packages the **Teldrive** project (https://github.com/tgdrive/teldrive) for one-click deployment on Railway. Teldrive is an upstream project of the tgdrive organization; this template is not affiliated with or endorsed by the Teldrive authors or Telegram.

This repository contains no upstream source code. The deployed application runs the official, unmodified container image `ghcr.io/tgdrive/teldrive` pinned by digest to the 2.0.13 release (linux/amd64 index digest `sha256:f1f221bd8c635b732257564798189520c5d5fd2331aa1bf86fdc81c856d41948`, upstream commit `c9c366aae22dae591475f8efabacf0e1bc211801`), with configuration supplied exclusively through environment variables and the database service documented in the README.

## License situation (checked 2026-11-20 at tag 2.0.13)

- The teldrive **2.0.x source tree ships without a LICENSE file** — the LICENSE that existed throughout the 1.x line (MIT, "Copyright (c) 2024 divyam234") was removed in the 2.0 rebuild commit ("feat!: rebuild Teldrive for 2.0 with a new backend, web UI, and release workflow", 2026-10-10), and GitHub reports no license for the repository at the 2.0.13 tag.
- Accordingly, this template makes no redistribution of upstream code at all: it references the official upstream-published container image and standard configuration only. Rights in Teldrive remain with its authors. If you need a license grant for the 2.0 line, consult the upstream authors.

## Trademarks

- "Teldrive" refers to the upstream project of the same name. Any product names mentioned are property of their respective owners.
- "Telegram" is a trademark of Telegram FZ-LLC / Telegram Messenger Inc. This template is not affiliated with, endorsed by, or sponsored by Telegram. Using Telegram's API for personal cloud storage is subject to Telegram's Terms of Service and API terms; see the honest-limits table in the README.

## Database image

Postgres runs the Railway-maintained `ghcr.io/railwayapp-templates/postgres-ssl` image (release 18, digest-pinned), which wraps the official PostgreSQL 18 image with TLS and operational wrappers. PostgreSQL is licensed under the PostgreSQL License (permissive).
