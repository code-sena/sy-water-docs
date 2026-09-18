# Traceability Matrix

> Maps each functional requirement to its user stories, owning service, and delivery.
> Answers: what validates each requirement and which service implements it.

---

## How to read this matrix

```
FR (functional requirement) → HU (user stories) → Service → Entrega

If an FR has no HU → the requirement is not planned
If an HU has no test → there is test debt
If code exists without an HU → unplanned work (gold plating)
```

---

## Functional requirements catalog

| FR ID | Area | Description | Priority |
|-------|------|-------------|----------|
| FR-001 | Authentication | Users register, login (email + Google), recover password, and manage sessions | Must Have |
| FR-002 | Place management | First place registration as part of onboarding, separate from account registration | Must Have |
| FR-003 | Account management | Users view/edit profile and delete account (soft delete) | Should Have |
| FR-004 | Place management extended | Owners create, edit, and delete places with address and stratum | Must Have |
| FR-005 | Device linking | Owners link/unlink ESP32 devices to places via serial number and MQTT | Must Have |
| FR-006 | Device monitoring | Users view device status (online/offline, 10-min threshold) and connectivity info | Must Have |
| FR-007 | Real-time consumption | Dashboard shows current consumption today and live flow rate | Must Have |
| FR-008 | Historical consumption | Users view consumption by day/week/month and export as CSV | Must Have |
| FR-009 | Consumption goals | Owners set monthly goals and receive threshold/exceeded alerts | Must Have |
| FR-010 | Leak detection | System detects continuous flow during inactivity and alerts the user | Must Have |
| FR-011 | Alert management | Users view alert history, configure notification channels, mark as resolved | Should Have |
| FR-012 | Valve control | Owners open/close the electrovalve remotely with confirmation code | Must Have |
| FR-013 | Valve audit | Users view valve status and action history (who, when, why) | Should Have |
| FR-014 | Member access | Owners invite members with owner-defined permissions; members view dashboard and receive alerts | Must Have |
| FR-015 | Bill management | Owners register bills and compare against sensor measurements | Must Have |
| FR-016 | Tariff configuration | Owners configure tariffs for cost estimation from sensor data | Should Have |
| FR-017 | Automatic shutoff | Rule-based valve closure with mandatory confirmation; manual reopen only | Must Have |
| FR-018 | Admin panel | Admin views users (no sensitive data), blocks/unblocks users, views devices and platform metrics, creates/manages generic recommendations | Must Have |
| FR-019 | Reports | Monthly/savings reports with charts and PDF export | Should Have |
| FR-020 | Multi-place | Users manage multiple places with place switcher and aggregated consumption | Must Have |

---

## FR → HU → Service traceability

### Entrega 1 — MVP

| FR | HU(s) | Service(s) | Test file (planned) | Status |
|----|-------|-----------|---------------------|--------|
| FR-001 | HU-001, HU-003, HU-004, HU-005 | auth-service | `auth-register.spec`, `auth-login.spec`, `auth-oauth.spec`, `auth-recovery.spec` | Planned |
| FR-002 | HU-002 | place-service | `place-register.spec` | Planned |
| FR-003 | HU-006, HU-007 | auth-service | `profile-edit.spec`, `account-delete.spec` | Planned |
| FR-004 | HU-008, HU-009, HU-010 | place-service | `place-crud.spec` | Planned |
| FR-005 | HU-011, HU-013 | device-service | `device-link.spec`, `device-unlink.spec` | Planned |
| FR-006 | HU-012 | device-service | `device-status.spec` | Planned |
| FR-007 | HU-014, HU-016, HU-018 | consumption-service, device-service | `consumption-today.spec`, `flow-rate.spec` | Planned |
| FR-008 | HU-015, HU-017 | consumption-service | `consumption-history.spec`, `consumption-export.spec` | Planned |

### Entrega 2 — Smart Monitoring

