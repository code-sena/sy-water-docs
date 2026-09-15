# System Architecture Overview

## 1. Adopted architectural style

**Style:** Microservices + Event-Driven

**Justification:** The system has 11 bounded contexts organized in 4 layers (Core, Supporting, Generic) that evolve at different rates and have different scaling needs. Event-driven communication decouples services and enables reactive features like leak detection and auto-shutoff.

**Reference:** `02-domain/domain-map.md` for bounded context decomposition and the strategic context map.

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
        │  (10 schemas: security, places, devices,     │
        │   monitoring, goals, billing, valve,         │
        │   alerts, notifications, content)            │
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
  │Schema:       │   │Schema:   │   │  │Schema:     │   │Schemas:     │
  │ security     │   │ places   │   │  │ valve      │   │ alerts,     │
  │ BC-01        │   │ BC-02    │   │  │ BC-07      │   │ notifications│
  │              │   │          │   │  │            │   │ content     │
  │              │   │          │   │  │            │   │ BC-08,09,11 │
  └──────────────┘   └──────────┘   │  └─────┬──────┘   └─────────────┘
                                    │        │
                                    ▼        │ REST (valve commands)
                           ┌────────────┐    │
                           │consumption-│    │
                           │  service   │◀───┘
                           │   :3004    │
                           │            │
                           │Schemas:    │
                           │monitoring, │
                           │goals,      │
                           │billing     │
                           │BC-04,05,   │
                           │  06,10     │
                           └────────────┘
                                    ▲
                                    │ Events (ReadingReceived)
                           ┌────────┴───┐
                           │device-     │◀──── MQTT Broker ◀──── ESP32
                           │ service    │        (Mosquitto)
                           │  :3003     │
                           │            │
                           │Schema:     │
                           │ devices    │
                           │ BC-03      │
                           └────────────┘

  All services ──events──▶ RabbitMQ ──events──▶ All services
  All services ──SQL──▶ SQL Server (sy_water_db) — one instance, schema-per-service
