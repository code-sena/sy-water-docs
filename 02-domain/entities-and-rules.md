# Entities, Value Objects, and Business Rules

> Building blocks of the Save Your Water domain following DDD tactical patterns.
> Focused on MVP entities (Entrega 1 & 2). Future entities are marked accordingly.

---

## System Entities

### Entity: User

**Context:** IAM
**Description:** A person registered on the platform who can own or be invited to places.
**Identifier:** `userId` (UUID)

**Attributes:**

| Attribute | Type | Description | Required | Rules |
|-----------|------|-------------|---------|-------|
| userId | UUID | Unique identifier | Yes | Auto-generated |
| fullName | string | Full name | Yes | Min 2 chars |
| email | Email (VO) | Account email | Yes | Unique across system, immutable |
| identityDocument | string | Government-issued ID number | Yes | Unique across system |
| phone | Phone (VO) | Mobile number | Yes | Valid format per country |
| passwordHash | string | Bcrypt/Argon2 hashed password | Yes (local accounts) | Never stored in plain text |
| authProvider | enum | LOCAL or GOOGLE | Yes | Set on creation, immutable |
| isVerified | boolean | Email verification status | Yes | Default: false (true for Google accounts) |
| isBlocked | boolean | Account temporarily blocked | Yes | Default: false |
| blockedUntil | DateTime | Block expiration timestamp | No | Set after 5 failed login attempts |
| failedLoginAttempts | int | Consecutive failed logins | Yes | Reset to 0 on successful login |
| photoUrl | string | Profile picture URL | No | — |
| createdAt | DateTime | Registration date | Yes | Immutable |
| updatedAt | DateTime | Last modification | Yes | Auto-updated |

**Lifecycle:**

```
[Unverified] ──(verify email)──▶ [Active] ──(5 failed logins)──▶ [Blocked]
                                    ▲                                  │
                                    └────────(15 min elapsed)──────────┘
```

| State | Description | Allowed transitions |
|-------|-------------|---------------------|
| Unverified | Registered but email not confirmed. Cannot log in. | → Active (via email link) |
| Active | Verified and operational | → Blocked (5 failed attempts) |
| Blocked | Temporarily locked out for 15 minutes | → Active (timer expires or admin unblocks) |

**Invariants:**

```
INV-U001: Password must have ≥ 8 chars, 1 uppercase, 1 number, 1 special character
INV-U002: An unverified account cannot start a session
INV-U003: A blocked account cannot start a session until blockedUntil has passed
INV-U004: Email cannot be changed after creation
INV-U005: Password is never stored or transmitted in plain text
INV-U006: Google OAuth accounts skip email verification (isVerified = true on creation)
INV-U007: After 5 consecutive failed login attempts, the account is blocked for 15 minutes
INV-U008: Failed login attempts reset to 0 on successful login
```

---

### Entity: Place

**Context:** Place Management
**Description:** A physical location (home or business) where water consumption is monitored.
**Identifier:** `placeId` (UUID)

**Attributes:**

| Attribute | Type | Description | Required | Rules |
|-----------|------|-------------|---------|-------|
| placeId | UUID | Unique identifier | Yes | Auto-generated |
| ownerId | UUID | User who created and owns this place | Yes | Immutable after creation |
| name | string | Reference name (e.g., "Mi Casa", "Oficina Centro") | Yes | Unique per owner |
| address | Address (VO) | Full address | Yes | Valid for the registered country |
| country | string | Country code | Yes | CO, EC, US |
| department | string | Department/State | Yes | — |
| city | string | City | Yes | — |
| stratum | int | Socioeconomic stratum (Colombia only) | No | 1–6, only when country = CO |
| inhabitants | int | Number of people at the place | Yes | Min 1 |
| measurementUnit | enum | LITERS, CUBIC_METERS, GALLONS | Yes | Default: LITERS |
| currency | string | Currency code based on country | Yes | COP, USD |
| linkedDeviceId | UUID | Currently linked device (if any) | No | At most one device per place |
| createdAt | DateTime | Creation date | Yes | Immutable |
| updatedAt | DateTime | Last modification | Yes | Auto-updated |

