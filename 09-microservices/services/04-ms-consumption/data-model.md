# Data Model — ms-consumption

> Schemas `monitoring` and `goals` in `sy-water-db`. Source of truth: `ms-consumption-db` (Liquibase, up to v1.1).
> ER diagram: `08-uml/diagrams/exports/ER-04_monitoring.png`.

## Database engine

| Item | Value |
|------|-------|
| Schema | `monitoring` |
| Runtime login | `consumption_app` |
| ORM | EF Core 10 without migrations |

## Tables

### `monitoring.readings`

Immutable readings received from the meters.

| Column | Type | Notes |
|--------|------|-------|
| `id` | UNIQUEIDENTIFIER PK | |
| `device_id` | FK → `devices.devices` | |
| `place_id` | FK → `places.places` | Place linked when the reading was taken |
| `flow_rate_lph` | DECIMAL(18,3) | Liters per **hour** (the event carries L/min; stored × 60) |
| `volume_liters` | DECIMAL(18,3) | Water of the interval; summing them gives the consumption |
| `cumulative_liters` | DECIMAL(18,3) | Device counter |
| `recorded_at` | DATETIME2 | Device time (UTC, NTP) |
| `received_at` | DATETIME2 | Server time |

Index: `UQ_reading_dev_time (device_id, recorded_at)` — no duplicates per device and instant.

### `monitoring.consumption_hourly`

Liters per place and UTC hour; base of the day, week and month charts.

| Column | Type | Notes |
|--------|------|-------|
| `id` | UNIQUEIDENTIFIER PK | |
| `place_id` | FK → `places.places` | |
| `hour_start` | DATETIME2 | UTC hour (13:47:10 → 13:00:00) |
| `total_liters` | DECIMAL(18,3) | Incremented atomically in the same transaction as the reading |
| `reading_count` | INT | |
| `created_at`, `updated_at` | DATETIME2 | |

Index: `UQ_hourly_place_hour (place_id, hour_start)`.

### `goals.consumption_goals` *(added 2026-10-28 — release v1.1, HU-023/HU-024)*

| Column | Type | Notes |
|--------|------|-------|
| `id` | UNIQUEIDENTIFIER PK | |
| `place_id` | FK → `places.places` | |
| `month_start` | DATE | Always day 1 (`CK_goal_day`) |
| `target_liters` | DECIMAL(18,3) | > 0 |
| `source` | NVARCHAR(15) | `MANUAL` (default) \| `SUGGESTED` \| `CARRIED_OVER` — only `MANUAL` is used by the code |
| `created_at`, `updated_at` | DATETIME2 | |

Constraint: `UQ_goal_place_month (place_id, month_start)` — one goal per place and month. Role `goals_rw` for the
runtime login.