```

---

## 4. Bounded Context to Service mapping

The system defines 11 bounded contexts (see `02-domain/domain-map.md`) but deploys them across 6 microservices. A team of 3 cannot operate 11 independent services — co-deploying BCs that share data gravity reduces operational cost without sacrificing domain clarity.

| Service | BCs deployed | Justification |
|---------|-------------|---------------|
| `auth-service` | BC-01 IAM | Isolated security boundary — only service managing credentials and tokens |
| `place-service` | BC-02 Place Management | Owns the place aggregate; member and invitation logic is tightly coupled to place |
| `device-service` | BC-03 Device Management | Owns MQTT ingestion and device lifecycle; hardware-facing boundary |
| `consumption-service` | BC-04 Consumption Monitoring, BC-05 Consumption Goals, BC-06 Billing & Tariffs, BC-10 Reports & Export | All share the consumption data gravity center — goals, billing, and reports all read from consumption_records. Splitting would create excessive REST chattiness |
| `valve-service` | BC-07 Valve Control | Owns MQTT commands to the electrovalve; latency-critical path must be isolated |
| `notification-service` | BC-08 Alerts & Detection, BC-09 Notification Delivery, BC-11 Savings Tips | Alert rules decide WHAT to notify, delivery decides HOW, tips is read-only content — all three operate on the same notification_preferences and alert tables |

### BC-to-Service visual mapping

```
┌─────────────────────────────────────────────────────────────┐
│                    6 DEPLOYED SERVICES                       │
├──────────────┬───────────────┬──────────────────────────────┤
│auth-service  │place-service  │device-service                │
│ ┌──────────┐ │ ┌───────────┐ │ ┌────────────┐               │
│ │ BC-01    │ │ │ BC-02     │ │ │ BC-03      │               │
│ │ IAM      │ │ │ Place Mgmt│ │ │ Device Mgmt│               │
│ └──────────┘ │ └───────────┘ │ └────────────┘               │
├──────────────┼───────────────┼──────────────────────────────┤
│consumption-service           │valve-service │notification-  │
│ ┌──────────┐ ┌─────────────┐ │ ┌──────────┐ │  service      │
│ │ BC-04    │ │ BC-05       │ │ │ BC-07    │ │ ┌───────────┐ │
│ │Monitoring│ │ Goals       │ │ │ Valve    │ │ │ BC-08     │ │
│ ├──────────┤ ├─────────────┤ │ │ Control  │ │ │ Alerts &  │ │
│ │ BC-06    │ │ BC-10       │ │ └──────────┘ │ │ Detection │ │
│ │Billing & │ │ Reports &   │ │              │ ├───────────┤ │
│ │Tariffs   │ │ Export      │ │              │ │ BC-09     │ │
│ └──────────┘ └─────────────┘ │              │ │ Delivery  │ │
│                              │              │ ├───────────┤ │
│                              │              │ │ BC-11     │ │
│                              │              │ │ Tips      │ │
│                              │              │ └───────────┘ │
└──────────────────────────────┴──────────────┴──────────────┘
```

### Evolution path

If the team grows or a BC's traffic demands it, the first candidates for extraction are:

1. **BC-08 Alerts & Detection** → own service (highest event fan-in: receives from BC-03, BC-05, BC-06, BC-07)
2. **BC-10 Reports & Export** → own service (heavy read queries should not compete with real-time consumption writes)
3. **BC-06 Billing & Tariffs** → own service (if billing logic grows to require its own data store)

---

## 5. Service catalog

| # | Service | Responsibility | Port | Technology | Schema | BCs | Communication |
|---|---------|---------------|------|-----------|--------|-----|---------------|
| 01 | `api-gateway` | Routing, JWT validation, CORS, rate limiting | 8080 | Spring Cloud Gateway | — | — | HTTP Proxy |
| 02 | `auth-service` | Registration, login, JWT tokens, Google OAuth, RBAC | 3001 | Spring Boot | `security` | BC-01 | REST + Events |
| 03 | `place-service` | Place CRUD, switching, invitations, shared access | 3002 | .NET 8 | `places` | BC-02 | REST + Events |
| 04 | `device-service` | Device linking, MQTT telemetry ingestion, device status | 3003 | .NET 8 | `devices` | BC-03 | MQTT + REST + Events |
| 05 | `consumption-service` | Consumption aggregation, goals, bill comparison, reports | 3004 | .NET 8 | `monitoring`, `goals`, `billing` | BC-04,05,06,10 | REST + Events |
| 06 | `valve-service` | Valve state, open/close commands, auto-shutoff rules | 3005 | .NET 8 | `valve` | BC-07 | REST + Events |
| 07 | `notification-service` | Alert engine, leak detection, multi-channel delivery, tips | 3006 | .NET 8 | `alerts`, `notifications`, `content` | BC-08,09,11 | REST + Events |

---

## 6. Architectural principles

### P1: API-First
Design the API contract (OpenAPI) before implementing the service. Contracts are the source of truth for consumers.

### P2: Shared Database, Schema Isolation
All services share a single SQL Server instance (`sy_water_db`) with 10 schemas separated for logical organization. Cross-schema foreign keys ARE enforced by SQL Server for referential integrity. However, application-level data access across services is through REST APIs or domain events via RabbitMQ — services must NOT JOIN across schemas at the application layer. See ADR-002.

### P3: Fail Fast, Recover Gracefully
Detect errors early with input validation at the API Gateway. When an external service or the MQTT broker is unavailable, log the failure and respond with a clear error — adopt resilience patterns (Circuit Breaker, Retry) when complexity justifies it.

### P4: Events for Decoupling
Use RabbitMQ domain events for all operations that do not require an immediate response. Synchronous REST calls are reserved for queries and operations the user is waiting for. See `02-domain/domain-events.md`.

### P5: Start Simple, Evolve When Needed
Do not adopt complex patterns (CQRS, Event Sourcing, Saga) upfront. Add them when a concrete problem demands it, and document the decision as an ADR.

### P6: Hexagonal Architecture per Service
Each service follows the Ports & Adapters pattern so the domain logic is independent of frameworks and infrastructure. See `05-architecture/hexagonal-architecture.md`.

### P7: Domain Integrity across Co-deployed BCs
When multiple BCs share a service, they are organized as separate modules with their own domain model. BCs in the same service communicate via in-process method calls (not REST or events) but respect aggregate boundaries.

---

## 7. Adopted architectural patterns

| Pattern | Adopted? | Justification |
|---------|---------|---------------|
| API Gateway | Yes | Single entry point, JWT validation, routing — Spring Cloud Gateway |
| Schema-per-Service (shared DB) | Yes | Team of 3, single VPS — schema isolation preserves logical boundaries. See ADR-002 |
| Hexagonal Architecture | Yes | Domain independence from frameworks; testability |
| Event-Driven (RabbitMQ) | Yes | Decouple services for async operations (readings, alerts, notifications) |
| BC-to-Service co-deployment | Yes | 11 BCs in 6 services — reduces operational overhead while preserving domain boundaries |
| Circuit Breaker | Not yet | Will evaluate when inter-service failures become a real concern |
| Saga | Not yet | Will evaluate when distributed transactions are needed (e.g., device linking flow) |
| Outbox Pattern | Not yet | Will evaluate when event delivery reliability becomes critical |
| CQRS | No | Read and write models are similar enough; unnecessary complexity |
| Event Sourcing | No | No audit/replay requirement justifies the complexity |
| BFF | No | Single React Native codebase for mobile and web — one API format suffices |

---

## 8. Communication patterns

### Synchronous (REST)

| Source | Destination | Purpose |
|--------|------------|---------|
| api-gateway | auth-service | JWT token validation on every request |
| api-gateway | all services | Route requests by path prefix |
| valve-service | device-service | Send open/close command to ESP32 via device-service MQTT bridge |
| Any service | auth-service | Verify user permissions (when needed beyond JWT claims) |
| consumption-service | place-service | Resolve place metadata (stratum for tariff calculation in BC-06) |

### Asynchronous (RabbitMQ — domain events)

Events are organized by origin BC. Full catalog with payloads in `02-domain/domain-events.md`.

| Source service | Origin BC | Event | Consumer service (BC) | Purpose |
|---------------|----------|-------|----------------------|---------|
| auth-service | BC-01 IAM | `UserRegistered` | notification-service (BC-09) | Send verification email |
| auth-service | BC-01 IAM | `AccountBlocked` | notification-service (BC-09) | Notify blocked account |
| place-service | BC-02 Place | `PlaceCreated` | consumption-service (BC-04) | Provision consumption storage |
| place-service | BC-02 Place | `PlaceDeleted` | consumption-service (BC-04), notification-service (BC-08) | Archive data, cancel alerts |
| place-service | BC-02 Place | `MemberInvited` | notification-service (BC-09) | Send invitation email |
| place-service | BC-02 Place | `MemberAccessRevoked` | auth-service (BC-01) | Invalidate sessions |
| device-service | BC-03 Device | `DeviceLinked` | consumption-service (BC-04), valve-service (BC-07) | Start monitoring, create valve record |
| device-service | BC-03 Device | `DeviceUnlinked` | valve-service (BC-07), consumption-service (BC-04) | Deactivate valve, stop monitoring |
| device-service | BC-03 Device | `ReadingReceived` | consumption-service (BC-04), notification-service (BC-08) | Aggregate readings, feed leak detection |
| device-service | BC-03 Device | `DeviceDisconnected` | notification-service (BC-08) | Sensor-down alert |
| device-service | BC-03 Device | `DeviceReconnected` | notification-service (BC-08) | Clear disconnect alert |
| consumption-service | BC-05 Goals | `GoalThresholdReached` | notification-service (BC-08) | Goal warning alert |
| consumption-service | BC-05 Goals | `GoalExceeded` | notification-service (BC-08), valve-service (BC-07) | Alert + auto-shutoff |
| consumption-service | BC-06 Billing | `BillMismatchDetected` | notification-service (BC-08) | Bill discrepancy alert |
| notification-service | BC-08 Alerts | `LeakSuspected` | valve-service (BC-07), notification-service (BC-09) | Auto-shutoff + critical notification |
| valve-service | BC-07 Valve | `ValveClosed` | notification-service (BC-08) | Critical alert |
| valve-service | BC-07 Valve | `ValveOpened` | notification-service (BC-09) | Water restored notification |
| valve-service | BC-07 Valve | `ValveCommandFailed` | notification-service (BC-08) | Command failure alert |

### IoT (MQTT — Mosquitto)

| Direction | Topic pattern | Purpose | QoS |
|-----------|--------------|---------|-----|
| ESP32 → device-service | `devices/{hardwareId}/telemetry` | Flow sensor readings (every 10s) | 1 (at-least-once) |
| device-service → ESP32 | `devices/{hardwareId}/commands` | Valve open/close commands | 1 (at-least-once) |
| ESP32 → device-service | `devices/{hardwareId}/status` | Device heartbeat / connectivity (every 60s) | 0 (at-most-once) |

### Communication flow diagram — Leak detection

```
 ESP32                device-service         notification-service      valve-service
  │                        │                        │                       │
  │──telemetry (MQTT)─────▶│                        │                       │
  │                        │──ReadingReceived───────▶│                       │
  │                        │     (RabbitMQ)          │                       │
  │                        │                   [LeakDetector                 │
  │                        │                    evaluates                    │
  │                        │                    continuous flow]             │
  │                        │                        │                       │
  │                        │                   LeakSuspected─────────────────▶│
  │                        │                        │   (RabbitMQ)          │
  │                        │                        │               [ShutoffRule
  │                        │                        │                evaluates]
  │                        │                        │                       │
  │                        │◀──── CloseValve (REST) ─────────────────────────│
  │◀──command (MQTT)───────│                        │                       │
  │                        │                        │                       │
  │──ack (MQTT)───────────▶│                        │                       │
  │                        │──ValveClosed──────────▶│                       │
  │                        │   (RabbitMQ)           │                       │
  │                        │                  [Push + Email                  │
  │                        │                   sent to owner]               │
