# Out-of-Scope Register (What Will Not Be Done)

> Historical record, written at project close (**2026-10-28**). Every item below was planned, started, requested or
> identified as a gap, and **will not be implemented**: the code is frozen (ADR-021). Each entry states what it is,
> where it stands in the code, and the concrete reason it is not done.
>
> "Backlog" = `04-requirements/user-stories.md`. "Branch" = `closure/branch-status.md`.

---

## Summary

| ID | Item | Type | Stories / reference |
|----|------|------|---------------------|
| E-01 | Goal alerts (exceeded / preventive) | Functional | HU-030, HU-031 |
| E-02 | Water bills | Functional (epic E11) | HU-048 – HU-052 |
| E-03 | Leak alert | Functional | HU-029 |
| E-04 | Automatic valve shutoff | Functional (epic E8) | HU-035, HU-036, HU-037 |
| E-05 | Reports and export (CSV, PDF) | Functional (epic E13) | HU-038, HU-057, HU-058 |
| E-06 | Pre-loaded tariff catalogs for Ecuador and the United States | Functional | HU-067, HU-068 |
| E-07 | Exact value tooltip in the consumption chart | Functional (partial) | HU-017 |
| E-08 | Date filter in the valve history (UI) | Functional (partial) | HU-022 |
| E-09 | SMS channel in production | Functional (partial) | HU-028 |
| E-10 | iOS app and iOS push | Platform | HU-026 |
| E-11 | Separate admin panel `sy-water-admin` | Architecture | ADR-011 |
| E-12 | ~~Confirmation of WhatsApp code delivery in production~~ — **closed 2026-10-28, works** | Resolved (kept for history) | HU-028, ADR-016 |
| E-13 | TLS on the MQTT broker | Security | R-006, ADR-009 |
| E-14 | Firmware in the release flow | Process | ADR-012 |
| E-15 | Automatic database migrations in the pipeline | Process | ADR-010 |
| E-16 | Centralized monitoring, logs and scheduled off-site backups | Operations | R-008, ADR-013 |
| E-17 | Staging environment | Process | ADR-012, ADR-013 |
| E-18 | Deleting a user's places when the account is deleted | Data | R-011 |
| E-19 | Revoked-token check inside ms-devices | Security | R-005, ADR-008 |
| E-20 | Back-merges `main` → `dev` (`iam-backend`, `infra`) | Repository | Branch §6 |
| E-21 | Integration of the 8 abandoned frontend branches | Repository | Branch §3 |
| E-22 | README updates inside the code repositories | Documentation | `repository-documentation-review.md` |
| E-23 | Static OpenAPI files per service in `07-api/contracts/` | Documentation | `07-api/README.md` |
| E-24 | Transactional outbox for RabbitMQ events; retention of raw readings | Architecture | ADR-006, ADR-018 |

---

## Functional scope

### E-01 — Goal alerts (HU-030, HU-031)

- **What:** notify when the monthly goal is exceeded (HU-030) or about to be exceeded (HU-031).
- **State in code:** goals are implemented (HU-023, HU-024: `goals.consumption_goals`, `/api/places/{id}/goals/current`,
  progress shown in the dashboard). No event `goal.*` is published and ms-notifications has no goal notification types.
  Suggested and carried-over goals are not implemented either: the column `source` accepts `SUGGESTED` and
  `CARRIED_OVER`, but the code only creates `MANUAL` goals.
- **Why not:** goals were delivered on 2026-10-23 as the last feature of their epic; the alert needs a new event in
  ms-consumption, new types in `CK_notif_type`, a database release and templates. It did not fit before the freeze.

### E-02 — Water bills (epic E11, HU-048 – HU-052)

- **What:** register a bill, history, compare bill vs sensor, mismatch alert, payment reminder.
- **State in code:** nothing. No `billing` schema exists (ADR-019).
- **Why not:** classified *Version 2* in the Sprint Review of 2026-10-15; capacity went to sharing (E9/E10), which had
  higher priority.

### E-03 — Leak alert (HU-029)

- **What:** detect a probable leak (continuous flow for a long period, e.g. at night) and alert.
- **State in code:** not implemented in any service. Branch `feature/hu-029-leak-alert` (frontend, abandoned) only
  has screens without a backend.
