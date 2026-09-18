# Delivery Roadmap

> This document maps the 4 deliveries (Entregas) to epics, features, and user stories.
> No fixed dates — deliveries are ordered by priority and executed sequentially.

---

## Delivery overview

```
Entrega 1 (MVP)          Entrega 2 (Smart)        Entrega 3 (Social)       Entrega 4 (Admin)
┌─────────────────┐      ┌─────────────────┐      ┌─────────────────┐      ┌─────────────────┐
│ E1: Auth         │      │ E5: Valve Ctrl   │      │ E8: Bill Mgmt    │      │ E13: Reports     │
│ E2: Place Mgmt   │      │ E6: Goals        │      │ E9: Tariffs      │      │ E14: Savings Tips│
│ E3: Device Link  │      │ E7: Alerts       │      │ E10: Auto-shutoff│      │ E15: Tariff Cat. │
│ E4: Dashboard    │      │                  │      │ E11: Members     │      │ E16: Admin Panel │
│                  │      │                  │      │ E12: Collaborator│      │                  │
└─────────────────┘      └─────────────────┘      └─────────────────┘      └─────────────────┘
        ~80 SP                   ~56 SP                   ~65 SP                   ~56 SP
                                                                          Total: ~257 SP
```

---

## Entrega 1 — MVP

> **Goal:** A user registers, creates a place, links a device, and sees real-time consumption on a dashboard.

### E1 — Accounts & Access

| ID | User story | SP | Acceptance criteria summary |
|----|-----------|----|-----------------------------|
| HU-001 | Register with email and password | 5 | First name, last name, email, phone, password; verification email sent |
| HU-003 | Login with email and password | 3 | JWT issued, session created |
| HU-004 | Login with Google OAuth | 5 | Google sign-in, auto-creates account if new |
| HU-005 | Recover password | 5 | Reset link via email or code via SMS, 15-minute expiration |
| HU-006 | View and edit my profile | 3 | Name, phone, photo editable |
| HU-007 | Delete my account | 3 | Soft-delete, device unlinked, sessions revoked |

**Epic total: 24 SP**
### E2 — Place Management

| ID | User story | SP | Acceptance criteria summary |
|----|-----------|----|-----------------------------|
| HU-002 | Register a home/place | 5 | Name, address, country, department/state, city, stratum, inhabitants; unique name per user |
| HU-008 | Register a place (edit) | 5 | Name, address, country, department/state, city, stratum, inhabitants; unique name per user |
| HU-009 | Edit place information | 3 | All fields editable except owner |
| HU-010 | Delete a place | 3 | Soft-delete, devices unlinked |

**Epic total: 16 SP** (was 11 SP)
### E3 — IoT Device Linking

| ID | User story | SP | Acceptance criteria summary |
|----|-----------|----|-----------------------------|
| HU-011 | Link a device to a place | 8 | Serial number input, device activates via MQTT, readings start |
| HU-012 | View device status | 3 | Online/offline (10-min threshold), last reading timestamp, battery/signal |
| HU-013 | Unlink a device | 3 | Device deactivated, readings stop, can be re-linked |

**Epic total: 14 SP**
### E4 — Real-time Consumption Monitoring

| ID | User story | SP | Acceptance criteria summary |
|----|-----------|----|-----------------------------|
| HU-014 | View current consumption (today) | 8 | Real-time liters consumed today, auto-refresh |
| HU-015 | View consumption history (daily/weekly/monthly) | 8 | Charts with date range selector |
| HU-016 | View current flow rate | 5 | Live liters/minute from last reading |
| HU-017 | Export consumption data | 5 | CSV download for selected period |
| HU-018 | View consumption summary on home screen | 5 | Quick glance: today, this week, this month |

**Epic total: 31 SP**
### Entrega 1 summary

| Metric | Value |
|--------|-------|
| Epics | 4 (E1–E4) |
| User stories | 18 |
| Story points | ~85 SP |
| Services involved | api-gateway, auth-service, place-service, device-service, consumption-service |
| Schemas activated | security, places, devices, monitoring (partial) |

---

## Entrega 2 — Smart Monitoring

> **Goal:** Users control the valve remotely, set consumption goals, and receive alerts for anomalies and leaks.

### E5 — Manual Valve Control

| ID | User story | SP | Acceptance criteria summary |
|----|-----------|----|-----------------------------|
| HU-027 | Close the valve remotely | 8 | Confirmation code sent, user confirms before MQTT command, warning shown |
| HU-028 | Open the valve remotely | 8 | MQTT command sent, confirmation received, status updated |
| HU-029 | View valve status | 3 | Open/closed, last action timestamp, who triggered it |
| HU-030 | View valve action history | 3 | List of all open/close actions with actor and reason |

