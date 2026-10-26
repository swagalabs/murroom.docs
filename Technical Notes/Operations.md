---
title: Operations
tags: [murrnails, deployment, runbook]
up: "[[TechnicalDocument]]"
---

# Operations

[[TechnicalDocument|← Core technical document]] · [[Security and Access]]

## Local development

```bash
docker compose up -d postgres
(cd backend && npm install && npm run start:dev)
(cd frontend && npm install && npm run dev)
```

The locally started backend defaults to port `3000`; Vite normally uses `5173`; PostgreSQL is bound to `127.0.0.1:5432`. Copy `backend/.env.example` to `backend/.env` and supply development-safe values before starting the backend.

Useful verification:

```bash
curl http://127.0.0.1:3000/api/health
(cd backend && npm test)
(cd frontend && npm run build)
```

## Production deployment

Compose starts:

- `postgres` (PostgreSQL 16), host-only `127.0.0.1:5432`;
- `backend`, host-only `127.0.0.1:3100` mapped to container 3000;
- `frontend`, host-only `127.0.0.1:8090` mapped to container 80.

nginx is public-facing. It proxies `/` to frontend and `/api/` to backend, forwards host/client/protocol headers, and caps request bodies at 25 MB. Telegram Mini Apps require HTTPS in production; set `MINI_APP_URL` to the public HTTPS origin and obtain a valid certificate (the repo deployment note describes certbot/nip.io and custom-domain paths).

```bash
docker compose up -d --build
docker compose ps
curl http://127.0.0.1:3100/api/health
```

Persistent volumes are `postgres_data`, `backend_uploads`, and `backend_backups`. Do not remove volumes as a normal redeploy step.

## Required configuration

| Variable | Why it matters |
|---|---|
| `TELEGRAM_BOT_TOKEN` | Bot updates and outbound Telegram actions. |
| `MINI_APP_URL` | Public Mini App origin; HTTPS in production. |
| `BACKEND_URL` | Backend public base URL where needed. |
| `JWT_SECRET` | JWT signing secret; use a long, unique random value. |
| `DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASSWORD`, `DB_NAME` | PostgreSQL connectivity. Compose overrides host settings for the container. |
| `UPLOADS_DIR` | Filesystem location for uploaded assets. |
| `BACKUP_DESTINATION`, `BACKUP_*` | Backup destination, path/timezone, and optional OneDrive/SMTP credentials. |

## Backups and restore

The backup service runs daily at `00:00` in `BACKUP_TZ` (or server timezone if absent). Destination is one of local, OneDrive, or email. In the Compose stack, `BACKUP_USE_DOCKER=true` is appropriate because PostgreSQL is inside Docker. On a directly run backend with `pg_dump` available, use `false`.

The protected `/api/backup/run` and `/api/backup/restore` endpoints support operator actions. Before a restore, preserve a fresh backup, identify the precise target database, and plan a maintenance window: restore is materially destructive.

## Scheduled work

| Work | Cadence/mechanism | Operational implication |
|---|---|---|
| Database backup | cron at midnight | Check destination and retention regularly. |
| Discount broadcast processing | at startup then every 5 min | Must have one active scheduler owner. |
| Appointment reminders/post-session/re-engagement | Nest scheduled services | Requires correct bot token, business time handling, and continuous backend runtime. |
| Master summary | scheduled summary service | Delivery is logged in `master_summary_logs`. |

## Time and timezone

The app retains appointment dates and times in separate database columns. `TZ=UTC` is set in Compose, while backups can use `Europe/Vilnius`; the repository also has `backend/TIMEZONE.md` and timezone utilities. Any change touching slot comparison, cron time, calendar output, or reminders must be tested around UTC offset and daylight-saving boundaries. Do not casually parse date-only values as UTC instants.

## Fast troubleshooting

| Symptom | First checks |
|---|---|
| Mini App opens but API fails | nginx `/api/` proxy, backend health, CORS/MINI_APP_URL, JWT. |
| Telegram says URL is invalid | Verify HTTPS, certificate, and exact `MINI_APP_URL`. |
| Bot not sending | Token, backend logs, outbound network reachability, recipient Telegram ID. |
| Slots/reminders at wrong time | Compare server `TZ`, application timezone utilities, stored date/time, and client timezone. |
| Upload unavailable | `UPLOADS_DIR`, backend volume, `/api/uploads/` static mapping, nginx size limit. |
| Backup fails | destination vars, credentials, `pg_dump` or Docker socket mode, mounted backup volume. |
