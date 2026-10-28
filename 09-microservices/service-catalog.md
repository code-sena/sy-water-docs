# Microservices Catalog

> Executive view of every service in Cuida Tu Agua — one row per service.
> Detail per service: `09-microservices/services/NN-service-name/`.
> Last aligned with the code: **2026-10-28 — final state at project close** (`main` of every repository, ADR-021).
> Previous alignment: 2026-10-10. Changes since then: sharing (E9, E10), goals (HU-023, HU-024), Google sign-in
> (HU-004), SMS/WhatsApp (HU-028), message templates, device administration API (R-004), offline after 1 minute
> (ADR-015), admin area web-only inside the app (ADR-011).

---

## Services map (as built)

```
   App web + Android (Expo)                                        ESP32 meters (MicroPython 2.0.1)
   cuida-tu-agua-frontNEW · admin area web-only                     cuida-tu-agua-device-ESP
        │ HTTPS + JWT                                                      │ MQTT 1883 (no TLS)
        ▼                                                                  ▼
   Caddy (sy-water.tech, api.sy-water.tech) ──► api-gateway :8080    Mosquitto 2.0.22
        │  /iam /places /devices /consumption /valve /notifications        │
        ▼                                                                  │
 ┌──────────┐ ┌───────────┐ ┌────────────┐ ┌────────────────┐ ┌──────────┐ │ ┌──────────────────┐
 │ ms-iam   │ │ ms-places │ │ ms-devices │◄┼────────────────┼─┼──────────┼─┘ │ ms-notifications │
 │  :3001   │ │   :3002   │ │   :3003    │ │ ms-consumption │ │ ms-valve │   │      :3006       │
 │ Java 21  │ │  .NET 10  │ │  .NET 10   │ │  :3004 .NET 10 │ │:3005 .NET│   │ .NET 10 · e-mail │
 └────┬─────┘ └─────┬─────┘ └─────┬──────┘ └───────┬────────┘ └────┬─────┘   │ push · WhatsApp  │
      │             │             │ publishes      │ consumes      │ consumes└────────┬─────────┘
      │             │             └──────────► RabbitMQ 4.1 ◄──────┴────────────────────┘ consumes
      │             │                         exchange sywater.events (dead letters: sywater.dlx)
      └─────────────┴──────────── SQL Server 2022 · database sy-water-db ───────
                                   (7 schemas, one owner each) + Redis 7 (denylist, rate limits)
```

Clients enter through the **API gateway** (`api-gateway`, port 8080): one URL with a prefix per service
(`/iam`, `/places`, `/devices`, `/consumption`, `/valve`, `/notifications`). The gateway validates the ms-iam JWT
(RS256 public key, issuer, expiry, Redis revocation list), answers CORS, applies Redis rate limits, adds
`X-Correlation-Id` and hides `/internal/**`. Every service **still** validates the JWT itself with the public key
of ms-iam (defense in depth, see ADR-008). Detail: `services/00-api-gateway/README.md`.

---

## Service registry

