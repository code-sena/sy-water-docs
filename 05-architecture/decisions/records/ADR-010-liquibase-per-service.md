# ADR-010 — Database Changes with Liquibase, One Repository per Service

| Field | Value |
|-------|-------|
| **ID** | ADR-010 |
| **Date** | 2026-09-10 (decision, approximate) · 2026-10-28 (recorded) |
| **Status** | Accepted |
| **Authors** | Juan Esteban Ome, Juan Manuel Gómez, Diego Alejandro Alvarado |
| **Reviewers** | Juan Esteban Ome, Juan Manuel Gómez, Diego Alejandro Alvarado |

> Recorded retroactively at project close. The decision was taken and implemented during the project; this record
> describes it as built in the `main` branches on 2026-10-28. Where the alternatives were not written down at the
> time, this is stated instead of reconstructing them.

---

## Context

ADR-002 set one SQL Server database (`sy-water-db`) with one schema per service. Each schema needs a versioned,
repeatable change history that can be applied to local Docker, and to the VPS without the services running
migrations on start-up.

## Evaluated alternatives

| Alternative | Pros | Cons | Reason for discarding |
|------------|------|------|-----------------------|
| **A — Liquibase, one repository per service DB (chosen)** | Same tool for Java and .NET services; formal-SQL changesets with rollback; independent release per schema | Six extra repositories | — (chosen) |
| B — EF Core migrations / Flyway inside each service | Code and schema together | Two tools (Java/.NET); the service would need DDL rights at runtime | One tool and least privilege |
| C — One repository for the whole DB | One history | Releases of one service block the others | Ownership per service (ADR-002) |

## Decision

**We decided:** repositories `ms-iam-db`, `ms-places-db`, `ms-device-db`, `ms-consumption-db`, `ms-valve-db`,
`ms-notification-db`. Each has a baseline (`01-ddl` … `04-tcl`, `05-rollbacks`) and `06-releases/vX.Y/` with formatted-SQL changesets and a rollback per changeset.
Rules:

1. Forward-only in production; a released changeset is never edited.
2. Every release is backward-compatible with the version of the service running at that moment (add, then use).
3. Releases are applied by hand with Liquibase **before** deploying the service that needs them, after a database
   backup (`10-devops/ci-cd.md`).

## Consequences

- Positive: schema history is explicit; see `06-data/migrations-history.md` for every release applied up to the close.
- Negative: applying migrations is a manual step outside Jenkins; nothing blocks a deploy if the release was forgotten
  (the service fails its health check and the deploy rolls back).
- Not taken: automatic migration on deploy (Jenkins stage) — not built for lack of time; recorded in
  `15-project-control/closure/out-of-scope-register.md` (E-15).

## References

- ADR-002, ADR-003, ADR-019
