# Data Models per Service

> Save Your Water — SQL Server (`sy_water_db`), schema-per-service.
> See ADR-002 for the shared database decision.
> All DDL uses T-SQL (SQL Server) syntax.

---

## Data modeling principles

### 1. Schema-per-Service (shared database)

All services share a single SQL Server instance. Each service owns its own schema.
No service may query another service's schema — inter-service data access is via REST or RabbitMQ events.

```
✓ auth-service  → iam.users
✓ place-service → place.places
✗ place-service → iam.users (direct JOIN)
```

### 2. Standard audit fields

All tables include:

```sql
id          UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
created_at  DATETIMEOFFSET   NOT NULL    DEFAULT SYSDATETIMEOFFSET(),
updated_at  DATETIMEOFFSET   NOT NULL    DEFAULT SYSDATETIMEOFFSET(),
deleted_at  DATETIMEOFFSET   NULL        -- NULL = active (soft delete)
```

### 3. Soft delete

All tables use soft delete. Records are never physically deleted.
All queries must include `WHERE deleted_at IS NULL` unless explicitly querying archived data.

### 4. Naming conventions

```sql
-- Schemas:     lowercase, service name          → iam, place, device
-- Tables:      snake_case, plural               → users, places, readings
-- Columns:     snake_case, descriptive          → flow_rate, last_report_at
-- FKs:         [referenced_table_singular]_id   → user_id, place_id
-- Indexes:     IX_[table]_[column(s)]           → IX_users_email
-- Constraints: CK_[table]_[rule]                → CK_users_auth_provider
-- Defaults:    DF_[table]_[column]              → DF_users_is_verified
-- Timestamps:  always DATETIMEOFFSET (with timezone)
-- IDs:         UNIQUEIDENTIFIER (UUID) — never INT/BIGINT
```

---

## Schema: iam (auth-service) — BC-01 IAM

**Owner:** auth-service (Spring Boot)
**Bounded Context:** BC-01 IAM (Supporting)

### Table: iam.users

**Purpose:** Registered platform users.

```sql
CREATE TABLE iam.users (
    id                    UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),

    -- Identity
    full_name             NVARCHAR(200)    NOT NULL,
    email                 NVARCHAR(320)    NOT NULL,
    identity_document     NVARCHAR(50)     NOT NULL,
    phone_country_code    NVARCHAR(5)      NOT NULL,
    phone_number          NVARCHAR(20)     NOT NULL,
    photo_url             NVARCHAR(500)    NULL,

    -- Authentication
    password_hash         NVARCHAR(500)    NULL,       -- NULL for Google OAuth accounts
    auth_provider         NVARCHAR(10)     NOT NULL DEFAULT 'LOCAL'
                          CONSTRAINT CK_users_auth_provider CHECK (auth_provider IN ('LOCAL', 'GOOGLE')),

    -- Verification & blocking
    is_verified           BIT              NOT NULL DEFAULT 0,
    is_blocked            BIT              NOT NULL DEFAULT 0,
    blocked_until         DATETIMEOFFSET   NULL,
    failed_login_attempts INT              NOT NULL DEFAULT 0,

    -- Audit
    created_at            DATETIMEOFFSET   NOT NULL DEFAULT SYSDATETIMEOFFSET(),
    updated_at            DATETIMEOFFSET   NOT NULL DEFAULT SYSDATETIMEOFFSET(),
    deleted_at            DATETIMEOFFSET   NULL,

    -- Constraints
    CONSTRAINT UQ_users_email             UNIQUE (email),
    CONSTRAINT UQ_users_identity_document UNIQUE (identity_document)
);

CREATE INDEX IX_users_email    ON iam.users (email) WHERE deleted_at IS NULL;
CREATE INDEX IX_users_deleted  ON iam.users (deleted_at) WHERE deleted_at IS NULL;
```

---

### Table: iam.verification_tokens

**Purpose:** Email verification and password reset tokens.

```sql
CREATE TABLE iam.verification_tokens (
    id          UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    user_id     UNIQUEIDENTIFIER NOT NULL REFERENCES iam.users(id),

    token       NVARCHAR(500)    NOT NULL,
    token_type  NVARCHAR(20)     NOT NULL
                CONSTRAINT CK_vtokens_type CHECK (token_type IN ('EMAIL_VERIFICATION', 'PASSWORD_RESET')),
    expires_at  DATETIMEOFFSET   NOT NULL,
    used_at     DATETIMEOFFSET   NULL,       -- NULL = not yet used

    created_at  DATETIMEOFFSET   NOT NULL DEFAULT SYSDATETIMEOFFSET(),
    updated_at  DATETIMEOFFSET   NOT NULL DEFAULT SYSDATETIMEOFFSET(),
    deleted_at  DATETIMEOFFSET   NULL
);

CREATE INDEX IX_vtokens_token   ON iam.verification_tokens (token) WHERE used_at IS NULL;
CREATE INDEX IX_vtokens_user_id ON iam.verification_tokens (user_id);
```

