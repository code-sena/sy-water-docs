# Domain Map — Bounded Contexts

## 1. Domain overview

Save Your Water manages the real-time monitoring and control of residential and commercial water consumption. An IoT device (ESP32 + YF-S201 flow sensor + electrovalve) installed at the property's main water inlet sends flow readings to the platform via MQTT. The platform aggregates readings into consumption data, displays charts, detects suspected leaks, allows remote valve control, compares measured consumption against utility bills, and delivers multi-channel alerts.

The domain is decomposed into **11 bounded contexts** organized in 4 strategic layers. These 11 contexts are deployed across **6 microservices** — a pragmatic mapping for a team of 3 developers. See `05-architecture/overview.md` for the deployment mapping.

---

## 2. Subdomain classification

### Core — what makes the product unique

| # | Subdomain | Justification |
|---|-----------|---------------|
| 1 | **Consumption Monitoring** | Real-time consumption visibility is the main value proposition — what differentiates the product from waiting for the monthly bill |
| 2 | **Valve Control** | Remote valve operation converts the platform from informative to actionable — a key differentiator vs. just reading the meter |
| 3 | **Alerts & Detection** | The rule engine that evaluates conditions (leak, goal breach, sensor down, bill mismatch) and classifies urgency — the reactive intelligence of the system |

### Supporting — necessary but not differentiating

| # | Subdomain | Justification |
|---|-----------|---------------|
| 4 | **IAM (Identity & Access)** | Has its own rules (login attempts, blocking, tokens, OAuth) but is infrastructure for the rest of the system |
| 5 | **Place Management** | Organizational model that connects users, devices, and data — necessary structure, not differentiating |
| 6 | **Device Management** | IoT bridge between hardware and domain — registration, linking, telemetry ingestion |
| 7 | **Billing & Tariffs** | Manual bill registration, tariff configuration, sensor-vs-bill comparison — has its own language (period, tariff, currency) |
| 8 | **Consumption Goals** | Monthly targets with percentage progress and suggested goals from history — feeds the alert engine but has its own logic |

### Generic — solvable with libraries or external services

| # | Subdomain | Justification |
|---|-----------|---------------|
| 9 | **Notification Delivery** | Multi-channel transport (push, email, SMS/WhatsApp) — no business logic, only delivers what Alerts & Detection decides |
| 10 | **Reports & Export** | PDF/CSV generation — data consumer with no domain rules |
| 11 | **Savings Tips** | Static catalog of water-saving recommendations created by Admin — no dependencies, no domain logic |

---

## 3. Bounded Contexts

### BC-01: IAM (Identity & Access Management)

| Field | Value |
|-------|-------|
| **Type** | Supporting |
| **Responsibility** | User identity lifecycle: registration, authentication, authorization, profile management, admin user panel |
| **Microservice** | `auth-service` |
| **Schema** | `security` |
| **Technology** | Spring Boot |
| **Ubiquitous Language** | User, Account, Session, Role (User/Owner/Member/Admin), Credential, Verification |

**Context-specific terms:**

| Term | Meaning in THIS context | Different in another context? |
|------|------------------------|-------------------------------|
| User | A person with an active account (email + password or Google OAuth) | No — User ID is shared across all contexts |
| Role | Permission level: User, Owner, Member, or Admin. User is the default authenticated role; Owner and Member are place-scoped | No |
| Session | An authenticated period with a JWT token (1h expiry, 7d refresh) | No |

**User Stories:** HU-01 to HU-06 (Auth), HU-45 to HU-48 (Admin panel)

---

### BC-02: Place Management

| Field | Value |
|-------|-------|
| **Type** | Supporting |
| **Responsibility** | Lifecycle of monitored locations, member access (invitations and permissions), multi-place management |
| **Microservice** | `place-service` |
| **Schema** | `places` |
| **Technology** | .NET |
| **Ubiquitous Language** | Place, Address, Stratum, Member, Invitation, Permissions |

**Context-specific terms:**

| Term | Meaning in THIS context | Different in another context? |
|------|------------------------|-------------------------------|
| Place | A registered location with address, country, city, stratum, and inhabitant count | In Device context, a Place is just an ID to associate a device with |
| Member | A user linked to a place with role (Owner/Member) and permissions defined by the Owner | No |
| Invitation | A pending request for a user to join a place as Member with predefined permissions | No |

**User Stories:** HU-07 to HU-09 (Place CRUD), HU-30 to HU-33 (Members), HU-53 to HU-56 (Multi-place)

---

### BC-03: Device Management

