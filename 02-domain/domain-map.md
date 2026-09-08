# Domain Map — Bounded Contexts

## 1. Domain overview

Save your water manages the real-time monitoring and control of residential and commercial water consumption. An IoT device (ESP32 + flow sensor + electrovalve) installed at the property's main water inlet sends flow readings to the platform via MQTT. The platform aggregates readings into consumption data, displays charts, detects suspected leaks, allows remote valve control, and compares measured consumption against utility bills.

---

## 2. Identified Bounded Contexts

### Bounded Context: IAM (Identity & Access Management)

| Field | Value |
|-------|-------|
| **Name** | IAM |
| **Responsibility** | User identity lifecycle: registration, authentication, authorization, profile management |
| **Owning team** | Full team (3 devs) |
| **Microservice(s)** | `auth-service` |
| **Database** | SQL Server — shared instance (`sy_water_db`), schema: `iam` |
| **Technology** | Spring Boot |
| **Ubiquitous Language** | User, Account, Session, Role (Owner/Guest/Admin), Credential |

**Context-specific terms:**

| Term | Meaning in THIS context | Different in another context? |
|------|------------------------|-------------------------------|
| User | A person with an active account (email + password or Google OAuth) | No — User ID is shared across all contexts |
| Role | Permission level: Owner, Guest, or Admin | No |
| Session | An authenticated period with a JWT token (1h expiry, 7d refresh) | No |

**User Stories:** HU-001, HU-003, HU-004, HU-005, HU-006, HU-007, HU-008

---

### Bounded Context: Place Management

| Field | Value |
|-------|-------|
| **Name** | Place Management |
| **Responsibility** | Lifecycle of monitored locations: creation, configuration, shared access (invitations) |
| **Owning team** | Full team |
| **Microservice(s)** | `place-service` |
| **Database** | SQL Server — shared instance (`sy_water_db`), schema: `place` |
| **Technology** | TBD (.NET under evaluation) |
| **Ubiquitous Language** | Place, Address, Stratum, Invitation, Member |

**Context-specific terms:**

| Term | Meaning in THIS context | Different in another context? |
|------|------------------------|-------------------------------|
| Place | A registered location with address, country, city, stratum, and inhabitant count | In Device context, a Place is just an ID to associate a device with |
| Member | A user linked to a place with a specific role (Owner or Guest) | No |
| Invitation | A pending request for a user to join a place as Guest | No |

**User Stories:** HU-002, HU-009, HU-010, HU-050, HU-051, HU-052, HU-053

---

### Bounded Context: Device Management

| Field | Value |
|-------|-------|
| **Name** | Device Management |
| **Responsibility** | Device registry, linking to places, telemetry ingestion via MQTT, device health monitoring |
| **Owning team** | Full team |
| **Microservice(s)** | `device-service` |
| **Database** | SQL Server — shared instance (`sy_water_db`), schema: `device` |
| **Technology** | TBD (.NET under evaluation) |
| **Ubiquitous Language** | Device, Reading, Telemetry, Link, Status |

**Context-specific terms:**

| Term | Meaning in THIS context | Different in another context? |
|------|------------------------|-------------------------------|
| Device | A registered IoT unit (ESP32 + sensor + valve) identified by a unique hardware ID | In Valve context, Device is just the communication channel for commands |
| Reading | A single flow measurement with timestamp, received via MQTT | In Consumption context, readings are aggregated into consumption records |
| Link | The association between a device and a place (one-to-one) | No |
| Status | Connectivity state: Connected / Disconnected / Never Reported | No |

**User Stories:** HU-011, HU-012, HU-013, HU-014

---

### Bounded Context: Consumption

| Field | Value |
|-------|-------|
| **Name** | Consumption |
| **Responsibility** | Aggregation of raw readings into consumption data, real-time display, charts, goals, and bill comparison |
| **Owning team** | Full team |
| **Microservice(s)** | `consumption-service` |
| **Database** | SQL Server — shared instance (`sy_water_db`), schema: `consumption` |
| **Technology** | TBD (.NET under evaluation) |
| **Ubiquitous Language** | Consumption, Period, Chart, Goal, Bill, Tariff, Comparison |

**Context-specific terms:**