---

### Table: iam.sessions

**Purpose:** Active user sessions (JWT refresh tokens).

```sql
CREATE TABLE iam.sessions (
    id              UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    user_id         UNIQUEIDENTIFIER NOT NULL REFERENCES iam.users(id),

    refresh_token   NVARCHAR(500)    NOT NULL,
    device_info     NVARCHAR(500)    NULL,       -- browser/device description
    ip_address      NVARCHAR(45)     NULL,
    expires_at      DATETIMEOFFSET   NOT NULL,
    revoked_at      DATETIMEOFFSET   NULL,       -- NULL = active

    created_at      DATETIMEOFFSET   NOT NULL DEFAULT SYSDATETIMEOFFSET(),
    updated_at      DATETIMEOFFSET   NOT NULL DEFAULT SYSDATETIMEOFFSET(),
    deleted_at      DATETIMEOFFSET   NULL
);

CREATE INDEX IX_sessions_user_id       ON iam.sessions (user_id) WHERE revoked_at IS NULL;
CREATE INDEX IX_sessions_refresh_token ON iam.sessions (refresh_token) WHERE revoked_at IS NULL;
```

---

### Table: iam.login_attempts

**Purpose:** Audit log of failed login attempts. Cleared on successful login.

```sql
CREATE TABLE iam.login_attempts (
    id          UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    user_id     UNIQUEIDENTIFIER NOT NULL REFERENCES iam.users(id),

    ip_address  NVARCHAR(45)     NULL,
    device_info NVARCHAR(500)    NULL,
    attempted_at DATETIMEOFFSET  NOT NULL DEFAULT SYSDATETIMEOFFSET(),

    created_at  DATETIMEOFFSET   NOT NULL DEFAULT SYSDATETIMEOFFSET(),
    updated_at  DATETIMEOFFSET   NOT NULL DEFAULT SYSDATETIMEOFFSET(),
    deleted_at  DATETIMEOFFSET   NULL
);

CREATE INDEX IX_login_attempts_user_id ON iam.login_attempts (user_id);
```

---

## Schema: place (place-service) — BC-02 Place Management

**Owner:** place-service
**Bounded Context:** BC-02 Place Management (Supporting)

### Table: place.places

**Purpose:** Monitored locations (homes or businesses).

```sql
CREATE TABLE place.places (
    id               UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    owner_id         UNIQUEIDENTIFIER NOT NULL,   -- references iam.users via app-level, no FK cross-schema

    -- Place info
    name             NVARCHAR(200)    NOT NULL,
    street_address   NVARCHAR(500)    NOT NULL,
    country          NVARCHAR(3)      NOT NULL
                     CONSTRAINT CK_places_country CHECK (country IN ('CO', 'EC', 'US')),
    department       NVARCHAR(200)    NOT NULL,
    city             NVARCHAR(200)    NOT NULL,
    postal_code      NVARCHAR(20)     NULL,
    stratum          INT              NULL
                     CONSTRAINT CK_places_stratum CHECK (stratum IS NULL OR stratum BETWEEN 1 AND 6),
    inhabitants      INT              NOT NULL DEFAULT 1
                     CONSTRAINT CK_places_inhabitants CHECK (inhabitants >= 1),

    -- Configuration
    measurement_unit NVARCHAR(15)     NOT NULL DEFAULT 'LITERS'
                     CONSTRAINT CK_places_unit CHECK (measurement_unit IN ('LITERS', 'CUBIC_METERS', 'GALLONS')),
    currency         NVARCHAR(3)      NOT NULL DEFAULT 'COP',
    linked_device_id UNIQUEIDENTIFIER NULL,       -- references device.devices via app-level

    -- Audit
    created_at       DATETIMEOFFSET   NOT NULL DEFAULT SYSDATETIMEOFFSET(),
    updated_at       DATETIMEOFFSET   NOT NULL DEFAULT SYSDATETIMEOFFSET(),
    deleted_at       DATETIMEOFFSET   NULL
);

CREATE UNIQUE INDEX UX_places_owner_name ON place.places (owner_id, name) WHERE deleted_at IS NULL;
CREATE INDEX IX_places_owner_id          ON place.places (owner_id) WHERE deleted_at IS NULL;
```

---

### Table: place.members *(Entrega 3)*

**Purpose:** Users with access to a place (Owner or Member).

