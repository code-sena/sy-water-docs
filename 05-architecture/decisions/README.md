# ADRs — Architecture Decision Records

ADRs document important architectural decisions. Each file = one decision.

## How to create an ADR

1. Copy `_template-adr.md`
2. Name it `ADR-NNN-short-title.md` (e.g.: `ADR-001-message-broker.md`)
3. Fill it in completely — especially the evaluated alternatives
4. Once accepted, the status is **permanent** (it is not deleted, it is "Superseded" by another ADR)

## Possible statuses

- `Proposed` — under discussion
- `Accepted` — approved by the team
- `Rejected` — evaluated and discarded (document why)
- `Superseded` — superseded by ADR-NNN (indicate which one)

## ADR register

| # | Title | Status | Date |
|---|-------|--------|------|
| ADR-001 | [Documentation Language](records/ADR-001-idioma-documentacion.md) | Accepted | 2026-09-08 |
| ADR-002 | [Shared Database with Schema-per-Service](records/ADR-002-shared-database-schema-per-service.md) | Accepted (rule 6 superseded by ADR-003; schema list superseded by ADR-019) | 2026-09-08 |
| ADR-003 | [Single UUID Primary Key](records/ADR-003-single-uuid-primary-key.md) | Accepted | 2026-09-26 |
| ADR-004 | [Market Scope: Colombia and Ecuador Only](records/ADR-004-market-scope-co-ec.md) | Accepted | 2026-09-26 |
| ADR-005 | [Hexagonal Architecture in Every Service and the Frontend](records/ADR-005-hexagonal-architecture.md) | Accepted (recorded at close) | 2026-09-15 |
| ADR-006 | [Service Integration: RabbitMQ Events plus Internal HTTP](records/ADR-006-events-and-internal-http.md) | Accepted (recorded at close) | 2026-09-20 |
| ADR-007 | [Polyglot Backend: Java for IAM and Gateway, .NET 10 for Domain Services](records/ADR-007-polyglot-backend.md) | Accepted (recorded at close) | 2026-09-12 |
| ADR-008 | [Single API Gateway with Defense-in-Depth JWT Validation](records/ADR-008-api-gateway-defense-in-depth.md) | Accepted (recorded at close) | 2026-10-14 |
| ADR-009 | [MQTT with Mosquitto for Devices](records/ADR-009-mqtt-mosquitto-devices.md) | Accepted (recorded at close) | 2026-09-22 |
| ADR-010 | [Database Changes with Liquibase, One Repository per Service](records/ADR-010-liquibase-per-service.md) | Accepted (recorded at close) | 2026-09-10 |
| ADR-011 | [Single Expo Codebase for Web and Android, Admin Integrated and Web-Only](records/ADR-011-single-expo-codebase.md) | Accepted (recorded at close) | 2026-10-09 / 2026-10-27 |
| ADR-012 | [Branching, Continuous Integration and Release Process](records/ADR-012-branching-ci-cd-release.md) | Accepted (recorded at close) | 2026-10-22 |
| ADR-013 | [Hosting on One VPS with Docker Compose and Caddy](records/ADR-013-single-vps-hosting.md) | Accepted (recorded at close) | 2026-10-01 |
| ADR-014 | [Place Sharing: Guest Links and Collaborators with Permissions](records/ADR-014-place-sharing-model.md) | Accepted (recorded at close) | 2026-10-22 |
| ADR-015 | [Device Liveness: Offline after One Minute](records/ADR-015-device-offline-one-minute.md) | Accepted (recorded at close; supersedes the 10-minute default of HU-013) | 2026-10-23 |
| ADR-016 | [Notification Channels and Providers](records/ADR-016-notification-channels.md) | Accepted (recorded at close) | 2026-10-10 / 2026-10-26 |
| ADR-017 | [Remote Valve Safety Rules](records/ADR-017-remote-valve-safety.md) | Accepted (recorded at close) | 2026-10-05 |
| ADR-018 | [Consumption Storage: Raw Readings plus Hourly Aggregate](records/ADR-018-consumption-hourly-aggregate.md) | Accepted (recorded at close) | 2026-09-28 |
| ADR-019 | [As-Built Database Schema Set](records/ADR-019-as-built-schema-set.md) | Accepted (supersedes the schema list of ADR-002) | 2026-10-28 |
| ADR-020 | [One-Time Six-Digit Codes Instead of Links](records/ADR-020-one-time-codes.md) | Accepted (recorded at close) | 2026-09-25 |
| ADR-021 | [Project Closure and Scope Freeze](records/ADR-021-project-closure.md) | Accepted | 2026-10-28 |
| ADR-022 | [Valve Orders for an Offline Device: Rejected, Not Queued (Saga Pattern Discarded)](records/ADR-022-offline-valve-command-saga-not-implemented.md) | Accepted — risk not resolved (recorded at close) | 2026-10-05 |

> ADR-005 … ADR-020 and ADR-022 were **recorded retroactively at project close (2026-10-28)**: the decisions were taken and
> implemented during the project; the date column is approximate (when it was first implemented in the code); the
> exact date of each decision was not recorded at the time. ADR-007 and ADR-008 close the dangling references noted in risk R-010.
> The project is closed (ADR-021); no new ADR is expected.
