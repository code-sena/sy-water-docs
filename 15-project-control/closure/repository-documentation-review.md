# Repository Documentation Review at Project Close

> Review of whether the documentation reflects the real state of each repository (**2026-10-28**).
> Reference for "real state": the `main` branch of each repository (`branch-status.md`).
> Code repositories are frozen (ADR-021): differences found **in them** are recorded here and **not fixed**.
> Differences found **in this documentation repository** (`sy-water-docs`) were fixed at close where they belong to
> sections 05, 06, 07, 09 and 15; the rest are listed in §3.

---

## 1. Code repositories (`cuida-tu-agua` organization)

| Repository | README | Last README change vs last code change | Matches the code? | Difference found |
|------------|--------|-----------------------------------------|-------------------|------------------|
| `api-gateway` | Yes | 2026-10-24 / 2026-10-24 | ✅ Yes | — |
| `iam-backend` | **No** | — | ❌ Missing | No README in the repository. The service is documented only here (`09-microservices/services/01-ms-iam/`) |
| `ms-places` | Yes | 2026-10-26 / 2026-10-26 | ✅ Yes | — |
| `ms-devices` | Yes | 2026-10-09 / 2026-10-24 | ⚠️ Partial | Describes the admin API (v1.1); predates the 1-minute offline threshold (2026-10-23) and collaborator access (2026-10-24) |
| `ms-consumption` | Yes | 2026-10-23 / 2026-10-24 | ⚠️ Partial | Predates guest consumption (2026-10-24) |
| `ms-valve` | Yes | 2026-10-04 / 2026-10-24 | ⚠️ Partial | Predates collaborator control (`CONTROL_VALVE`, 2026-10-24) |
| `ms-notifications` | Yes | 2026-10-26 / 2026-10-26 | ✅ Yes | — |
| `ms-iam-db` | Yes | 2026-10-23 / 2026-10-23 | ✅ Yes | — |
| `ms-places-db` | Yes | 2026-09-26 / 2026-10-24 | ❌ No | Lists the baseline only; releases v1.1 – v1.4 are not described |
| `ms-device-db` | Yes | 2026-09-29 / 2026-10-23 | ❌ No | Lists the baseline only; v1.1 and v1.2 are not described. Refers to a guide outside the repository |
| `ms-consumption-db` | Yes | 2026-10-23 / 2026-10-23 | ✅ Yes | — |
| `ms-valve-db` | Yes | 2026-10-03 / 2026-10-22 | ✅ Yes | No releases after the baseline, so still accurate |
| `ms-notification-db` | Yes | 2026-10-26 / 2026-10-26 | ✅ Yes | — |
| `infra` | Yes | 2026-10-26 / 2026-10-26 | ✅ Yes | `dev` lacks the `IAM_MAIL_APP_URL` hotfix (branch §6), not a README issue |
| `cuida-tu-agua-frontNEW` | Yes | 2026-10-19 / 2026-10-27 | ❌ No | Says the app talks to "the five services (ms-iam, ms-places, ms-device, ms-consumption, ms-valve)" on ports 3001 – 3005: ms-notifications (3006) and the gateway are missing |
| `cuida-tu-agua-device-ESP` | **No** | — | ❌ Missing | No README; `config.py` comment still says "Desconectado after 10 min" (server uses 1 minute, ADR-015) |
| `cuida-tu-agua-frontend` (legacy) | — | — | n/a | Superseded; not reviewed in detail |

These items are recorded in `out-of-scope-register.md` E-22: they will not be corrected in the code repositories.
**This repository is the source of truth** for the final state.

---

## 2. This repository (`sy-water-docs`) — sections updated at close

| Section | Finding before the update | Action |
|---------|---------------------------|--------|
| `05-architecture/decisions/` | Only ADR-001 … ADR-004. ADR-007 and ADR-008 referenced but missing (R-010). Many decisions only in service `decisions.md` files or not written | ADR-005 … ADR-021 added; register updated |
| `06-data/README.md` | Schema summary with design-time names (`iam`, `place`, `device`, `consumption`, `notification`) and tables that do not exist | Replaced by the as-built summary (7 schemas, 38 tables) |
| `06-data/models.md` | As-built note of 2026-10-10: 6 schemas / 31 tables, `place_members` and `invitations` "not created yet" | As-built section updated to 2026-10-28 |
| `06-data/` | No release history | `migrations-history.md` added |
| `07-api/README.md` | Template only; `auth-service.yaml` and `api-gateway.yaml` are scaffold files of 2026-08-25 that do not describe the services; `ms-iam.yaml` / `ms-places.yaml` referenced but missing; `guidelines.md`, `authentication.md` missing | As-built section: runtime OpenAPI is the source; files marked historical |
| `07-api/` | No endpoint inventory | `endpoint-catalog.md` added (from the controllers of `main`) |
| `09-microservices/README.md` | Said the api-gateway "was never built"; no row for it | Corrected; row 00 and closure links added |
| `09-microservices/service-catalog.md` | Aligned 2026-10-10: `sy-water-admin` panel, admin device API "not in the code", ms-places "events: none yet", no sharing, goals, HU-028, guest routes | Aligned to 2026-10-28 |
| `09-microservices/services/*/README.md` | Stale "out of scope" bullets (Google, members/invitations, admin devices API, goals, SMS/WhatsApp); 10-minute offline values | Bullets corrected; "As-built at project close" section added to each |
| `09-microservices/services/*/data-model.md` | Release lists stopped at iam v1.3, places v1.1, notifications v1.4; devices threshold 10; no sharing/goals/SMS/templates tables | Updated |
| `15-project-control/` | No closure record | `closure/` added; README links it |

The detailed list of edits to 06, 07 and 09 is in [`docs-changes-06-07-09.md`](./docs-changes-06-07-09.md).

---

## 3. This repository — differences found and **not** updated at close

Outside the focus of the closure update (sections 06, 07, 09). Recorded so the reader knows they are stale.

| Document | Difference with the code |
|----------|--------------------------|
| `04-requirements/user-stories.md` | "Scope of the current release (v2.1 — 2026-10-15)": says 40 stories implemented and lists as *Version 2* stories that were implemented afterwards — HU-004 (Google), HU-023/024 (goals), HU-028 (SMS/WhatsApp), HU-039 – HU-047 (guest and collaborator). HU-013 still says 10 minutes (ADR-015). HU-002/HU-005 say "enlace" (ADR-020). Real status per story: `closure/README.md` §3 |
| `15-project-control/risks.md` | Statuses as of 2026-10-10. Status at close is in `closure/README.md` §4 |
| `08-uml/diagrams/` (ER diagrams) | **Not verified** at close; they were drawn before `places` v1.4, `notification` v1.6/v1.7, `devices` v1.1 and the `goals` schema, so they very likely lack those tables |
| `09-microservices/services/*/events.md`, `runbook.md`, `decisions.md` | **Not reviewed** line by line; the decisions they contain are referenced from the new ADRs |
| `02-domain/`, `05-architecture/overview.md`, `10-devops/` | **Not reviewed** at close |

---

## 4. What could not be verified

| Item | Reason |
|------|--------|
| Remote branches of `code-sena/sy-water-docs` | Private repository; no credentials in the review environment. Only the local clone (fetched 2026-10-22) was read |
| Image tag running in production per service | Not recorded in any repository; only in Jenkins history and on the VPS (`docker ps`). Not accessed |
| Liquibase releases applied in production | Only visible in the `DATABASECHANGELOG` tables of the production database. Not accessed |