**Epic total: 22 SP**
### E6 — Consumption Goals

| ID | User story | SP | Acceptance criteria summary |
|----|-----------|----|-----------------------------|
| HU-019 | Set a monthly consumption goal | 5 | Goal in liters, per place |
| HU-020 | View goal progress | 5 | Progress bar, percentage consumed vs. goal |
| HU-021 | Receive alert when approaching goal | 3 | Notification at configurable threshold (e.g., 80%) |
| HU-022 | Receive alert when goal is exceeded | 3 | Notification when 100% reached |

**Epic total: 16 SP**
### E7 — Alerts & Notifications

| ID | User story | SP | Acceptance criteria summary |
|----|-----------|----|-----------------------------|
| HU-023 | Receive leak detection alert | 8 | Alert when continuous flow detected during inactivity hours |
| HU-024 | View alert history | 3 | List of all alerts with timestamp, type, status |
| HU-025 | Configure notification preferences | 5 | Push, email, WhatsApp toggle per alert type |
| HU-026 | Mark alert as resolved | 2 | Manual acknowledgment |

**Epic total: 18 SP**
### Entrega 2 summary

| Metric | Value |
|--------|-------|
| Epics | 3 (E5–E7) |
| User stories | 12 |
| Story points | ~56 SP |
| New services involved | valve-service, notification-service |
| Schemas activated | valve, goals, alerts, notifications |

---

## Entrega 3 — Social & Billing

> **Goal:** Owners manage utility bills and tariffs, configure automatic shutoff rules, and invite members or collaborators.

### E8 — Utility Bill Management

| ID | User story | SP | Acceptance criteria summary |
|----|-----------|----|-----------------------------|
| HU-035 | Register a water bill | 5 | Amount, period, liters billed, provider |
| HU-036 | Compare bill vs. sensor measurement | 8 | Side-by-side for the same period, difference highlighted |
| HU-037 | View bill history | 3 | List of all registered bills |
| HU-038 | Receive alert for bill discrepancy | 5 | Alert when sensor vs. bill difference exceeds threshold |

**Epic total: 21 SP**
### E9 — Tariffs & Measurement Units

| ID | User story | SP | Acceptance criteria summary |
|----|-----------|----|-----------------------------|
| HU-039 | Configure water tariff for a place | 5 | Rate per m³, stratum, provider |
| HU-040 | View estimated cost from sensor data | 5 | Cost calculated from consumption × tariff |
| HU-041 | Tariff updates apply to future estimates | 2 | Historical estimates unchanged |

**Epic total: 12 SP**
### E10 — Automatic Valve Shutoff

| ID | User story | SP | Acceptance criteria summary |
|----|-----------|----|-----------------------------|
| HU-042 | Configure automatic shutoff rule | 8 | Trigger conditions (e.g., leak detected, goal exceeded) |
| HU-043 | Automatic shutoff activates on rule trigger | 8 | Confirmation request sent, user confirms before closing valve |
| HU-044 | Disable automatic shutoff | 2 | Rules deactivated, valve stays manual |
| HU-045 | Reopen valve after auto-shutoff (manual only) | 3 | Safety: never auto-reopens |

**Epic total: 21 SP** (was 18 SP)
### E11 — Shared Users (Owner / Guest)

| ID | User story | SP | Acceptance criteria summary |
|----|-----------|----|-----------------------------|
| HU-031 | Invite a member to a place | 5 | Email invitation, permissions assigned by owner |
| HU-032 | Member views consumption dashboard | 3 | Access to place consumption per owner-defined permissions |
| HU-033 | Revoke member access | 3 | Immediate access removal |
| HU-034 | Member receives alerts for the place | 3 | Alert types based on owner-defined permissions |

**Epic total: 14 SP**
### E12 — Shared Users (Owner / Collaborator)

> User stories for this epic will be defined during sprint planning for Entrega 3.
> Covers inviting collaborators with extended permissions (beyond read-only guest access) to a place.

### Entrega 3 summary

| Metric | Value |
|--------|-------|
| Epics | 5 (E8–E12) |
| User stories | 15 + E12 TBD |
| Story points | ~65 SP + E12 TBD |
| Schema changes | billing (bills, tariffs), valve (shutoff_rules), places (members, invitations) |
| Note | HU-043 (Auto-shutoff) upgraded to 8 SP (was 5 SP) to include mandatory confirmation flow |

---

## Entrega 4 — Administration & Analytics

> **Goal:** Downloadable reports, savings tips, pre-loaded tariff catalogs, an admin panel, and multi-place support.

### E13 — Downloadable Reports

