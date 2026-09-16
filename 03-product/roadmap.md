# Delivery Roadmap

> This document maps the 4 deliveries (Entregas) to epics, features, and user stories.
> No fixed dates — deliveries are ordered by priority and executed sequentially.

---

## Delivery overview

```
Entrega 1 (MVP)          Entrega 2 (Smart)        Entrega 3 (Social)       Entrega 4 (Admin)
┌─────────────────┐      ┌─────────────────┐      ┌─────────────────┐      ┌─────────────────┐
│ E1: Auth         │      │ E5: Goals        │      │ E8: Members      │      │ E12: Admin Panel │
│ E2: Place Mgmt   │      │ E6: Alerts       │      │ E9: Bill Mgmt    │      │ E13: Reports     │
│ E3: Device Link  │      │ E7: Valve Ctrl   │      │ E10: Tariffs     │      │ E14: Multi-place │
│ E4: Dashboard    │      │                  │      │ E11: Auto-shutoff│      │                  │
└─────────────────┘      └─────────────────┘      └─────────────────┘      └─────────────────┘
        ~80 SP                   ~70 SP                   ~66 SP                   ~50 SP
                                                                          Total: ~266 SP
```

---

## Entrega 1 — MVP

> **Goal:** A user registers, creates a place, links a device, and sees real-time consumption on a dashboard.

### E1 — Authentication & Account Management

| ID | User story | SP | Acceptance criteria summary |
|----|-----------|----|-----------------------------|
| HU-001 | Register with email and password | 5 | Email/password validation, verification email sent |
| HU-002 | Login with email and password | 3 | JWT issued, session created |
| HU-003 | Login with Google OAuth | 5 | Google sign-in, auto-creates account if new |
| HU-004 | Recover password | 5 | Reset token via email, password updated |
| HU-005 | View and edit my profile | 3 | Name, phone, photo editable |
| HU-006 | Delete my account | 3 | Soft-delete, device unlinked, sessions revoked |

**Epic total: 24 SP**

### E2 — Place Management

| ID | User story | SP | Acceptance criteria summary |
|----|-----------|----|-----------------------------|
| HU-007 | Register a place | 5 | Name, address, stratum (estrato), linked to owner |
| HU-008 | Edit place information | 3 | All fields editable except owner |
| HU-009 | Delete a place | 3 | Soft-delete, devices unlinked |

**Epic total: 11 SP**

### E3 — Device Linking

| ID | User story | SP | Acceptance criteria summary |
|----|-----------|----|-----------------------------|
| HU-010 | Link a device to a place | 8 | Serial number input, device activates via MQTT, readings start |
| HU-011 | View device status | 3 | Online/offline, last reading timestamp, battery/signal |
| HU-012 | Unlink a device | 3 | Device deactivated, readings stop, can be re-linked |

**Epic total: 14 SP**

### E4 — Consumption Dashboard

| ID | User story | SP | Acceptance criteria summary |
|----|-----------|----|-----------------------------|
| HU-013 | View current consumption (today) | 8 | Real-time liters consumed today, auto-refresh |
| HU-014 | View consumption history (daily/weekly/monthly) | 8 | Charts with date range selector |
| HU-015 | View current flow rate | 5 | Live liters/minute from last reading |
| HU-016 | Export consumption data | 5 | CSV download for selected period |
| HU-017 | View consumption summary on home screen | 5 | Quick glance: today, this week, this month |

**Epic total: 31 SP**

### Entrega 1 summary

| Metric | Value |
|--------|-------|
| Epics | 4 (E1–E4) |
| User stories | 17 |
| Story points | ~80 SP |
| Services involved | api-gateway, auth-service, place-service, device-service, consumption-service |
| Schemas activated | iam, place, device, consumption (partial) |

---

## Entrega 2 — Smart Monitoring

> **Goal:** Users set consumption goals, receive alerts for anomalies and leaks, and can manually open/close the valve.

### E5 — Consumption Goals

