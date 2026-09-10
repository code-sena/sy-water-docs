# Product Vision

> The vision of Save Your Water is to give every head of household and business owner
> real-time control over their water consumption — transforming a monthly surprise bill
> into an informed, daily decision.

---

## Vision statement

**For** heads of household and business owners
**who** pay a water bill but have no visibility into their consumption until it arrives,
**the** Save Your Water platform
**is a** IoT-powered water monitoring solution (mobile + web)
**that** provides real-time consumption tracking, leak detection, remote valve control, and bill comparison,
**unlike** waiting for the monthly bill or manually reading the meter,
**our product** gives users live data and actionable alerts on their phone, so they can save water, detect problems early, and verify what the utility charges.

---

## Team mission

We build an affordable and accessible tool that turns water consumption from an invisible, uncontrollable expense into a visible, manageable resource — using low-cost IoT hardware and a mobile-first platform.

---

## Strategic pillars

| Pillar | Description | How we measure it |
|--------|-------------|-------------------|
| Visibility | Users see their consumption in real time, not after 30 days | Dashboard usage; users check consumption at least 3x/week |
| Prevention | Leaks and abnormal consumption are detected before they cause damage | Alert response rate > 80%; leak detection true positive > 70% |
| Control | Users can set goals, receive alerts, and shut off water remotely | % of users with active goals; valve commands per month |
| Verification | Users can compare their sensor data against the utility bill | % of users who register at least one bill for comparison |

---

## User roles

| Role | Who they are | What they do | Entrega |
|------|-------------|-------------|---------|
| Owner (Dueño) | The person who manages the place and pays the bill | Registers, links devices, monitors consumption, controls the valve, sets goals, compares bills | MVP |
| Guest (Invitado) | A family member, roommate, or employee invited by the Owner | Views consumption dashboard (read-only); receives alerts | Entrega 3 |
| Admin (Administrador) | Platform operator (the development team during university phase) | Manages users, views platform-wide metrics | Entrega 4 |

---

## High-level roadmap

```
Entrega 1 ──── Entrega 2 ──── Entrega 3 ──── Entrega 4
(MVP)          (Smart)        (Social)       (Admin)
    │               │              │              │
 Auth +         Goals +       Guests +       Admin panel +
 Device +       Alerts +      Bill mgmt +    Reports +
 Dashboard      Valve ctrl    Auto-shutoff   Multi-place
```

| Entrega | Objective | Epics | Story points |
|---------|----------|-------|-------------|
| **Entrega 1 (MVP)** | A user registers, links a device, and sees consumption in real time | E1 (Auth), E2 (Place), E3 (Device), E4 (Consumption Dashboard) | ~80 SP |
| **Entrega 2** | Smart monitoring with goals, alerts, and manual valve control | E5 (Goals), E6 (Alerts), E7 (Valve Control) | ~70 SP |
| **Entrega 3** | Social features and bill management | E8 (Guests), E9 (Bill Management), E10 (Tariffs), E11 (Auto-shutoff) | ~66 SP |
| **Entrega 4** | Administration, analytics, and multi-place | E12 (Admin Panel), E13 (Reports), E14 (Multi-place) | ~50 SP |

> **Note:** No fixed dates. Deliveries are ordered by priority. This is a university project; the team advances through each Entrega sequentially.

---

## Product principles

1. **Real data, not estimates** — every number comes from the sensor; we never guess.
2. **Alerts must be actionable** — if the user can't do anything about it, don't send the alert.
3. **Safe by default** — auto-shutoff is always opt-in; reopening the valve is always manual; no one gets locked out of their water accidentally.
4. **Mobile first, web too** — the primary experience is on the phone (React Native); the web extension ensures desktop access from the same codebase.
5. **Affordable hardware** — the ESP32 + YF-S201 + electrovalve stack must remain under a reasonable consumer budget.
6. **Incremental value** — each Entrega delivers a usable product, not a half-finished feature.

---

## Definition of Done (product level)

> The product is successful when a user can install the device, see their consumption, and take action — all from their phone.

| Key result | How to measure | Target |
|------------|---------------|--------|
| Device successfully linked and reporting | % of registered users with an active device | > 90% |
| User monitors consumption regularly | Dashboard visits per week per active user | >= 3 |
| Alerts are acknowledged | % of critical alerts acted on within 1 hour | > 80% |
| Bill comparison is used | % of users with 2+ months who register a bill | > 50% |
| Device stays connected | Uptime % of connected devices | > 95% |

> **Note:** As a university project, these are aspirational product design targets. Validation depends on scope of user testing.

---

## Correlations

- Problem framing (the why) → `03-product/problem-framing.md`
- Delivery detail → `03-product/roadmap.md`
- Backlog with all user stories → `04-requirements/user-stories.md`
- Domain bounded contexts → `02-domain/domain-map.md`
- Architecture overview → `05-architecture/overview.md`
