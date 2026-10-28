# ADR-018 — Consumption Storage: Raw Readings plus Hourly Aggregate

| Field | Value |
|-------|-------|
| **ID** | ADR-018 |
| **Date** | 2026-09-28 (decision, approximate) · 2026-10-28 (recorded) |
| **Status** | Accepted |
| **Authors** | Juan Esteban Ome, Juan Manuel Gómez, Diego Alejandro Alvarado |
| **Reviewers** | Juan Esteban Ome, Juan Manuel Gómez, Diego Alejandro Alvarado |

> Recorded retroactively at project close. The decision was taken and implemented during the project; this record
> describes it as built in the `main` branches on 2026-10-28. Where the alternatives were not written down at the
> time, this is stated instead of reconstructing them.

---

## Context

A meter sends a reading every 10 s (8,640 per day). The app shows consumption by day, week and month, in liters or
gallons, in the user's time zone.

## Evaluated alternatives

| Alternative | Pros | Cons | Reason for discarding |
|------------|------|------|-----------------------|
| **A — Store readings + one hourly aggregate; compute day/week/month on request (chosen)** | One aggregate to keep correct; any time zone can be derived | Monthly queries sum ~720 rows | — (chosen) |
| B — Pre-aggregate hourly, daily, weekly, monthly | Fast reads | Daily/monthly buckets depend on the time zone; four aggregates to maintain | Time-zone correctness |
| C — Query raw readings only | No aggregate | Too many rows per chart | Performance |
| D — Time-series database (InfluxDB, TimescaleDB) | Built for this | Another engine (ADR-002 single engine) | ADR-002 |

## Decision

**We decided:** `monitoring.readings` (append-only, `NEWSEQUENTIALID()`) and `monitoring.consumption_hourly`
(UTC hour buckets). The API takes `period`, a reference date and a `tz` parameter and builds the buckets on request.
Volumes are stored in **liters**; the unit (L/gal) is converted in the client.

## Consequences

- Positive: changing the unit or time zone needs no data migration.
- Negative: no retention policy for raw readings (they grow forever; not a problem at the project volume).
- Not taken: CSV/PDF export and consumption reports (HU-038, HU-057, HU-058) — E-05.

## References

- `09-microservices/services/04-ms-consumption/README.md`
- `09-microservices/services/04-ms-consumption/decisions.md` DEC-002, DEC-003, DEC-004
