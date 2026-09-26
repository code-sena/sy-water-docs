# ADR-002 — Shared Database with Schema-per-Service

| Field | Value |
|-------|-------|
| **ID** | ADR-002 |
| **Date** | 2026-09-08 |
| **Status** | Accepted — rule 6 superseded by [ADR-003](ADR-003-single-uuid-primary-key.md) |
| **Authors** | Development team |
| **Reviewers** | Juan Esteban Ome, Juan Manuel Gómez, Diego Alejandro Alvarado |

---

## Context

The standard microservices recommendation is "database per service" — each service owns a separate database instance. This maximizes independence but increases operational overhead: multiple database servers to manage, backup, monitor, and pay for.

Save Your Water is built by a team of 3 developers, deployed on a single VPS with Docker. The project has 11 bounded contexts deployed across 6 microservices (see `05-architecture/overview.md` § 4 for the mapping), and maintaining 6+ SQL Server instances (or even containers) is impractical given the team size and infrastructure budget (university project, no cloud budget).

At the same time, we need to preserve the logical independence of each service's data to avoid tight coupling. Schemas are separated for logical organization, NOT as a preparation for future migration to separate databases.

---

## Evaluated alternatives

### Alternative A — Database per service (separate instances)
- **Pros:** Maximum independence; each service can use a different DB engine; no query contention; standard microservices practice
- **Cons:** 6+ SQL Server instances to manage; significant resource overhead on a single VPS; licensing cost; operational burden for a team of 3

### Alternative B — Shared database with schema-per-service (CHOSEN)
- **Pros:** Single instance to manage; lower resource usage; schema isolation preserves logical boundaries; easy to evolve to separate instances later by extracting schemas
- **Cons:** Services share compute resources (CPU, RAM, I/O); a misbehaving query in one schema can affect others; requires discipline to not cross-query schemas

### Alternative C — Single database, no schema separation
- **Pros:** Simplest to set up
- **Cons:** No logical boundaries; services can JOIN across each other's tables; extremely hard to split later; violates service independence

---

## Decision

**Alternative B:** Use a single SQL Server instance (`sy_water_db`) with one schema per service.

### Schemas (10 schemas for 11 BCs)

| Schema | Owner service | BCs |
|--------|-------------|-----|
| `security` | auth-service | BC-01 IAM |
| `places` | place-service | BC-02 Place Management |
| `devices` | device-service | BC-03 Device Management |
| `monitoring` | consumption-service | BC-04 Consumption Monitoring |
| `goals` | consumption-service | BC-05 Consumption Goals |
| `billing` | consumption-service | BC-06 Billing & Tariffs |
| `valve` | valve-service | BC-07 Valve Control |
| `alerts` | notification-service | BC-08 Alerts & Detection |
| `notifications` | notification-service | BC-09 Notification Delivery |
| `content` | notification-service | BC-11 Savings Tips |

BC-10 Reports & Export has no tables — it reads from `monitoring`.

### Rules

1. A service may only read/write tables in its own schemas
2. Cross-service data access at the application layer is exclusively via REST API or RabbitMQ events
3. Cross-schema foreign keys ARE used — SQL Server enforces referential integrity across schemas
4. Application-layer cross-schema JOINs are NOT allowed — use the owning service's API
5. Each service manages its own migrations, scoped to its schemas
6. ~~All tables use `BIGINT IDENTITY` as internal PK + separate `UNIQUEIDENTIFIER` column for public API exposure~~ — **Superseded by [ADR-003](ADR-003-single-uuid-primary-key.md) (2026-09-26):** every table uses a single `id UNIQUEIDENTIFIER` as PK and public ID
7. Database users/roles per service (with permissions limited to their schemas) are recommended but optional for the MVP

---

## Consequences

**Positive:**
- Single instance reduces operational overhead (backup, monitoring, deployment)
- Schema names make ownership explicit in every SQL query
- Cross-schema FKs enforce referential integrity at the database level
- 10 schemas provide fine-grained logical separation per BC

**Negative:**
- Shared compute: a heavy query in `monitoring.readings` could slow down `security.users`
- Cross-schema FKs create physical coupling — removing a schema requires careful FK cleanup
- Requires team discipline: no application-level cross-schema JOINs
- Single point of failure for the database

**Mitigation:**
- Monitor query performance per schema
- Code reviews explicitly check for cross-schema access at the application layer
- If contention becomes a problem, the first candidate to extract is `monitoring` (highest write volume due to telemetry readings)

---

## References

- Architecture overview → `05-architecture/overview.md`
- Domain bounded contexts → `02-domain/domain-map.md`
- Data models → `06-data/models.md`