**Lifecycle:**

```
[Active] ──(owner deletes)──▶ [Deleted]
```

**Invariants:**

```
INV-P001: Place name must be unique within the same owner's places
INV-P002: A place cannot be deleted if it has a linked device — unlink first
INV-P003: A place has at most one linked device at any time
INV-P004: Owner cannot be changed after creation
INV-P005: Stratum is required and valid (1–6) only when country is CO
INV-P006: Deletion is a soft-delete; consumption history is preserved for archive
```

---

### Entity: Device

**Context:** Device Management
**Description:** An IoT unit (ESP32 + YF-S201 sensor + electrovalve) that measures water flow and controls the valve.
**Identifier:** `deviceId` (UUID), `hardwareId` (string — physical identifier printed on the device)

**Attributes:**

| Attribute | Type | Description | Required | Rules |
|-----------|------|-------------|---------|-------|
| deviceId | UUID | Internal unique identifier | Yes | Auto-generated on first registration |
| hardwareId | string | Physical identifier (QR code / serial number) | Yes | Unique, immutable |
| linkedPlaceId | UUID | Place this device is linked to | No | Null when unlinked |
| status | enum | CONNECTED, DISCONNECTED, NEVER_REPORTED | Yes | Default: NEVER_REPORTED |
| lastReportAt | DateTime | Timestamp of last received reading | No | Null until first reading |
| disconnectThreshold | Duration | Time without reports before marking disconnected | Yes | Default: 15 minutes |
| createdAt | DateTime | First registration date | Yes | Immutable |
| updatedAt | DateTime | Last modification | Yes | Auto-updated |

**Lifecycle:**

```
[Unlinked] ──(link to place)──▶ [Linked / Never Reported]
                                        │
                                  (first reading)
                                        ▼
                                [Linked / Connected] ◀──(reading received)──┐
                                        │                                    │
                                  (no reading for 15 min)                    │
                                        ▼                                    │
                                [Linked / Disconnected] ─────────────────────┘
                                        │
                                  (unlink)
                                        ▼
                                   [Unlinked]
```

**Invariants:**

```
INV-D001: A device can be linked to at most one place at a time
INV-D002: Only authenticated/recognized devices can be linked (hardware ID must exist in the registry)
INV-D003: Unlinking preserves the place's consumption history
INV-D004: Duplicate readings (same device + same timestamp) are rejected
INV-D005: Readings with negative values or physically implausible flow rates are rejected
INV-D006: Device is considered DISCONNECTED after disconnectThreshold without a reading
```

---

### Entity: Reading

**Context:** Device Management
**Description:** A single flow measurement sent by an ESP32 device via MQTT.
**Identifier:** `readingId` (UUID)

**Attributes:**

| Attribute | Type | Description | Required | Rules |
|-----------|------|-------------|---------|-------|
| readingId | UUID | Unique identifier | Yes | Auto-generated |
| deviceId | UUID | Device that sent this reading | Yes | Must reference a linked device |
| placeId | UUID | Place associated at the time of reading | Yes | Denormalized from device link |
| flowRate | decimal | Instantaneous flow rate (liters/min) | Yes | Must be ≥ 0 and within physical range |
| volume | decimal | Volume measured in this interval (liters) | Yes | Must be ≥ 0 |
| timestamp | DateTime | When the measurement was taken (device clock) | Yes | Cannot be in the future |
| receivedAt | DateTime | When the backend received the reading | Yes | Server clock |

**Invariants:**

```
INV-R001: No two readings from the same device can have the same timestamp (deduplication)
INV-R002: flowRate must be ≥ 0 and ≤ max physical range of YF-S201 (≈ 30 L/min)
INV-R003: volume must be ≥ 0
INV-R004: timestamp cannot be in the future (tolerance: 5 minutes for clock drift)
```

---

### Entity: Valve *(Entrega 2)*

**Context:** Valve Control
**Description:** Logical representation of the electrovalve at a place's water inlet.
**Identifier:** `valveId` (UUID)

