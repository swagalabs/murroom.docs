---
title: Security and Access
tags: [murrnails, security, authentication]
up: "[[TechnicalDocument]]"
---

# Security and Access

[[TechnicalDocument|← Core technical document]] · [[API Reference]] · [[Operations]]

## Authentication methods

- **Telegram:** `POST /api/auth/telegram` is guarded by Telegram init-data validation. The server derives the Telegram user from validated data, rather than trusting a client-supplied user ID.
- **Password:** a user can register/sign in with login and password. Password hashes use bcrypt (cost 12); `passwordHash` is excluded from standard entity selection.
- **Session:** successful authentication returns a signed JWT. Protected calls pass it as a bearer token and `JwtAuthGuard` resolves the user.

## Authorization

| Role/guard | Scope |
|---|---|
| `JwtAuthGuard` | Signed-in user required. |
| `MasterOrAdminGuard` | Master capabilities plus administrators. |
| `AdminOnlyGuard` | Global user/role and broadcast administration. |
| Service-level ownership | Prevents a master/client from acting on another owner’s resource. |

The application includes a hard-coded super-admin Telegram identifier that is enforced when that identity signs in. This is an operational break-glass mechanism and should be reviewed with the rest of access policy when ownership changes.

## Security controls already present

- Global whitelist validation rejects unexpected request fields.
- CORS is restricted to Telegram Web, local development, and the configured Mini App origin.
- Database and application services bind to loopback in Compose; nginx is the intended public edge.
- The Mini App’s public URL is expected to use HTTPS in production.
- Uploads are served only through an application-controlled static prefix.

## Review checklist

1. Confirm each sensitive controller endpoint has JWT and appropriate role guard.
2. Confirm service methods filter by `masterId`, authenticated user, or a verified client relationship.
3. Keep `JWT_SECRET` and bot/database/backup credentials out of git, screenshots, logs, and this vault.
4. Rotate secrets after suspected exposure and redeploy all dependent services.
5. Treat `/backup/restore`, role updates, broadcast, content deletion, and calendar token regeneration as high-impact actions; preserve audit context.
6. The backend’s Docker socket mount is powerful. Restrict host access and do not expose the backend port publicly.

> [!warning]
> Frontend route metadata is only navigation gating. A user can call an API directly; server authorization remains the real control plane.
