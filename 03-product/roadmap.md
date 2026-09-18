# Delivery Roadmap

> This document maps the 4 deliveries (Entregas) to epics, features, and user stories.
> No fixed dates — deliveries are ordered by priority and executed sequentially.

---

## Delivery overview

```
Entrega 1 (MVP)          Entrega 2 (Smart)         Entrega 3 (Social)       Entrega 4 (Admin)
┌─────────────────┐      ┌──────────────────┐      ┌──────────────────┐      ┌─────────────────┐
│ E1: Auth         │      │ E5: Valve Ctrl    │      │ E9: Guest         │      │ E13: Reports     │
│ E2: Place Mgmt   │      │ E6: Goals         │      │ E10: Collaborator  │      │ E14: Admin Panel │
│ E3: Device Link  │      │ E7: Alerts        │      │ E11: Bills         │      │ E15: Tariffs CO  │
│ E4: Dashboard    │      │ E8: Auto-shutoff  │      │ E12: Tariffs       │      │                  │
│                  │      │                   │      │                    │      │                  │
└─────────────────┘      └──────────────────┘      └──────────────────┘      └─────────────────┘
        ~80 SP                   ~74 SP                    ~69 SP                   ~42 SP
                                                                          Total: ~265 SP
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
| HU-014 | View current consumption (today) | 8 | Real-time liters consumed today, auto-refresh ≥1 min, manual refresh, last update timestamp |
| HU-015 | View consumption history (daily/weekly/monthly) | 8 | Charts with date range selector |
| HU-016 | View current flow rate | 5 | Live liters/minute from last reading |

**Epic total: 21 SP**
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
| HU-019 | Close the valve remotely | 5 | Confirmation code sent, user confirms before MQTT command, warning shown |
| HU-020 | Open the valve remotely | 5 | MQTT command sent, confirmation received, status updated |
| HU-021 | View valve status | 3 | Open/closed, last action timestamp, who triggered it |
| HU-022 | View valve action history | 3 | List of all open/close actions with actor and reason |

**Epic total: 16 SP**
### E6 — Consumption Goals

| ID | User story | SP | Priority | Acceptance criteria summary |
|----|-----------|----|-----------|-----------------------------|
| HU-023 | Set a monthly consumption goal | 5 | Should | Goal in liters, per place |
| HU-024 | View goal progress | 5 | Should | Progress bar, percentage consumed vs. goal |
| HU-025 | Receive alert when approaching goal | 3 | Should | Notification at configurable threshold (e.g., 80%) |
| HU-026 | Receive alert when goal is exceeded | 3 | Should | Notification when 100% reached |

**Epic total: 16 SP**
### E7 — Alerts & Notifications

| ID | User story | SP | Acceptance criteria summary |
|----|-----------|----|-----------------------------|
| HU-027 | Receive leak detection alert | 8 | Alert when continuous flow detected during inactivity hours |
| HU-028 | View alert history | 3 | List of all alerts with timestamp, type, status (read/unread) |
| HU-029 | Configure notification preferences | 5 | Push, email, WhatsApp toggle per alert type |

**Epic total: 16 SP**
### E8 — Automatic Valve Shutoff

| ID | User story | SP | Acceptance criteria summary |
|----|-----------|----|-----------------------------|
| HU-030 | Configure automatic shutoff rule | 5 | Trigger: LeakSuspected only; requires confirmation before closing |
| HU-031 | Automatic shutoff activates on rule trigger | 8 | Confirmation request sent, user confirms or timeout, valve closes |
| HU-032 | Disable automatic shutoff | 2 | Rules deactivated, valve stays manual |
| HU-033 | Reopen valve after auto-shutoff (manual only) | 3 | Safety: never auto-reopens |

**Epic total: 18 SP**

### Entrega 2 summary

| Metric | Value |
|--------|-------|
| Epics | 4 (E5–E8) |
| User stories | 16 |
| Story points | ~74 SP |
| New services involved | valve-service, notification-service |
| Schemas activated | valve, goals, alerts, notifications |

---

## Entrega 3 — Social & Billing

> **Goal:** Owners invite guests and collaborators, manage utility bills, and configure tariffs.

### E9 — Guest (Anonymous access)

| ID | User story | SP | Priority | Acceptance criteria summary |
|----|-----------|----|---------|-----------------------------|
| HU-034 | Generate guest access link | 5 | Should | Owner creates shareable link, can disable it |
| HU-035 | View consumption as guest (anonymous) | 3 | Should | Access without registration, read-only |
| HU-036 | Read-only access restriction | 5 | Must | Validated in backend, restricted actions hidden |

**Epic total: 13 SP**

### E10 — Collaborator (Registered access with permissions)

| ID | User story | SP | Priority | Acceptance criteria summary |
|----|-----------|----|-----------------------------|
| HU-037 | Invite collaborator by email | 5 | Should | Requires registration, Owner defines permissions, 7-day expiry |
| HU-038 | Accept invitation | 3 | Should | Signup if needed, auto-accept on dashboard |
| HU-039 | Define collaborator permissions | 5 | Should | Per collaborator, per place, editable after |
| HU-040 | Collaborator executes per permissions | 5 | Must | Backend validation, restricted actions hidden |
| HU-041 | Revoke collaborator access | 3 | Should | Immediate cutoff, even if logged in, audit log |
| HU-042 | Collaborator receives alerts | 3 | Could | Per permissions, preferred channel |

**Epic total: 24 SP**

### E11 — Utility Bill Management

| ID | User story | SP | Acceptance criteria summary |
|----|-----------|----|-----------------------------|
| HU-043 | Register a water bill | 5 | Amount, period, liters billed, provider |
| HU-044 | Compare bill vs. sensor measurement | 8 | Side-by-side for the same period, difference highlighted |
| HU-045 | View bill history | 3 | List of all registered bills |
| HU-046 | Receive alert for bill discrepancy | 5 | Alert when sensor vs. bill difference exceeds threshold |
| HU-047 | Bill payment reminder | 3 | Configurable days before due date (default 3), important alert; mark as paid to stop reminder |

**Epic total: 24 SP**

### E12 — Tariffs & Measurement Units

| ID | User story | SP | Acceptance criteria summary |
|----|-----------|----|-----------------------------|
| HU-048 | Configure water tariff for a place | 5 | Rate per m³, stratum, provider |
| HU-049 | View estimated cost from sensor data | 5 | Cost calculated from consumption × tariff |
| HU-050 | Tariff updates apply to future estimates | 2 | Historical estimates unchanged |

**Epic total: 12 SP**

### Entrega 3 summary

| Metric | Value |
|--------|-------|
| Epics | 4 (E9–E12) |
| User stories | 18 |
| Story points | ~73 SP |
| Schema changes | billing (bills, tariffs), places (members, invitations, permissions) |

---

## Entrega 4 — Administration & Analytics

> **Goal:** Downloadable reports, admin panel (with savings tips), and pre-loaded tariffs for Colombia.

### E13 — Downloadable Reports

| ID | User story | SP | Acceptance criteria summary |
|----|-----------|----|-----------------------------|
| HU-051 | View monthly consumption report | 5 | Summary with charts, comparison to previous months |
| HU-052 | Download report as PDF | 5 | Formatted report with charts and data |

**Epic total: 10 SP**

### E14 — Admin Panel (includes Savings Tips)

| ID | User story | SP | Acceptance criteria summary |
|----|-----------|----|-----------------------------|
| HU-053 | Admin views all registered users | 3 | Paginated list, search by email/name. No sensitive data |
| HU-054 | Admin blocks/unblocks a user | 3 | User sessions revoked on block |
| HU-055 | Admin views all devices | 3 | Status, linked place, last reading |
| HU-056 | Admin views platform metrics | 5 | Total users, devices, places, active/inactive |
| HU-057 | Admin creates a recommendation | 5 | Title, body, category (saving tip, maintenance, seasonal); visible to all users |
| HU-058 | Admin edits or deactivates recommendation | 3 | Deactivated recommendations hidden but not deleted |

**Epic total: 22 SP**

### E15 — Preloaded Tariffs (Colombia only)

| ID | User story | SP | Priority | Acceptance criteria summary |
|----|-----------|----|---------|-----------------------------|
| HU-059 | Colombia tariff catalog | 8 | Could | Select city and stratum (1–6); show consumption tiers (basic/additional/luxury); display update date; allow override |
| HU-060 | Auto-calculation with preloaded tariff | 5 | Could | Use preloaded tariff by default if available; show tariff source; historical calcs unchanged on update |

**Note:** Ecuador and USA documented as Won't (deferred).

**Epic total: 13 SP**

### Entrega 4 summary

| Metric | Value |
|--------|-------|
| Epics | 3 (E13–E15) |
| User stories | 11 |
| Story points | ~45 SP |
| New role | Admin (Administrator) |

---

## Cumulative progress by delivery

| Entrega | Epics | User stories | Story points | Cumulative SP |
|---------|-------|-------------|-------------|---------------|
| Entrega 1 (MVP) | E1–E4 | 13 | ~80 | ~80 |
| Entrega 2 (Smart) | E5–E8 | 16 | ~74 | ~154 |
| Entrega 3 (Social) | E9–E12 | 18 | ~73 | ~227 |
| Entrega 4 (Admin) | E13–E15 | 11 | ~45 | ~272 |
| **Total** | **15 epics** | **58 HUs** | **~272 SP** | |

> **Changes in this release:**
> - Restructured to 15 epics (E1–E15) matching Product Backlog
> - Moved Automatic Shutoff (E8) from Entrega 3 to Entrega 2
> - Eliminated multi-place as separate epic (HU-054 now covered by E2 place selector)
> - Removed HU-018 (home screen summary — redundant with HU-014 period selector)
> - Removed HU-026 (mark alert resolved — only read/unread states)
> - Moved HU-017 (CSV export) from Entrega 1 to Entrega 2
> - Added auto-refresh criteria to HU-014 (≥1 min, manual refresh, timestamp)
> - Changed all Goal HUs (E6) from Must to Should priority
> - Added E9 (Guest) and E10 (Collaborator) with invitations and permissions
> - Updated E11 (Bills) and E12 (Tariffs) with corrected HU numbers and payment reminder
> - E8 (Shutoff) now triggers on LeakSuspected only (removed GoalExceeded trigger)
> - Consolidated E14 (Admin Panel) with Savings Tips
> - Added E15 (Colombia preloaded tariffs only; Ecuador/USA deferred as Won't)

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
