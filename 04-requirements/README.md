# 04 — Requirements

> **What is this?** The formal contract between product, engineering, and stakeholders
> about what Save Your Water must do (functional) and how well it must do it (non-functional).

---

## Documents in this section

| File | Purpose | Status |
|------|---------|--------|
| [user-stories.md](./user-stories.md) | Complete backlog: 56 user stories across 14 epics and 4 Entregas, with Gherkin acceptance criteria | Complete |
| [non-functional.md](./non-functional.md) | 8 NFRs: performance, availability, scalability, security, IoT reliability, observability, maintainability, deployment | Complete |
| [traceability-matrix.md](./traceability-matrix.md) | 19 functional requirements mapped to 56 HUs, services, test files, and delivery | Complete |

---

## Requirements at a glance

| Metric | Value |
|--------|-------|
| Functional requirements | 19 (FR-001 to FR-019) |
| Non-functional requirements | 8 (NFR-001 to NFR-008) |
| User stories | 56 (HU-01 to HU-56) |
| Epics | 14 (E1 to E14) |
| Planned test files | 32 |
| Services involved | 7 (api-gateway + 6 domain services) |

---

## NFR priorities

| Priority | NFRs |
|----------|------|
| P1 (Critical) | Performance, Security, IoT Reliability, Deployment |
| P2 (Important) | Availability, Maintainability |
| P3 (Aspirational) | Observability, Scalability |

---

## Correlations

| This section is fed by... | And feeds into... |
|---------------------------|-------------------|
| `03-product/roadmap.md` → epics and deliveries | HU organization by Entrega |
| `03-product/problem-framing.md` → problem and users | FR definitions |
| `02-domain/domain-events.md` → events | Alert and IoT requirement details |
| `user-stories.md` → HUs | `07-api/` → API contracts and endpoints |
| `user-stories.md` → HUs | `09-microservices/` → service-level requirements |
| `non-functional.md` → NFRs | `05-architecture/overview.md` → architectural decisions |
| `traceability-matrix.md` → test files | `11-quality/testing-strategy.md` → test plan |
