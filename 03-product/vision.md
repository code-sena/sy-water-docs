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
| User (base) | Newly registered person | Can see places, create new ones, or accept invitations as collaborator. No access to consumption until linked to a place | MVP |
| Owner (Dueño) | Manages a place; pays the bill | Complete control: consumption, valve, goals, bills, tariffs, alerts, invitations. All permissions. | MVP |
| Guest (Invitado) | Anonymous visitor via shared link | Read-only consumption and charts. No registration required. No invitations or actions. | Entrega 3 |
| Collaborator (Colaborador) | Registered user invited by Owner | Permissions defined by Owner per place: partial actions (view/control/alerts). Revocable. | Entrega 3 |
| Admin (Administrador) | Platform operator (university team) | Views users (no sensitive data), blocks/unblocks accounts, monitors platform metrics, creates/edits recommendations | Entrega 4 |

---

## High-level roadmap

```
Entrega 1 ──── Entrega 2 ──── Entrega 3 ──── Entrega 4
(MVP)          (Smart)        (Social)       (Admin)
    │               │              │              │
 Auth +         Valve ctrl +  Guest +        Reports +
 Places +       Goals +       Collaborator + Admin panel +
 Device +       Alerts +      Bills +        Preloaded
 Dashboard      Auto-shutoff  Tariffs        tariffs
```

| Entrega | Objective | Epics | HUs | Story points |
|---------|----------|-------|-----|-------------|
| **Entrega 1 (MVP)** | A user registers, links a device, and sees consumption in real time | E1 (Cuentas y acceso), E2 (Gestión de lugares), E3 (Vinculación IoT), E4 (Monitoreo) | 18 | 73 SP |
| **Entrega 2** | Smart monitoring with valve control, goals, alerts, and auto-shutoff | E5 (Control válvula), E6 (Metas), E7 (Alertas), E8 (Cierre automático) + HU-038 | 20 | 88 SP |
| **Entrega 3** | Shared access, billing, and tariffs | E9 (Invitado), E10 (Colaborador), E11 (Recibos), E12 (Tarifas) | 18 | 70 SP |
| **Entrega 4** | Reports, admin panel with savings tips, and preloaded tariffs | E13 (Reportes), E14 (Admin Panel, includes savings tips), E15 (Tarifas precargadas) | 13 | 48 SP |

> **Totals:** 15 epics (E1–E15), 69 HUs, 279 SP (266 active without Won't).

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
