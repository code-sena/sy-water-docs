# Data Model — ms-places

> Schema `places` in `sy-water-db`. Source of truth: `ms-places-db` (Liquibase).

## Database engine

| Item | Value |
|------|-------|
| Schema | `places` |
| Runtime login | `places_app` — role `places_rw`: SELECT/INSERT/UPDATE, **no DELETE**, read-only on the catalog |
| Migration login | `places_migrator` (has `REFERENCES` on `security.users`) |
| ORM | EF Core 10 without migrations |

## Releases

| Tag | Content |
|-----|---------|
| `v1.0-baseline` | Catalog tables, `places`, seeds for Colombia and Ecuador |
| `v1.1-places-e2` | `activity_log` (HU-011) |

## Tables

### Catalog (read-only for the service)
| Table | Columns |
|-------|---------|
| `countries` | `id`, `code` UQ (CO, EC), `name`, `default_currency` (COP, USD), `default_unit` |
| `subdivisions` | `id`, `country_id` FK, `code`, `name` — UQ (`country_id`, `code`) |
| `cities` | `id`, `subdivision_id` FK, `code`, `name` — UQ (`subdivision_id`, `code`) |

Official codes: DANE DIVIPOLA (Colombia) and INEC DPA (Ecuador).

### `places.places`
| Column | Type | Notes |
|--------|------|-------|
| `id` | UNIQUEIDENTIFIER PK | |
| `owner_id` | FK → `security.users` | From the JWT `sub`; no cascade |
| `city_id` | FK → `cities` | The country comes from the city |
| `name` | NVARCHAR(200) | |
| `place_type` | NVARCHAR(15) | RESIDENTIAL \| COMMERCIAL |
| `address` | NVARCHAR(500) | |
| `currency` | NVARCHAR(3) | COP \| USD, copied from the country |
| `measurement_unit` | NVARCHAR(15) | LITERS \| CUBIC_METERS \| GALLONS (display only) |
| `is_default` | BIT | Filtered unique index: one selected place per owner |
| `created_at`, `updated_at`, `deleted_at` | DATETIME2 | Soft delete |

### `places.activity_log`
`id` BIGINT · `place_id` FK · `owner_id` · `action` (PLACE_DELETED) · `metadata` JSON · `created_at`. Append-only.

## Planned
- `place_members`, `invitations` (Entrega 3, E9/E10).
- Tariff tables (E12/E15) — pending decision between the `billing` schema (`06-data/models.md`) and `places` (2-day plan).