```sql
CREATE TABLE place.members (
    id          UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    place_id    UNIQUEIDENTIFIER NOT NULL REFERENCES place.places(id),
    user_id     UNIQUEIDENTIFIER NOT NULL,   -- references iam.users via app-level

    role        NVARCHAR(10)     NOT NULL
                CONSTRAINT CK_members_role CHECK (role IN ('OWNER', 'MEMBER')),

    -- Permissions (comma-separated, only applies to MEMBER role; OWNER has all implicitly)
    permissions NVARCHAR(500)    NULL,
    -- Valid values: VIEW_CONSUMPTION, VIEW_DEVICE_STATUS, CONTROL_VALVE, VIEW_ALERTS, VIEW_BILLS

    created_at  DATETIMEOFFSET   NOT NULL DEFAULT SYSDATETIMEOFFSET(),
    updated_at  DATETIMEOFFSET   NOT NULL DEFAULT SYSDATETIMEOFFSET(),
    deleted_at  DATETIMEOFFSET   NULL
);

CREATE UNIQUE INDEX UX_members_place_user ON place.members (place_id, user_id) WHERE deleted_at IS NULL;
```

---

### Table: place.invitations *(Entrega 3)*

**Purpose:** Pending invitations to join a place as Member.

```sql
CREATE TABLE place.invitations (
    id             UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    place_id       UNIQUEIDENTIFIER NOT NULL REFERENCES place.places(id),
    invited_email  NVARCHAR(320)    NOT NULL,
    invited_by     UNIQUEIDENTIFIER NOT NULL,   -- references iam.users via app-level

    -- Permissions the member will receive upon acceptance
    permissions    NVARCHAR(500)    NOT NULL,
    -- Valid values: VIEW_CONSUMPTION, VIEW_DEVICE_STATUS, CONTROL_VALVE, VIEW_ALERTS, VIEW_BILLS

    status         NVARCHAR(10)     NOT NULL DEFAULT 'PENDING'
                   CONSTRAINT CK_invitations_status CHECK (status IN ('PENDING', 'ACCEPTED', 'EXPIRED', 'REJECTED')),
    expires_at     DATETIMEOFFSET   NOT NULL,
    responded_at   DATETIMEOFFSET   NULL,

    created_at     DATETIMEOFFSET   NOT NULL DEFAULT SYSDATETIMEOFFSET(),
    updated_at     DATETIMEOFFSET   NOT NULL DEFAULT SYSDATETIMEOFFSET(),
    deleted_at     DATETIMEOFFSET   NULL
);

CREATE INDEX IX_invitations_place_id ON place.invitations (place_id) WHERE deleted_at IS NULL;
CREATE INDEX IX_invitations_email    ON place.invitations (invited_email) WHERE status = 'PENDING';
```

---

## Schema: device (device-service) — BC-03 Device Management

**Owner:** device-service
**Bounded Context:** BC-03 Device Management (Supporting)

### Table: device.devices

**Purpose:** Registered IoT units (ESP32 + sensor + electrovalve).

```sql
CREATE TABLE device.devices (
    id                    UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),

    hardware_id           NVARCHAR(100)    NOT NULL,   -- physical serial number / QR code
    linked_place_id       UNIQUEIDENTIFIER NULL,       -- references place.places via app-level
    linked_by             UNIQUEIDENTIFIER NULL,       -- user who linked it

    -- Status
    status                NVARCHAR(20)     NOT NULL DEFAULT 'NEVER_REPORTED'
                          CONSTRAINT CK_devices_status CHECK (status IN ('CONNECTED', 'DISCONNECTED', 'NEVER_REPORTED')),
    last_report_at        DATETIMEOFFSET   NULL,
    disconnect_threshold  INT              NOT NULL DEFAULT 15,  -- minutes

    -- Link timestamps
    linked_at             DATETIMEOFFSET   NULL,
    unlinked_at           DATETIMEOFFSET   NULL,

    -- Audit
    created_at            DATETIMEOFFSET   NOT NULL DEFAULT SYSDATETIMEOFFSET(),
    updated_at            DATETIMEOFFSET   NOT NULL DEFAULT SYSDATETIMEOFFSET(),
    deleted_at            DATETIMEOFFSET   NULL,

    CONSTRAINT UQ_devices_hardware_id UNIQUE (hardware_id)
);

CREATE INDEX IX_devices_linked_place ON device.devices (linked_place_id) WHERE linked_place_id IS NOT NULL;
CREATE INDEX IX_devices_status       ON device.devices (status) WHERE deleted_at IS NULL;
```

---

### Table: device.readings

**Purpose:** Raw flow sensor measurements received via MQTT.

