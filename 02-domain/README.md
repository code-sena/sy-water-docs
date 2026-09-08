# 02 — Problem Domain

> **What is this?** The mental model of the business. It is not technology — it is understanding
> the problem the system solves before writing code.

---

## Documents in this section

| File | Purpose | Status |
|------|---------|--------|
| [domain-map.md](./domain-map.md) | Bounded contexts, context map, relationships, and strategic classification (Core/Supporting/Generic) | Complete |
| [entities-and-rules.md](./entities-and-rules.md) | Entities, value objects, aggregates, invariants, and domain services | Complete |
| [domain-events.md](./domain-events.md) | Event catalog with payloads, consumers, topics, policies, and resilience patterns | Complete |

---

## Bounded Contexts Summary

| Context | Type | Microservice | Entrega |
|---------|------|-------------|---------|
| IAM | Generic | auth-service | MVP |
| Place Management | Supporting | place-service | MVP |
| Device Management | Core | device-service | MVP |
| Consumption | Core | consumption-service | MVP |
| Valve Control | Core | valve-service | 2 |
| Notification | Supporting | notification-service | 2 |

---

## Correlations with other sections

| This section feeds... | Why |
|-----------------------|-----|
| `05-architecture/overview.md` | Bounded contexts → microservices |
| `06-data/models.md` | Entities → database tables |
| `07-api/contracts/` | Domain events → async API contracts |
| `09-microservices/service-catalog.md` | Context map → service registry |
| `04-requirements/` | Business rules → acceptance criteria |

---

## Recommended reading order

1. `domain-map.md` — understand the contexts and how they relate
2. `entities-and-rules.md` — learn the entities, their rules, and aggregates
3. `domain-events.md` — see how contexts communicate asynchronously