| ID | User story | SP | Acceptance criteria summary |
|----|-----------|----|-----------------------------|
| HU-018 | Set a monthly consumption goal | 5 | Goal in liters, per place |
| HU-019 | View goal progress | 5 | Progress bar, percentage consumed vs. goal |
| HU-020 | Receive alert when approaching goal | 3 | Notification at configurable threshold (e.g., 80%) |
| HU-021 | Receive alert when goal is exceeded | 3 | Notification when 100% reached |

**Epic total: 16 SP**

### E6 — Alerts & Notifications

| ID | User story | SP | Acceptance criteria summary |
|----|-----------|----|-----------------------------|
| HU-022 | Receive leak detection alert | 8 | Alert when continuous flow detected during inactivity hours |
| HU-023 | View alert history | 3 | List of all alerts with timestamp, type, status |
| HU-024 | Configure notification preferences | 5 | Push, email, WhatsApp toggle per alert type |
| HU-025 | Mark alert as resolved | 2 | Manual acknowledgment |

**Epic total: 18 SP**

### E7 — Valve Control

| ID | User story | SP | Acceptance criteria summary |
|----|-----------|----|-----------------------------|
| HU-026 | Close the valve remotely | 8 | MQTT command sent, confirmation received, status updated |
| HU-027 | Open the valve remotely | 8 | MQTT command sent, confirmation received, status updated |
| HU-028 | View valve status | 3 | Open/closed, last action timestamp, who triggered it |
| HU-029 | View valve action history | 3 | List of all open/close actions with actor and reason |

**Epic total: 22 SP**

### Entrega 2 summary

| Metric | Value |
|--------|-------|
| Epics | 3 (E5–E7) |
| User stories | 12 |
| Story points | ~56 SP |
| New services involved | valve-service, notification-service |
| Schemas activated | consumption (goals), valve, notification |

---

## Entrega 3 — Social & Billing

> **Goal:** Owners invite members, manage bills and tariffs, and configure automatic shutoff rules.

### E8 — Member Management

| ID | User story | SP | Acceptance criteria summary |
|----|-----------|----|-----------------------------|
| HU-030 | Invite a member to a place | 5 | Email invitation, permissions assigned by owner |
| HU-031 | Member views consumption dashboard | 3 | Access to place consumption per owner-defined permissions |
| HU-032 | Revoke member access | 3 | Immediate access removal |
| HU-033 | Member receives alerts for the place | 3 | Alert types based on owner-defined permissions |

**Epic total: 14 SP**

### E9 — Bill Management

| ID | User story | SP | Acceptance criteria summary |
|----|-----------|----|-----------------------------|
| HU-034 | Register a water bill | 5 | Amount, period, liters billed, provider |
| HU-035 | Compare bill vs. sensor measurement | 8 | Side-by-side for the same period, difference highlighted |
| HU-036 | View bill history | 3 | List of all registered bills |
| HU-037 | Receive alert for bill discrepancy | 5 | Alert when sensor vs. bill difference exceeds threshold |

**Epic total: 21 SP**

### E10 — Tariff Configuration

| ID | User story | SP | Acceptance criteria summary |
|----|-----------|----|-----------------------------|
| HU-038 | Configure water tariff for a place | 5 | Rate per m³, stratum, provider |
| HU-039 | View estimated cost from sensor data | 5 | Cost calculated from consumption × tariff |
| HU-040 | Tariff updates apply to future estimates | 2 | Historical estimates unchanged |

**Epic total: 12 SP**

### E11 — Automatic Shutoff

| ID | User story | SP | Acceptance criteria summary |
|----|-----------|----|-----------------------------|
| HU-041 | Configure automatic shutoff rule | 8 | Trigger conditions (e.g., leak detected, goal exceeded) |
| HU-042 | Automatic shutoff activates on rule trigger | 5 | Valve closes, alert sent, action logged |
| HU-043 | Disable automatic shutoff | 2 | Rules deactivated, valve stays manual |
| HU-044 | Reopen valve after auto-shutoff (manual only) | 3 | Safety: never auto-reopens |