```sql
CREATE TABLE device.readings (
    id          UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    device_id   UNIQUEIDENTIFIER NOT NULL REFERENCES device.devices(id),
    place_id    UNIQUEIDENTIFIER NOT NULL,   -- denormalized from device link at ingestion time

    -- Measurement
    flow_rate   DECIMAL(10, 4)   NOT NULL,   -- liters per minute
    volume      DECIMAL(10, 4)   NOT NULL,   -- liters in this interval
    timestamp   DATETIMEOFFSET   NOT NULL,   -- device clock
    received_at DATETIMEOFFSET   NOT NULL DEFAULT SYSDATETIMEOFFSET(),  -- server clock

    -- Audit (no updated_at — readings are immutable)
    created_at  DATETIMEOFFSET   NOT NULL DEFAULT SYSDATETIMEOFFSET(),
    deleted_at  DATETIMEOFFSET   NULL,

    -- Deduplication
    CONSTRAINT UQ_readings_device_timestamp UNIQUE (device_id, timestamp)
);

CREATE INDEX IX_readings_place_timestamp ON device.readings (place_id, timestamp DESC);
CREATE INDEX IX_readings_device_id       ON device.readings (device_id, timestamp DESC);

-- Constraint: flow_rate and volume must be non-negative
ALTER TABLE device.readings ADD CONSTRAINT CK_readings_flow_rate CHECK (flow_rate >= 0);
ALTER TABLE device.readings ADD CONSTRAINT CK_readings_volume    CHECK (volume >= 0);
```

**Performance note:** This table will have the highest write volume. If it grows beyond SQL Server's comfort zone, consider partitioning by month on `timestamp` or extracting to a time-series database.

---

## Schema: consumption (consumption-service) — BC-04, BC-05, BC-06, BC-10

**Owner:** consumption-service
**Bounded Contexts:** BC-04 Consumption Monitoring (Core), BC-05 Consumption Goals (Supporting), BC-06 Billing & Tariffs (Supporting), BC-10 Reports & Export (Generic)

### Table: consumption.consumption_records — BC-04 Consumption Monitoring

**Purpose:** Aggregated consumption data by place and time interval.

```sql
CREATE TABLE consumption.consumption_records (
    id            UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    place_id      UNIQUEIDENTIFIER NOT NULL,   -- references place.places via app-level

    -- Time window
    period_type   NVARCHAR(10)     NOT NULL
                  CONSTRAINT CK_consumption_period CHECK (period_type IN ('HOUR', 'DAY', 'WEEK', 'MONTH')),
    period_start  DATETIMEOFFSET   NOT NULL,
    period_end    DATETIMEOFFSET   NOT NULL,

    -- Aggregated data
    total_volume  DECIMAL(12, 4)   NOT NULL DEFAULT 0,   -- liters
    reading_count INT              NOT NULL DEFAULT 0,

    -- Audit
    created_at    DATETIMEOFFSET   NOT NULL DEFAULT SYSDATETIMEOFFSET(),
    updated_at    DATETIMEOFFSET   NOT NULL DEFAULT SYSDATETIMEOFFSET(),
    deleted_at    DATETIMEOFFSET   NULL
);

CREATE UNIQUE INDEX UX_consumption_place_period ON consumption.consumption_records
    (place_id, period_type, period_start) WHERE deleted_at IS NULL;
CREATE INDEX IX_consumption_place_id ON consumption.consumption_records
    (place_id, period_start DESC) WHERE deleted_at IS NULL;
```

---

### Table: consumption.goals — BC-05 Consumption Goals *(Entrega 2)*

**Purpose:** Monthly water consumption targets per place.

```sql
CREATE TABLE consumption.goals (
    id             UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    place_id       UNIQUEIDENTIFIER NOT NULL,
    set_by         UNIQUEIDENTIFIER NOT NULL,   -- user who set the goal

    month          NVARCHAR(7)      NOT NULL,   -- format: '2026-09'
    target_volume  DECIMAL(12, 4)   NOT NULL,   -- in the place's measurement unit
    unit           NVARCHAR(15)     NOT NULL,
    source         NVARCHAR(10)     NOT NULL DEFAULT 'MANUAL'
                   CONSTRAINT CK_goals_source CHECK (source IN ('MANUAL', 'SUGGESTED')),

    created_at     DATETIMEOFFSET   NOT NULL DEFAULT SYSDATETIMEOFFSET(),
    updated_at     DATETIMEOFFSET   NOT NULL DEFAULT SYSDATETIMEOFFSET(),
    deleted_at     DATETIMEOFFSET   NULL
);

CREATE UNIQUE INDEX UX_goals_place_month ON consumption.goals (place_id, month) WHERE deleted_at IS NULL;
```

---

### Table: consumption.bills — BC-06 Billing & Tariffs *(Entrega 3)*

**Purpose:** Utility bills registered by the user for comparison against sensor data.

