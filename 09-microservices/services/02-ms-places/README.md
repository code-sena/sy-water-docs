# ms-places

> Owner of the **places** of each user (house, shop, warehouse) and of the read-only **location
> catalog** (countries → subdivisions → cities). Every other service asks ms-places whether a place
> belongs to the caller.

---

## Location in the architecture

| Field | Value |
|-------|-------|
| Number in catalog | 02 |
| Bounded context | BC-02 Place Management (Supporting) |
| Repository | `sy-water-backend/ms-places` |
| Database repository | `sy-water-db/ms-places-db` (Liquibase) |
| Local port | **3002** (`http://0.0.0.0:3002`, reachable from the phone on the LAN) |
| Stack | .NET 10 · ASP.NET Core · EF Core SqlServer 10 · JwtBearer · StackExchange.Redis |
| Storage | SQL Server, schema `places` (login `places_app`) · Redis (token denylist check) |
| Calls | ms-devices (`GET /api/places/{id}/device`) before deleting a place |
| Called by | App; ms-devices, ms-consumption and ms-valve (`GET /api/places/{id}` with the user's JWT) |

---

## Responsibilities (what this service DOES)

| HU | Capability |
|----|------------|
| HU-009 | Create a place: name, type (RESIDENTIAL/COMMERCIAL), city, address, display unit. The first place becomes the selected one |
| HU-010 | List only the caller's places; select one (`is_default`), remembered between sessions |
| HU-011 | Edit; delete with soft delete, refused if a meter is linked; deletion logged in `places.activity_log` |
| HU-053 | The country of the city defines the currency (COP for Colombia, USD for Ecuador) |
| HU-055 | Display unit per place: LITERS, CUBIC_METERS or GALLONS (data is always stored in liters) |
| — | Location catalog for the form: Colombia (departments → municipalities) and Ecuador (provinces → cantons) |

A place that does not belong to the caller answers **404** (never 403), so its existence is not revealed.

## Out of scope

- Members and invitations (E9/E10) — planned (`place_members`, `invitations`).
- Tariffs (E12/E15) — planned. `06-data/models.md` places them in the `billing` schema; see the 2-day plan for the chosen location.
- Consumption, devices and valve data.

---

## How to run it locally

```powershell
cd sy-water-backend\ms-places
dotnet user-secrets set "ConnectionStrings:Places" "Server=localhost,1433;Database=sy-water-db;User Id=places_app;Password=********;TrustServerCertificate=True" --project src/SyWater.Places.Api
Copy-Item ..\ms-iam\keys\public.pem src\SyWater.Places.Api\keys\iam-public.pem
dotnet test
dotnet run --project src/SyWater.Places.Api       # http://localhost:3002/health
```

## Configuration

| Key | Purpose |
|-----|---------|
| `ConnectionStrings:Places` | `places_app` connection (user-secrets) |
| `Jwt:Issuer`, `Jwt:KeyId`, `Jwt:PublicKeyPath` | ms-iam token validation |
| `Redis:Configuration` | Denylist check (fails closed) |
| `Services:DevicesBaseUrl` | ms-devices; empty = "no device" adapter for isolated runs |
| `Cors:AllowedOrigins` | Web origins |

## Related documents

- [data-model.md](./data-model.md) · [events.md](./events.md) · [decisions.md](./decisions.md) · [runbook.md](./runbook.md)
- API contract: `07-api/contracts/openapi/ms-places.yaml`
