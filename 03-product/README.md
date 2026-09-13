# 03 — Product Definition

> **What is this?** The answer to "what are we going to build and why?".
> Not "how" — that comes in architecture. Here we define the validated problem,
> the product vision, and the delivery plan.

---

## Documents in this section

| File | Purpose | Status |
|------|---------|--------|
| [problem-framing.md](./problem-framing.md) | The problem we solve, affected users, JTBD, solution hypothesis, risks | Complete |
| [vision.md](./vision.md) | Product vision statement, strategic pillars, user roles, roadmap overview, product principles | Complete |
| [roadmap.md](./roadmap.md) | Delivery plan: 4 Entregas mapped to 14 epics and 56 user stories, dependencies | Complete |

---

## Product at a glance

| Attribute | Value |
|-----------|-------|
| Product name | Save Your Water |
| Problem | No visibility into water consumption until the monthly bill arrives |
| Solution | IoT device (ESP32 + YF-S201 + electrovalve) + mobile/web platform |
| Users | User (default authenticated), Owner (head of household), Member (family/roommate with owner-defined permissions), Admin (platform operator) |
| Deliveries | 4 Entregas (no fixed dates, ordered by priority) |
| Epics | 14 |
| User stories | 58 |
| Total story points | ~257 SP |

---

## Delivery summary

| Entrega | Theme | Epics | HUs | SP |
|---------|-------|-------|-----|-----|
| 1 — MVP | Auth + Device + Dashboard | E1–E4 | 17 | ~80 |
| 2 — Smart | Goals + Alerts + Valve | E5–E7 | 12 | ~56 |
| 3 — Social | Members + Bills + Auto-shutoff | E8–E11 | 15 | ~65 |
| 4 — Admin | Admin panel + Recommendations + Reports + Multi-place | E12–E14 | 14 | ~56 |

---

## Correlations

| This section is fed by... | And feeds into... |
|---------------------------|-------------------|
| Team's own experience with water bills | Problem framing |
| Hardware prototyping (ESP32 + YF-S201) | Solution hypothesis |
| `problem-framing.md` → problem + users | `02-domain/domain-map.md` → bounded contexts |
| `vision.md` → user roles | `02-domain/entities-and-rules.md` → User entity |
| `roadmap.md` → epics and HUs | `04-requirements/user-stories.md` → detailed requirements |
| `roadmap.md` → Entrega mapping | `06-data/models.md` → tables by Entrega |
| `roadmap.md` → dependencies | `05-architecture/overview.md` → service interactions |

---

## Questions this section answers

- What problem exactly are we solving?
- What does product success look like?
- Who are the users and what do they need?
- What do we build first and why?
- What do we NOT build?