```sql
CREATE TABLE consumption.bills (
    id                UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    place_id          UNIQUEIDENTIFIER NOT NULL,
    registered_by     UNIQUEIDENTIFIER NOT NULL,

    -- Bill data
    billed_volume     DECIMAL(12, 4)   NOT NULL,   -- consumption billed by utility
    total_amount      DECIMAL(12, 2)   NOT NULL,   -- total bill amount
    currency          NVARCHAR(3)      NOT NULL,
    period_start      DATE             NOT NULL,
    period_end        DATE             NOT NULL,
    payment_due_date  DATE             NOT NULL,
    is_paid           BIT              NOT NULL DEFAULT 0,

    created_at        DATETIMEOFFSET   NOT NULL DEFAULT SYSDATETIMEOFFSET(),
    updated_at        DATETIMEOFFSET   NOT NULL DEFAULT SYSDATETIMEOFFSET(),
    deleted_at        DATETIMEOFFSET   NULL,

    CONSTRAINT CK_bills_period CHECK (period_start < period_end)
);

CREATE INDEX IX_bills_place_id ON consumption.bills (place_id, period_start DESC) WHERE deleted_at IS NULL;
```

---

### Table: consumption.tariffs — BC-06 Billing & Tariffs *(Entrega 3)*

**Purpose:** Price per unit of water, used to estimate monetary cost of consumption.

```sql
CREATE TABLE consumption.tariffs (
    id              UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    place_id        UNIQUEIDENTIFIER NOT NULL,

    price_per_unit  DECIMAL(10, 4)   NOT NULL,   -- price per measurement unit
    unit            NVARCHAR(15)     NOT NULL,
    currency        NVARCHAR(3)      NOT NULL,
    fixed_charge    DECIMAL(10, 2)   NOT NULL DEFAULT 0,   -- optional monthly fixed charge
    effective_from  DATE             NOT NULL,
    effective_until DATE             NULL,       -- NULL = current tariff

    created_at      DATETIMEOFFSET   NOT NULL DEFAULT SYSDATETIMEOFFSET(),
    updated_at      DATETIMEOFFSET   NOT NULL DEFAULT SYSDATETIMEOFFSET(),
    deleted_at      DATETIMEOFFSET   NULL
);

CREATE INDEX IX_tariffs_place_id ON consumption.tariffs (place_id, effective_from DESC) WHERE deleted_at IS NULL;
```

---

## Schema: valve (valve-service) — BC-07 Valve Control *(Entrega 2)*

**Owner:** valve-service
**Bounded Context:** BC-07 Valve Control (Core)

### Table: valve.valves

**Purpose:** Logical representation of the electrovalve at each place.

```sql
CREATE TABLE valve.valves (
    id                    UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    place_id              UNIQUEIDENTIFIER NOT NULL,
    device_id             UNIQUEIDENTIFIER NOT NULL,

    state                 NVARCHAR(20)     NOT NULL DEFAULT 'OPEN'
                          CONSTRAINT CK_valves_state CHECK (state IN ('OPEN', 'CLOSED', 'NO_COMMUNICATION')),
    last_confirmed_at     DATETIMEOFFSET   NULL,
    auto_shutoff_enabled  BIT              NOT NULL DEFAULT 0,

    created_at            DATETIMEOFFSET   NOT NULL DEFAULT SYSDATETIMEOFFSET(),
    updated_at            DATETIMEOFFSET   NOT NULL DEFAULT SYSDATETIMEOFFSET(),
    deleted_at            DATETIMEOFFSET   NULL
);

CREATE UNIQUE INDEX UX_valves_place ON valve.valves (place_id) WHERE deleted_at IS NULL;
```

---

### Table: valve.valve_actions

**Purpose:** Audit log of every open/close action on a valve.

```sql
CREATE TABLE valve.valve_actions (
    id            UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    valve_id      UNIQUEIDENTIFIER NOT NULL REFERENCES valve.valves(id),
    user_id       UNIQUEIDENTIFIER NOT NULL,

    action        NVARCHAR(10)     NOT NULL
                  CONSTRAINT CK_vactions_action CHECK (action IN ('OPEN', 'CLOSE')),
    origin        NVARCHAR(15)     NOT NULL
                  CONSTRAINT CK_vactions_origin CHECK (origin IN ('MANUAL', 'AUTO_LEAK', 'AUTO_GOAL')),
    result        NVARCHAR(10)     NOT NULL
                  CONSTRAINT CK_vactions_result CHECK (result IN ('SUCCESS', 'FAILED', 'TIMEOUT')),
    requested_at  DATETIMEOFFSET   NOT NULL,
    confirmed_at  DATETIMEOFFSET   NULL,

    created_at    DATETIMEOFFSET   NOT NULL DEFAULT SYSDATETIMEOFFSET(),
    updated_at    DATETIMEOFFSET   NOT NULL DEFAULT SYSDATETIMEOFFSET(),
    deleted_at    DATETIMEOFFSET   NULL
);

CREATE INDEX IX_vactions_valve_id ON valve.valve_actions (valve_id, requested_at DESC);
```

---

### Table: valve.shutoff_rules *(Entrega 3)*