| FR | HU(s) | Service(s) | Test file (planned) | Status |
|----|-------|-----------|---------------------|--------|
| FR-009 | HU-019, HU-020, HU-021, HU-022 | consumption-service, notification-service | `goal-crud.spec`, `goal-alerts.spec` | Planned |
| FR-010 | HU-023 | consumption-service, notification-service | `leak-detection.spec` | Planned |
| FR-011 | HU-024, HU-025, HU-026 | notification-service | `alert-history.spec`, `notification-prefs.spec` | Planned |
| FR-012 | HU-027, HU-028 | valve-service, device-service, notification-service | `valve-close.spec`, `valve-open.spec` | Planned |
| FR-013 | HU-029, HU-030 | valve-service | `valve-status.spec`, `valve-history.spec` | Planned |

### Entrega 3 — Social & Billing

| FR | HU(s) | Service(s) | Test file (planned) | Status |
|----|-------|-----------|---------------------|--------|
| FR-014 | HU-031, HU-032, HU-033, HU-034 | place-service, notification-service | `member-invite.spec`, `member-access.spec` | Planned |
| FR-015 | HU-035, HU-036, HU-037, HU-038 | consumption-service, notification-service | `bill-register.spec`, `bill-compare.spec` | Planned |
| FR-016 | HU-039, HU-040, HU-041 | consumption-service | `tariff-config.spec`, `cost-estimate.spec` | Planned |
| FR-017 | HU-042, HU-043, HU-044, HU-045 | valve-service, notification-service | `shutoff-rule.spec`, `auto-shutoff.spec` | Planned |

### Entrega 4 — Administration & Analytics

| FR | HU(s) | Service(s) | Test file (planned) | Status |
|----|-------|-----------|---------------------|--------|
| FR-018 | HU-046, HU-047, HU-048, HU-049, HU-058, HU-059 | auth-service, device-service, place-service, notification-service | `admin-users.spec`, `admin-devices.spec`, `admin-recommendations.spec` | Planned |
| FR-019 | HU-050, HU-051, HU-052, HU-053 | consumption-service | `report-monthly.spec`, `report-pdf.spec` | Planned |
| FR-020 | HU-054, HU-055, HU-056, HU-057 | place-service, consumption-service, device-service | `multi-place.spec` | Planned |

---

## NFR → Validation traceability

| NFR | Description | Validation method | Tool | Status |
|-----|-------------|-------------------|------|--------|
| NFR-001 | API performance (p95 targets) | Manual timing / load test | Postman / k6 | Planned |
| NFR-002 | 99% availability | Health check monitoring | Docker health checks | Planned |
| NFR-003 | 50 concurrent users, 100 devices | Capacity test | k6 (aspirational) | Planned |
| NFR-004 | JWT + RBAC + input validation | Code review + manual test | Manual | Planned |
| NFR-005 | IoT reliability (reconnect, QoS, idempotency) | Integration test with MQTT | Mosquitto test client | Planned |
| NFR-006 | Structured logs with correlation ID | Log inspection | Manual | Planned |
| NFR-007 | 80% domain test coverage | Coverage report | JaCoCo / dotnet-coverage | Planned |
| NFR-008 | Docker deployment | Smoke test | docker-compose up | Planned |

---

## Inverse traceability: HU → FR

