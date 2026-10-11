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

### `places.place_tariffs`
History of the tariff of a place: `id` BIGINT · `place_id` FK · `source` (`MANUAL` \| `CATALOG`) · `unit_price` · `fixed_charge` · `stratum` (catalog) · `valid_from` · `created_by`. The current tariff is the latest `valid_from`.

### `places.tariff_catalog`
Water tariffs per city and stratum: `city_id` FK · `stratum` · `currency` · `fixed_charge` · `basic_price` / `complementary_price` / `luxury_price` · `basic_limit_m3` / `complementary_limit_m3` · `effective_from` · `source`. Read-only for the app.

### `places.tips` and `places.tip_favorites`
`tips`: `id` · `title` (150) · `body` (1000) · `category` · `is_active` (deactivated, never deleted) · `created_by` / `updated_by` · timestamps.
`tip_favorites`: PK (`user_id`, `tip_id`) · `created_at`.

ER diagram: `08-uml/diagrams/exports/ER-02_places.png`.

## Planned
- `place_members`, `invitations` (Entrega 3, E9/E10).