```

---

## 9. Deployment view

### Infrastructure (single VPS — Docker Compose)

```
┌─────────────────────────────────────────────────────────────┐
│  VPS (Docker Host)                                          │
│                                                             │
│  ┌─────────────────┐  ┌─────────────────┐                   │
│  │  api-gateway    │  │  auth-service   │                   │
│  │  :8080          │  │  :3001          │                   │
│  └────────┬────────┘  └─────────────────┘                   │
│           │                                                 │
│  ┌────────┴────────┬──────────────┬──────────────┐          │
│  │                 │              │              │          │
│  │ place-service   │device-service│consumption-  │          │
│  │ :3002           │:3003         │service :3004 │          │
│  │                 │              │              │          │
│  └─────────────────┴──────┬───────┴──────────────┘          │
│                           │                                 │
│  ┌─────────────────┐  ┌───┴─────────────┐                   │
│  │ valve-service   │  │notification-    │                   │
│  │ :3005           │  │service :3006    │                   │
│  └─────────────────┘  └─────────────────┘                   │
│                                                             │
│  ┌── Infrastructure ────────────────────────────────────┐   │
│  │                                                      │   │
│  │  ┌─────────────┐ ┌──────────┐ ┌──────────────────┐   │   │
│  │  │ SQL Server  │ │ RabbitMQ │ │ Mosquitto (MQTT)│   │   │
│  │  │ :1433       │ │ :5672    │ │ :1883           │   │   │
│  │  │ sy_water_db │ │ :15672   │ │                 │   │   │
│  │  │             │ │ (mgmt UI)│ │                 │   │   │
│  │  └─────────────┘ └──────────┘ └──────────────────┘   │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌── Volumes ───────────────────────────────────────────┐   │
│  │  sqlserver-data · rabbitmq-data · mosquitto-config   │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### Container configuration summary