**Purpose:** Auto-shutoff rule configuration per valve.

```sql
CREATE TABLE valve.shutoff_rules (
    id            UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    valve_id      UNIQUEIDENTIFIER NOT NULL REFERENCES valve.valves(id),

    trigger_type  NVARCHAR(10)     NOT NULL
                  CONSTRAINT CK_srules_trigger CHECK (trigger_type IN ('LEAK', 'GOAL')),
    is_enabled    BIT              NOT NULL DEFAULT 0,

    created_at    DATETIMEOFFSET   NOT NULL DEFAULT SYSDATETIMEOFFSET(),
    updated_at    DATETIMEOFFSET   NOT NULL DEFAULT SYSDATETIMEOFFSET(),
    deleted_at    DATETIMEOFFSET   NULL
);

CREATE UNIQUE INDEX UX_srules_valve_trigger ON valve.shutoff_rules (valve_id, trigger_type) WHERE deleted_at IS NULL;
```

---

## Schema: notification (notification-service) — BC-08, BC-09, BC-11 *(Entrega 2)*

**Owner:** notification-service
**Bounded Contexts:** BC-08 Alerts & Detection (Core), BC-09 Notification Delivery (Generic), BC-11 Savings Tips (Generic)

### Table: notification.alerts — BC-08 Alerts & Detection

**Purpose:** System-generated alerts triggered by domain events.

```sql
CREATE TABLE notification.alerts (
    id          UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    place_id    UNIQUEIDENTIFIER NOT NULL,
    user_id     UNIQUEIDENTIFIER NOT NULL,   -- alert recipient

    type        NVARCHAR(30)     NOT NULL
                CONSTRAINT CK_alerts_type CHECK (type IN (
                    'LEAK_SUSPECTED', 'VALVE_CLOSED', 'GOAL_EXCEEDED', 'GOAL_WARNING',
                    'SENSOR_DISCONNECTED', 'BILL_MISMATCH', 'BILL_REMINDER', 'PAYMENT_REMINDER'
                )),
    urgency     NVARCHAR(15)     NOT NULL
                CONSTRAINT CK_alerts_urgency CHECK (urgency IN ('CRITICAL', 'IMPORTANT', 'INFORMATIVE')),
    title       NVARCHAR(200)    NOT NULL,
    body        NVARCHAR(2000)   NOT NULL,
    is_read     BIT              NOT NULL DEFAULT 0,
    channels    NVARCHAR(200)    NOT NULL,   -- comma-separated: 'IN_APP,PUSH,EMAIL,SMS'

    created_at  DATETIMEOFFSET   NOT NULL DEFAULT SYSDATETIMEOFFSET(),
    updated_at  DATETIMEOFFSET   NOT NULL DEFAULT SYSDATETIMEOFFSET(),
    deleted_at  DATETIMEOFFSET   NULL
);

CREATE INDEX IX_alerts_user_id   ON notification.alerts (user_id, created_at DESC) WHERE deleted_at IS NULL;
CREATE INDEX IX_alerts_place_id  ON notification.alerts (place_id, created_at DESC) WHERE deleted_at IS NULL;
CREATE INDEX IX_alerts_unread    ON notification.alerts (user_id) WHERE is_read = 0 AND deleted_at IS NULL;
```

---

### Table: notification.notification_preferences — BC-09 Notification Delivery

**Purpose:** User's channel preferences per urgency level.

```sql
CREATE TABLE notification.notification_preferences (
    id              UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    user_id         UNIQUEIDENTIFIER NOT NULL,

    urgency_level   NVARCHAR(15)     NOT NULL
                    CONSTRAINT CK_nprefs_urgency CHECK (urgency_level IN ('CRITICAL', 'IMPORTANT', 'INFORMATIVE')),
    in_app_enabled  BIT              NOT NULL DEFAULT 1,
    push_enabled    BIT              NOT NULL DEFAULT 1,
    email_enabled   BIT              NOT NULL DEFAULT 0,
    sms_enabled     BIT              NOT NULL DEFAULT 0,

    created_at      DATETIMEOFFSET   NOT NULL DEFAULT SYSDATETIMEOFFSET(),
    updated_at      DATETIMEOFFSET   NOT NULL DEFAULT SYSDATETIMEOFFSET(),
    deleted_at      DATETIMEOFFSET   NULL
);

CREATE UNIQUE INDEX UX_nprefs_user_urgency ON notification.notification_preferences
    (user_id, urgency_level) WHERE deleted_at IS NULL;
```

---

### Table: notification.recommendations — BC-11 Savings Tips *(Entrega 4)*

**Purpose:** Generic water-saving tips and recommendations created by Admins. Displayed to all users.