- **Why not:**
  1. Moved to Version 2 on 2026-10-15 (Sprint Review) as "the first improvement of the next version".
  2. The detection rule needs real consumption data to calibrate thresholds and avoid false alarms; the project only
     had one meter in a mock-up.
  3. The sensor used (YF-S201, about 1–30 L/min) does not register drips below its minimum flow, so the most common
     small leaks would not be detected even with the rule in place. A reliable detector needs a lower-range sensor.

### E-04 — Automatic valve shutoff (epic E8, HU-035 – HU-037)

- **What:** rules to close the valve automatically on a leak, with confirmation and re-opening.
- **State in code:** only the value `AUTO_LEAK` exists in the valve command origin; no rule produces it.
- **Why not:** depends on E-03 (HU-035 depends on HU-029). Without a reliable leak detector, closing water
  automatically is a safety risk (ADR-017).

### E-05 — Reports and export (HU-038, HU-057, HU-058)

- **What:** export consumption to CSV/Excel, period report (total, average, peak, gaps), report as PDF.
- **State in code:** not in `main`. Abandoned branches `feature/hu-038-export-csv`, `feature/hu-057-consumption-report`,
  `feature/hu-058-report-pdf` implement it client-side over the obsolete frontend structure (based on 2026-10-06).
- **Why not:** the branches were not rebased after the frontend restructure (web + mobile in one project) and the
  history rewrite of 2026-10-23; re-implementing them is new development, forbidden by the freeze.

### E-06 — Tariff catalogs for Ecuador and the United States (HU-067, HU-068)

- **What:** pre-loaded tariffs by city/category for Ecuador; by district for the United States.
- **State in code:** the pre-loaded catalog has Colombian cities only; Ecuador uses the manual tariff (HU-054).
- **Why not:** HU-067 — classified *Won't (for now)* in the backlog: the pre-loaded catalog was built for Colombia
  only and Ecuador is covered by the manual tariff (HU-054). HU-068 — the United States is outside the market scope
  (ADR-004).

### E-07 — Exact value tooltip in the chart (HU-017, partial)

- **What:** show the exact value of a bar on tap/hover.
- **State in code:** the chart in `main` labels the peak only. Branch `feature/hu-017-chart-tooltip` (abandoned) has
  `bucketTooltip`.
- **Why not:** same reason as E-05 (branch on the obsolete structure).

### E-08 — Date filter in the valve history (HU-022, partial)

- **What:** filter the history of valve actions by date range.
- **State in code:** ms-valve accepts `from`/`to` (`GET /api/places/{id}/valve/commands`, default last 30 days); the app
  does not send them. Branch `feature/hu-022-valve-history-filter` (abandoned) has the UI.
- **Why not:** same reason as E-05. The backend part is done; only the UI is missing.

### E-09 — SMS channel in production (HU-028, partial)

- **What:** send codes and critical alerts by SMS.
- **State in code:** implemented (Twilio adapter, `sms_outbox`, templates). In production `TWILIO_SMS_FROM` is empty,
  so only WhatsApp is offered.
- **Why not:** cost — about US$0.06–0.34 per SMS to Colombia, paid per message by the team; WhatsApp templates cover
  the same use (ADR-016).

### E-10 — iOS app and iOS push (HU-026, partial)

- **What:** iOS build of the app and push notifications on iPhone.
- **State in code:** the Expo project is cross-platform; only the Android build (EAS) and FCM V1 credentials exist.
- **Why not:** publishing and push on iOS require a paid Apple Developer account (US$99/year) and an Apple device for
  testing; neither was available within the budget (ADR-013).

---

## Architecture, security and operations

### E-11 — Separate admin panel `sy-water-admin`

- **What:** a Vite web panel for administrators, described in the first service catalog.
- **State:** not part of the delivered system (no repository in the `cuida-tu-agua` organization). The admin area is
  part of the app, web only (ADR-011).
- **Why not:** a third frontend for three people; the integrated area reuses authentication, components and i18n.

### E-12 — WhatsApp code delivery in production (resolved)

- **What:** confirm that the phone verification code (HU-028) arrives by WhatsApp in production.
- **State:** **resolved on 2026-10-28** — the code is delivered by WhatsApp in production. The entry is kept so the
  numbering of the register does not change; it is no longer an exclusion.

### E-13 — TLS on the MQTT broker (R-006)

