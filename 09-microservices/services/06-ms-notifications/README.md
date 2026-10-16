# ms-notifications

> Delivers **alerts** to the users: in-app notification center, push to the phone (Expo) and e-mail.
> It turns device events into notifications, respects the channels each user chose per urgency
> level and detects meters that stopped reporting.

---

## Location in the architecture

| Field | Value |
|-------|-------|
| Number in catalog | 06 |
| Bounded context | BC-09 Notification Delivery (Generic) — also covers the device alerts of BC-08 |
| Repository | `sy-water-backend/ms-notification` (solution `SyWater.Notifications.slnx`) |
| Database repository | `sy-water-db/ms-notification-db` (Liquibase, up to tag `v1.4-device-liveness`) |
| Local port | **3006** |
| Stack | .NET 10 · ASP.NET Core · EF Core SqlServer · JwtBearer · RabbitMQ.Client · StackExchange.Redis · MailKit 4.18 |
| Storage | SQL Server, schema `notification` (login `notification_app`) · Redis (token denylist check) |
| Calls | ms-iam (`GET /internal/users/{id}/contact`, `X-Internal-Key`) · SMTP · Expo Push API |
| Called by | App |
| Consumes | `device.linked`, `device.unlinked`, `device.valve.reported`, `device.reading.received` (queue `notification.device-events`) |

---

## Responsibilities (what this service DOES)

| HU | Capability |
|----|------------|
| HU-025 | Notification center: list (newest first, 30 per page, max 100), unread count, mark one or all as read |
| HU-026 | Push to the phone through Expo; the app registers and removes its push token |
| HU-027 | E-mail through an outbox with retries (1 min, 5 min, 15 min, 1 h; `FAILED` at the 5th failure) |
| HU-032 | "Sensor disconnected": a meter that reported before and has been silent for 10 min gets **one** WARNING alert until it reports again |
| HU-033 | "Valve closed" as CRITICAL (and "valve opened" as INFO); only changes notify, not the periodic reports |
| HU-034 | Channels per urgency level (`INFO`, `WARNING`, `CRITICAL`); defaults: CRITICAL all channels, WARNING app + push, INFO app only. CRITICAL can never lose the in-app channel |
| HU-012/014 | Informative notifications when a meter is linked or unlinked |

Notifications can target a user or a role (`ADMIN`), and are de-duplicated by the source event id.

## Out of scope

- SMS / WhatsApp (HU-028): the preference is stored (`sms`), but no SMS provider is integrated.
- Leak (HU-029) and goal alerts (HU-030/031): the source events do not exist yet.
- Valve orders → ms-valve; device status → ms-devices.

---

## How to run it locally

```powershell
cd sy-water-backend\ms-notification
dotnet user-secrets set "ConnectionStrings:Notification" "Server=localhost,1433;Database=sy-water-db;User Id=notification_app;Password=********;TrustServerCertificate=True" --project src/SyWater.Notifications.Api
dotnet user-secrets set "RabbitMq:Password" "********" --project src/SyWater.Notifications.Api
dotnet user-secrets set "Internal:ApiKey" "********" --project src/SyWater.Notifications.Api
Copy-Item ..\ms-iam\keys\public.pem src\SyWater.Notifications.Api\keys\iam-public.pem   # missing in the repo
dotnet test
dotnet run --project src/SyWater.Notifications.Api    # http://localhost:3006/health
```

Development e-mail goes to Mailpit (`localhost:1025`, inbox at `http://localhost:8025`).

## Configuration

| Key | Default | Purpose |
|-----|---------|---------|
| `ConnectionStrings:Notification` | — (user-secrets) | `notification_app` connection |
| `Jwt:*`, `Redis:Configuration` | as in every .NET service | Token validation and denylist (fails closed) |
| `RabbitMq:*` | `localhost`, `5672`, `sywater`, `sywater.events` | Consumer, dead letters to `sywater.dlx` |
| `Services:IamBaseUrl` + `Internal:ApiKey` | `http://localhost:3001` | User contact for e-mail (required only when e-mail is on) |
| `Devices:OfflineAfterMinutes` | `10` | HU-032 threshold |
| `App:PublicUrl` | `http://localhost:8081` | Links inside e-mails |
| `Smtp:Host` / `Port` / `Security` / `Username` / `Password` / `FromAddress` / `FromName` | `""` (Dev `localhost`) / `587` (Dev `1025`) / `StartTls` (Dev `None`) / — / — / `no-reply@cuidatuagua.local` / `Cuida Tu Agua` | Empty host = e-mail channel off |
| `Push:Enabled` / `ExpoUrl` / `ExpoAccessToken` | `true` / Expo endpoint / optional | Push channel |

Background jobs: e-mail delivery every 10 s; offline detection every 60 s.

## Endpoints

| Method | Route | Auth |
|--------|-------|------|
| GET | `/api/notifications?unreadOnly&before&limit` | user |
| GET | `/api/notifications/unread-count` | user |
| POST | `/api/notifications/{id}/read` | user |
| POST | `/api/notifications/read-all` | user |
| GET | `/api/notification-preferences` | user |
| PUT | `/api/notification-preferences/{severity}` `{inApp,push,email,sms}` | user |
| POST | `/api/push-tokens` `{token,platform}` → 204 | user |
| DELETE | `/api/push-tokens` `{token}` → 204 | user |
| GET | `/health` (database) | anonymous |

## Related documents

- [data-model.md](./data-model.md) · [events.md](./events.md) · [decisions.md](./decisions.md) · [runbook.md](./runbook.md)
- Diagrams: `08-uml/diagrams/exports/CL-06_ms-notification.png`, `ER-06_notification.png`, `SQ-12_sensor_desconectado.png`