**Epic total: 18 SP**

### Entrega 3 summary

| Metric | Value |
|--------|-------|
| Epics | 4 (E8–E11) |
| User stories | 15 |
| Story points | ~65 SP |
| Schema changes | place (members, invitations), consumption (bills, tariffs), valve (shutoff_rules) |

---

## Entrega 4 — Administration & Analytics

> **Goal:** An admin panel for the platform, analytics reports, and support for multiple places per user.

### E12 — Admin Panel

| ID | User story | SP | Acceptance criteria summary |
|----|-----------|----|-----------------------------|
| HU-045 | Admin views all registered users | 3 | Paginated list, search by email/name. No sensitive data (no passwords, identity documents, or billing info) |
| HU-046 | Admin blocks/unblocks a user | 3 | User sessions revoked on block |
| HU-047 | Admin views all devices | 3 | Status, linked place, last reading |
| HU-048 | Admin views platform metrics | 5 | Total users, devices, places, active/inactive |
| HU-057 | Admin creates a recommendation | 5 | Title, body, category (saving tip, maintenance, seasonal, general); visible to all users |
| HU-058 | Admin edits or deactivates a recommendation | 3 | Deactivated recommendations hidden from users but not deleted |

**Epic total: 22 SP**

### E13 — Reports & Analytics

| ID | User story | SP | Acceptance criteria summary |
|----|-----------|----|-----------------------------|
| HU-049 | View monthly consumption report | 5 | Summary with charts, comparison to previous months |
| HU-050 | View water savings report | 5 | Savings vs. goal, trend over time |
| HU-051 | Download report as PDF | 5 | Formatted report with charts and data |
| HU-052 | View consumption ranking across places | 3 | Leaderboard for multi-place users |

**Epic total: 18 SP**

### E14 — Multi-place

| ID | User story | SP | Acceptance criteria summary |
|----|-----------|----|-----------------------------|
| HU-053 | Register multiple places | 5 | Each place independent, own device |
| HU-054 | Switch between places in the app | 3 | Place selector in navigation |
| HU-055 | View aggregated consumption across places | 5 | Total and per-place breakdown |
| HU-056 | Manage devices per place | 3 | Link/unlink per place |

**Epic total: 16 SP**

### Entrega 4 summary

| Metric | Value |
|--------|-------|
| Epics | 3 (E12–E14) |
| User stories | 14 |
| Story points | ~56 SP |
| New role | Admin (Administrador) |

---

## Cumulative progress by delivery

| Entrega | Epics | User stories | Story points | Cumulative SP |
|---------|-------|-------------|-------------|---------------|
| Entrega 1 (MVP) | E1–E4 | 17 | ~80 | ~80 |
| Entrega 2 (Smart) | E5–E7 | 12 | ~56 | ~136 |
| Entrega 3 (Social) | E8–E11 | 15 | ~65 | ~201 |
| Entrega 4 (Admin) | E12–E14 | 14 | ~56 | ~257 |
| **Total** | **14 epics** | **58 HUs** | **~257 SP** | |

> **Note:** Story point totals are approximate. Some stories may be split or merged during sprint planning. The original backlog references 61 HUs / 266 SP — the difference accounts for stories that may be consolidated or re-estimated during detailed planning.

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
| Device + Dashboard | E1 | E2 | Goals and alerts need consumption data flowing |
| Alerts | E2 (E6) | E3 (E11) | Auto-shutoff rules trigger alerts |
| Valve control | E2 (E7) | E3 (E11) | Auto-shutoff sends valve commands |
| All data | E1+E2 | E4 (E12–E13) | Admin and reports need all domain data |

---

## Correlations

- Product vision → `03-product/vision.md`
- Problem framing → `03-product/problem-framing.md`
- User stories (detailed) → `04-requirements/user-stories.md`
- Domain events by Entrega → `02-domain/domain-events.md`
- Database tables by Entrega → `06-data/models.md`
- Architecture overview → `05-architecture/overview.md`
