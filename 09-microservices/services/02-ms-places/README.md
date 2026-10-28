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
| Calls | ms-devices (`GET /api/places/{id}/device`) before deleting a place · ms-consumption (`GET /api/places/{id}/consumption`) for the cost of the period |
| Called by | App; ms-devices, ms-consumption and ms-valve (`GET /api/places/{id}` with the user's JWT); ms-iam (`GET /internal/metrics`) |

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
| HU-054, HU-066 | Tariff of a place: manual (unit price + fixed charge) or from the Colombian catalog by city and stratum (basic / complementary / luxury ranges) |
| HU-056, HU-069 | Consumption in money: estimated cost of the period with the liters of ms-consumption and the current tariff |
| HU-063/064 | Admin: create, edit and activate/deactivate savings tips (never deleted) |
| HU-065 | Users read active tips and mark favorites |
| HU-062 | Place counts for the admin dashboard (`GET /internal/metrics`, `X-Internal-Key`) |

A place that does not belong to the caller answers **404** (never 403), so its existence is not revealed.

## Out of scope

- ~~Members and invitations (E9/E10) — planned~~ → **implemented** with guest links (see the as-built section).
- Bills and comparison against the sensor (E11) — **not implemented, will not be done** (E-02). Tariffs live in the `places` schema (not in the `billing` schema planned in `06-data/models.md`).
- Consumption, devices and valve data.

---

## As-built at project close (2026-10-28)

> Added at project close (ADR-021). Describes the `main` branch of the repository; where the sections above or
> below say otherwise, this section wins.

| HU | Capability added after the sections above were written |
|----|----------------------------------------------------------|
| HU-039 – HU-041 | Guest link: one active token per place (`places.guest_links`); owner reads, regenerates or deactivates it (`/api/places/{id}/guest-link`); anonymous read-only view `GET /api/guest/{token}` |
| HU-042, HU-043 | Invitations by e-mail (`places.invitations`, PENDING/ACCEPTED/EXPIRED/REJECTED); `GET /api/invitations`, `POST /api/invitations/{id}/accept|reject`; e-mail through ms-iam, in-app notice through ms-notifications |
| HU-044 – HU-046 | Collaborators (`places.place_members`) with permissions `VIEW_CONSUMPTION`, `VIEW_DEVICE_STATUS`, `VIEW_ALERTS`, `CONTROL_VALVE`, `VIEW_BILLS`; change permissions, revoke, leave (`DELETE /api/places/{id}/membership`); `GET /api/shared-places` |
| HU-047 | `GET /internal/places/{id}/recipients?permission=…` for ms-notifications |
| — | `GET /api/places/{id}/access`: access authority for ms-devices, ms-consumption and ms-valve (ADR-014) |

- Last change in `main`: `57889f4` (2026-10-26). Database: `ms-places-db` v1.4.
- `VIEW_BILLS` exists as a permission but has no feature behind it (bills not built, E-02).
- Not done: tariff catalogs for Ecuador and the United States (HU-067, HU-068 — E-06).

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
| `Services:ConsumptionBaseUrl` | ms-consumption (`http://localhost:3004`), required for the cost |
| `Internal:ApiKey` | Shared `X-Internal-Key` for `/internal/metrics` |
| `Cors:AllowedOrigins` | Web origins |

## Related documents

- [data-model.md](./data-model.md) · [events.md](./events.md) · [decisions.md](./decisions.md) · [runbook.md](./runbook.md)
- API contract: `07-api/contracts/openapi/ms-places.yaml`