| Container | Image | Ports | Volumes | Depends on |
|-----------|-------|-------|---------|-----------|
| `api-gateway` | Custom (Spring Cloud GW) | 8080:8080 | — | auth-service |
| `auth-service` | Custom (Spring Boot) | 3001:3001 | — | sqlserver, rabbitmq |
| `place-service` | Custom (.NET 8) | 3002:3002 | — | sqlserver, rabbitmq |
| `device-service` | Custom (.NET 8) | 3003:3003 | — | sqlserver, rabbitmq, mosquitto |
| `consumption-service` | Custom (.NET 8) | 3004:3004 | — | sqlserver, rabbitmq |
| `valve-service` | Custom (.NET 8) | 3005:3005 | — | sqlserver, rabbitmq, device-service |
| `notification-service` | Custom (.NET 8) | 3006:3006 | — | sqlserver, rabbitmq |
| `sqlserver` | mcr.microsoft.com/mssql/server:2022 | 1433:1433 | sqlserver-data | — |
| `rabbitmq` | rabbitmq:3-management | 5672:5672, 15672:15672 | rabbitmq-data | — |
| `mosquitto` | eclipse-mosquitto:2 | 1883:1883 | mosquitto-config | — |

### Network topology

All containers share a Docker bridge network (`sy-water-net`). Services resolve each other by container name (e.g., `http://auth-service:3001`).

---

## 10. Cross-cutting concerns

| Concern | Solution | Where configured |
|---------|---------|-----------------|
| Authentication | JWT (issued by auth-service BC-01, validated at API Gateway) | api-gateway config |
| Authorization | Role-based (User/Owner/Member/Admin) in JWT claims + 5 granular permissions for Member role | Each service validates role; place-service validates member permissions |
| Logging | Structured JSON with correlation ID | Shared logging config per service |
| Health Checks | `GET /health` (liveness) + `GET /health/ready` (readiness) | Each service |
| Error format | Standard error response (code, message, details) | `07-api/contracts/openapi/_shared.yaml` |
| Rate Limiting | At the API Gateway level | Spring Cloud Gateway config |
| CORS | Configured at API Gateway for React Native web | api-gateway config |
| Correlation ID | Generated at API Gateway, propagated in `X-Correlation-Id` header, included in all events and logs | api-gateway generates; services propagate |

---

## 11. Database strategy

**Single instance:** SQL Server — `sy_water_db`

**Schema isolation:** 10 schemas separated for logical organization. See ADR-002.

