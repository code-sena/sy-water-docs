# System Architecture Overview

## 1. Adopted architectural style

**Style:** Microservices + Event-Driven

**Justification:** The system has naturally distinct bounded contexts (IAM, devices, consumption, valve control, notifications) that evolve at different rates and have different scaling needs. Event-driven communication decouples services and enables reactive features like leak detection and auto-shutoff.

**Reference:** `02-domain/domain-map.md` for bounded context decomposition.

---

## 2. C4 Diagram — System Level (Context)

```
  ┌────────────┐     ┌────────────┐     ┌──────────────────┐
  │  Mobile App│     │  Web App   │     │  ESP32 Devices   │
  │(React Native)    │(RN Web Ext)│     │(Sensor + Valve)  │
  └─────┬──────┘     └─────┬──────┘     └────────┬─────────┘
        │                  │                      │
        │     HTTPS        │     HTTPS            │ MQTT
        └──────────┬───────┘                      │
                   ▼                              ▼
        ┌─────────────────┐            ┌─────────────────┐
        │   API Gateway   │            │   MQTT Broker   │
        │ (Spring Cloud   │            │  (Mosquitto)    │
        │   Gateway)      │            └────────┬────────┘
        └────────┬────────┘                     │
                 │                              │
        ┌────────┴──────────────────────────────┴──────┐
        │           Save Your Water Platform           │
        │                                              │
        │  auth · place · device · consumption         │
        │  valve · notification                        │
        └──────────────────┬───────────────────────────┘
                           │
              ┌────────────┼────────────────┐
              ▼            ▼                ▼
     ┌──────────────┐ ┌──────────┐ ┌──────────────┐
     │  SQL Server  │ │ RabbitMQ │ │External APIs  │
     │ (sy_water_db)│ │ (Events) │ │(Email, Push,  │
     │              │ │          │ │ Google OAuth)  │
     └──────────────┘ └──────────┘ └──────────────┘
```

---

## 3. C4 Diagram — Container Level

```
                        ┌──────────────────────────────────┐
    Mobile / Web ──────▶│  API Gateway (Spring Cloud GW)   │
                        │  :8080                           │
                        │  JWT validation · routing · CORS │
                        └──┬────┬────┬────┬────┬───────────┘
                           │    │    │    │    │
          ┌────────────────┘    │    │    │    └──────────────────┐
          ▼                     ▼    │    ▼                      ▼
  ┌──────────────┐   ┌──────────┐   │  ┌────────────┐   ┌─────────────┐
  │ auth-service │   │  place-  │   │  │   valve-   │   │notification-│
  │ (Spring Boot)│   │ service  │   │  │  service   │   │   service   │
  │   :3001      │   │  :3002   │   │  │   :3005    │   │    :3006    │
  │              │   │          │   │  │            │   │             │
  │ Schema: iam  │   │Schema:   │   │  │Schema:     │   │Schema:      │
  │              │   │ place    │   │  │ valve      │   │ notification│
  └──────────────┘   └──────────┘   │  └─────┬──────┘   └─────────────┘
                                    │        │
                                    ▼        │ REST (valve commands)
                           ┌────────────┐    │
                           │consumption-│    │
                           │  service   │◀───┘
                           │   :3004    │
                           │            │
                           │Schema:     │
                           │consumption │
                           └────────────┘
                                    ▲
                                    │ Events (ReadingReceived)
                           ┌────────┴───┐
                           │device-     │◀──── MQTT Broker ◀──── ESP32
                           │ service    │        (Mosquitto)
                           │  :3003     │
                           │            │
                           │Schema:     │
                           │ device     │
                           └────────────┘

  All services ──events──▶ RabbitMQ ──events──▶ All services
  All services ──SQL──▶ SQL Server (sy_water_db) — one instance, schema-per-service
```

---

## 4. Service catalog

