# Data Model — ms-devices

> Schema `devices` in `sy-water-db`. Source of truth: `ms-devices-db` (Liquibase).
> ER diagram: `08-uml/diagrams/exports/ER-03_devices.png`.

## Database engine

| Item | Value |
|------|-------|
| Schema | `devices` |
| Runtime login | `devices_app` — SELECT/INSERT/UPDATE, **no DELETE** |
| Migration login | `devices_migrator` (has `REFERENCES` on `security.users` and `places.places`) |
| ORM | EF Core 10 without migrations (the service never migrates at startup) |
| Seeds | `02-dml/00-inserts/001-seed-dev-devices.sql`, context `dev` only: `SW-ESP32-000001` … `000005` |

## Tables

### `devices.devices`

Aggregate root `Device`. Secrets are stored only as SHA-256 hashes.

| Column | Type | Notes |
|--------|------|-------|
| `id` | UNIQUEIDENTIFIER PK | |
| `serial_number` | NVARCHAR(32) UQ | Printed on the device; MQTT client id |
| `auth_token_hash` | VARCHAR(64) | SHA-256 of the factory token sent in every MQTT payload |
| `auth_token_last_used_at` | DATETIME2 NULL | |
| `auth_token_revoked_at` | DATETIME2 NULL | Revoked device = `device.revoked` |
| `pairing_code_hash` | VARCHAR(64) | SHA-256 of `serial:code` |
| `pairing_failed_attempts` | INT | Reset on success |
| `pairing_locked_until` | DATETIME2 NULL | 15 min after 5 failures |
| `place_id` | FK → `places.places` NULL | Filtered unique index `UX_devices_place` (one device per place) |
| `linked_by` | FK → `security.users` NULL | Only this user sees the device |
| `linked_at` | DATETIME2 NULL | |
| `status` | NVARCHAR(15) | `CONNECTED` \| `DISCONNECTED` \| `NEVER_REPORTED` |
| `last_report_at` | DATETIME2 NULL | Last authenticated heartbeat/telemetry |
| `firmware_version` | NVARCHAR(20) NULL | From the heartbeat |
| `inactivity_threshold_min` | INT | Minutes without reports before `DISCONNECTED` — default **1** since `ms-device-db` v1.2 (was 10; ADR-015) |
| `created_at`, `updated_at` | DATETIME2 | |

Indexes: `UQ_devices_serial`, `UX_devices_place (place_id) WHERE place_id IS NOT NULL`, `IX_devices_status_report (status, last_report_at)`.

### `devices.device_link_history`

Append-only history of links (never deleted).

| Column | Type | Notes |
|--------|------|-------|
| `id` | UNIQUEIDENTIFIER PK | |
| `device_id` | FK → `devices.devices` | |
| `place_id` | FK → `places.places` | |
| `linked_by` / `unlinked_by` | FK → `security.users` | `unlinked_by` NULL while active |
| `linked_at` / `unlinked_at` | DATETIME2 | `unlinked_at` NULL while active |
| `created_at` | DATETIME2 | |

Indexes: `IX_linkh_device_time`, `IX_linkh_place`, `UX_linkh_device_active (device_id) WHERE unlinked_at IS NULL` (one open link per device).

### `devices.device_admin_log` *(added 2026-10-28 — release v1.1)*

Audit of administrator actions on devices (R-004).

| Column | Type | Notes |
|--------|------|-------|
| `id` | UNIQUEIDENTIFIER PK | |
| `device_id` | FK → `devices.devices` | |
| `action` | NVARCHAR(30) | `REGISTERED` \| `CREDENTIALS_REGENERATED` \| `DECOMMISSIONED` |
| `admin_user_id` | UNIQUEIDENTIFIER | Administrator who acted |
| `admin_name` | NVARCHAR(150) NULL | Name stored with the row |
| `occurred_at` | DATETIME2 | |

## Releases *(added 2026-10-28)*

| Release | Content |
|---------|---------|
| v1.0 baseline | `devices`, `device_link_history` |
| v1.1 | `device_admin_log`; grants for factory registration |
| v1.2 | Default `inactivity_threshold_min` = 1; existing devices patched to 1 |
