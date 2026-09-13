# 05 — Architecture

> **What is this?** The system's design decisions: how it is organized, why,
> what alternatives were evaluated, and how it is deployed.

---

## Documents in this section

| File | Purpose | Status |
|------|---------|--------|
| [overview.md](./overview.md) | C4 diagrams, service catalog, communication patterns, cross-cutting concerns, DB strategy | Complete |
| [hexagonal-architecture.md](./hexagonal-architecture.md) | Ports & Adapters pattern guide — folder structure, ports, adapters, dependency rule, TDD | Reference (generic) |
| [pattern-guide.md](./pattern-guide.md) | Design patterns (GoF) and microservices patterns catalog with project adoption status | Updated |
| [decisions/](./decisions/) | Architecture Decision Records (ADRs) | Living — add ADRs as decisions are made |

---

## Architecture Decision Records (ADRs)

| ADR | Decision | Status |
|-----|---------|--------|
| [ADR-001](./decisions/records/ADR-001-idioma-documentacion.md) | All documentation and code in English | Accepted |
| [ADR-002](./decisions/records/ADR-002-shared-database-schema-per-service.md) | Shared SQL Server instance with schema-per-service isolation | Accepted |

---

## Key architectural decisions summary

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Architectural style | Microservices + Event-Driven | Natural bounded contexts with different scaling needs |
| API Gateway | Spring Cloud Gateway | Integrates with Spring Boot auth-service |
| Backend (IAM) | Spring Boot | Mature security ecosystem |
| Backend (other services) | .NET (under evaluation) | Team expertise |
| Database | SQL Server — single instance, schema-per-service | Team of 3, single VPS; ADR-002 |
| Domain event broker | RabbitMQ | Reliable pub/sub for inter-service events |
| IoT protocol | MQTT (Mosquitto) | Lightweight, optimized for ESP32 devices |
| Frontend | React Native (mobile + web extension) | Single codebase for all clients |
| Internal architecture | Hexagonal (Ports & Adapters) | Domain independence, testability |

---

## Correlations

| This section is fed by... | And feeds... |
|--------------------------|-------------|
| `02-domain/domain-map.md` → 11 bounded contexts | `overview.md` → BC-to-service mapping (11 BCs → 6 services) |
| Domain events in `02-domain/` | Communication patterns in `overview.md` |
| ADRs here | Implementation in `09-microservices/` |
| DB strategy in `overview.md` | Data models in `06-data/models.md` |