| # | Service | Responsibility | Port | Technology | Schema | Communication |
|---|---------|---------------|------|-----------|--------|---------------|
| 01 | `api-gateway` | Routing, JWT validation, CORS, rate limiting | 8080 | Spring Cloud Gateway | — | HTTP Proxy |
| 02 | `auth-service` | Registration, login, JWT tokens, Google OAuth, RBAC | 3001 | Spring Boot | `iam` | REST + Events |
| 03 | `place-service` | Place CRUD, switching, invitations, shared access | 3002 | TBD (.NET eval) | `place` | REST + Events |
| 04 | `device-service` | Device linking, MQTT telemetry ingestion, device status | 3003 | TBD (.NET eval) | `device` | MQTT + REST + Events |
| 05 | `consumption-service` | Consumption aggregation, charts, goals, bill comparison | 3004 | TBD (.NET eval) | `consumption` | REST + Events |
| 06 | `valve-service` | Valve state, open/close commands, auto-shutoff rules | 3005 | TBD (.NET eval) | `valve` | REST + Events |
| 07 | `notification-service` | Alert engine, multi-channel delivery (in-app, push, email) | 3006 | TBD (.NET eval) | `notification` | REST + Events |

> Full detail per service in `09-microservices/service-catalog.md`

---

## 5. Architectural principles

### P1: API-First
Design the API contract (OpenAPI) before implementing the service. Contracts are the source of truth for consumers.

### P2: Shared Database, Schema Isolation
All services share a single SQL Server instance (`sy_water_db`). Each service owns its schema and must NOT query another service's schema directly. Inter-service data access is always through REST APIs or domain events via RabbitMQ.

### P3: Fail Fast, Recover Gracefully
Detect errors early with input validation at the API Gateway. When an external service or the MQTT broker is unavailable, log the failure and respond with a clear error — adopt resilience patterns (Circuit Breaker, Retry) when complexity justifies it.

### P4: Events for Decoupling
Use RabbitMQ domain events for all operations that do not require an immediate response. Synchronous REST calls are reserved for queries and operations the user is waiting for.

### P5: Start Simple, Evolve When Needed
Do not adopt complex patterns (CQRS, Event Sourcing, Saga) upfront. Add them when a concrete problem demands it, and document the decision as an ADR.

### P6: Hexagonal Architecture per Service
Each service follows the Ports & Adapters pattern so the domain logic is independent of frameworks and infrastructure. See `05-architecture/hexagonal-architecture.md`.

---

## 6. Adopted architectural patterns

| Pattern | Adopted? | Justification |
|---------|---------|---------------|
| API Gateway | Yes | Single entry point, JWT validation, routing — Spring Cloud Gateway |
| Schema-per-Service (shared DB) | Yes | Team of 3, single VPS — schema isolation preserves logical boundaries. See ADR-002. |
| Hexagonal Architecture | Yes | Domain independence from frameworks; testability |
| Event-Driven (RabbitMQ) | Yes | Decouple services for async operations (readings, alerts, notifications) |
| Circuit Breaker | Not yet | Will evaluate when inter-service failures become a real concern |
| Saga | Not yet | Will evaluate when distributed transactions are needed (e.g., device linking flow) |
| Outbox Pattern | Not yet | Will evaluate when event delivery reliability becomes critical |
| CQRS | No | Read and write models are similar enough; unnecessary complexity |
| Event Sourcing | No | No audit/replay requirement justifies the complexity |
| BFF | No | Single React Native codebase for mobile and web — one API format suffices |

---

## 7. Communication patterns

### Synchronous (REST)

| Source | Destination | Purpose |
|--------|------------|---------|
| api-gateway | auth-service | JWT token validation on every request |
| api-gateway | all services | Route requests by path prefix |
| valve-service | device-service | Send open/close command to ESP32 via device-service |
| Any service | auth-service | Verify user permissions (when needed beyond JWT claims) |

### Asynchronous (RabbitMQ — domain events)