| Field | Value |
|-------|-------|
| **Type** | Supporting |
| **Responsibility** | Device registry, linking to places, MQTT telemetry ingestion, device health monitoring |
| **Microservice** | `device-service` |
| **Schema** | `devices` |
| **Technology** | .NET |
| **Ubiquitous Language** | Device, HardwareId, Reading, Telemetry, Link, Status |

**Context-specific terms:**

| Term | Meaning in THIS context | Different in another context? |
|------|------------------------|-------------------------------|
| Device | A registered IoT unit (ESP32 + sensor + valve) identified by a unique hardware ID | In Valve Control, Device is the communication channel for commands |
| Reading | A single flow measurement with timestamp, received via MQTT | In Consumption Monitoring, readings are aggregated into consumption records |
| Link | The association between a device and a place (one-to-one) | No |
| Status | Connectivity state: Connected / Disconnected / Never Reported | No |

**User Stories:** HU-10 to HU-12

---

### BC-04: Consumption Monitoring *(Core)*

| Field | Value |
|-------|-------|
| **Type** | Core |
| **Responsibility** | Aggregation of raw readings into consumption data, real-time display, charts by period (hour/day/week/month) |
| **Microservice** | `consumption-service` |
| **Schema** | `monitoring` |
| **Technology** | .NET |
| **Ubiquitous Language** | Consumption, ConsumptionRecord, Period, Aggregation, Chart, Volume |

**Context-specific terms:**

| Term | Meaning in THIS context | Different in another context? |
|------|------------------------|-------------------------------|
| Consumption | Aggregated water usage for a place in a specific period (hour/day/week/month) | In Billing & Tariffs, consumption is translated into monetary cost |
| Period | A time window (HOUR, DAY, WEEK, MONTH) over which readings are aggregated | In Billing & Tariffs, period is the billing cycle defined by the utility company |
| Aggregation | The process of combining raw readings into hourly/daily/monthly totals | No |

**User Stories:** HU-13 to HU-17

---

### BC-05: Consumption Goals *(Supporting)*

| Field | Value |
|-------|-------|
| **Type** | Supporting |
| **Responsibility** | Monthly consumption targets per place, percentage progress tracking, suggested goals based on historical data |
| **Microservice** | `consumption-service` |
| **Schema** | `goals` |
| **Technology** | .NET |
| **Ubiquitous Language** | Goal, Target, Progress, Threshold (80%), Exceeded (100%), Suggestion |

**Context-specific terms:**

| Term | Meaning in THIS context | Different in another context? |
|------|------------------------|-------------------------------|
| Goal | A monthly consumption target set by the Owner for a place | No |
| Threshold | The 80% mark of a goal that triggers a warning event | In Alerts & Detection, threshold is a generic condition evaluation concept |
| Suggestion | An automatically proposed goal based on historical consumption (3-month average) | No |

**User Stories:** HU-18 to HU-21

---

### BC-06: Billing & Tariffs *(Supporting)*

| Field | Value |
|-------|-------|
| **Type** | Supporting |
| **Responsibility** | Manual bill registration, tariff configuration, sensor-vs-bill comparison, cost estimation |
| **Microservice** | `consumption-service` |
| **Schema** | `billing` |
| **Technology** | .NET |
| **Ubiquitous Language** | Bill, Tariff, BillingPeriod, BilledVolume, Comparison, CostEstimate, Mismatch |

**Context-specific terms:**

| Term | Meaning in THIS context | Different in another context? |
|------|------------------------|-------------------------------|
| Bill | A utility company invoice manually entered by the Owner for comparison | No |
| Tariff | Price per unit of water (liters/m3/gallons) with effective date range | No |
| Comparison | The difference (absolute and %) between sensor-measured and billed consumption | No |
| Mismatch | A comparison where the difference exceeds 15% — triggers an alert | No |

**User Stories:** HU-34 to HU-40

---

### BC-07: Valve Control *(Core)*

| Field | Value |
|-------|-------|
| **Type** | Core |
| **Responsibility** | Valve state management, manual open/close commands with security checks, auto-shutoff rules, action audit log |
| **Microservice** | `valve-service` |
| **Schema** | `valve` |
| **Technology** | .NET |
| **Ubiquitous Language** | Valve, Command, Confirmation, ShutoffRule, Action, State (Open/Closed/No Communication) |

**Context-specific terms:**

| Term | Meaning in THIS context | Different in another context? |
|------|------------------------|-------------------------------|
| Valve | The logical representation of the electrovalve, with state confirmed by the device | In Device Management, the valve is a physical component |
| Command | An instruction to open or close, sent to the ESP32 via device-service | No |
| Confirmation | A security code sent to the Owner before executing a close command | No |
| ShutoffRule | An automatic rule that triggers closure on specific conditions (leak, goal exceeded) | No |