| Term | Meaning in THIS context | Different in another context? |
|------|------------------------|-------------------------------|
| Consumption | Aggregated water usage for a place in a specific period (day/week/month) | No |
| Goal | A monthly consumption target set by the owner for a place | No |
| Bill | A manually registered utility bill used for comparison against sensor data | No |
| Tariff | Price per water unit, used to estimate monetary cost | No |

**User Stories:** HU-015, HU-016, HU-017, HU-018, HU-023, HU-024, HU-025, HU-036, HU-037, HU-038, HU-042, HU-043, HU-044, HU-045

---

### Bounded Context: Valve Control

| Field | Value |
|-------|-------|
| **Name** | Valve Control |
| **Responsibility** | Valve state management, manual open/close commands with security checks, auto-shutoff rules |
| **Owning team** | Full team |
| **Microservice(s)** | `valve-service` |
| **Database** | SQL Server — shared instance (`sy_water_db`), schema: `valve` |
| **Technology** | TBD (.NET under evaluation) |
| **Ubiquitous Language** | Valve, Command, Confirmation, ShutoffRule, Action Log |

**Context-specific terms:**

| Term | Meaning in THIS context | Different in another context? |
|------|------------------------|-------------------------------|
| Valve | The logical representation of the electrovalve, with state (Open/Closed/No Communication) | In Device context, the valve is a physical component of the device |
| Command | An instruction to open or close the valve, sent to the ESP32 via MQTT through device-service | No |
| Confirmation | A security code sent to the owner before executing a close command | No |
| ShutoffRule | An automatic rule that triggers valve closure (leak detected, goal exceeded) | No |

**User Stories:** HU-019, HU-020, HU-021, HU-022, HU-046, HU-047, HU-048, HU-049

---

### Bounded Context: Notification

| Field | Value |
|-------|-------|
| **Name** | Notification |
| **Responsibility** | Alert engine, multi-channel delivery (in-app, push, email, WhatsApp/SMS), notification center, user preferences |
| **Owning team** | Full team |
| **Microservice(s)** | `notification-service` |
| **Database** | SQL Server — shared instance (`sy_water_db`), schema: `notification` |
| **Technology** | TBD (.NET under evaluation) |
| **Ubiquitous Language** | Alert, Notification, Channel, Preference, Urgency Level |

**Context-specific terms:**

| Term | Meaning in THIS context | Different in another context? |
|------|------------------------|-------------------------------|
| Alert | A system-generated event classified by urgency (Critical/Important/Informative) | No |
| Channel | A delivery medium: In-App, Push, Email, WhatsApp/SMS | No |
| Preference | The user's choice of which channels to receive for each urgency level | No |

**User Stories:** HU-026, HU-027, HU-028, HU-029, HU-030, HU-031, HU-032, HU-033, HU-034, HU-035, HU-039, HU-040, HU-041

---

## 3. Context Map

```
                              ┌─────────────────────┐
         External clients     │    API Gateway       │
         (Mobile, Web)   ───▶ │    :8080             │
                              └──────────┬──────────┘
                                         │ routes by path
            ┌────────────┬───────────────┼───────────────┬──────────────┐
            ▼            ▼               ▼               ▼              ▼
    ┌──────────────┐ ┌──────────┐ ┌────────────┐ ┌────────────┐ ┌─────────────┐
    │ auth-service │ │  place-  │ │consumption-│ │   valve-   │ │notification-│
    │   (IAM)      │ │ service  │ │  service   │ │  service   │ │   service   │
    │   :3001      │ │  :3002   │ │   :3004    │ │   :3005    │ │    :3006    │
    └──────┬───────┘ └────┬─────┘ └─────┬──────┘ └─────┬──────┘ └──────┬──────┘
           │              │             │               │               │
           │              │             │    ┌──────────┘               │
           │              │             │    │                          │
           │              │        ┌────┴────┴───┐                     │
           │              │        │device-service│                    │
           │              │        │    :3003     │                    │
           │              │        └──────┬───────┘                    │
           │              │               │                            │
           │              │               │ MQTT                       │
           │              │        ┌──────┴───────┐                    │
           │              │        │  MQTT Broker  │                    │
           │              │        │  (Mosquitto)  │                    │
           │              │        └──────┬───────┘                    │
           │              │               │                            │
           │              │        ┌──────┴───────┐                    │
           │              │        │  ESP32 + IoT  │                    │
           │              │        │   Devices     │                    │
           │              │        └──────────────┘                    │
           │              │                                            │
           └──────────────┴──────────────┬─────────────────────────────┘
                                         ▼
                                 ┌───────────────┐
                                 │ Message Broker │
                                 │(Domain Events) │
                                 └───────────────┘
```