**Attributes:**

| Attribute | Type | Description | Required | Rules |
|-----------|------|-------------|---------|-------|
| valveId | UUID | Unique identifier | Yes | Auto-generated |
| placeId | UUID | Place this valve belongs to | Yes | One-to-one with place's device |
| deviceId | UUID | Device that physically controls this valve | Yes | — |
| state | enum | OPEN, CLOSED, NO_COMMUNICATION | Yes | Reflects last confirmed state from device |
| lastConfirmedAt | DateTime | When the device last confirmed the state | No | — |
| autoShutoffEnabled | boolean | Whether auto-shutoff rules are active | Yes | Default: false |

**Invariants:**

```
INV-V001: Only a user with the Owner role on the place can send open/close commands
INV-V002: A close command requires a confirmation code sent to the owner
INV-V003: State reflects the device's confirmed state, not the last command sent
INV-V004: If the device does not confirm within 30 seconds, the command is marked as failed
INV-V005: Auto-shutoff rules are disabled by default and can only be enabled by the owner
INV-V006: Auto-shutoff never executes without explicit user confirmation
INV-V007: The system never reopens the valve automatically — reopening is always manual
```

---

### Entity: ValveAction *(Entrega 2)*

**Context:** Valve Control
**Description:** An auditable record of every open/close action on a valve.
**Identifier:** `actionId` (UUID)

**Attributes:**

| Attribute | Type | Description | Required | Rules |
|-----------|------|-------------|---------|-------|
| actionId | UUID | Unique identifier | Yes | Auto-generated |
| valveId | UUID | Valve acted upon | Yes | — |
| userId | UUID | User who initiated the action | Yes | Must be Owner of the place |
| action | enum | OPEN, CLOSE | Yes | — |
| origin | enum | MANUAL, AUTO_LEAK, AUTO_GOAL | Yes | — |
| result | enum | SUCCESS, FAILED, TIMEOUT | Yes | — |
| requestedAt | DateTime | When the command was sent | Yes | — |
| confirmedAt | DateTime | When the device confirmed | No | Null if failed/timeout |

---

### Entity: Alert *(Entrega 2)*

**Context:** Notification
**Description:** A system-generated notification triggered by a domain event.
**Identifier:** `alertId` (UUID)

**Attributes:**

| Attribute | Type | Description | Required | Rules |
|-----------|------|-------------|---------|-------|
| alertId | UUID | Unique identifier | Yes | Auto-generated |
| placeId | UUID | Place this alert relates to | Yes | — |
| type | enum | LEAK_SUSPECTED, VALVE_CLOSED, GOAL_EXCEEDED, GOAL_WARNING, SENSOR_DISCONNECTED, BILL_MISMATCH, BILL_REMINDER, PAYMENT_REMINDER | Yes | — |
| urgency | enum | CRITICAL, IMPORTANT, INFORMATIVE | Yes | Derived from type |
| title | string | Short description | Yes | — |
| body | string | Detailed message | Yes | — |
| isRead | boolean | Whether the user has seen it | Yes | Default: false |
| channels | list | Channels used for delivery | Yes | Based on urgency + user preferences |
| createdAt | DateTime | When the alert was generated | Yes | Immutable |

---

## System Value Objects

### Value Object: Email

| Attribute | Type | Description |
|-----------|------|-------------|
| value | string | Email address in lowercase |

**Validation:** Must match standard email format. Converted to lowercase on creation.

---

### Value Object: Phone

| Attribute | Type | Description |
|-----------|------|-------------|
| countryCode | string | Country dialing code (e.g., +57) |
| number | string | Phone number without country code |

**Validation:** Must be a valid phone number for the given country code.

---

### Value Object: Address

| Attribute | Type | Description |
|-----------|------|-------------|
| street | string | Street address |
| city | string | City |
| department | string | Department or state |
| country | string | Country code (CO, EC, US) |
| postalCode | string | Postal/ZIP code (optional) |

**Validation:** Address must be valid for the specified country.

---