| Source | Event | Consumers | Purpose |
|--------|-------|-----------|---------|
| auth-service | `UserRegistered` | notification-service | Send verification email |
| place-service | `PlaceCreated` | consumption-service | Provision consumption storage |
| place-service | `PlaceDeleted` | consumption, notification | Archive data, cancel alerts |
| device-service | `ReadingReceived` | consumption, notification | Aggregate readings, feed leak detection |
| device-service | `DeviceDisconnected` | notification-service | Sensor-down alert |
| consumption-service | `GoalExceeded` | notification, valve | Alert + auto-shutoff evaluation |
| consumption-service | `LeakSuspected` | notification, valve | Critical alert + auto-shutoff |
| valve-service | `ValveClosed` | notification-service | Critical alert on all channels |

### IoT (MQTT — Mosquitto)

| Direction | Topic pattern | Purpose |
|-----------|--------------|---------|
| ESP32 → device-service | `devices/{hardwareId}/telemetry` | Flow sensor readings |
| device-service → ESP32 | `devices/{hardwareId}/commands` | Valve open/close commands |
| ESP32 → device-service | `devices/{hardwareId}/status` | Device heartbeat / connectivity |

---

## 8. Cross-cutting concerns

| Concern | Solution | Where configured |
|---------|---------|-----------------|
| Authentication | JWT (issued by auth-service, validated at API Gateway) | api-gateway config |
| Authorization | Role-based (Owner/Guest/Admin) in JWT claims | Each service validates role |
| Logging | Structured JSON with correlation ID | Shared logging config per service |
| Health Checks | `GET /health` (liveness) + `GET /health/ready` (readiness) | Each service |
| Error format | Standard error response (code, message, details) | `07-api/contracts/openapi/_shared.yaml` |
| Rate Limiting | At the API Gateway level | Spring Cloud Gateway config |
| CORS | Configured at API Gateway for React Native web | api-gateway config |

---

## 9. Database strategy

**Single instance:** SQL Server — `sy_water_db`

**Schema isolation:** Each service owns its schema. Tables are prefixed by schema name.

| Schema | Owner service | Main tables |
|--------|-------------|-------------|
| `iam` | auth-service | users, sessions, verification_tokens |
| `place` | place-service | places, members, invitations |
| `device` | device-service | devices, readings |
| `consumption` | consumption-service | consumption_records, goals, bills, tariffs |
| `valve` | valve-service | valves, valve_actions, shutoff_rules |
| `notification` | notification-service | alerts, notification_preferences |

**Rules:**
- A service may only read/write tables in its own schema
- Cross-service data access is via REST API or RabbitMQ events
- Foreign keys across schemas are NOT allowed — use application-level references (UUIDs)
- Migrations are managed per service, scoped to its schema

> Detailed data model in `06-data/models.md`

---

## 10. Registered architectural technical debt

| ID | Description | Impact | Priority | Target |
|----|-------------|--------|---------|--------|
| AT-001 | No Circuit Breaker on inter-service calls | Medium — cascade failures possible | P3 | Evaluate after MVP |
| AT-002 | No Outbox Pattern for event publishing | Medium — events could be lost on crash | P3 | Evaluate after MVP |
| AT-003 | No distributed tracing (OpenTelemetry) | Low — debugging across services is harder | P3 | Entrega 2 |

---

## 11. Planned evolution

| Phase | Architectural change | Motivation | Target |
|-------|---------------------|------------|--------|
| Post-MVP | Add Circuit Breaker on device-service calls | MQTT/device communication is the most failure-prone path | Entrega 2 |
| Post-MVP | Add distributed tracing | As service count grows, debugging cross-service flows becomes critical | Entrega 2 |
| If needed | Split shared DB into separate instances | If query contention becomes a bottleneck | When load demands it |
| If needed | Add CQRS for consumption reads | If read volume vastly exceeds writes (many users refreshing dashboards) | When load demands it |

---

## Key correlations

- Domain bounded contexts → `02-domain/domain-map.md`
- ADRs → `05-architecture/decisions/`
- Hexagonal architecture → `05-architecture/hexagonal-architecture.md`
- Pattern catalog → `05-architecture/pattern-guide.md`
- Per-service detail → `09-microservices/service-catalog.md`
- Data model → `06-data/models.md`
- API contracts → `07-api/contracts/openapi/`