| # | Service | Repository | Main responsibility | Port | Stack | Schema | Status |
|---|---------|------------|---------------------|------|-------|--------|--------|
| 01 | `ms-iam` | `cuida-tu-agua/iam-backend` · DB `ms-iam-db` | Identity, sessions, JWT, Google sign-in, profile, phone verification, one-time codes | 3001 | Java 21 · Spring Boot 4.1.1 | `security` | 🟢 Implemented |
| 02 | `ms-places` | `ms-places` · DB `ms-places-db` | Places, location catalog (Colombia and Ecuador), tariffs and cost, tips, sharing (guest links, collaborators, invitations) — access authority | 3002 | .NET 10 · ASP.NET Core | `places` | 🟢 Implemented |
| 03 | `ms-devices` | `ms-devices` · DB `ms-device-db` | Meter linking, status, MQTT ingestion, valve orders, device administration (factory registration) | 3003 | .NET 10 · MQTTnet · RabbitMQ.Client | `devices` | 🟢 Implemented |
| 04 | `ms-consumption` | `ms-consumption` · DB `ms-consumption-db` | Readings and consumption per hour, day, week and month; monthly goals; guest consumption | 3004 | .NET 10 · RabbitMQ.Client | `monitoring` | 🟢 Implemented |
| 05 | `ms-valve` | `ms-valve` · DB `ms-valve-db` | Confirmed valve state, open/close orders, history | 3005 | .NET 10 · RabbitMQ.Client | `valve` | 🟢 Implemented |
| 06 | `ms-notifications` | `ms-notifications` · DB `ms-notification-db` | In-app, push, e-mail and WhatsApp/SMS notifications (E7, HU-028); message templates | 3006 | .NET 10 · RabbitMQ.Client · MailKit · Twilio / Meta | `notification` | 🟢 Implemented |
| 00 | `api-gateway` | `cuida-tu-agua/api-gateway` | Single entry point: prefix routing, JWT validation, CORS, Redis rate limiting, correlation id | 8080 | Java 21 · Spring Boot 4.1 · Spring Cloud Gateway 5 | — | ✅ Built |

**Statuses:** 🟢 Implemented | ✅ Built | 🟡 In development | 🔴 Planned | ⏸ Deprecated / not built

### Clients and infrastructure (not microservices)

| Component | Repository | Port | Purpose |
|-----------|------------|------|---------|
| App | `cuida-tu-agua/cuida-tu-agua-frontNEW` (local folder `save-your-water`) | 19006 / 8081 (Expo) | Web + Android app with the admin area (web only): Expo SDK 57, React Native 0.86, React 19 (ADR-011) |
| ~~`sy-water-admin`~~ | — (no repository in `cuida-tu-agua`) | — | **Not delivered**; admin is part of the app (ADR-011, E-11) |
| Firmware | `cuida-tu-agua/cuida-tu-agua-device-ESP` (branch `dev` only) | — | MicroPython 2.0.1 for the ESP32 + PC simulator |
| Legacy app | `cuida-tu-agua/cuida-tu-agua-frontend` | — | First frontend, superseded; nothing deployed from it |
| SQL Server 2022 | created by `ms-iam-db` | 1433 | Database `sy-water-db` |
| Mosquitto 2.0.22 | `infra/compose.yml` (standalone: `ms-devices/compose.yml`) | 1883 | MQTT broker with users and per-topic ACL |
| RabbitMQ 4.1 | `infra/compose.yml` (standalone: `ms-devices/compose.yml`) | 5672 / 15672 | Integration events between services |
| Redis 7 | `infra/compose.yml` (standalone: `ms-iam/compose.yml`) | 6379 | Revoked access tokens (HU-006), gateway rate limits |
| Mailpit | `ms-iam/compose.yml` | 1025 / 8025 | Development SMTP |
| Full system | `infra` | 80 / 443 (prod) · 8080 / 8082 (local) | Docker Compose for local and production, Caddy, Mosquitto config, Jenkins release pipeline (`10-devops/ci-cd.md`) |

---

## Detail per service

### 01 — `ms-iam`

| Field | Value |
|-------|-------|
| **Folder** | `09-microservices/services/01-ms-iam/` |
| **Bounded context** | BC-01 IAM (Supporting) |
| **Responsibility** | Registration, e-mail verification, login with lockout, Google sign-in (HU-004), refresh-token rotation, logout, password recovery, profile and avatar, account deletion, one-time action codes, phone verification by SMS/WhatsApp (HU-028), collaborator invitation e-mail |
| **Calls** | ms-devices and ms-places (`/internal/*`, `X-Internal-Key`) · ms-notifications (`/internal/short-messages/verification-code`) · SMTP · Redis · Google OAuth |
| **Called by** | App, ms-valve (`/action-codes/verify`), ms-notifications (`/internal/users/{id}/contact`), ms-places (`/internal/users/by-email`, `/internal/users/{id}/contact`, `/internal/invitation-emails`) |

