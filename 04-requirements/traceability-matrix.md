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
| FR-002 | Account management | Users view/edit profile and delete account (soft delete) | Should Have |
| FR-003 | Place management | Owners create, edit, and delete places with address and stratum | Must Have |
| FR-004 | Device linking | Owners link/unlink ESP32 devices to places via serial number and MQTT | Must Have |
| FR-005 | Device monitoring | Users view device status (online/offline) and connectivity info | Must Have |
| FR-006 | Real-time consumption | Dashboard shows current consumption today and live flow rate | Must Have |
| FR-007 | Historical consumption | Users view consumption by day/week/month and export as CSV | Must Have |
| FR-008 | Consumption goals | Owners set monthly goals and receive threshold/exceeded alerts | Must Have |
| FR-009 | Leak detection | System detects continuous flow during inactivity and alerts the user | Must Have |
| FR-010 | Alert management | Users view alert history, configure notification channels, mark as resolved | Should Have |
| FR-011 | Valve control | Owners open/close the electrovalve remotely via MQTT commands | Must Have |
| FR-012 | Valve audit | Users view valve status and action history (who, when, why) | Should Have |
| FR-013 | Guest access | Owners invite guests with read-only access; guests view dashboard and receive alerts | Must Have |
| FR-014 | Bill management | Owners register bills and compare against sensor measurements | Must Have |
| FR-015 | Tariff configuration | Owners configure tariffs for cost estimation from sensor data | Should Have |
| FR-016 | Automatic shutoff | Rule-based valve closure on events (leak, goal exceeded); manual reopen only | Must Have |
| FR-017 | Admin panel | Admin views all users and devices; blocks/unblocks users; views platform metrics | Must Have |
| FR-018 | Reports | Monthly/savings reports with charts and PDF export | Should Have |
| FR-019 | Multi-place | Users manage multiple places with place switcher and aggregated consumption | Must Have |

---

## FR → HU → Service traceability

### Entrega 1 — MVP

| FR | HU(s) | Service(s) | Test file (planned) | Status |
|----|-------|-----------|---------------------|--------|
| FR-001 | HU-01, HU-02, HU-03, HU-04 | auth-service | `auth-register.spec`, `auth-login.spec`, `auth-oauth.spec`, `auth-recovery.spec` | Planned |
| FR-002 | HU-05, HU-06 | auth-service | `profile-edit.spec`, `account-delete.spec` | Planned |
| FR-003 | HU-07, HU-08, HU-09 | place-service | `place-crud.spec` | Planned |
| FR-004 | HU-10, HU-12 | device-service | `device-link.spec`, `device-unlink.spec` | Planned |
| FR-005 | HU-11 | device-service | `device-status.spec` | Planned |
| FR-006 | HU-13, HU-15, HU-17 | consumption-service, device-service | `consumption-today.spec`, `flow-rate.spec` | Planned |
| FR-007 | HU-14, HU-16 | consumption-service | `consumption-history.spec`, `consumption-export.spec` | Planned |

### Entrega 2 — Smart Monitoring

| FR | HU(s) | Service(s) | Test file (planned) | Status |
|----|-------|-----------|---------------------|--------|
| FR-008 | HU-18, HU-19, HU-20, HU-21 | consumption-service, notification-service | `goal-crud.spec`, `goal-alerts.spec` | Planned |
| FR-009 | HU-22 | consumption-service, notification-service | `leak-detection.spec` | Planned |
| FR-010 | HU-23, HU-24, HU-25 | notification-service | `alert-history.spec`, `notification-prefs.spec` | Planned |
| FR-011 | HU-26, HU-27 | valve-service, device-service | `valve-close.spec`, `valve-open.spec` | Planned |
| FR-012 | HU-28, HU-29 | valve-service | `valve-status.spec`, `valve-history.spec` | Planned |

### Entrega 3 — Social & Billing

| FR | HU(s) | Service(s) | Test file (planned) | Status |
|----|-------|-----------|---------------------|--------|
| FR-013 | HU-30, HU-31, HU-32, HU-33 | place-service, notification-service | `guest-invite.spec`, `guest-access.spec` | Planned |
| FR-014 | HU-34, HU-35, HU-36, HU-37 | consumption-service, notification-service | `bill-register.spec`, `bill-compare.spec` | Planned |
| FR-015 | HU-38, HU-39, HU-40 | consumption-service | `tariff-config.spec`, `cost-estimate.spec` | Planned |
| FR-016 | HU-41, HU-42, HU-43, HU-44 | valve-service, notification-service | `shutoff-rule.spec`, `auto-shutoff.spec` | Planned |

### Entrega 4 — Administration & Analytics

