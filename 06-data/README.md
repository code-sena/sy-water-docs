# 06 — Data

> **What is this?** How the system stores, structures, and migrates data.

## Fundamental principle

> **Schema-per-service, shared database.**

All services use a single SQL Server instance (`sy_water_db`). Each service owns its schema and must NOT query another service's schema directly. Inter-service data access is via REST API or RabbitMQ events. See [ADR-002](../05-architecture/decisions/records/ADR-002-shared-database-schema-per-service.md).

---

## Documents in this section

| File | Purpose | Status |
|------|---------|--------|
| [models.md](./models.md) | DDL for all schemas and tables (T-SQL), ER diagrams, data ownership matrix, migration strategy | Complete |

---

## Schemas summary

| Schema | Owner service | Tables | Entrega |
|--------|-------------|--------|---------|
| `iam` | auth-service | users, verification_tokens, sessions, login_attempts | MVP |
| `place` | place-service | places, members, invitations | MVP (members/invitations: Entrega 3) |
| `device` | device-service | devices, readings | MVP |
| `consumption` | consumption-service | consumption_records, goals, bills, tariffs | MVP (goals: E2, bills/tariffs: E3) |
| `valve` | valve-service | valves, valve_actions, shutoff_rules | Entrega 2 (shutoff_rules: E3) |
| `notification` | notification-service | alerts, notification_preferences | Entrega 2 |

---

## Correlations

| This section is fed by... | And feeds into... |
|---------------------------|-------------------|
| `02-domain/entities-and-rules.md` → domain entities | DB tables in `models.md` |
| `05-architecture/overview.md` → DB strategy | Schema structure |
| `models.md` | `07-api/contracts/` → API field names and types |
| `models.md` | `09-microservices/[service]/data-model.md` → per-service detail |