**Key endpoints:** `POST /api/auth/register`, `POST /api/auth/login`, `POST /api/auth/refresh`, `POST /api/auth/logout`, `GET|PATCH|DELETE /api/users/me`, `POST /api/users/me/action-codes`, `/api/auth/google/*`, `POST /api/users/me/phone-verification[/confirm]`, `GET /api/admin/users`, `PUT /api/admin/users/{id}/block|unblock`, `GET /api/admin/metrics`.
**Events:** none (it only uses REST). Full list: `07-api/endpoint-catalog.md`.

### 02 — `ms-places`

| Field | Value |
|-------|-------|
| **Folder** | `09-microservices/services/02-ms-places/` |
| **Bounded context** | BC-02 Place Management (Supporting) |
| **Responsibility** | Create, list, select, edit and soft-delete places; location catalog; tariffs and cost; savings tips; **sharing**: guest links, invitations, collaborators with permissions (ADR-014) |
| **Calls** | ms-devices (does the place have a linked meter?) · ms-consumption (liters for the cost) · ms-iam (user by e-mail, contact, invitation e-mail) · ms-notifications (in-app invitation notices) |
| **Called by** | App, ms-devices, ms-consumption, ms-valve (`/api/places/{id}/access` with the user's JWT), ms-consumption (`/internal/guest-links/{token}`), ms-notifications (`/internal/places/{id}/recipients`) |

**Key endpoints:** `GET|POST /api/places`, `GET|PUT|DELETE /api/places/{id}`, `PUT /api/places/{id}/default`, `GET /api/geography/...`, `/api/places/{id}/tariff|cost`, `/api/tips`, `/api/admin/tips`, `/api/places/{id}/access|guest-link|invitations|members`, `/api/shared-places`, `/api/invitations`, `GET /api/guest/{token}` (anonymous).
**Events:** none (sharing is integrated by HTTP, ADR-006).

### 03 — `ms-devices`

| Field | Value |
|-------|-------|
| **Folder** | `09-microservices/services/03-ms-devices/` |
| **Bounded context** | BC-03 Device Management (Supporting) |
| **Responsibility** | Link/unlink meters, connection status, heartbeat/telemetry/valve-state ingestion over MQTT, valve orders |
| **Calls** | ms-places · Mosquitto · RabbitMQ |
| **Called by** | App, ms-places, ms-valve and ms-iam (`X-Internal-Key`) |

**Key endpoints:** `GET|POST|DELETE /api/places/{id}/device`, `POST /api/places/{id}/device/valve-commands` (internal), `GET|POST /api/admin/devices`, `GET /api/admin/devices/{id}`, `POST /api/admin/devices/{id}/credentials|decommission`, `DELETE /internal/users/{id}/devices`, `GET /internal/metrics`.
> **Updated 2026-10-28:** the factory registration / fleet API (`/api/admin/devices/*`) exists since `ms-device-db` v1.1 (risk R-004 resolved). Offline after 1 minute (ADR-015).
**Events published:** `device.linked`, `device.unlinked`, `device.reading.received`, `device.valve.reported`.

### 04 — `ms-consumption`

| Field | Value |
|-------|-------|
| **Folder** | `09-microservices/services/04-ms-consumption/` |
| **Bounded context** | BC-04 Consumption Monitoring (Core) |
| **Responsibility** | Store every reading and aggregate liters per place and hour; answer day/week/month queries; monthly goals (HU-023, HU-024); consumption for guest links |
| **Calls** | ms-places (`/access`, `/internal/guest-links/{token}`) |
| **Called by** | App, ms-places (cost of the period) |

**Key endpoints:** `GET /api/places/{id}/consumption?period=day|week|month&tz=…`, `GET|PUT /api/places/{id}/goals/current`, `GET /api/guest/{token}/consumption` (anonymous).
**Events consumed:** `device.reading.received` (queue `consumption.readings`).

### 05 — `ms-valve`

| Field | Value |
|-------|-------|
| **Folder** | `09-microservices/services/05-ms-valve/` |
| **Bounded context** | BC-07 Valve Control (Core) |
| **Responsibility** | Keep the state confirmed by the device, create and track open/close orders (close needs an e-mail code), history |
| **Calls** | ms-iam, ms-places, ms-devices |
| **Called by** | App (owner, or collaborator with `CONTROL_VALVE`) |

**Key endpoints:** `GET /api/places/{id}/valve`, `POST /api/places/{id}/valve/close|open`, `GET /api/places/{id}/valve/commands`.
**Events consumed:** `device.linked`, `device.unlinked`, `device.valve.reported` (queue `valve.device-events`).

### 06 — `ms-notifications`

| Field | Value |
|-------|-------|
| **Folder** | `09-microservices/services/06-ms-notifications/` |
| **Bounded context** | BC-09 Notification Delivery (Generic) |
| **Responsibility** | Notification center, push (Expo), e-mail with outbox, SMS/WhatsApp with outbox (HU-028), message templates editable by admins, channel preferences per urgency level, offline-meter and valve-closed alerts to owner and collaborators, invitation notices |
| **Calls** | ms-iam (`/internal/users/{id}/contact`) · ms-places (`/internal/places/{id}/recipients`) · SMTP · Expo Push · Twilio (or Meta WhatsApp Cloud API) |
| **Called by** | App, ms-iam (verification code), ms-places (invitation notices) |

**Key endpoints:** `GET /api/notifications`, `GET /api/notifications/unread-count`, `POST /api/notifications/{id}/read|read-all`, `GET|PUT /api/notification-preferences`, `POST|DELETE /api/push-tokens`, `/api/admin/message-templates/*`, `GET /api/short-messages/channels`.
**Events consumed:** `device.linked`, `device.unlinked`, `device.valve.reported`, `device.reading.received` (queue `notification.device-events`).

---

## Service communication matrix

| Source | Destination | Channel | Type | Description |
|--------|-------------|---------|------|-------------|
| App (user and admin area) | all services, through api-gateway | HTTP/REST + JWT | Synchronous | User and admin operations |
| Anonymous browser | ms-places, ms-consumption (guest routes) | HTTP/REST, no token, rate-limited | Synchronous | Guest link view (HU-039 – HU-041) |
| ESP32 | ms-devices | MQTT via Mosquitto | Asynchronous | `heartbeat` (30 s), `telemetry` (10 s), `valve/state` (60 s and after each order) |
| ms-devices | ESP32 | MQTT via Mosquitto | Asynchronous | `valve/set` — each meter only reads its own topic |
| ms-devices | ms-places | HTTP/REST | Synchronous | Access check (`GET /api/places/{id}/access` and `/api/places/{id}` with the user's JWT) |
| ms-places | ms-devices | HTTP/REST | Synchronous | Refuse to delete a place with a linked meter |
| ms-devices | ms-consumption | RabbitMQ `device.reading.received` | Asynchronous | Validated readings |
| ms-devices | ms-valve | RabbitMQ `device.linked`, `device.unlinked`, `device.valve.reported` | Asynchronous | Valve lifecycle and confirmed state |
| ms-consumption | ms-places | HTTP/REST | Synchronous | Access check; resolve guest token (`/internal/guest-links/{token}`) |
| ms-valve | ms-places | HTTP/REST | Synchronous | Access check (`CONTROL_VALVE` for collaborators) |
| ms-valve | ms-iam | HTTP/REST | Synchronous | Verify the close-valve e-mail code |
| ms-valve | ms-devices | HTTP/REST + `X-Internal-Key` | Synchronous | Send the order to the device |
| ms-devices | ms-notifications | RabbitMQ `device.*` | Asynchronous | Linked/unlinked, valve changes, proof of life |
| ms-notifications | ms-iam | HTTP/REST + `X-Internal-Key` | Synchronous | E-mail address and phone of the user |
| ms-iam | ms-devices | HTTP/REST + `X-Internal-Key` | Synchronous | Release meters of a deleted account (HU-008); device metrics (HU-062) |
| ms-iam | ms-places | HTTP/REST + `X-Internal-Key` | Synchronous | Place metrics (HU-062) |
| ms-places | ms-consumption | HTTP/REST | Synchronous | Liters of the period for the cost (HU-056) |
| ms-places | ms-iam | HTTP/REST + `X-Internal-Key` | Synchronous | User by e-mail, contact, invitation e-mail with button (HU-042, HU-043) |
| ms-places | ms-notifications | HTTP/REST + `X-Internal-Key` | Synchronous | In-app notices of new and answered invitations |
| ms-notifications | ms-places | HTTP/REST + `X-Internal-Key` | Synchronous | Recipients of a place alert (`?permission=VIEW_ALERTS`, HU-047) |
| ms-iam | ms-notifications | HTTP/REST + `X-Internal-Key` | Synchronous | Send the phone verification code (HU-028) |
| ms-notifications | Twilio / Meta, Expo, SMTP | HTTPS / SMTP | Asynchronous (outbox) | WhatsApp/SMS, push, e-mail delivery |

> **Known cycles (accepted at close):** ms-places ↔ ms-devices, ms-places ↔ ms-iam and ms-places ↔ ms-notifications
> call each other. Original note on the first one: ms-places ↔ ms-devices call each other (ownership check one way, "has a linked
> meter?" the other way). Both calls are read-only and fail closed, so there is no deadlock risk,
> the proposal to remove it with `device.linked`/`device.unlinked` events was **not implemented** (project closed).

---

## Data ownership matrix

| Entity / data | Owner (source of truth) | How others access it |
|---------------|-------------------------|----------------------|
| User, credentials, roles, sessions | ms-iam (`security`) | JWT claims (`sub`, `roles`); `/action-codes/verify` |
| Place, location catalog | ms-places (`places`) | REST `GET /api/places/{id}` (404 if no access) |
| Collaborators, invitations, guest links | ms-places (`places`) | REST `GET /api/places/{id}/access`; internal recipients and guest-link resolution |
| Device, link history, admin audit | ms-devices (`devices`) | REST; RabbitMQ `device.*` events |
| Readings, hourly consumption | ms-consumption (`monitoring`) | REST |
| Monthly goals | ms-consumption (`goals`) | REST |
| Valve state and commands | ms-valve (`valve`) | REST |
| Tariffs, tariff catalog, savings tips | ms-places (`places`) | REST |
| Notifications, preferences, push tokens, e-mail and SMS outboxes, message templates | ms-notifications (`notification`) | REST |

Cross-schema foreign keys are allowed (single shared database, ADR-002), but **no service writes
into another service's schema**: each one connects with its own login (`security_app`,
`places_app`, `devices_app`, `consumption_app`, `valve_app`, `notification_app`).

---

## How to add a new service

> Kept for reference. The project is closed (ADR-021): no new service is planned.

1. Assign the next number in this catalog (next: `07`).
2. `cp -r 09-microservices/_template/service 09-microservices/services/NN-new-service`.
3. Create its database repository (`ms-xxx-db`) with the same Liquibase layout as `ms-iam-db`.
4. Fill in README, data-model, events, decisions and runbook.
5. Expose the runtime OpenAPI document and add it to the documentation UI; add its endpoints to `07-api/endpoint-catalog.md`.
6. Add the service here and to `05-architecture/overview.md`.
7. Record the decision in `05-architecture/decisions/records/`.

---

## Correlations

- Architecture and ADRs → `05-architecture/`
- API endpoints → `07-api/endpoint-catalog.md` (runtime OpenAPI at `https://api.sy-water.tech/docs`)
- Events → `02-domain/domain-events.md`
- Data model → `06-data/models.md`
- Local setup → `10-devops/local-setup.md`
