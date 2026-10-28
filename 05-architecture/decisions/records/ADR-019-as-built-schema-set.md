# ADR-019 — As-Built Database Schema Set

| Field | Value |
|-------|-------|
| **ID** | ADR-019 |
| **Date** | 2026-10-28 |
| **Status** | Accepted (supersedes the schema list of ADR-002; ADR-002 rules stay in force) |
| **Authors** | Juan Esteban Ome, Juan Manuel Gómez, Diego Alejandro Alvarado |
| **Reviewers** | Juan Esteban Ome, Juan Manuel Gómez, Diego Alejandro Alvarado |

> Recorded retroactively at project close. The decision was taken and implemented during the project; this record
> describes it as built in the `main` branches on 2026-10-28. Where the alternatives were not written down at the
> time, this is stated instead of reconstructing them.

---

## Context

ADR-002 and `06-data/README.md` listed the schemas planned at design time (including `alerts`, `billing` and
`content`, and names such as `iam`, `place`, `device`). The database deployed at close is different. ADR-003 requires a
single `UNIQUEIDENTIFIER` PK, and a few tables do not follow it.

## Decision

**We record:** the database `sy-water-db` has **7 schemas and 38 tables**:

| Schema | Owner | Tables |
|--------|-------|--------|
| `security` | ms-iam | 10 |
| `places` | ms-places | 12 (includes tariffs and tips) |
| `devices` | ms-devices | 3 |
| `monitoring` | ms-consumption | 2 |
| `goals` | ms-consumption (repository `ms-consumption-db`) | 1 |
| `valve` | ms-valve | 2 |
| `notification` | ms-notifications | 8 |

- `alerts`, `billing` and `content` were **never created**: tariffs and tips live in `places` (`02-ms-places/decisions.md` DEC-009);
  alerts are `notification.notifications`.
- **Accepted deviations from ADR-003:** `places.activity_log` and `places.place_tariffs` use a `BIGINT IDENTITY` PK
  (internal, never exposed); `places.tip_favorites`, `notification.notification_reads` and
  `notification.notification_preferences` use composite PKs; `notification.place_devices` uses `place_id` as PK; `notification.message_templates` uses its natural key
  `template_key` as PK.
  They are internal or association tables with no own identity in the API.

## Evaluated alternatives

| Alternative | Reason for discarding |
|------------|-----------------------|
| Rewrite the deviating tables to `id UNIQUEIDENTIFIER` | Code is frozen; the tables are internal and the deviation has no functional effect |
| Create empty `alerts` / `billing` / `content` schemas to match the design | No owner and no use |

## Consequences

- `06-data/README.md` and `06-data/models.md` are updated to this set; `06-data/migrations-history.md` lists the
  releases.

## References

- ADR-002, ADR-003, ADR-010
