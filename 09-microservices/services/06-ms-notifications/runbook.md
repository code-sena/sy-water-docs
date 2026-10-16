# Runbook — ms-notifications

## 1. Quick information

| Item | Value |
|------|-------|
| Port | 3006 |
| Health | `GET /health` (database) |
| Dependencies | SQL Server, Redis, RabbitMQ, ms-iam (e-mail), SMTP/Mailpit, Expo Push, public key of ms-iam |

## 2. Frequent problems

| Symptom | Cause | Action |
|---------|-------|--------|
| The service fails at startup reading the key | `keys/iam-public.pem` is missing (only `.gitkeep` in the repo) | Copy `ms-iam/keys/public.pem` |
| No notification after linking a meter | Missing `RabbitMq:*` or the queue `notification.device-events` was not bound | Configure RabbitMQ and restart; check bindings in `http://localhost:15672` |
| E-mails stay `PENDING` | `Smtp:Host` empty, Mailpit down, or ms-iam unreachable / wrong `Internal:ApiKey` | Start Mailpit; check `Services:IamBaseUrl` and the key |
| E-mails `FAILED` | 5 failed attempts | Read `last_error` in `notification.email_outbox` |
| No push on the phone | Push token not registered, Expo Go limits, or `Push:Enabled=false` | Register the token from the app; use a development build |
| 400 `preferences.critical_requires_in_app` | The app tried to turn off in-app for CRITICAL | Expected |

## 3. Maintenance

- Apply `ms-notification-db` up to `v1.4-device-liveness` before running this version.
- Production SMTP: set `Smtp:Host`, `Port` 587, `Security` `StartTls`, user and password in user-secrets or environment variables.