**User Stories:** HU-26 to HU-29 (Manual control + audit), HU-41 to HU-44 (Auto-shutoff)

---

### BC-08: Alerts & Detection *(Core)*

| Field | Value |
|-------|-------|
| **Type** | Core |
| **Responsibility** | Rule engine that evaluates conditions (leak, goal breach, sensor disconnection, bill mismatch), classifies urgency, deduplicates alerts, and dispatches to delivery |
| **Microservice** | `notification-service` |
| **Schema** | `alerts` |
| **Technology** | .NET |
| **Ubiquitous Language** | Alert, Rule, Condition, Urgency (Critical/Important/Informative), Detection, Deduplication |

**Context-specific terms:**

| Term | Meaning in THIS context | Different in another context? |
|------|------------------------|-------------------------------|
| Alert | A system-generated fact triggered by a domain event, classified by urgency | No |
| Detection | The process of analyzing patterns (e.g., continuous flow = suspected leak) | No |
| Urgency | Classification that determines delivery channels: Critical (all), Important (push+in-app), Informative (in-app) | No |
| Deduplication | Preventing repeated alerts for the same ongoing condition (e.g., one leak alert per hour max) | No |

**User Stories:** HU-22 (Leak detection), HU-23 to HU-25 (Alert management)

---

### BC-09: Notification Delivery *(Generic)*

| Field | Value |
|-------|-------|
| **Type** | Generic |
| **Responsibility** | Multi-channel transport: push (Android/iOS), email, SMS/WhatsApp, in-app. No business logic — only delivers what Alerts & Detection decides |
| **Microservice** | `notification-service` |
| **Schema** | `notifications` |
| **Technology** | .NET |
| **Ubiquitous Language** | Channel, Preference, Delivery, Push, Email, SMS |

**Context-specific terms:**

| Term | Meaning in THIS context | Different in another context? |
|------|------------------------|-------------------------------|
| Channel | A delivery medium: In-App, Push, Email, SMS/WhatsApp | No |
| Preference | The user's choice of which channels to receive for each urgency level | No |

**User Stories:** (Cross-cutting — implicit in HU-23 to HU-25, HU-27 to HU-29)

---

### BC-10: Reports & Export *(Generic)*

| Field | Value |
|-------|-------|
| **Type** | Generic |
| **Responsibility** | Generation of consumption reports (monthly, savings), PDF/CSV export. Data consumer with no domain rules |
| **Microservice** | `consumption-service` |
| **Schema** | — (reads from `monitoring`) |
| **Technology** | .NET |
| **Ubiquitous Language** | Report, Export, PDF, CSV, Summary |

**User Stories:** HU-49 to HU-52

---

### BC-11: Savings Tips *(Generic)*

| Field | Value |
|-------|-------|
| **Type** | Generic |
| **Responsibility** | Static catalog of water-saving recommendations created by Admin. No dependencies on other contexts |
| **Microservice** | `notification-service` |
| **Schema** | `content` |
| **Technology** | .NET |
| **Ubiquitous Language** | Recommendation, Tip, Category (Saving, Maintenance, Seasonal, General) |

**User Stories:** HU-57, HU-58

---

## 4. Context Map

### Strategic view (layered)

