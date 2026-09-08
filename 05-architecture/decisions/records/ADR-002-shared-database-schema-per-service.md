# ADR-002 — Shared Database with Schema-per-Service

| Field | Value |
|-------|-------|
| **ID** | ADR-002 |
| **Date** | 2026-09-08 |
| **Status** | Accepted |
| **Authors** | Development team |
| **Reviewers** | Juan Esteban Ome, Juan Manuel Gómez, Diego Alejandro Alvarado |

---

## Context

The standard microservices recommendation is "database per service" — each service owns a separate database instance. This maximizes independence but increases operational overhead: multiple database servers to manage, backup, monitor, and pay for.

Save Your Water is built by a team of 3 developers, deployed on a single VPS with Docker. The project has 6 domain services, and maintaining 6+ SQL Server instances (or even containers) is impractical given the team size and infrastructure budget (university project, no cloud budget).

At the same time, we need to preserve the logical independence of each service's data to avoid tight coupling and to allow a future migration to separate databases if scale demands it.

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

### Schemas

| Schema | Owner service |
|--------|-------------|
| `iam` | auth-service |
| `place` | place-service |
| `device` | device-service |
| `consumption` | consumption-service |
| `valve` | valve-service |
| `notification` | notification-service |

### Rules

1. A service may only read/write tables in its own schema
2. Cross-service data access is exclusively via REST API or RabbitMQ events
3. Foreign keys across schemas are NOT allowed — use application-level UUID references
4. Each service manages its own migrations, scoped to its schema
5. Database users/roles per service (with permissions limited to their schema) are recommended but optional for the MVP

---

## Consequences

**Positive:**
- Single instance reduces operational overhead (backup, monitoring, deployment)
- Schema names make ownership explicit in every SQL query
- Migration to separate databases is straightforward: extract a schema into its own instance and update the connection string

**Negative:**
- Shared compute: a heavy query in `device.readings` could slow down `iam.users`
- Requires team discipline: no cross-schema JOINs, even when the data is "right there"
- Single point of failure for the database

**Mitigation:**
- Monitor query performance per schema
- Code reviews explicitly check for cross-schema access
- If contention becomes a problem, the first candidate to extract is `device` (highest write volume due to telemetry)

---

## References

- Architecture overview → `05-architecture/overview.md`
- Domain bounded contexts → `02-domain/domain-map.md`
- Data models → `06-data/models.md`
