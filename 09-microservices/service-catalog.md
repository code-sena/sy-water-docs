# Microservices Catalog

> Executive view of every service in Cuida Tu Agua — one row per service.
> Detail per service: `09-microservices/services/NN-service-name/`.
> Last aligned with the code: **2026-10-10** (deliveries E1–E5, E7 notifications, admin HU-059/060/062–064).

---

## Services map (as built)

```
  Mobile/Web app (Expo)        Admin panel (Vite)              ESP32 meters (MicroPython)
  save-your-water              sy-water-admin                  firmware 2.0
        │ HTTPS + JWT                │ /iam, /devices (Vite proxy)        │ MQTT 1883
        │                            │                                    ▼
        │                            │                          ┌──────────────────┐
        │                            │                          │ Mosquitto 2.0.22 │
        │                            │                          └────────┬─────────┘
        ▼                            ▼                                   │
 ┌──────────┐ ┌───────────┐ ┌────────────┐ ┌────────────────┐ ┌──────────┐
 │ ms-iam   │ │ ms-places │ │ ms-devices │ │ ms-consumption │ │ ms-valve │
 │  :3001   │ │   :3002   │ │   :3003    │ │     :3004      │ │  :3005   │
 │ Java 21  │ │  .NET 10  │ │  .NET 10   │ │    .NET 10     │ │ .NET 10  │
 └────┬─────┘ └─────┬─────┘ └─────┬──────┘ └───────┬────────┘ └────┬─────┘
      │             │             │ publishes      │ consumes      │ consumes
      │             │             └──────────► RabbitMQ 4.1 ◄──────┴───────┘
      │             │                         exchange sywater.events ──► ms-notifications :3006
      │             │                         (dead letters: sywater.dlx)   .NET 10 · e-mail · Expo push
      └─────────────┴──────────── SQL Server 2022 · database sy-water-db ───────
                                   (one schema per service) + Redis 7 (token denylist)
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
| 01 | `ms-iam` | `sy-water-backend/ms-iam` · DB `ms-iam-db` | Identity, sessions, JWT, profile, one-time codes | 3001 | Java 21 · Spring Boot 4.1.1 | `security` | 🟢 Implemented |
| 02 | `ms-places` | `ms-places` · DB `ms-places-db` | Places of each user and location catalog (Colombia and Ecuador) | 3002 | .NET 10 · ASP.NET Core | `places` | 🟢 Implemented |
| 03 | `ms-devices` | `ms-device` · DB `ms-devices-db` | Meter linking, status, MQTT ingestion, valve orders | 3003 | .NET 10 · MQTTnet · RabbitMQ.Client | `devices` | 🟢 Implemented |
| 04 | `ms-consumption` | `ms-consumption` · DB `ms-consumption-db` | Readings and consumption per hour, day, week and month | 3004 | .NET 10 · RabbitMQ.Client | `monitoring` | 🟢 Implemented |
| 05 | `ms-valve` | `ms-valve` · DB `ms-valve-db` | Confirmed valve state, open/close orders, history | 3005 | .NET 10 · RabbitMQ.Client | `valve` | 🟢 Implemented |
| 06 | `ms-notifications` | `ms-notification` · DB `ms-notification-db` | In-app, push and e-mail alerts (E7) | 3006 | .NET 10 · RabbitMQ.Client · MailKit | `notification` | 🟢 Implemented |
| 00 | `api-gateway` | `cuida-tu-agua/api-gateway` | Single entry point: prefix routing, JWT validation, CORS, Redis rate limiting, correlation id | 8080 | Java 21 · Spring Boot 4.1 · Spring Cloud Gateway 5 | — | ✅ Built |

**Statuses:** 🟢 Implemented | ✅ Built | 🟡 In development | 🔴 Planned | ⏸ Deprecated / not built

### Clients and infrastructure (not microservices)

| Component | Repository | Port | Purpose |
|-----------|------------|------|---------|
| `save-your-water` | `sy-water-frontend/save-your-water` | 19006 / 8081 (Expo) | User app: Expo SDK 57, React Native 0.86, React 19 |
| `sy-water-admin` | `sy-water-frontend/sy-water-admin` | 5173 | Admin web panel: React 19 + Vite 8 (HU-061) |
| Firmware | `sy-water-hardware/firmware` | — | MicroPython 2.0 for the ESP32 + PC simulator |
| SQL Server 2022 | created by `ms-iam-db` | 1433 | Database `sy-water-db` |
| Mosquitto 2.0.22 | `ms-device/compose.yml` | 1883 | MQTT broker with users and per-topic ACL |
| RabbitMQ 4.1 | `ms-device/compose.yml` | 5672 / 15672 | Integration events between services |
| Redis 7 | `ms-iam/compose.yml` | 6379 | Revoked access tokens (HU-006) |
| Mailpit | `ms-iam/compose.yml` | 1025 / 8025 | Development SMTP |

---

## Detail per service

### 01 — `ms-iam`

| Field | Value |
|-------|-------|
| **Folder** | `09-microservices/services/01-ms-iam/` |
| **Bounded context** | BC-01 IAM (Supporting) |
| **Responsibility** | Registration, e-mail verification, login with lockout, refresh-token rotation, logout, password recovery, profile and avatar, account deletion, one-time action codes |
| **Calls** | ms-devices and ms-places (`/internal/*`, `X-Internal-Key`) · SMTP · Redis |
| **Called by** | App, admin panel, ms-valve (`/action-codes/verify`), ms-notifications (`/internal/users/{id}/contact`) |

**Key endpoints:** `POST /api/auth/register`, `POST /api/auth/login`, `POST /api/auth/refresh`, `POST /api/auth/logout`, `GET|PATCH|DELETE /api/users/me`, `POST /api/users/me/action-codes`, `GET /api/admin/users`, `PUT /api/admin/users/{id}/block|unblock`, `GET /api/admin/metrics`.
**Events:** none (it only uses REST).

### 02 — `ms-places`

| Field | Value |
|-------|-------|
| **Folder** | `09-microservices/services/02-ms-places/` |
| **Bounded context** | BC-02 Place Management (Supporting) |
| **Responsibility** | Create, list, select, edit and soft-delete places; location catalog; tariffs and cost; savings tips |
| **Calls** | ms-devices (does the place have a linked meter?) · ms-consumption (liters for the cost) |
| **Called by** | App, ms-devices, ms-consumption, ms-valve (ownership check with the user's JWT) |

**Key endpoints:** `GET|POST /api/places`, `GET|PUT|DELETE /api/places/{id}`, `PUT /api/places/{id}/default`, `GET /api/geography/...`, `/api/places/{id}/tariff|cost`, `/api/tips`, `/api/admin/tips`.
**Events:** none yet.

### 03 — `ms-devices`

| Field | Value |
|-------|-------|
| **Folder** | `09-microservices/services/03-ms-devices/` |
| **Bounded context** | BC-03 Device Management (Supporting) |
| **Responsibility** | Link/unlink meters, connection status, heartbeat/telemetry/valve-state ingestion over MQTT, valve orders |
| **Calls** | ms-places · Mosquitto · RabbitMQ |
| **Called by** | App, ms-places, ms-valve and ms-iam (`X-Internal-Key`) |

**Key endpoints:** `GET|POST|DELETE /api/places/{id}/device`, `POST /api/places/{id}/device/valve-commands` (internal), `DELETE /internal/users/{id}/devices`, `GET /internal/metrics`.
> The factory registration / fleet API (`/api/admin/devices/*`) described in the guides is **not** in the code (risk R-004).
**Events published:** `device.linked`, `device.unlinked`, `device.reading.received`, `device.valve.reported`.

### 04 — `ms-consumption`

| Field | Value |
|-------|-------|
| **Folder** | `09-microservices/services/04-ms-consumption/` |
| **Bounded context** | BC-04 Consumption Monitoring (Core) |
| **Responsibility** | Store every reading and aggregate liters per place and hour; answer day/week/month queries |
| **Calls** | ms-places |
| **Called by** | App, ms-places (cost of the period) |

**Key endpoints:** `GET /api/places/{id}/consumption?period=day|week|month&tz=…`.
**Events consumed:** `device.reading.received` (queue `consumption.readings`).

### 05 — `ms-valve`

| Field | Value |
|-------|-------|
| **Folder** | `09-microservices/services/05-ms-valve/` |
| **Bounded context** | BC-07 Valve Control (Core) |
| **Responsibility** | Keep the state confirmed by the device, create and track open/close orders (close needs an e-mail code), history |
| **Calls** | ms-iam, ms-places, ms-devices |
| **Called by** | App |

**Key endpoints:** `GET /api/places/{id}/valve`, `POST /api/places/{id}/valve/close|open`, `GET /api/places/{id}/valve/commands`.
**Events consumed:** `device.linked`, `device.unlinked`, `device.valve.reported` (queue `valve.device-events`).

### 06 — `ms-notifications`

| Field | Value |
|-------|-------|
| **Folder** | `09-microservices/services/06-ms-notifications/` |
| **Bounded context** | BC-09 Notification Delivery (Generic) |
| **Responsibility** | Notification center, push (Expo), e-mail with outbox, channel preferences per urgency level, offline-meter and valve-closed alerts |
| **Calls** | ms-iam (`/internal/users/{id}/contact`) · SMTP · Expo Push |
| **Called by** | App |

**Key endpoints:** `GET /api/notifications`, `GET /api/notifications/unread-count`, `POST /api/notifications/{id}/read|read-all`, `GET|PUT /api/notification-preferences`, `POST|DELETE /api/push-tokens`.
**Events consumed:** `device.linked`, `device.unlinked`, `device.valve.reported`, `device.reading.received` (queue `notification.device-events`).

---

## Service communication matrix

| Source | Destination | Channel | Type | Description |
|--------|-------------|---------|------|-------------|
| App / admin panel | all services | HTTP/REST + JWT | Synchronous | User and admin operations |
| ESP32 | ms-devices | MQTT via Mosquitto | Asynchronous | `heartbeat` (30 s), `telemetry` (10 s), `valve/state` (60 s and after each order) |
| ms-devices | ESP32 | MQTT via Mosquitto | Asynchronous | `valve/set` — each meter only reads its own topic |
| ms-devices | ms-places | HTTP/REST | Synchronous | Ownership check (`GET /api/places/{id}` with the user's JWT) |
| ms-places | ms-devices | HTTP/REST | Synchronous | Refuse to delete a place with a linked meter |
| ms-devices | ms-consumption | RabbitMQ `device.reading.received` | Asynchronous | Validated readings |
| ms-devices | ms-valve | RabbitMQ `device.linked`, `device.unlinked`, `device.valve.reported` | Asynchronous | Valve lifecycle and confirmed state |
| ms-consumption | ms-places | HTTP/REST | Synchronous | Ownership check |
| ms-valve | ms-places | HTTP/REST | Synchronous | Ownership check |
| ms-valve | ms-iam | HTTP/REST | Synchronous | Verify the close-valve e-mail code |
| ms-valve | ms-devices | HTTP/REST + `X-Internal-Key` | Synchronous | Send the order to the device |
| ms-devices | ms-notifications | RabbitMQ `device.*` | Asynchronous | Linked/unlinked, valve changes, proof of life |
| ms-notifications | ms-iam | HTTP/REST + `X-Internal-Key` | Synchronous | E-mail address of the user |
| ms-iam | ms-devices | HTTP/REST + `X-Internal-Key` | Synchronous | Release meters of a deleted account (HU-008); device metrics (HU-062) |
| ms-iam | ms-places | HTTP/REST + `X-Internal-Key` | Synchronous | Place metrics (HU-062) |
| ms-places | ms-consumption | HTTP/REST | Synchronous | Liters of the period for the cost (HU-056) |

> **Known cycle:** ms-places ↔ ms-devices call each other (ownership check one way, "has a linked
> meter?" the other way). Both calls are read-only and fail closed, so there is no deadlock risk,
> but the cycle should disappear when ms-places learns the link from `device.linked`/`device.unlinked`
> events instead of asking ms-devices.

---

## Data ownership matrix

| Entity / data | Owner (source of truth) | How others access it |
|---------------|-------------------------|----------------------|
| User, credentials, roles, sessions | ms-iam (`security`) | JWT claims (`sub`, `roles`); `/action-codes/verify` |
| Place, location catalog | ms-places (`places`) | REST `GET /api/places/{id}` (404 if not the owner) |
| Device, link history, admin audit | ms-devices (`devices`) | REST; RabbitMQ `device.*` events |
| Readings, hourly consumption | ms-consumption (`monitoring`) | REST |
| Valve state and commands | ms-valve (`valve`) | REST |
| Tariffs, tariff catalog, savings tips | ms-places (`places`) | REST |
| Notifications, preferences, push tokens, e-mail outbox | ms-notifications (`notification`) | REST |

Cross-schema foreign keys are allowed (single shared database, ADR-002), but **no service writes
into another service's schema**: each one connects with its own login (`security_app`,
`places_app`, `devices_app`, `consumption_app`, `valve_app`, `notification_app`).

---

## How to add a new service

1. Assign the next number in this catalog (next: `07`).
2. `cp -r 09-microservices/_template/service 09-microservices/services/NN-new-service`.
3. Create its database repository (`ms-xxx-db`) with the same Liquibase layout as `ms-iam-db`.
4. Fill in README, data-model, events, decisions and runbook.
5. Create the OpenAPI contract in `07-api/contracts/openapi/ms-xxx.yaml`.
6. Add the service here and to `05-architecture/overview.md`.
7. Record the decision in `05-architecture/decisions/records/`.

---

## Correlations

- Architecture and ADRs → `05-architecture/`
- API contracts → `07-api/contracts/openapi/`
- Events → `02-domain/domain-events.md`
- Data model → `06-data/models.md`
- Local setup → `10-devops/local-setup.md`