| HU | Title | FR(s) | Entrega |
|----|-------|-------|---------|
| HU-001 | Register with email and password | FR-001 | 1 |
| HU-002 | Register a home/place | FR-002 | 1 |
| HU-003 | Login with email and password | FR-001 | 1 |
| HU-004 | Login with Google OAuth | FR-001 | 1 |
| HU-005 | Recover password | FR-001 | 1 |
| HU-006 | View and edit profile | FR-003 | 1 |
| HU-007 | Delete account | FR-003 | 1 |
| HU-008 | Register a place | FR-004 | 1 |
| HU-009 | Edit place information | FR-004 | 1 |
| HU-010 | Delete a place | FR-004 | 1 |
| HU-011 | Link a device | FR-005 | 1 |
| HU-012 | View device status | FR-006 | 1 |
| HU-013 | Unlink a device | FR-005 | 1 |
| HU-014 | View current consumption | FR-007 | 1 |
| HU-015 | View consumption history | FR-008 | 1 |
| HU-016 | View current flow rate | FR-007 | 1 |
| HU-017 | Export consumption data | FR-008 | 1 |
| HU-018 | Consumption summary on home | FR-007 | 1 |
| HU-019 | Set monthly goal | FR-009 | 2 |
| HU-020 | View goal progress | FR-009 | 2 |
| HU-021 | Alert approaching goal | FR-009 | 2 |
| HU-022 | Alert goal exceeded | FR-009 | 2 |
| HU-023 | Leak detection alert | FR-010 | 2 |
| HU-024 | View alert history | FR-011 | 2 |
| HU-025 | Configure notifications | FR-011 | 2 |
| HU-026 | Mark alert resolved | FR-011 | 2 |
| HU-027 | Close valve remotely | FR-012 | 2 |
| HU-028 | Open valve remotely | FR-012 | 2 |
| HU-029 | View valve status | FR-013 | 2 |
| HU-030 | View valve action history | FR-013 | 2 |
| HU-031 | Invite member | FR-014 | 3 |
| HU-032 | Member views dashboard | FR-014 | 3 |
| HU-033 | Revoke member access | FR-014 | 3 |
| HU-034 | Member receives alerts | FR-014 | 3 |
| HU-035 | Register a bill | FR-015 | 3 |
| HU-036 | Compare bill vs. sensor | FR-015 | 3 |
| HU-037 | View bill history | FR-015 | 3 |
| HU-038 | Alert for bill discrepancy | FR-015 | 3 |
| HU-039 | Configure tariff | FR-016 | 3 |
| HU-040 | View estimated cost | FR-016 | 3 |
| HU-041 | Tariff updates (future only) | FR-016 | 3 |
| HU-042 | Configure auto-shutoff rule | FR-017 | 3 |
| HU-043 | Auto-shutoff activates | FR-017 | 3 |
| HU-044 | Disable auto-shutoff | FR-017 | 3 |
| HU-045 | Manual reopen after auto-shutoff | FR-017 | 3 |
| HU-046 | Admin views all users | FR-018 | 4 |
| HU-047 | Admin blocks/unblocks user | FR-018 | 4 |
| HU-048 | Admin views all devices | FR-018 | 4 |
| HU-049 | Admin views platform metrics | FR-018 | 4 |
| HU-050 | Monthly consumption report | FR-019 | 4 |
| HU-051 | Water savings report | FR-019 | 4 |
| HU-052 | Download report as PDF | FR-019 | 4 |
| HU-053 | Consumption ranking | FR-019 | 4 |
| HU-054 | Register multiple places | FR-020 | 4 |
| HU-055 | Switch between places | FR-020 | 4 |
| HU-056 | Aggregated consumption | FR-020 | 4 |
| HU-057 | Manage devices per place | FR-020 | 4 |
| HU-058 | Admin creates recommendation | FR-018 | 4 |
| HU-059 | Admin edits/deactivates recommendation | FR-018 | 4 |

---

## Coverage summary

| Entrega | FRs covered | HUs | Test files (planned) |
|---------|------------|-----|---------------------|
| 1 | FR-001 to FR-008 | 18 | 12 |
| 2 | FR-009 to FR-013 | 12 | 9 |
| 3 | FR-014 to FR-017 | 15 | 8 |
| 4 | FR-018 to FR-020 | 14 | 5 |
| **Total** | **20 FRs** | **59 HUs** | **34 test files** |

---

## Identified gaps

| Gap type | Description | Action | Owner |
|----------|-------------|--------|-------|
| No test implementation yet | All test files are planned, none implemented | Create test files as each Entrega begins | Team |
| NFR validation manual | Most NFR validation relies on manual testing | Automate gradually; minimum: structured logs | Team |
| No E2E tests planned | End-to-end flows not yet defined | Define critical path E2E tests before Entrega 1 launch | Team |

---

## Status legend

| Status | Meaning |
|--------|---------|
| Planned | In backlog, not yet started |
| In progress | Under development |
| Done | Implemented, tested, validated |
| Blocked | Requires external dependency |

---

## Correlations

- User stories → `04-requirements/user-stories.md`
- Non-functional requirements → `04-requirements/non-functional.md`
- Delivery roadmap → `03-product/roadmap.md`
- Testing strategy → `11-quality/testing-strategy.md`
- Domain events → `02-domain/domain-events.md`
