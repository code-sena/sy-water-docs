# Project Closure — Cuida Tu Agua

> Closure record of the project (**2026-10-28**). The code is frozen and will not be modified (ADR-021).
> This folder documents the final state; it does not plan work.

---

## 1. Documents

| Document | Answers |
|----------|---------|
| [branch-status.md](./branch-status.md) | What exists in every branch of every repository; what is integrated, abandoned, or missing a back-merge |
| [repository-documentation-review.md](./repository-documentation-review.md) | Does each repository's documentation match its code? What could not be verified? |
| [docs-changes-06-07-09.md](./docs-changes-06-07-09.md) | What changed in sections 06 (data), 07 (API) and 09 (microservices) at close |
| [out-of-scope-register.md](./out-of-scope-register.md) | What will not be done, and why (E-01 … E-24) |
| [`05-architecture/decisions/`](../../05-architecture/decisions/README.md) | Architecture decisions ADR-001 … ADR-021 (ADR-005 … ADR-021 recorded at close) |

---

## 2. Final state

| Area | Final state |
|------|-------------|
| Source of the final state | `main` branch of each repository of the `cuida-tu-agua` organization |
| Services | api-gateway, ms-iam (Java 21 / Spring Boot 4); ms-places, ms-devices, ms-consumption, ms-valve, ms-notifications (.NET 10) |
| Clients | App web + Android (`cuida-tu-agua-frontNEW`, Expo SDK 57) with the admin area on web only; firmware ESP32 (`cuida-tu-agua-device-ESP`, branch `dev`) |
| Database | SQL Server 2022, `sy-water-db`: 7 schemas, 38 tables, 6 Liquibase repositories (`06-data/migrations-history.md`) |
| Integration | RabbitMQ `sywater.events` (device events), internal HTTP with `X-Internal-Key`, MQTT (Mosquitto, 1883) |
| Hosting | One Hostinger VPS, Docker Compose, Caddy; `sy-water.tech`, `api.sy-water.tech` (ADR-013) |
| Release | PR checks in GitHub Actions; Jenkins release with approval, GHCR images by commit SHA, automatic rollback (ADR-012) |
| Pending integration into `main` | **None** |

---

## 3. User stories at close

Backlog: `04-requirements/user-stories.md` (69 stories). That file's own status table dates from 2026-10-15 and is
outdated; this table is the status at close.

| Status | Count | Stories |
|--------|-------|---------|
| Implemented | 49 | HU-001 – HU-016, HU-018 – HU-021, HU-023 – HU-025, HU-027, HU-032 – HU-034, HU-039 – HU-047, HU-053 – HU-056, HU-059 – HU-066, HU-069 |
| Partial | 4 | HU-017 (no exact-value tooltip, E-07) · HU-022 (no date filter in the UI, E-08) · HU-026 (Android only, E-10) · HU-028 (WhatsApp works in production; SMS disabled for cost, E-09) |
| Not implemented | 14 | HU-029 (E-03) · HU-030, HU-031 (E-01) · HU-035 – HU-037 (E-04) · HU-038, HU-057, HU-058 (E-05) · HU-048 – HU-052 (E-02) |
| Won't | 2 | HU-067, HU-068 (E-06, ADR-004) |

Accepted deviations from the written stories: HU-002 and HU-005 use 6-digit codes instead of links (ADR-020); HU-013
uses a 1-minute threshold instead of 10 (ADR-015); HU-023 creates manual goals only (suggested goals not built).

---

## 4. Risks at close

Register: `15-project-control/risks.md` (statuses there are from 2026-10-10).

| ID | Risk | Status at close |
|----|------|-----------------|
| R-001 | No container images | **Resolved** — images in GHCR built by Jenkins (ADR-012) |
| R-002 | Frontend/backend contracts out of sync | **Resolved** — frontend integrated against the services; contracts are the runtime OpenAPI (07-api) |
| R-003 | Must stories still open | **Mostly resolved** — HU-004, HU-041, HU-045 implemented; HU-029 not implemented (E-03) |
| R-004 | Device administration API missing | **Resolved** — `/api/admin/devices` (ms-device-db v1.1) |
| R-005 | Revoked tokens accepted by ms-devices | **Open, mitigated** at the gateway (E-19) |
| R-006 | MQTT without TLS | **Open, accepted** (ADR-009, E-13) |
| R-007 | Configuration drift between services | **Not reassessed** at close. Single `infra/compose.yml` and `.env` for production; the missing `dev` back-merge in `infra` is a known drift (`branch-status.md` §6) |
| R-008 | No backups and no centralized observability | **Partial** — database backup before each release; no observability (E-16) |
| R-009 | Gaps in automated testing | **Not reassessed** at close. Every service repository, database repository and the app run their tests in CI on each PR (check `test`); coverage was not measured. The firmware has no CI |
| R-010 | Documentation drift | **Reduced** — ADR-007/ADR-008 now exist; 06, 07, 09 aligned. Code-repository READMEs and other sections remain stale (`repository-documentation-review.md`) |
| R-011 | Places survive a deleted account | **Open** (E-18) |

---

## 5. Limits of this record

- The remote of this documentation repository (`code-sena/sy-water-docs`) was not accessible; its branches were read
  from the local clone only.
- Which image tag runs in production per service and which Liquibase releases are applied in production were not
  verified (Jenkins history and the production `DATABASECHANGELOG` tables are the authority).
- Everything else was read from the `main` (and other) branches of the repositories on 2026-10-28.