| ID | User story | SP | Acceptance criteria summary |
|----|-----------|----|-----------------------------|
| HU-050 | View monthly consumption report | 5 | Summary with charts, comparison to previous months |
| HU-051 | View water savings report | 5 | Savings vs. goal, trend over time |
| HU-052 | Download report as PDF | 5 | Formatted report with charts and data |
| HU-053 | View consumption ranking across places | 3 | Leaderboard for multi-place users |

**Epic total: 18 SP**
### E14 — Savings Tips

> User stories for this epic will be defined during sprint planning for Entrega 4.
> Covers generic water-saving recommendations displayed to users, differentiated by place type (residential / commercial).

### E15 — Preloaded Tariffs by Country

> User stories for this epic will be defined during sprint planning for Entrega 4.
> Covers pre-loaded tariff catalogs for Colombia (by city and stratum), Ecuador (by city and service category), and the United States (by state and water district).

### E16 — Admin Panel

| ID | User story | SP | Acceptance criteria summary |
|----|-----------|----|-----------------------------|
| HU-046 | Admin views all registered users | 3 | Paginated list, search by email/name. No sensitive data (no passwords, identity documents, or billing info) |
| HU-047 | Admin blocks/unblocks a user | 3 | User sessions revoked on block |
| HU-048 | Admin views all devices | 3 | Status, linked place, last reading |
| HU-049 | Admin views platform metrics | 5 | Total users, devices, places, active/inactive |
| HU-058 | Admin creates a recommendation | 5 | Title, body, category (saving tip, maintenance, seasonal, general); visible to all users |
| HU-059 | Admin edits or deactivates a recommendation | 3 | Deactivated recommendations hidden from users but not deleted |

**Epic total: 22 SP**
### Multi-place Extensions (extends E2 / E3)

| ID | User story | SP | Acceptance criteria summary |
|----|-----------|----|-----------------------------|
| HU-054 | Register multiple places | 5 | Each place independent, own device |
| HU-055 | Switch between places in the app | 3 | Place selector in navigation |
| HU-056 | View aggregated consumption across places | 5 | Total and per-place breakdown |
| HU-057 | Manage devices per place | 3 | Link/unlink per place |

**Epic total: 16 SP**
### Entrega 4 summary

| Metric | Value |
|--------|-------|
| Epics | 4 (E13–E16) + multi-place extensions |
| User stories | 14 + E14/E15 TBD |
| Story points | ~56 SP + E14/E15 TBD |
| New role | Admin (Administrador) |

---

## Cumulative progress by delivery

| Entrega | Epics | User stories | Story points | Cumulative SP |
|---------|-------|-------------|-------------|---------------|
| Entrega 1 (MVP) | E1–E4 | 18 | ~85 | ~85 |
| Entrega 2 (Smart) | E5–E7 | 12 | ~56 | ~141 |
| Entrega 3 (Social) | E8–E12 | 15 | ~65 | ~206 |
| Entrega 4 (Admin) | E13–E16 | 14 | ~56 | ~262 |
| **Total** | **16 epics** | **59 HUs** | **~262 SP** | |

> **Note:** Story point totals are approximate. Changes in this release:
> - Added HU-002 (Register a home/place) — 5 SP — independent of HU-001, placed before login stories
> - HU-043 (Auto-shutoff) upgraded from 5 SP to 8 SP to include mandatory confirmation flow
> - HU-005 (Recover password) changed to accept email OR phone with 15-minute expiration
> - HU-012 (View device status) offline threshold changed from 5 to 10 minutes
> - HU-027 (Close valve remotely) now requires confirmation code + explicit warning
> - Renumbered all downstream HUs (HU-003 onwards) to accommodate HU-002
> - Epics E12, E14, and E15 do not yet have user stories and will add to these totals

---

## Dependencies between deliveries

```
Entrega 1 (MVP)
    ├──→ Entrega 2 (Smart)
    │        ├──→ Entrega 3 (Social)  [needs alerts + valve from E2]
    │        └──→ Entrega 4 (Admin)   [needs all data from E1+E2]
    └──→ Entrega 3 (Social)          [needs auth + place from E1]
```

| Dependency | From | To | Reason |
|-----------|------|-----|--------|
| Auth & Place | E1 | All others | Every feature requires authenticated users and registered places |
| Device + Dashboard | E3, E4 | E5–E7 | Goals and alerts need consumption data flowing |
| Alerts | E7 | E10 | Auto-shutoff rules trigger alerts |
| Valve control | E5 | E10 | Auto-shutoff sends valve commands |
| All data | E1–E7 | E13, E16 | Admin and reports need all domain data |

---

## Correlations

- Product vision → `03-product/vision.md`
- Problem framing → `03-product/problem-framing.md`
- User stories (detailed) → `04-requirements/user-stories.md`
- Domain events by Entrega → `02-domain/domain-events.md`
- Database tables by Entrega → `06-data/models.md`
- Architecture overview → `05-architecture/overview.md`