- **What:** Mosquitto on 8883 with TLS and the CA in the firmware.
- **State:** plain MQTT on 1883 (ADR-009).
- **Why not:** certificate handling and memory limits of MicroPython on the ESP32, plus certificate renewal on the
  device, did not fit the schedule. Risk accepted; device tokens give application-level identity.

### E-14 — Firmware in the release flow

- **What:** `main` branch, CI and versioned releases for `cuida-tu-agua-device-ESP`.
- **State:** only `dev`; flashed by hand; no README.
- **Why not:** a single physical meter, flashed by USB by its owner; there is no over-the-air update mechanism, so a
  pipeline would not deliver the firmware to the device. Not prioritized.

### E-15 — Automatic database migrations

- **What:** a Jenkins stage that runs Liquibase before the service deploy.
- **State:** releases applied by hand after a backup (ADR-010).
- **Why not:** not built before the freeze. It needs SQL Server credentials in Jenkins and a data rollback strategy,
  which the manual procedure (backup + Liquibase rollback per changeset) covers for the project's release frequency.

### E-16 — Monitoring, centralized logs and scheduled off-site backups (R-008)

- **What:** metrics and alerts (e.g. Prometheus/Grafana), log aggregation, daily backups stored outside the VPS.
- **State:** health checks, `docker logs`, and a database backup before each release only.
- **Why not:** memory on the single VPS and budget (ADR-013).

### E-17 — Staging environment

- **What:** a pre-production copy of the stack.
- **Why not:** a second VPS doubles the monthly cost (ADR-013); the local compose (`10-devops/local-setup.md`) is the
  pre-production check.

### E-18 — Deleting places when the account is deleted (R-011)

- **What:** `DELETE /internal/users/{id}/places` called by ms-iam, as it already does for devices.
- **State:** account deletion releases the devices (iam DEC-008) but the places of the user stay active (orphans).
- **Why not:** identified on 2026-10-10 with low impact (orphan rows, no data exposure) and not prioritized before the
  freeze. With sharing in place, the rule would also have to decide what happens to the collaborators of those places;
  that was not decided.

### E-19 — Revoked-token check inside ms-devices (R-005)

- **What:** check the Redis denylist in ms-devices like ms-iam does.
- **State:** checked at the gateway; ms-devices validates signature and expiry only.
- **Why not:** mitigated by the gateway (ms-devices is not published outside the Docker network); low priority.

### E-24 — Event outbox and readings retention

- **What:** transactional outbox for every RabbitMQ publisher; retention/archiving of `monitoring.readings`.
- **Why not:** at the project's volume (one meter) neither the loss window of a direct publish nor the table growth is
  material. Recorded in ADR-006 and ADR-018.

---

## Repositories and documentation

### E-20 — Back-merges `main` → `dev`

- **What:** merge hotfix `585375e` (`iam-backend`) and `595857a` (`infra`) back into `dev`.
- **Why not:** the closure mandate forbids repository changes; `main` already contains them and is the reference
  (ADR-021).

### E-21 — Integration of the abandoned frontend branches

- **What:** the 8 stacked branches `feature/hu-025…hu-058` of `cuida-tu-agua-frontNEW`.
- **Why not:** based on a structure that no longer exists (122 commits behind, ~300 files different); HU-025 and HU-033
  were already delivered another way. See E-03, E-05, E-07, E-08 for the remaining functionality.

### E-22 — README updates inside the code repositories

- **What:** missing README in `iam-backend` and the firmware; outdated READMEs in the frontend, `ms-devices`,
  `ms-valve`, `ms-device-db`, `ms-valve-db`.
- **Why not:** they live in code repositories, which are frozen. This repository is the source of truth; the
  differences are listed in `repository-documentation-review.md`.

### E-23 — Static OpenAPI files per service

- **What:** `07-api/contracts/openapi/ms-*.yaml`, referenced by some service documents.
- **Why not:** the contracts are generated at runtime by each service and aggregated at `https://api.sy-water.tech/docs`;
  a static copy would drift (risk R-002). The endpoint inventory at close is `07-api/endpoint-catalog.md`.

---

## Correlations

- Decisions → `05-architecture/decisions/README.md` (ADR-005 … ADR-021)
- Branches → `branch-status.md`
- Risks → `15-project-control/risks.md`