```sql
CREATE TABLE notification.recommendations (
    id          UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    created_by  UNIQUEIDENTIFIER NOT NULL,   -- references iam.users via app-level (Admin)

    title       NVARCHAR(200)    NOT NULL,
    body        NVARCHAR(2000)   NOT NULL,
    category    NVARCHAR(20)     NOT NULL
                CONSTRAINT CK_recommendations_category CHECK (category IN ('SAVING_TIP', 'MAINTENANCE', 'SEASONAL', 'GENERAL')),
    is_active   BIT              NOT NULL DEFAULT 1,

    created_at  DATETIMEOFFSET   NOT NULL DEFAULT SYSDATETIMEOFFSET(),
    updated_at  DATETIMEOFFSET   NOT NULL DEFAULT SYSDATETIMEOFFSET(),
    deleted_at  DATETIMEOFFSET   NULL
);

CREATE INDEX IX_recommendations_active   ON notification.recommendations (is_active) WHERE deleted_at IS NULL;
CREATE INDEX IX_recommendations_category ON notification.recommendations (category) WHERE is_active = 1 AND deleted_at IS NULL;
```

---

## Data ownership matrix

| Entity / Data | Owner schema | Owner BC | How other services access |
|---------------|-------------|---------|--------------------------|
| User identity | `iam` | BC-01 IAM | REST API (`GET /users/:id`) or JWT claims |
| Place info | `place` | BC-02 Place Mgmt | REST API (`GET /places/:id`) |
| Members & invitations | `place` | BC-02 Place Mgmt | REST API (`GET /places/:id/members`) |
| Device registry | `device` | BC-03 Device Mgmt | REST API + `DeviceLinked` / `DeviceUnlinked` events |
| Raw readings | `device` | BC-03 Device Mgmt | `ReadingReceived` events via RabbitMQ |
| Aggregated consumption | `consumption` | BC-04 Monitoring | REST API (`GET /consumption/:placeId`) |
| Goals | `consumption` | BC-05 Goals | REST API |
| Bills & tariffs | `consumption` | BC-06 Billing | REST API |
| Reports | `consumption` | BC-10 Reports | REST API (read-only projections) |
| Valve state | `valve` | BC-07 Valve Control | REST API + `ValveClosed` / `ValveOpened` events |
| Valve action log | `valve` | BC-07 Valve Control | REST API |
| Alerts | `notification` | BC-08 Alerts | REST API (`GET /alerts`) |
| Notification preferences | `notification` | BC-09 Delivery | REST API |
| Recommendations | `notification` | BC-11 Tips | REST API (`GET /recommendations`) |

---

## Migration strategy

**Tool:** TBD — Flyway (Spring Boot services), EF Core Migrations (.NET services), or a unified tool.

**File naming convention:**

```
V{version}__{snake_case_description}.sql

Examples:
  V001__create_iam_schema.sql
  V002__create_users_table.sql
  V003__create_verification_tokens_table.sql
```

**Migration rules:**

- Migrations are ALWAYS forward-only
- One migration per logical change
- Seed data goes in separate migrations with prefix `S`: `S001__seed_...`
- Never modify a migration already executed in any environment
- Incompatible changes (rename, drop) use a 2-phase approach across releases

---

## ER Diagrams

### iam schema

```
┌──────────────────────┐
│      iam.users       │
├──────────────────────┤       ┌─────────────────────────────┐
│ id (PK)              │──┐    │  iam.verification_tokens    │
│ full_name            │  │    ├─────────────────────────────┤
│ email (UQ)           │  ├───▶│ user_id (FK)                │
│ identity_document(UQ)│  │    │ token, token_type           │
│ phone                │  │    │ expires_at, used_at         │
│ password_hash        │  │    └─────────────────────────────┘
│ auth_provider        │  │
│ is_verified          │  │    ┌─────────────────────────────┐
│ is_blocked           │  │    │      iam.sessions           │
│ failed_login_attempts│  │    ├─────────────────────────────┤
│ created_at           │  ├───▶│ user_id (FK)                │
│ deleted_at           │  │    │ refresh_token               │
└──────────────────────┘  │    │ expires_at, revoked_at      │
                          │    └─────────────────────────────┘
                          │
                          │    ┌─────────────────────────────┐
                          │    │    iam.login_attempts        │
                          │    ├─────────────────────────────┤
                          └───▶│ user_id (FK)                │
                               │ ip_address, attempted_at    │
                               └─────────────────────────────┘
```

### place schema

