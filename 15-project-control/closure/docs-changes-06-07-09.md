# Documentation Changes at Close — Sections 06, 07 and 09

> What each section said before the closure update, what it says now, and why (**2026-10-28**).
> Base: `sy-water-docs` local branch `docs/ci-cd-release-process` (`3065672`). Only documentation changed.

---

## 06 — Data

### State before

- `README.md`: schema table from the design phase — `iam`, `place`, `device`, `consumption`, `notification` with
  tables such as `verification_tokens`, `sessions`, `members`, `bills`, `tariffs`, `shutoff_rules`, `alerts`. None of
  those names exists in the database.
- `models.md`: target model v5 plus an "As-built status (2026-10-10)" note: 6 schemas, 31 tables, `place_members` and
  `invitations` not created, no `goals`.
- No history of Liquibase releases.

### State now

| File | Change | Reason |
|------|--------|--------|
| `README.md` | Schema summary replaced by the as-built set: **7 schemas, 38 tables**, owner and last release per schema; list of schemas never created; document index and correlations updated | The table did not describe the database |
| `models.md` | Header: design source vs authoritative source (the six Liquibase repositories). As-built section rewritten for 2026-10-28 (per-schema table list and differences from the target model, ADR-003 deviations). Note on top of the ownership matrix with the real owners. The target model v5 below is unchanged and kept as design reference | 7 releases since 2026-10-10 (iam v1.5, places v1.4, devices v1.2, consumption v1.1, notification v1.5 – v1.7) |
| `migrations-history.md` | **New.** Every baseline and release of the six repositories with date and content (6 baselines, 19 releases) | No document listed what was applied |

### What changed in the database since the previous as-built note (2026-10-10)

| Schema | Added |
|--------|-------|
| `security` | v1.4 official admins seed; v1.5 phone verification (`PHONE_VERIFY`, `target_phone`, `channel`, `users.phone_channel`) |
| `places` | v1.4 `place_members`, `invitations`, `guest_links` |
| `devices` | v1.1 `device_admin_log`; v1.2 threshold 1 minute |
| `goals` | **New schema**, v1.1 `consumption_goals` (repository `ms-consumption-db`) |
| `notification` | v1.5 `push_tokens`; v1.6 `sms_outbox`; v1.7 `message_templates` and invitation notification types |

Related ADRs: ADR-019 (schema set), ADR-010 (Liquibase), ADR-003 (PK, deviations recorded).

---

## 07 — API

### State before

- `README.md`: template guidance only (contract-first, YAML per service). It referenced `guidelines.md` and
  `authentication.md`, which do not exist.
- `contracts/openapi/`: `auth-service.yaml`, `api-gateway.yaml`, `_shared.yaml`, `_template-service.yaml`, all from the
  framework scaffold of 2026-08-25, never updated. Service documents referenced `ms-iam.yaml` and `ms-places.yaml`,
  which were never created.

### State now

| File | Change | Reason |
|------|--------|--------|
| `README.md` | New "As-built status at project close" section: the source of truth is the runtime OpenAPI of each service (`/v3/api-docs`, `/openapi/v1.json`) aggregated at `https://api.sy-water.tech/docs`; state of every file in the folder (historical / template / missing); conventions actually applied (no URL version, JWT RS256, internal key, RFC 7807 errors, 404 for foreign resources, 202 + polling). Template guidance kept below | The folder described a process that was not followed |
| `endpoint-catalog.md` | **New.** Every endpoint of `main` per service (method, path, access, purpose/HU), gateway routes and rate limiters, MQTT topics. Taken from the controller route attributes | No inventory existed; the static contracts were wrong |
| `contracts/openapi/*.yaml` | **Not modified**; marked historical in the README | Kept as history; rewriting them would produce a second copy that drifts (E-23) |

Related ADRs: ADR-006 (internal HTTP), ADR-008 (gateway and security).

---

## 09 — Microservices

### State before