### Value Object: DateRange

| Attribute | Type | Description |
|-----------|------|-------------|
| startDate | Date | Start of the period |
| endDate | Date | End of the period |

**Validation:** `startDate` must be before `endDate`.

---

### Value Object: Money *(Entrega 3)*

| Attribute | Type | Description |
|-----------|------|-------------|
| amount | decimal | Monetary value | 
| currency | string | Currency code (COP, USD) |

**Validation:** `amount` must be ≥ 0. `currency` must be a supported code.

---

## System Aggregates

### Aggregate: User

**Aggregate Root:** User
**Value Objects:** Email, Phone
**Why:** User identity, credentials, and verification state must always be consistent as a unit.

---

### Aggregate: Place

**Aggregate Root:** Place
**Internal entities:** Invitation (pending invitations for this place), Member (users with access)
**Value Objects:** Address
**Why:** A place, its members, and pending invitations must be consistent — an invitation cannot exist without a place, and member roles must be validated against the place's owner.

**Aggregate invariants:**
```
AGGR-P001: A place always has exactly one Owner member
AGGR-P002: An Invitation cannot be created for a user who is already a Member
AGGR-P003: The Owner cannot be removed as a Member
```

---

### Aggregate: Device

**Aggregate Root:** Device
**Internal entities:** — (readings are stored separately for performance)
**Why:** Device identity, link state, and health status must be consistent. Readings are stored in a separate collection/table for write performance but are validated against the device on ingestion.

---

### Aggregate: Valve *(Entrega 2)*

**Aggregate Root:** Valve
**Internal entities:** ShutoffRule (auto-shutoff configuration)
**Why:** The valve state and its auto-shutoff rules must be consistent — a rule cannot trigger if the valve is already closed, and enabling a rule requires checking the current state.

**Aggregate invariants:**
```
AGGR-V001: Auto-shutoff rules can only be enabled if the valve is in OPEN or CLOSED state (not NO_COMMUNICATION)
AGGR-V002: Only one rule per trigger type (leak / goal) per valve
```

---

## Summary Table

| Name | Type | Bounded Context | Aggregate Root? | Entrega |
|------|------|----------------|----------------|---------|
| User | Entity | IAM | Yes | MVP |
| Place | Entity | Place Management | Yes | MVP |
| Invitation | Entity | Place Management | No (inside Place) | 3 |
| Member | Entity | Place Management | No (inside Place) | 3 |
| Device | Entity | Device Management | Yes | MVP |
| Reading | Entity | Device Management | No (stored separately) | MVP |
| Valve | Entity | Valve Control | Yes | 2 |
| ValveAction | Entity | Valve Control | No (audit log) | 2 |
| ShutoffRule | Entity | Valve Control | No (inside Valve) | 3 |
| Alert | Entity | Notification | Yes | 2 |
| Email | Value Object | Shared | N/A | MVP |
| Phone | Value Object | Shared | N/A | MVP |
| Address | Value Object | Place Management | N/A | MVP |
| DateRange | Value Object | Shared | N/A | 3 |
| Money | Value Object | Consumption | N/A | 3 |

---

## Domain Services

### ConsumptionAggregator

**Context:** Consumption
**Description:** Aggregates raw readings into consumption records for display (by hour, day, week, month). Stateless — reads from the readings store and writes aggregated results.

### LeakDetector *(Entrega 2)*

**Context:** Notification
**Description:** Analyzes reading patterns to detect suspected leaks. Triggers when continuous flow exceeds a threshold for a sustained period (default: 30 min). Applies higher weight to nighttime consumption (e.g., 11 PM – 5 AM).

### GoalEvaluator *(Entrega 2)*

**Context:** Consumption
**Description:** Compares current monthly consumption against the defined goal. Emits events when thresholds are reached (80% warning, 100% exceeded).

### BillComparator *(Entrega 3)*

**Context:** Consumption
**Description:** Compares sensor-measured consumption against a registered utility bill for the same period. Calculates absolute and percentage difference. Flags if data coverage is insufficient for a reliable comparison.