### Relationships table

| Context A | Relationship | Context B | Communication | Description |
|-----------|-------------|-----------|---------------|-------------|
| IAM | U → D (OHS) | All contexts | JWT validation | All services validate JWT tokens issued by auth-service |
| Place Management | U → D | Device Management | REST + Event | A device is linked to a place; `PlaceDeleted` triggers device unlinking |
| Place Management | U → D | Consumption | Event | `PlaceCreated` provisions consumption storage; `PlaceDeleted` archives data |
| Device Management | U → D | Consumption | Event | `ReadingReceived` events feed consumption aggregation |
| Device Management | U → D | Notification | Event | `DeviceDisconnected` triggers sensor-down alert |
| Consumption | U → D | Notification | Event | `GoalExceeded`, `GoalThresholdReached` trigger alerts |
| Consumption | U → D | Valve Control | Event | `GoalExceeded` can trigger auto-shutoff rule evaluation |
| Valve Control | C/S | Device Management | REST (command) | Valve-service sends open/close commands; device-service forwards to ESP32 via MQTT |
| Device Management | U → D | Valve Control | Event | `ValveStateChanged` confirms command execution |
| Valve Control | U → D | Notification | Event | `ValveClosed`, `ValveCommandFailed` trigger critical alerts |
| Notification | ACL | External (Email/Push/SMS) | API | Anti-corruption layer translates internal alerts to external provider formats |

---

## 4. Core Domain, Supporting, Generic

| Bounded Context | Type | Justification |
|----------------|------|---------------|
| **Consumption** | Core | Real-time consumption visibility is the main value proposition — what differentiates the product |
| **Device Management** | Core | IoT telemetry ingestion is the data foundation; without it, the product has no data |
| **Valve Control** | Core | Remote valve control is a key differentiator vs. just reading the meter |
| **Place Management** | Supporting | Necessary structure for organizing devices and data, but not differentiating |
| **Notification** | Supporting | Alerts add value but the delivery infrastructure is standard |
| **IAM** | Generic | Authentication and authorization — standard solution, not differentiating |
| **API Gateway** | Generic | Infrastructure routing — commodity |

---

## 5. Modeling decisions

### How were these decisions made?

- **Source:** Product backlog v1.0 (2026-09-02) — 61 user stories across 14 epics
- **Method:** Backlog analysis and context grouping based on cohesion, coupling, and rate of change
- **Date:** 2026-09-08
- **Participants:** Development team + Claude (analysis)

### Key decisions and discarded alternatives

| Decision | Discarded alternative | Reason |
|----------|----------------------|--------|
| Separate Device and Valve contexts | Merge into one `device-service` | Different criticality (valve = security-sensitive), different scaling needs (telemetry = high volume, valve = low volume), different rate of change |
| Consumption owns goals and bills | Separate goal-service and billing-service | For a team of 3, the overhead of separate services for closely related data is not justified; they share the same aggregated consumption data |
| Notification as its own context | Inline alerts in each service | Multi-channel delivery (push, email, SMS, in-app) is cross-cutting; centralizing avoids duplicating channel logic |
| Single shared database (`sy_water_db`) with schema-per-service | Database-per-service (separate SQL Server instances) | Team of 3 and single VPS make separate instances impractical; schema separation preserves logical boundaries while sharing one instance. Services must NOT query another service's schema directly — use events or REST instead |

---

## 6. How to update this map

1. Before adding a new microservice, verify whether it belongs to an existing bounded context.
2. If a context's ubiquitous language is changing, review whether the context should be split.
3. The context map MUST be synchronized with the C4 diagram in `05-architecture/overview.md`.
4. Record significant changes as ADRs in `05-architecture/decisions/records/`.
