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
| ADR-002 | [Shared Database with Schema-per-Service](records/ADR-002-shared-database-schema-per-service.md) | Accepted | 2026-09-08 |

> Add rows here as you create ADRs in `records/`
