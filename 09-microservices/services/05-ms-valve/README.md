# ms-valve

> Owner of the **valve of each place**: the state confirmed by the device, the open/close orders and
> their history. Closing the water is critical, so it needs a code sent by e-mail and it is only
> considered done when the device confirms it.

---

## Location in the architecture

| Field | Value |
|-------|-------|
| Number in catalog | 05 |
| Bounded context | BC-07 Valve Control (Core) |
| Repository | `sy-water-backend/ms-valve` (solution `SyWater.Valve.slnx`) |
| Database repository | `sy-water-db/ms-valve-db` (Liquibase) |
| Local port | **3005** |
| Stack | .NET 10 · ASP.NET Core · EF Core SqlServer · JwtBearer · RabbitMQ.Client · StackExchange.Redis |
| Storage | SQL Server, schema `valve` (login `valve_app`) · Redis (token denylist check) |
| Calls | ms-places (ownership) · ms-iam (verify the e-mailed code) · ms-devices (send the order, `X-Internal-Key`) |
| Called by | App |
| Consumes | `device.linked`, `device.unlinked`, `device.valve.reported` (queue `valve.device-events`) |

---

## Responsibilities (what this service DOES)

| HU | Capability |
|----|------------|
| HU-019 | Valve state of the place: `OPEN`, `CLOSED` or `UNKNOWN` (never reported), plus "no communication" when the device has not confirmed for 3 min, and the order in progress |
| HU-020 | Close by hand with the 6-digit code that ms-iam e-mails (`VALVE_CLOSE` action code). The order is `SENT`; it becomes `ACK_SUCCESS` when the device reports the same `commandId`, or `ACK_TIMEOUT` after 30 s |
| HU-021 | Open by hand (no code) with the same order lifecycle |
| HU-022 | History of orders of a place: who, action, origin, result and times; default range the last 30 days |
| HU-012/014 | A valve row appears when a meter is linked and disappears when it is unlinked |

Only one order can be in progress per place (filtered unique index). The state only changes when the
**device** reports it, never when an order is sent.

## Out of scope

- Talking to the device → ms-devices.
- Notifying the owner that the valve closed (HU-033) → ms-notifications (from `device.valve.reported`).
- Automatic close on suspected leak (HU-035/036/037, origin `AUTO_LEAK`) — the origin exists in the model, the flow is not implemented.

---

## How to run it locally

```powershell
cd sy-water-backend\ms-valve
dotnet user-secrets set "ConnectionStrings:Valve" "Server=localhost,1433;Database=sy-water-db;User Id=valve_app;Password=********;TrustServerCertificate=True" --project src/SyWater.Valve.Api
dotnet user-secrets set "RabbitMq:Password" "********" --project src/SyWater.Valve.Api
dotnet user-secrets set "Internal:ApiKey" "********" --project src/SyWater.Valve.Api   # same value as ms-devices
Copy-Item ..\ms-iam\keys\public.pem src\SyWater.Valve.Api\keys\iam-public.pem
dotnet test
dotnet run --project src/SyWater.Valve.Api        # http://localhost:3005/health
```

## Configuration

| Key | Default | Purpose |
|-----|---------|---------|
| `ConnectionStrings:Valve` | — (user-secrets) | `valve_app` connection |
| `Jwt:*`, `Redis:Configuration` | as in every .NET service | Token validation and denylist (fails closed) |
| `Services:IamBaseUrl` / `PlacesBaseUrl` / `DevicesBaseUrl` | `http://localhost:3001` / `3002` / `3003` | All three are required |
| `Internal:ApiKey` | — (≥ 24 chars) | Sent to ms-devices as `X-Internal-Key` |
| `RabbitMq:*` | `localhost`, `5672`, `sywater`, `sywater.events` | Consumer, dead letters to `sywater.dlx` |

Fixed in code (`ValvePolicy.Default`): acknowledgement timeout 30 s, communication window 3 min; the timeout sweeper runs every 5 s.

## Endpoints

| Method | Route | Auth |
|--------|-------|------|
| GET | `/api/places/{placeId}/valve` | user |
| POST | `/api/places/{placeId}/valve/close` `{ "code": "123456" }` → 202 | user |
| POST | `/api/places/{placeId}/valve/open` → 202 | user |
| GET | `/api/places/{placeId}/valve/commands/{commandId}` | user (the app polls it) |
| GET | `/api/places/{placeId}/valve/commands?from&to` | user |
| GET | `/health` (database) | anonymous |

## Related documents

- [data-model.md](./data-model.md) · [events.md](./events.md) · [decisions.md](./decisions.md) · [runbook.md](./runbook.md)
- Diagrams: `08-uml/diagrams/exports/CL-05_ms-valve.png`, `ER-05_valve.png`, `SQ-05_cerrar_valvula.png`, `ST-03_orden_valvula.png`, `ST-04_valvula.png`