```
╔══════════════════════════════════════════════════════════════════════╗
║                    SUPPORT LAYER (upper)                           ║
║                                                                    ║
║  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────────┐  ║
║  │     IAM      │  │    Place     │  │   Device Management      │  ║
║  │ (Supporting) │  │  Management  │  │     (Supporting)          │  ║
║  │              │  │ (Supporting) │  │                           │  ║
║  │ Auth, roles, │  │ CRUD, members│  │ Link ESP32, MQTT,        │  ║
║  │ admin panel  │  │ invitations  │  │ telemetry ingestion      │  ║
║  └──────┬───────┘  └──────┬───────┘  └────────────┬─────────────┘  ║
║         │U                │U                      │U (OHS)         ║
╠═════════╪════════════════╪════════════════════════╪════════════════╣
║         │        CORE + BILLING LAYER             │                ║
║         ▼                ▼                        ▼                ║
║  ┌──────────────┐  ╔══════════════╗  ╔════════════════════╗       ║
║  │  Billing &   │  ║ CONSUMPTION  ║  ║   VALVE CONTROL    ║       ║
║  │   Tariffs    │  ║  MONITORING  ║  ║     (Core)         ║       ║
║  │ (Supporting) │  ║   (Core)     ║  ║                    ║       ║
║  │              │  ║              ║  ║ Manual/auto valve, ║       ║
║  │ Bills, cost  │──║ Aggregate,   ║  ║ shutoff rules,     ║       ║
║  │ estimation   │AC║ charts, RT   ║  ║ audit log          ║       ║
║  └──────┬───────┘L └──────┬───────┘  └────────┬───────────┘       ║
║         │U               │U                   │Partnership       ║
╠═════════╪════════════════╪════════════════════╪═══════════════════╣
║         │    REACTIVE INTELLIGENCE LAYER      │                    ║
║         ▼                ▼                    ▼                    ║
║  ┌──────────────┐  ╔══════════════════╗  ┌──────────────┐         ║
║  │ Consumption  │  ║    ALERTS &      ║  │ Savings Tips │         ║
║  │    Goals     │  ║   DETECTION      ║  │  (Generic)   │         ║
║  │ (Supporting) │  ║    (Core)        ║  │              │         ║
║  │              │  ║                  ║  │ Static tips, │         ║
║  │ Monthly goal,│─▶║ Rule engine,     ║  │ no deps      │         ║
║  │ progress %   │  ║ leak detection,  ║  └──────────────┘         ║
║  └──────────────┘  ║ urgency levels   ║                           ║
║                    ╚════════┬═════════╝                           ║
║                             │U                                     ║
╠═════════════════════════════╪═════════════════════════════════════╣
║                    OUTPUT LAYER                                    ║
║                             ▼                                      ║
║           ┌──────────────────────┐    ┌──────────────────┐        ║
║           │ Notification Delivery│    │ Reports & Export  │        ║
║           │     (Generic)        │    │    (Generic)      │        ║
║           │                      │    │                   │        ║
║           │ Push, email, SMS     │    │ PDF, CSV          │        ║
║           └──────────────────────┘    └──────────────────┘        ║
╚══════════════════════════════════════════════════════════════════════╝

Legend: ╔══╗ = Core    ┌──┐ = Supporting/Generic
        U = Upstream   D = Downstream
        ACL = Anti-Corruption Layer
        OHS = Open Host Service
```

---

### Relationship table

Each row is a dependency between bounded contexts with its DDD integration pattern.

| # | Upstream (U) | Downstream (D) | Pattern | What flows | Justification |
|---|-------------|----------------|---------|-----------|---------------|
| 1 | IAM | Place Management | Customer / Supplier | Authenticated user ID, role | Places needs to know who the user is; IAM defines the contract |
| 2 | IAM | Valve Control | Customer / Supplier | Verified role (Owner) | Only the Owner can operate the valve; Valve queries role from IAM |
| 3 | Place Management | Consumption Monitoring | Customer / Supplier | Place ID, active place context | Consumption filters and groups readings by place |
| 4 | Place Management | Device Management | Partnership | Place↔Device association | Bidirectional: linking a device requires a place and vice versa; both evolve together |
| 5 | Device Management | Consumption Monitoring | Open Host Service | Flow readings (MQTT → event) | Devices publishes readings in an open protocol; Monitoring consumes without coupling to hardware |
| 6 | Device Management | Valve Control | Partnership | Open/close commands, state confirmation | Bidirectional: Valve sends commands, Devices confirms execution; both teams must coordinate |
| 7 | Billing & Tariffs | Consumption Monitoring | ACL | Measured consumption for the billing period | Billing translates Monitoring's model (readings, intervals) into its own language (period, tariff, currency) without contaminating it |
| 8 | Billing & Tariffs | Alerts & Detection | Customer / Supplier | Event: bill mismatch detected (>15%) | Billing detects the deviation and emits the event; Alerts decides how to notify |
| 9 | Consumption Monitoring | Consumption Goals | Customer / Supplier | Monthly accumulated volume, historical data | Goals needs current consumption to calculate progress % and suggest targets |
| 10 | Consumption Monitoring | Alerts & Detection | Customer / Supplier | Events: continuous flow data, consumption updated | Alerts evaluates consumption patterns (leak detection) from what Monitoring publishes |
| 11 | Consumption Goals | Alerts & Detection | Customer / Supplier | Events: 80% threshold reached, 100% exceeded | Goals emits threshold events; Alerts classifies and dispatches them |
| 12 | Valve Control | Alerts & Detection | Partnership | Event: valve closed. Command: close by rule | Bidirectional: Alerts can request auto-closure; Valve informs closure for critical alert |
| 13 | Alerts & Detection | Notification Delivery | Conformist | Formatted alert with urgency and channels | Delivery accepts the Alerts model as-is; no translation or enrichment |
| 14 | Consumption Monitoring | Reports & Export | Conformist | Aggregated data for the period | Reports consumes what Monitoring exposes without transforming the model |