```
┌──────────────────────────┐
│      place.places        │
├──────────────────────────┤
│ id (PK)                  │──┐
│ owner_id                 │  │    ┌──────────────────────────────┐
│ name (UQ w/owner)        │  │    │      place.members           │
│ street_address           │  │    │         *(Entrega 3)*        │
│ country, department, city│  │    ├──────────────────────────────┤
│ stratum, inhabitants     │  ├───▶│ place_id (FK)                │
│ measurement_unit         │  │    │ user_id (UQ w/place)         │
│ currency                 │  │    │ role (OWNER / MEMBER)        │
│ linked_device_id         │  │    │ permissions                  │
└──────────────────────────┘  │    └──────────────────────────────┘
                              │
                              │    ┌──────────────────────────────┐
                              │    │    place.invitations         │
                              │    │         *(Entrega 3)*        │
                              │    ├──────────────────────────────┤
                              └───▶│ place_id (FK)                │
                                   │ invited_email, invited_by    │
                                   │ permissions                  │
                                   │ status, expires_at           │
                                   └──────────────────────────────┘
```

### device schema

```
┌────────────────────────┐
│    device.devices      │
├────────────────────────┤       ┌─────────────────────────┐
│ id (PK)                │──────▶│    device.readings      │
│ hardware_id (UQ)       │       ├─────────────────────────┤
│ linked_place_id        │       │ device_id (FK)          │
│ status                 │       │ place_id                │
│ last_report_at         │       │ flow_rate, volume       │
│ disconnect_threshold   │       │ timestamp (UQ w/device) │
└────────────────────────┘       │ received_at             │
                                 └─────────────────────────┘
```

### consumption schema

```
┌────────────────────────────────┐
│ consumption.consumption_records│
├────────────────────────────────┤
│ id (PK)                        │
│ place_id                       │       ┌─────────────────────────┐
│ period_type (UQ w/place+start) │       │   consumption.goals     │
│ period_start, period_end       │       │      *(Entrega 2)*      │
│ total_volume, reading_count    │       ├─────────────────────────┤
└────────────────────────────────┘       │ id (PK)                 │
                                         │ place_id (UQ w/month)   │
       ┌─────────────────────────┐       │ set_by                  │
       │  consumption.bills      │       │ month, target_volume    │
       │     *(Entrega 3)*       │       │ unit, source            │
       ├─────────────────────────┤       └─────────────────────────┘
       │ id (PK)                 │
       │ place_id                │       ┌─────────────────────────┐
       │ registered_by           │       │ consumption.tariffs     │
       │ billed_volume           │       │     *(Entrega 3)*       │
       │ total_amount, currency  │       ├─────────────────────────┤
       │ period_start, period_end│       │ id (PK)                 │
       │ payment_due_date        │       │ place_id                │
       │ is_paid                 │       │ price_per_unit, unit    │
       └─────────────────────────┘       │ currency, fixed_charge  │
                                         │ effective_from/until    │
                                         └─────────────────────────┘

All tables linked by place_id (app-level, no FK cross-schema)
```

### valve schema *(Entrega 2)*

```
┌────────────────────────┐
│     valve.valves       │
├────────────────────────┤       ┌───────────────────────────┐
│ id (PK)                │──┬───▶│   valve.valve_actions     │
│ place_id (UQ)          │  │    ├───────────────────────────┤
│ device_id              │  │    │ valve_id (FK)             │
│ state                  │  │    │ action, origin, result    │
│ auto_shutoff_enabled   │  │    │ requested_at, confirmed_at│
└────────────────────────┘  │    └───────────────────────────┘
                            │
                            │    ┌───────────────────────────┐
                            │    │   valve.shutoff_rules     │
                            │    │      *(Entrega 3)*        │
                            │    ├───────────────────────────┤
                            └───▶│ valve_id (FK)             │
                                 │ trigger_type (UQ w/valve) │
                                 │ is_enabled                │
                                 └───────────────────────────┘
```

### notification schema *(Entrega 2)*

```
┌─────────────────────────────┐
│     notification.alerts     │
├─────────────────────────────┤
│ id (PK)                     │
│ place_id, user_id           │
│ type, urgency               │       ┌─────────────────────────────────────┐
│ title, body                 │       │ notification.notification_preferences│
│ is_read, channels           │       ├─────────────────────────────────────┤
└─────────────────────────────┘       │ id (PK)                             │
                                      │ user_id (UQ w/urgency)              │
┌─────────────────────────────┐       │ urgency_level                       │
│notification.recommendations│       │ in_app, push, email, sms _enabled   │
│      *(Entrega 4)*         │       └─────────────────────────────────────┘
├─────────────────────────────┤
│ id (PK)                     │
│ created_by (Admin)          │
│ title, body                 │
│ category, is_active         │
└─────────────────────────────┘
```

---

## Correlations

- Bounded contexts and domain map → `02-domain/domain-map.md`
- Domain entities and aggregates → `02-domain/entities-and-rules.md`
- Domain events → `02-domain/domain-events.md`
- BC-to-service mapping → `05-architecture/overview.md` § 4
- Schema-per-service decision → `05-architecture/decisions/records/ADR-002-shared-database-schema-per-service.md`
- API contracts exposing this data → `07-api/contracts/openapi/`