- `README.md`: six services, and "the `api-gateway` placeholder was never built".
- `service-catalog.md`: aligned 2026-10-10; map with a `sy-water-admin` Vite panel; admin device API "not in the
  code (R-004)"; ms-places "Events: none yet"; no sharing, goals, Google sign-in, SMS/WhatsApp, guest routes or message
  templates; communication matrix without ms-places ↔ ms-iam / ms-notifications calls.
- Service READMEs: stale "out of scope" bullets — Google postponed (iam), members/invitations planned (places), admin
  device API missing (devices), goals planned (consumption), no SMS provider (notifications); offline 10 minutes and
  sweep 60 s (devices, notifications).
- `data-model.md` files: release lists stopped at iam v1.3, places v1.1, notifications v1.4; devices threshold "(10)";
  no tables for sharing, goals, device admin log, SMS outbox, templates.

### State now

| File | Change |
|------|--------|
| `README.md` | Service table with row 00 (gateway), repositories and stacks; clients and firmware listed; correction note about the gateway; closure links; note that the cross-cutting documents described in the template were never created |
| `service-catalog.md` | Date 2026-10-28 with the list of changes since 2026-10-10; new map (Caddy → gateway, no `sy-water-admin`); registry rows with real repositories and responsibilities; clients table (frontNEW, firmware, legacy app; `sy-water-admin` marked not delivered); detail per service with sharing, goals, HU-004, HU-028, templates, admin device API; communication matrix with guest access and the new internal calls; known cycles; data ownership with collaborators, guest links, goals, outboxes, templates; "how to add a service" marked as reference only |
| `services/00-api-gateway/README.md` | `/*/openapi/**` is no longer blocked (corrected); as-built section: guest routes, the three rate limiters, Google routes, open risk R-005 |
| `services/01-ms-iam/README.md` | Repository name; Google no longer "postponed"; as-built section (HU-004, HU-028, invitation e-mail button only in `main`, configuration, missing README in the code repository); API reference points to the endpoint catalog |
| `services/02-ms-places/README.md` | Members/invitations no longer "planned"; bills marked will-not-do; as-built section (guest links, invitations, collaborators and permissions, recipients, access authority) |
| `services/03-ms-devices/README.md` | HU-013 values marked superseded; admin API no longer "not present"; admin endpoints added to the table; as-built section (1 minute / 15 s, admin log, access check, telemetry guards, open risks) |
| `services/04-ms-consumption/README.md` | Goals no longer "planned"; goal and guest endpoints added; as-built section |
| `services/05-ms-valve/README.md` | Automatic close marked will-not-do; as-built section (`CONTROL_VALVE`, safety rules, history filter UI missing) |
| `services/06-ms-notifications/README.md` | Offline 1 minute and worker 15 s; SMS/WhatsApp no longer "not integrated"; as-built section (outbox, providers, templates, recipients, invitation notices, WhatsApp delivery confirmed in production) |
| `services/01-ms-iam/data-model.md` | Releases v1.4, v1.5; `action_codes` with `PHONE_VERIFY` |
| `services/02-ms-places/data-model.md` | Releases v1.2 – v1.4; "Planned" replaced by the three sharing tables |
| `services/03-ms-devices/data-model.md` | Threshold default 1; `device_admin_log`; release list |
| `services/04-ms-consumption/data-model.md` | Schema `goals` and `consumption_goals` |
| `services/06-ms-notifications/data-model.md` | Up to v1.7; new notification types; `sms_outbox`, `message_templates`; release list; PK deviations |

Not changed in 09: `services/05-ms-valve/data-model.md` (still accurate), `events.md`, `decisions.md`, `runbook.md`
of every service (not reviewed line by line, see `repository-documentation-review.md` §3).

---

## Other sections touched at close

| Section | Change |
|---------|--------|
| `05-architecture/decisions/` | ADR-005 … ADR-021 and register rows |
| `15-project-control/` | `closure/` folder and link in the README |