| FR | HU(s) | Service(s) | Test file (planned) | Status |
|----|-------|-----------|---------------------|--------|
| FR-017 | HU-45, HU-46, HU-47, HU-48 | auth-service, device-service, place-service | `admin-users.spec`, `admin-devices.spec` | Planned |
| FR-018 | HU-49, HU-50, HU-51, HU-52 | consumption-service | `report-monthly.spec`, `report-pdf.spec` | Planned |
| FR-019 | HU-53, HU-54, HU-55, HU-56 | place-service, consumption-service, device-service | `multi-place.spec` | Planned |

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
| HU-01 | Register with email and password | FR-001 | 1 |
| HU-02 | Login with email and password | FR-001 | 1 |
| HU-03 | Login with Google OAuth | FR-001 | 1 |
| HU-04 | Recover password | FR-001 | 1 |
| HU-05 | View and edit profile | FR-002 | 1 |
| HU-06 | Delete account | FR-002 | 1 |
| HU-07 | Register a place | FR-003 | 1 |
| HU-08 | Edit place information | FR-003 | 1 |
| HU-09 | Delete a place | FR-003 | 1 |
| HU-10 | Link a device | FR-004 | 1 |
| HU-11 | View device status | FR-005 | 1 |
| HU-12 | Unlink a device | FR-004 | 1 |
| HU-13 | View current consumption | FR-006 | 1 |
| HU-14 | View consumption history | FR-007 | 1 |
| HU-15 | View current flow rate | FR-006 | 1 |
| HU-16 | Export consumption data | FR-007 | 1 |
| HU-17 | Consumption summary on home | FR-006 | 1 |
| HU-18 | Set monthly goal | FR-008 | 2 |
| HU-19 | View goal progress | FR-008 | 2 |
| HU-20 | Alert approaching goal | FR-008 | 2 |
| HU-21 | Alert goal exceeded | FR-008 | 2 |
| HU-22 | Leak detection alert | FR-009 | 2 |
| HU-23 | View alert history | FR-010 | 2 |
| HU-24 | Configure notifications | FR-010 | 2 |
| HU-25 | Mark alert resolved | FR-010 | 2 |
| HU-26 | Close valve remotely | FR-011 | 2 |
| HU-27 | Open valve remotely | FR-011 | 2 |
| HU-28 | View valve status | FR-012 | 2 |
| HU-29 | View valve action history | FR-012 | 2 |
| HU-30 | Invite guest | FR-013 | 3 |
| HU-31 | Guest views dashboard | FR-013 | 3 |
| HU-32 | Revoke guest access | FR-013 | 3 |
| HU-33 | Guest receives alerts | FR-013 | 3 |
| HU-34 | Register a bill | FR-014 | 3 |
| HU-35 | Compare bill vs. sensor | FR-014 | 3 |
| HU-36 | View bill history | FR-014 | 3 |
| HU-37 | Alert for bill discrepancy | FR-014 | 3 |
| HU-38 | Configure tariff | FR-015 | 3 |
| HU-39 | View estimated cost | FR-015 | 3 |
| HU-40 | Tariff updates (future only) | FR-015 | 3 |
| HU-41 | Configure auto-shutoff rule | FR-016 | 3 |
| HU-42 | Auto-shutoff activates | FR-016 | 3 |
| HU-43 | Disable auto-shutoff | FR-016 | 3 |
| HU-44 | Manual reopen after auto-shutoff | FR-016 | 3 |
| HU-45 | Admin views all users | FR-017 | 4 |
| HU-46 | Admin blocks/unblocks user | FR-017 | 4 |
| HU-47 | Admin views all devices | FR-017 | 4 |
| HU-48 | Admin views platform metrics | FR-017 | 4 |
| HU-49 | Monthly consumption report | FR-018 | 4 |
| HU-50 | Water savings report | FR-018 | 4 |
| HU-51 | Download report as PDF | FR-018 | 4 |
| HU-52 | Consumption ranking | FR-018 | 4 |
| HU-53 | Register multiple places | FR-019 | 4 |
| HU-54 | Switch between places | FR-019 | 4 |
| HU-55 | Aggregated consumption | FR-019 | 4 |
| HU-56 | Manage devices per place | FR-019 | 4 |

---

## Coverage summary

| Entrega | FRs covered | HUs | Test files (planned) |
|---------|------------|-----|---------------------|
| 1 | FR-001 to FR-007 | 17 | 11 |
| 2 | FR-008 to FR-012 | 12 | 9 |
| 3 | FR-013 to FR-016 | 15 | 8 |
| 4 | FR-017 to FR-019 | 12 | 4 |
| **Total** | **19 FRs** | **56 HUs** | **32 test files** |

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