| Schema | Owner service | Owner BCs | Main tables |
|--------|-------------|----------|-------------|
| `security` | auth-service | BC-01 | users, user_credentials, email_verifications, password_resets, refresh_tokens, login_attempts, activity_log |
| `places` | place-service | BC-02 | places, place_members, invitations |
| `devices` | device-service | BC-03 | devices, device_link_history |
| `monitoring` | consumption-service | BC-04 | readings, consumption_hourly, consumption_daily, consumption_monthly |
| `goals` | consumption-service | BC-05 | consumption_goals, goal_suggestions |
| `billing` | consumption-service | BC-06 | water_bills, tariffs, tariff_catalog |
| `valve` | valve-service | BC-07 | valve_states, valve_commands, valve_rules |
| `alerts` | notification-service | BC-08 | alert_type_config, alert_events, notification_preferences |
| `notifications` | notification-service | BC-09 | notification_log, push_subscriptions |
| `content` | notification-service | BC-11 | savings_tips, tip_favorites |

**Rules:**
- A service may only read/write tables in its own schemas
- Cross-service data access at application level is via REST API or RabbitMQ events
- Cross-schema foreign keys ARE enforced by SQL Server for referential integrity
- Application-layer cross-schema JOINs are NOT allowed — use the owning service's API
- Migrations are managed per service, scoped to its schemas

> Detailed data model in `06-data/models.md`

---

## 12. Technology stack summary

| Layer | Technology | Justification |
|-------|-----------|---------------|
| Mobile app | React Native (Expo) | Single codebase for iOS + Android |
| Web app | React Native Web (Expo) | Code sharing with mobile; same UX |
| API Gateway | Spring Cloud Gateway | Routing, JWT validation, rate limiting; Spring ecosystem |
| Auth service | Spring Boot 3 + Spring Security | Mature OAuth2/JWT support; Spring team familiarity |
| Domain services | .NET 8 (ASP.NET Core) | Team's primary backend technology; strong typing |
| Database | SQL Server 2022 | Team familiarity; T-SQL support; single VPS licensing |
| Message broker | RabbitMQ 3 | Simpler than Kafka for current scale; topic exchanges for event routing |
| MQTT broker | Eclipse Mosquitto 2 | Lightweight; standard IoT protocol; ESP32 native support |
| IoT device | ESP32 + YF-S201 (flow sensor) + electrovalve | Low cost; WiFi built-in; Arduino ecosystem |
| Containerization | Docker + Docker Compose | Single VPS deployment; reproducible environments |

---

## 13. Registered architectural technical debt

| ID | Description | Impact | Priority | Target |
|----|-------------|--------|---------|--------|
| AT-001 | No Circuit Breaker on inter-service calls | Medium — cascade failures possible | P3 | Evaluate after MVP |
| AT-002 | No Outbox Pattern for event publishing | Medium — events could be lost on crash | P3 | Evaluate after MVP |
| AT-003 | No distributed tracing (OpenTelemetry) | Low — debugging across services is harder | P3 | Entrega 2 |
| AT-004 | All BCs co-deployed — some could benefit from extraction | Low — current load does not require it | P4 | When team grows or load demands |

---

## 14. Planned evolution

| Phase | Architectural change | Motivation | Target |
|-------|---------------------|------------|--------|
| Post-MVP | Add Circuit Breaker on device-service and valve-service calls | MQTT/device communication is the most failure-prone path | Entrega 2 |
| Post-MVP | Add distributed tracing | As event flows cross 3+ services, debugging becomes critical | Entrega 2 |
| Post-MVP | Add health dashboard | Centralized view of service status and RabbitMQ queues | Entrega 2 |
| If needed | Extract BC-08 Alerts into own service | Highest event fan-in — receives from 4 other BCs | When notification-service becomes bottleneck |
| If needed | Extract BC-10 Reports into own service | Heavy read queries competing with real-time writes | When report generation slows consumption writes |
| If needed | Split shared DB into separate instances | If query contention becomes a bottleneck | When load demands it |
| If needed | Add CQRS for consumption reads | If read volume vastly exceeds writes (many users refreshing dashboards) | When load demands it |

---

## Key correlations

- Domain bounded contexts → `02-domain/domain-map.md`
- Domain events catalog → `02-domain/domain-events.md`
- Entities and aggregates → `02-domain/entities-and-rules.md`
- ADRs → `05-architecture/decisions/`
- Hexagonal architecture → `05-architecture/hexagonal-architecture.md`
- Pattern catalog → `05-architecture/pattern-guide.md`
- Data model → `06-data/models.md`
- API contracts → `07-api/contracts/openapi/`
