# Data Model — ms-valve

> Schema `valve` in `sy-water-db`. Source of truth: `ms-valve-db` (Liquibase).
> ER diagram: `08-uml/diagrams/exports/ER-05_valve.png`.

## Database engine

| Item | Value |
|------|-------|
| Schema | `valve` |
| Runtime login | `valve_app` |
| ORM | EF Core 10 without migrations |

## Tables

### `valve.valve_states`

One row per place with a linked meter.

| Column | Type | Notes |
|--------|------|-------|
| `id` | UNIQUEIDENTIFIER PK | |
| `place_id` | FK → `places.places` UQ | |
| `device_id` | FK → `devices.devices` UQ | |
| `state` | NVARCHAR(10) | `OPEN` \| `CLOSED` \| `UNKNOWN` — only what the device confirmed |
| `last_confirmed_at` | DATETIME2 NULL | Drives "no communication" (3 min) |
| `created_at`, `updated_at` | DATETIME2 | |

### `valve.valve_commands`

Every order ever sent (never deleted: HU-022 history).

| Column | Type | Notes |
|--------|------|-------|
| `id` | UNIQUEIDENTIFIER PK | Also the `commandId` sent to the device |
| `place_id` | FK → `places.places` | |
| `device_id` | FK → `devices.devices` | |
| `action` | NVARCHAR(10) | `OPEN` \| `CLOSE` |
| `origin` | NVARCHAR(15) | `MANUAL` \| `AUTO_LEAK` (planned) |
| `status` | NVARCHAR(15) | `SENT` \| `ACK_SUCCESS` \| `ACK_TIMEOUT` \| `FAILED` |
| `requested_by` | FK → `security.users` NULL | NULL for automatic orders |
| `requested_by_name` | NVARCHAR(MAX) NULL | Name copied at request time, so the history needs no call to ms-iam |
| `confirmed_with_code` | BIT | True for manual closes |
| `created_at`, `sent_at`, `timeout_at`, `acknowledged_at` | DATETIME2 | `timeout_at` = `created_at` + 30 s |
| `failure_reason` | NVARCHAR(500) NULL | |

Index: `UX_vcmd_place_pending (place_id) WHERE status = 'SENT'` — one order in progress per place.
