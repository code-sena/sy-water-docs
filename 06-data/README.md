# 06 — Data

> **What is this?** How the system stores, structures, and migrates data.

## Fundamental principle

> **Schema-per-service, shared database.**

All services use a single SQL Server 2022 database (`sy-water-db`). Each service owns its schema and must NOT query another service's schema directly. Inter-service data access is via REST API or RabbitMQ events. See [ADR-002](../05-architecture/decisions/records/ADR-002-shared-database-schema-per-service.md).

---

## Documents in this section

| File | Purpose | Status |
|------|---------|--------|
| [models.md](./models.md) | Target model v5 (DDL, ER diagrams, ownership) plus the **as-built status at project close** | Updated 2026-10-28 |
| [migrations-history.md](./migrations-history.md) | Every Liquibase baseline and release applied, per repository | New 2026-10-28 |

---

## Schemas summary (as built at project close, 2026-10-28)

> The previous table of this section listed the schemas planned at design time (`iam`, `place`, `device`,
> `consumption`, `notification` with tables such as `alerts`, `bills`, `shutoff_rules`). None of those names matched
> the database. Replaced by the real set (ADR-019).

**7 schemas, 38 tables.**

| Schema | Owner service | Liquibase repository (last release) | Tables |
|--------|---------------|-------------------------------------|--------|
| `security` | ms-iam | `ms-iam-db` (v1.5) | users, user_credentials, email_verifications, password_resets, refresh_tokens, login_attempts, activity_log, roles, user_roles, action_codes |
| `places` | ms-places | `ms-places-db` (v1.4) | countries, subdivisions, cities, places, activity_log, tariff_catalog, place_tariffs, tips, tip_favorites, place_members, invitations, guest_links |
| `devices` | ms-devices | `ms-device-db` (v1.2) | devices, device_link_history, device_admin_log |
| `monitoring` | ms-consumption | `ms-consumption-db` (baseline) | readings, consumption_hourly |
| `goals` | ms-consumption | `ms-consumption-db` (v1.1) | consumption_goals |
| `valve` | ms-valve | `ms-valve-db` (baseline) | valve_states, valve_commands |
| `notification` | ms-notifications | `ms-notification-db` (v1.7) | notifications, notification_reads, notification_preferences, place_devices, email_outbox, push_tokens, sms_outbox, message_templates |

**Never created:** `alerts`, `billing`, `content` — tariffs and tips are in `places`; alerts are
`notification.notifications`; bills were not built (`15-project-control/closure/out-of-scope-register.md` E-02).

---

## Correlations

| This section is fed by... | And feeds into... |
|---------------------------|-------------------|
| `02-domain/entities-and-rules.md` → domain entities | DB tables in `models.md` |
| `05-architecture/overview.md` → DB strategy | Schema structure |
| ADR-002, ADR-003, ADR-010, ADR-019 | Rules, PK strategy, migrations, as-built schema set |
| `models.md` | `07-api/endpoint-catalog.md` → API resources |
| `models.md` | `09-microservices/[service]/data-model.md` → per-service detail |
