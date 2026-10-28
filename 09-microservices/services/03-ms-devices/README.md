# ms-devices

> Owner of the **IoT meters** (ESP32 + YF-S201 flow sensor + normally-closed solenoid valve): factory
> credentials, link to a place, connection status and the MQTT channel. It is the only service that
> talks to the devices; the rest of the system learns what happens through RabbitMQ events.

---

## Location in the architecture

| Field | Value |
|-------|-------|
| Number in catalog | 03 |
| Bounded context | BC-03 Device Management (Supporting) |
| Repository | `sy-water-backend/ms-device` (solution `SyWater.Devices.slnx`) |
| Database repository | `sy-water-db/ms-devices-db` (Liquibase) |
| Local port | **3003** (`http://0.0.0.0:3003`, reachable from the phone on the LAN) |
| Stack | .NET 10 · ASP.NET Core · EF Core SqlServer · JwtBearer · MQTTnet 5.2 · RabbitMQ.Client 7.2 |
| Storage | SQL Server, schema `devices` (login `devices_app`) |
| Brokers | Mosquitto 2.0.22 (MQTT 1883, user `ms-devices`) · RabbitMQ 4.1 (exchange `sywater.events`) |
| Calls | ms-places (`GET /api/places/{id}` with the user's JWT) |
| Called by | App; ms-places (has the place a meter?); ms-valve (`valve-commands`, `X-Internal-Key`); ms-iam (`/internal/*`) |

---

## Responsibilities (what this service DOES)

| HU | Capability |
|----|------------|
| HU-012 | Link a meter to a place with serial + pairing code. The device must be online (authenticated heartbeat), one device per place and one place per device. 5 wrong codes lock the device for 15 min; each user has 10 attempts per 15 min (429 + `Retry-After`) |
| HU-013 | Connection status `CONNECTED` / `DISCONNECTED` / `NEVER_REPORTED`, from heartbeats and a sweeper that runs every 60 s against `inactivity_threshold_min` (10 min) — *superseded: 15 s / 1 min, see the as-built section and ADR-015* |
| HU-014 | Unlink a meter; the link history is kept forever (`device_link_history`) |
| HU-008 | Release every device linked by a user whose account was deleted (`DELETE /internal/users/{id}/devices`) |
| HU-020/021 | Forward valve orders from ms-valve to the device (`valve/set`) |
| HU-062 | Device counts for the admin dashboard (`GET /internal/metrics`) |
| — | MQTT ingestion: authenticates every message with the device token (SHA-256 hash), validates telemetry and publishes integration events |

A meter of another user answers **404**, like a place without meter.

## Out of scope

- Storing readings or aggregating consumption → ms-consumption.
- Valve state, order lifecycle and history → ms-valve.
- Alerts to the user → ms-notifications.
- ~~Factory registration / fleet administration endpoints (`/api/admin/devices`) — not present in the code (R-004)~~ → **implemented** (see the as-built section).

---

## As-built at project close (2026-10-28)

> Added at project close (ADR-021). Describes the `main` branch of the repository; where the sections above or
> below say otherwise, this section wins.

| Item | State |
|------|-------|
| Liveness (ADR-015) | `DefaultInactivityThresholdMinutes = 1`, `Devices:StatusSweepSeconds = 15`; heartbeat (30 s) and telemetry (10 s) both count as proof of life. `ms-device-db` v1.2 set the default to 1 and patched existing devices |
| Device administration (R-004 resolved) | `/api/admin/devices` (list, register with factory credentials, detail, regenerate credentials, decommission), audited in `devices.device_admin_log` (`ms-device-db` v1.1) |
| Sharing (ADR-014) | Access checked with `GET /api/places/{id}/access` of ms-places; collaborators need `VIEW_DEVICE_STATUS` |
| Telemetry guards | Readings more than 5 min ahead of the server clock or older than 24 h are rejected |
| Last change in `main` | `cff60ad` (2026-10-24) |
| Open | R-005: no Redis denylist check in this service (mitigated at the gateway, E-19). MQTT without TLS (R-006, E-13) |

---

## How to run it locally

```powershell
cd sy-water-backend\ms-device
Copy-Item .env.example .env                 # set RABBITMQ_PASSWORD
docker compose up -d                        # Mosquitto + RabbitMQ
dotnet user-secrets set "ConnectionStrings:Devices" "Server=localhost,1433;Database=sy-water-db;User Id=devices_app;Password=********;TrustServerCertificate=True" --project src/SyWater.Devices.Api
dotnet user-secrets set "Mqtt:Password" "********" --project src/SyWater.Devices.Api
dotnet user-secrets set "RabbitMq:Password" "********" --project src/SyWater.Devices.Api
dotnet user-secrets set "Internal:ApiKey" "********" --project src/SyWater.Devices.Api
Copy-Item ..\ms-iam\keys\public.pem src\SyWater.Devices.Api\keys\iam-public.pem
dotnet test
dotnet run --project src/SyWater.Devices.Api      # http://localhost:3003/health
```

Prerequisites: `ms-iam-db`, `ms-places-db` and `ms-devices-db` applied; ms-iam and ms-places running.
To simulate a meter without hardware: `scripts/simulate-device.ps1 -Serial <serial> -Token <token>`.

## Configuration

| Key | Default | Purpose |
|-----|---------|---------|
| `ConnectionStrings:Devices` | — (user-secrets) | `devices_app` connection |
| `Jwt:Issuer`, `Jwt:KeyId`, `Jwt:PublicKeyPath` | `https://iam.cuidatuagua.local`, `iam-key-1`, `keys/iam-public.pem` | ms-iam token validation |
| `Services:PlacesBaseUrl` | `http://localhost:3002` | Ownership check |
| `Mqtt:Host` / `Port` / `Username` / `ClientId` | `localhost` / `1883` / `ms-devices` / `ms-devices` | Broker connection (reconnects every 5 s) |
| `RabbitMq:Host` / `Port` / `Username` / `Exchange` | `localhost` / `5672` / `sywater` / `sywater.events` | Empty host = events are only logged |
| `Devices:RequireOnlineToLink` | `true` | HU-012 |
| `Devices:MaxFailedPairingAttempts` / `PairingLockMinutes` | `5` / `15` | Pairing lock |
| `Devices:LinkAttemptsPerUser` / `LinkAttemptsWindowMinutes` | `10` / `15` | Per-user rate limiter |
| `Devices:StatusSweepSeconds` | `60` | Status sweeper |
| `Internal:ApiKey` | — (user-secrets, ≥ 24 chars) | `X-Internal-Key` for internal endpoints |

## Endpoints

| Method | Route | Auth |
|--------|-------|------|
| GET | `/api/places/{placeId}/device` | user |
| POST | `/api/places/{placeId}/device` | user (rate limited) |
| DELETE | `/api/places/{placeId}/device` | user |
| POST | `/api/places/{placeId}/device/valve-commands` | user + `X-Internal-Key` (ms-valve) |
| DELETE | `/internal/users/{userId}/devices` | `X-Internal-Key` (ms-iam) |
| GET | `/internal/metrics` | `X-Internal-Key` (ms-iam) |
| GET / POST | `/api/admin/devices` | ADMIN (list, factory registration) |
| GET | `/api/admin/devices/{id}` | ADMIN |
| POST | `/api/admin/devices/{id}/credentials` | ADMIN (regenerate) |
| POST | `/api/admin/devices/{id}/decommission` | ADMIN |
| GET | `/health` (database + MQTT) | anonymous |

## Related documents

- [data-model.md](./data-model.md) · [events.md](./events.md) · [decisions.md](./decisions.md) · [runbook.md](./runbook.md)
- Diagrams: `08-uml/diagrams/exports/CL-03_ms-device.png`, `ER-03_devices.png`, `SQ-03_vincular_medidor.png`, `ST-02_medidor.png`
