# ms-consumption

> Owner of the **water readings** and of the **consumption per place**. It stores every reading the
> meter sends (once), adds it to an hourly total and answers the consumption panel of the app.

---

## Location in the architecture

| Field | Value |
|-------|-------|
| Number in catalog | 04 |
| Bounded context | BC-04 Consumption Monitoring (Core) |
| Repository | `sy-water-backend/ms-consumption` (solution `SyWater.Consumption.slnx`) |
| Database repository | `sy-water-db/ms-consumption-db` (Liquibase) |
| Local port | **3004** |
| Stack | .NET 10 · ASP.NET Core · EF Core SqlServer · JwtBearer · RabbitMQ.Client · StackExchange.Redis |
| Storage | SQL Server, schema `monitoring` (login `consumption_app`) · Redis (token denylist check) |
| Calls | ms-places (`GET /api/places/{id}` with the user's JWT) |
| Called by | App; ms-places (cost of the period) |
| Consumes | `device.reading.received` (queue `consumption.readings`) |

---

## Responsibilities (what this service DOES)

| HU | Capability |
|----|------------|
| HU-015 | Panel of the selected place: consumption of today by default, current flow (if the last reading is less than 2 min old) and time of the last reading |
| HU-016 | Scales `day` (bars per hour), `week` and `month` (bars per day) in the user's time zone (`tz`, default `America/Bogota`) |
| HU-017/018 | Totals of the period and chart data (always in liters; the app converts to the place's display unit) |
| — | Idempotent ingestion: a reading is stored only once per device and `recordedAt`; duplicates from RabbitMQ are ignored |

## Out of scope

- Talking to the devices or validating their token → ms-devices (readings arrive already authenticated).
- ~~Goals (E6)~~ → **implemented** (HU-023, HU-024; see the as-built section).
- Leak detection (HU-029), bills (E11) and reports/export (E13, HU-038) — **will not be done** (E-03, E-02, E-05).
- Daily and monthly tables: `06-data/models.md` plans them, but the service aggregates from `consumption_hourly` on demand (DEC-002).

---

## As-built at project close (2026-10-28)

> Added at project close (ADR-021). Describes the `main` branch of the repository; where the sections above or
> below say otherwise, this section wins.

| HU | Capability added after the sections above were written |
|----|----------------------------------------------------------|
| HU-023 | Monthly goal per place: `PUT /api/places/{id}/goals/current` (`goals.consumption_goals`, `ms-consumption-db` v1.1). Only manual goals are created: the column `source` accepts MANUAL/SUGGESTED/CARRIED_OVER, but suggested and carried-over goals were not implemented |
| HU-024 | Progress against the goal: `GET /api/places/{id}/goals/current` |
| HU-040 | Guest consumption: `GET /api/guest/{token}/consumption`, token resolved with ms-places `/internal/guest-links/{token}` |
| — | Collaborators need `VIEW_CONSUMPTION` (access checked with ms-places) |

- Last change in `main`: `ef3b048` (2026-10-24).
- Not done: goal alerts (HU-030, HU-031 — E-01); tooltip with the exact value of a bar (HU-017, UI only — E-07).

---

## How to run it locally

```powershell
cd sy-water-backend\ms-consumption
dotnet user-secrets set "ConnectionStrings:Consumption" "Server=localhost,1433;Database=sy-water-db;User Id=consumption_app;Password=********;TrustServerCertificate=True" --project src/SyWater.Consumption.Api
dotnet user-secrets set "RabbitMq:Password" "********" --project src/SyWater.Consumption.Api
Copy-Item ..\ms-iam\keys\public.pem src\SyWater.Consumption.Api\keys\iam-public.pem
dotnet test
dotnet run --project src/SyWater.Consumption.Api     # http://localhost:3004/health
```

Without `RabbitMq:Host` the service starts but receives no readings.

## Configuration

| Key | Default | Purpose |
|-----|---------|---------|
| `ConnectionStrings:Consumption` | — (user-secrets) | `consumption_app` connection |
| `Jwt:*` | as in every .NET service | ms-iam token validation |
| `Redis:Configuration` | `localhost:6379` | Denylist check (fails closed: Redis down = 401) |
| `Services:PlacesBaseUrl` | `http://localhost:3002` | Ownership check |
| `RabbitMq:Host` / `Port` / `Username` / `Password` | `localhost` / `5672` / `sywater` / — | Consumer; `Prefetch` 20, reconnect 5 s, dead letters to `sywater.dlx` |

## Endpoints

| Method | Route | Auth |
|--------|-------|------|
| GET | `/api/places/{placeId}/consumption?period=day\|week\|month&tz=America/Bogota` | user |
| GET | `/api/places/{placeId}/goals/current` | user (place access) |
| PUT | `/api/places/{placeId}/goals/current` | user (owner) |
| GET | `/api/guest/{token}/consumption` | anonymous (guest link, rate-limited at the gateway) |
| GET | `/health` (database) | anonymous |

Errors: `consumption.invalid_period` (400), `consumption.invalid_time_zone` (400), `place.not_found` (404).

## Related documents

- [data-model.md](./data-model.md) · [events.md](./events.md) · [decisions.md](./decisions.md) · [runbook.md](./runbook.md)
- Diagrams: `08-uml/diagrams/exports/CL-04_ms-consumption.png`, `ER-04_monitoring.png`, `SQ-04_telemetria_consumo.png`
