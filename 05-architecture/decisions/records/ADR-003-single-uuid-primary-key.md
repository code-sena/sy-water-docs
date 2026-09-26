# ADR-003 — Single UUID Primary Key

| Field | Value |
|-------|-------|
| **ID** | ADR-003 |
| **Date** | 2026-09-26 |
| **Status** | Accepted |
| **Supersedes** | ADR-002, rule 6 (only that rule — the rest of ADR-002 stays in force) |
| **Authors** | Juan Esteban Ome |
| **Reviewers** | Juan Esteban Ome, Juan Manuel Gómez, Diego Alejandro Alvarado |

---

## Context

ADR-002 rule 6 required every table to carry two identifiers: a `BIGINT IDENTITY` internal PK used for JOINs and FKs, plus a separate `UNIQUEIDENTIFIER` column (`uuid`) exposed through the REST API and JWT.

Once implementation started in `sy-water-db`, this dual-ID scheme created friction:

- **Cross-service FKs need the internal ID.** `places.places.owner_id` must reference `security.users.id` (BIGINT), but place-service only ever receives the user's public UUID from the JWT. Every write would need a lookup through auth-service's API to translate UUID → BIGINT, which is exactly the cross-service coupling ADR-002 tries to avoid.
- **Two IDs per aggregate in every layer.** Domain entities, DTOs, repositories and events would all have to carry and map `id` + `uuid`, and it is easy to leak the internal BIGINT by mistake.
- **The domain can't assign its own identity.** With `IDENTITY`, the ID only exists after the INSERT, so aggregates cannot be created with a known identity (useful for domain events and idempotent message handling).

`ms-iam-db` release v1.1 already moved `security.users` to a single `id UNIQUEIDENTIFIER`, and `ms-places-db` v1.0-baseline followed. The documentation has to be aligned with what is deployed.

**Known constraints:**
- SQL Server is the only engine (ADR-002).
- Cross-schema FKs must have matching types on both sides: if `security.users.id` is `UNIQUEIDENTIFIER`, every FK pointing to it must be too.
- `monitoring.readings` is the highest write-volume table (telemetry every few seconds per device).

---

## Decision

**We decided:** every table has **one** identifier — `id UNIQUEIDENTIFIER` — which is both the primary key and the ID exposed by the API. All FKs (same-schema and cross-schema) are `UNIQUEIDENTIFIER`.

```sql
id UNIQUEIDENTIFIER NOT NULL CONSTRAINT DF_<table>_id DEFAULT (NEWID()),
-- ...
CONSTRAINT PK_<table> PRIMARY KEY (id)
```

**Rules:**

1. Column name is `id`; there is no `uuid` column and no `BIGINT IDENTITY` column.
2. Default is `NEWID()`. The backend may supply the ID (generated in the domain layer); if it doesn't, SQL Server generates one.
3. High-volume, append-only tables (`monitoring.readings`, `notifications.notification_log`) use `DEFAULT (NEWSEQUENTIALID())` and the application does **not** supply the ID, so inserts append to the end of the clustered index instead of splitting random pages.
4. FK columns keep the `[referenced_table_singular]_id` naming and are `UNIQUEIDENTIFIER`.
5. All constraints are named (`PK_`, `DF_`, `FK_`, `CK_`, `UQ_`, `UX_`, `IX_`), so rollbacks work the same on every machine.

**Justification:** removing the UUID → BIGINT translation removes the need for a synchronous call to the owning service before every cross-schema write. That decoupling, plus a single identity across DB, domain, API and events, is worth more than the storage and index-locality cost for this project's data volume.

---

## Evaluated alternatives

| Alternative | Pros | Cons | Reason for discarding |
|------------|------|------|-----------------------|
| **A — Single `UNIQUEIDENTIFIER` PK (chosen)** | One ID everywhere; domain can assign identity; no UUID→BIGINT lookups across services; IDs are not guessable | 16 bytes vs 8; `NEWID()` fragments the clustered index | — (chosen) |
| B — `BIGINT IDENTITY` PK + public `uuid` (ADR-002 rule 6) | Compact, sequential clustered index; fast JOINs | Cross-service FKs need the internal ID → API lookups; two IDs to map in every layer; risk of exposing internal IDs | Coupling and mapping cost outweigh the performance gain at our scale |
| C — `BIGINT IDENTITY` exposed in the API | Simplest; smallest | Enumerable IDs (IDOR risk: `/places/1`, `/places/2`…); identity only known after INSERT | Security risk, rejected |
| D — ULID / UUIDv7 stored as `BINARY(16)` | Time-ordered and globally unique | No native SQL Server type or generator; needs app-side generation and custom mapping in EF Core and JPA | Extra tooling not justified; `NEWSEQUENTIALID()` covers the hot tables |

---

## Consequences

**Positive:**
- A single identity flows from the DB through the domain, REST API, JWT claims and RabbitMQ events.
- Services can insert cross-schema FKs directly with the UUID they already have (e.g. `owner_id` from the JWT `sub`).
- IDs are not enumerable, which removes a class of IDOR attacks.

**Negative / Trade-offs:**
- PK and FK columns grow from 8 to 16 bytes; every non-clustered index carries the 16-byte clustered key.
- `NEWID()` causes page splits and fragmentation on the clustered index. For low-volume tables (users, places, devices, rules) this is negligible.
- UUIDs are harder to read when debugging by hand.

**Impact on the system:**
- Affected services: all (auth-service, place-service, device-service, consumption-service, valve-service, notification-service).
- Documents that must be updated: `06-data/models.md` (§ PK strategy and every table), `02-domain/entities-and-rules.md` (ID types), ADR-002 (rule 6 marked as superseded).

---

## Risks

| Risk | Probability | Impact | Mitigation |
|------|------------|--------|-----------|
| Fragmentation on `monitoring.readings` degrades insert throughput | Medium | Medium | `NEWSEQUENTIALID()` default (rule 3); partition by `recorded_at` past ~50M rows (see `06-data/models.md`) |
| A service creates an FK column as `BIGINT` by habit and the FK fails to create | Medium | Low | Liquibase precondition checks; code review checklist; SQL Server rejects mismatched FK types at migration time |
| `NEWSEQUENTIALID()` order resets after a server restart | Low | Low | Ordering is only used for index locality, never for business ordering (use `recorded_at` / `created_at`) |

---

## References

- ADR-002 — Shared Database with Schema-per-Service → `ADR-002-shared-database-schema-per-service.md` (rule 6 superseded by this ADR)
- Data models → `06-data/models.md`
- Implementation: `sy-water-db/ms-iam-db` release v1.1 (`security.users.id`), `sy-water-db/ms-places-db` v1.0-baseline
- Related to: ADR-004 (country / subdivision / city catalog tables follow this rule)