---

## 5. BC-to-Service deployment mapping

The 11 bounded contexts are deployed across 6 microservices. Multiple BCs in one service are separated by internal packages/modules — they share a runtime and schema but maintain distinct domain logic.

| Service | Port | BCs hosted | Schemas | Technology |
|---------|------|-----------|---------|------------|
| `auth-service` | 3001 | BC-01 (IAM) | `security` | Spring Boot |
| `place-service` | 3002 | BC-02 (Place Management) | `places` | .NET |
| `device-service` | 3003 | BC-03 (Device Management) | `devices` | .NET |
| `consumption-service` | 3004 | BC-04 (Monitoring), BC-05 (Goals), BC-06 (Billing), BC-10 (Reports) | `monitoring`, `goals`, `billing` | .NET |
| `valve-service` | 3005 | BC-07 (Valve Control) | `valve` | .NET |
| `notification-service` | 3006 | BC-08 (Alerts), BC-09 (Delivery), BC-11 (Tips) | `alerts`, `notifications`, `content` | .NET |

**Why 6 services for 11 contexts?**

A team of 3 cannot operate 11 independent services. The deployment grouping follows two rules:

1. **Shared data gravity:** BCs that read/write related data live in the same service (Consumption Monitoring + Goals + Billing all pivot on place consumption data; Alerts + Delivery both access alert and notification tables)
2. **Distinct scaling/criticality:** BCs with different SLAs or scaling profiles get their own service (Device Management handles high-volume MQTT ingestion; Valve Control is security-critical with low volume)

---

## 6. Modeling decisions

### How were these decisions made?

- **Source:** Product backlog v2.0 — 58 user stories across 14 epics
- **Method:** Subdomain analysis based on ubiquitous language boundaries, rate of change, and strategic value
- **Date:** 2026-09-12
- **Participants:** Development team + DDD strategic design analysis

### Key decisions and discarded alternatives

| Decision | Discarded alternative | Reason |
|----------|----------------------|--------|
| Split Consumption into Monitoring + Goals + Billing | Keep as one monolithic Consumption BC | Each has a distinct ubiquitous language: Monitoring speaks in readings and aggregations; Goals in targets and percentages; Billing in tariffs and currency. The ACL between Billing and Monitoring confirms they speak different languages |
| Split Notification into Alerts & Detection + Delivery | Keep as one Notification BC | The rule engine (what to alert, when, deduplication) is core business logic; the delivery layer (push, email, SMS) is generic infrastructure. Different rates of change and different strategic value |
| Alerts & Detection as Core, not Supporting | Classify as Supporting | The intelligence that detects leaks, evaluates conditions, and classifies urgency is what makes the system reactive — it's a key differentiator, not standard infrastructure |
| Device Management as Supporting, not Core | Keep as Core | Device linking and telemetry ingestion are necessary infrastructure. The data foundation is important, but the value is in what Consumption Monitoring and Alerts do with the data, not in receiving it |
| Savings Tips as a separate Generic BC | Fold into Notification | Tips have zero dependencies on other BCs and zero domain rules — a static catalog with CRUD. Keeping it separate clarifies that it's independent content, not alert logic |
| Separate Device and Valve contexts | Merge into one device-service | Different criticality (valve = security-sensitive), different scaling needs (telemetry = high volume, valve = low volume), different rate of change |
| Single shared database with schema-per-service | Database-per-service | Team of 3 and single VPS — schema separation preserves logical boundaries. See ADR-002 |

---

## 7. How to update this map

1. Before adding a new microservice, verify whether it belongs to an existing bounded context.
2. If a context's ubiquitous language is changing, review whether the context should be split.
3. The context map MUST be synchronized with the C4 diagram in `05-architecture/overview.md`.
4. Record significant changes as ADRs in `05-architecture/decisions/records/`.
5. When adding a new BC, update the deployment mapping table (section 5) and the data ownership matrix in `06-data/models.md`.

---

## Correlations

- Entities per BC → `02-domain/entities-and-rules.md`
- Domain events per BC → `02-domain/domain-events.md`
- Architecture overview (C4, service catalog) → `05-architecture/overview.md`
- Data models per schema → `06-data/models.md`
- User stories referenced → `04-requirements/user-stories.md`
- ADRs → `05-architecture/decisions/records/`
