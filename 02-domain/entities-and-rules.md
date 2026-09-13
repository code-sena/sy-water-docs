# Entities, Value Objects, and Business Rules

> Building blocks of the Save Your Water domain following DDD tactical patterns.
> Organized by bounded context (see `02-domain/domain-map.md` for strategic design).
> Each entity is tagged with its target delivery (Entrega).

---

## BC-01: IAM (Identity & Access Management) — Supporting

### Entity: User

**Aggregate Root:** Yes
**Description:** A person registered on the platform. The default role is "User" (can create or join places). Place-scoped roles (Owner, Member) are assigned per place through the Member entity.
**Identifier:** `userId` (UUID)
**Entrega:** MVP

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

## BC-02: Place Management — Supporting

### Entity: Place

**Aggregate Root:** Yes
**Description:** A physical location (home or business) where water consumption is monitored.
**Identifier:** `placeId` (UUID)
**Entrega:** MVP

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

### Entity: Member *(Entrega 3)*

**Aggregate Root:** No (inside Place aggregate)
**Description:** A user linked to a place with a specific role (Owner or Member) and permissions defined by the Owner.
**Identifier:** `memberId` (UUID)

**Attributes:**

| Attribute | Type | Description | Required | Rules |
|-----------|------|-------------|---------|-------|
| memberId | UUID | Unique identifier | Yes | Auto-generated |
| placeId | UUID | Place this membership belongs to | Yes | Must reference an active place |
| userId | UUID | User who is a member | Yes | Unique per place |
| role | enum | OWNER or MEMBER | Yes | Exactly one OWNER per place |
| permissions | Permissions (VO) | Set of permissions granted by the Owner | Yes (for MEMBER role) | Owner has all permissions implicitly |
| createdAt | DateTime | When the membership was created | Yes | Immutable |
| updatedAt | DateTime | Last modification | Yes | Auto-updated |

**Permissions (for MEMBER role, assigned by Owner):**

| Permission | Description |
|------------|-------------|
| VIEW_CONSUMPTION | View consumption dashboard, charts, and history |
| VIEW_DEVICE_STATUS | View device connectivity status |
| CONTROL_VALVE | Open/close the valve remotely |
| VIEW_ALERTS | View and acknowledge alerts |
| VIEW_BILLS | View registered utility bills and comparisons |

**Invariants:**

```
INV-M001: A place always has exactly one member with role OWNER
INV-M002: The OWNER member cannot be removed or have their role changed
INV-M003: Only one membership per user per place (unique: placeId + userId)
INV-M004: OWNER role has all permissions implicitly — permissions list only applies to MEMBER role
INV-M005: A MEMBER must have at least VIEW_CONSUMPTION permission
```

---

### Entity: Invitation *(Entrega 3)*

**Aggregate Root:** No (inside Place aggregate)
**Description:** A pending request from an Owner for a user to join a place as a Member.
**Identifier:** `invitationId` (UUID)

**Attributes:**

| Attribute | Type | Description | Required | Rules |
|-----------|------|-------------|---------|-------|
| invitationId | UUID | Unique identifier | Yes | Auto-generated |
| placeId | UUID | Place the user is invited to | Yes | Must reference an active place |
| invitedEmail | Email (VO) | Email of the invited person | Yes | — |
| invitedBy | UUID | User (Owner) who sent the invitation | Yes | Must be the Owner of the place |
| permissions | Permissions (VO) | Permissions the Member will receive | Yes | At least VIEW_CONSUMPTION |
| status | enum | PENDING, ACCEPTED, EXPIRED, REJECTED | Yes | Default: PENDING |
| expiresAt | DateTime | When the invitation expires | Yes | Default: 7 days from creation |
| respondedAt | DateTime | When the user responded | No | Null while PENDING |
| createdAt | DateTime | Creation timestamp | Yes | Immutable |
| updatedAt | DateTime | Last modification | Yes | Auto-updated |

**Lifecycle:**

```
[PENDING] ──(user accepts)──▶ [ACCEPTED] ──▶ (creates Member record)
    │
    ├──(user rejects)──▶ [REJECTED]
    │
    └──(time expires)──▶ [EXPIRED]
```

**Invariants:**

```
INV-I001: Only the Owner of a place can create an invitation
INV-I002: An invitation cannot be created for a user who is already a Member of that place
INV-I003: An invitation cannot be created if a PENDING invitation for the same email + place already exists
INV-I004: Accepting an invitation creates a Member record with the permissions defined in the invitation
INV-I005: An expired invitation cannot be accepted — the Owner must send a new one
```

---

## BC-03: Device Management — Supporting

### Entity: Device

**Aggregate Root:** Yes
**Description:** An IoT unit (ESP32 + YF-S201 sensor + electrovalve) that measures water flow and controls the valve.
**Identifier:** `deviceId` (UUID), `hardwareId` (string — physical identifier printed on the device)
**Entrega:** MVP

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

**Aggregate Root:** No (stored separately for performance)
**Description:** A single flow measurement sent by an ESP32 device via MQTT. Immutable after creation.
**Identifier:** `readingId` (UUID)
**Entrega:** MVP

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

## BC-04: Consumption Monitoring — Core

### Entity: ConsumptionRecord

**Aggregate Root:** Yes
**Description:** Aggregated water consumption for a place during a specific time interval. Created and updated by the ConsumptionAggregator domain service as readings arrive.
**Identifier:** `recordId` (UUID)
**Entrega:** MVP

**Attributes:**

| Attribute | Type | Description | Required | Rules |
|-----------|------|-------------|---------|-------|
| recordId | UUID | Unique identifier | Yes | Auto-generated |
| placeId | UUID | Place this consumption belongs to | Yes | Must reference an active place |
| periodType | enum | HOUR, DAY, WEEK, MONTH | Yes | — |
| periodStart | DateTime | Start of the aggregation window | Yes | — |
| periodEnd | DateTime | End of the aggregation window | Yes | Must be after periodStart |
| totalVolume | decimal | Total liters consumed in this period | Yes | Default: 0, must be ≥ 0 |
| readingCount | int | Number of readings aggregated | Yes | Default: 0, must be ≥ 0 |
| createdAt | DateTime | When the record was first created | Yes | Immutable |
| updatedAt | DateTime | Last aggregation update | Yes | Auto-updated |

**Invariants:**

```
INV-CR001: Only one record per (placeId + periodType + periodStart) — no duplicate windows
INV-CR002: totalVolume must be ≥ 0
INV-CR003: readingCount must be ≥ 0
INV-CR004: periodEnd must be after periodStart
INV-CR005: HOUR records feed DAY aggregation; DAY records feed WEEK and MONTH aggregation
```

---

## BC-05: Consumption Goals — Supporting

### Entity: Goal *(Entrega 2)*

**Aggregate Root:** Yes (single-entity aggregate)
**Description:** A monthly water consumption target set by the Owner for a specific place. Used by the GoalEvaluator domain service to emit threshold and exceeded events.
**Identifier:** `goalId` (UUID)

**Attributes:**

| Attribute | Type | Description | Required | Rules |
|-----------|------|-------------|---------|-------|
| goalId | UUID | Unique identifier | Yes | Auto-generated |
| placeId | UUID | Place this goal applies to | Yes | — |
| setBy | UUID | User (Owner) who set the goal | Yes | Must be Owner of the place |
| month | string | Target month (format: "2026-09") | Yes | — |
| targetVolume | decimal | Target consumption in the place's measurement unit | Yes | Must be > 0 |
| unit | enum | LITERS, CUBIC_METERS, GALLONS | Yes | Matches place's measurementUnit |
| source | enum | MANUAL or SUGGESTED | Yes | Default: MANUAL |
| createdAt | DateTime | When the goal was set | Yes | Immutable |
| updatedAt | DateTime | Last modification | Yes | Auto-updated |

**Invariants:**

```
INV-G001: Only one goal per place per month
INV-G002: targetVolume must be > 0
INV-G003: Only the Owner can set or modify a goal
INV-G004: A goal for a past month cannot be modified
```

---

## BC-06: Billing & Tariffs — Supporting

### Entity: Bill *(Entrega 3)*

**Aggregate Root:** Yes (single-entity aggregate)
**Description:** A water utility bill manually registered by the Owner to compare against sensor-measured consumption for the same period.
**Identifier:** `billId` (UUID)

**Attributes:**

| Attribute | Type | Description | Required | Rules |
|-----------|------|-------------|---------|-------|
| billId | UUID | Unique identifier | Yes | Auto-generated |
| placeId | UUID | Place this bill belongs to | Yes | — |
| registeredBy | UUID | User who entered the bill | Yes | Must be Owner of the place |
| billedVolume | decimal | Consumption billed by the utility company | Yes | Must be > 0 |
| totalAmount | Money (VO) | Total bill amount with currency | Yes | Must be ≥ 0 |
| billingPeriod | DateRange (VO) | Start and end date of the billing period | Yes | Must not overlap with another bill for the same place |
| paymentDueDate | Date | When the bill is due | Yes | — |
| isPaid | boolean | Whether the bill has been paid | Yes | Default: false |
| createdAt | DateTime | When the bill was registered | Yes | Immutable |
| updatedAt | DateTime | Last modification | Yes | Auto-updated |

**Invariants:**

```
INV-B001: Billing periods must not overlap for the same place
INV-B002: billedVolume must be > 0
INV-B003: Only the Owner can register or modify a bill
INV-B004: period start must be before period end
```

---

### Entity: Tariff *(Entrega 3)*

**Aggregate Root:** Yes (single-entity aggregate)
**Description:** The price per unit of water for a place, used to estimate the monetary cost of sensor-measured consumption.
**Identifier:** `tariffId` (UUID)

**Attributes:**

| Attribute | Type | Description | Required | Rules |
|-----------|------|-------------|---------|-------|
| tariffId | UUID | Unique identifier | Yes | Auto-generated |
| placeId | UUID | Place this tariff applies to | Yes | — |
| pricePerUnit | Money (VO) | Price per measurement unit | Yes | Must be > 0 |
| unit | enum | LITERS, CUBIC_METERS, GALLONS | Yes | Matches place's measurementUnit |
| fixedCharge | Money (VO) | Optional monthly fixed charge | No | Default: 0 |
| effectiveFrom | Date | Start date of tariff validity | Yes | — |
| effectiveUntil | Date | End date of tariff validity | No | Null = currently active tariff |
| createdAt | DateTime | When the tariff was configured | Yes | Immutable |
| updatedAt | DateTime | Last modification | Yes | Auto-updated |

**Invariants:**

```
INV-T001: Only one active tariff per place at any given date (no overlapping ranges)
INV-T002: pricePerUnit must be > 0
INV-T003: A new tariff automatically closes the previous one (sets effectiveUntil)
INV-T004: Tariff changes apply to future cost estimates only — historical estimates remain unchanged
INV-T005: Only the Owner can configure tariffs
```

---

## BC-07: Valve Control — Core

### Entity: Valve *(Entrega 2)*

**Aggregate Root:** Yes
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
INV-V001: Only a user with the Owner role (or CONTROL_VALVE permission) can send open/close commands
INV-V002: A close command requires a confirmation code sent to the owner
INV-V003: State reflects the device's confirmed state, not the last command sent
INV-V004: If the device does not confirm within 30 seconds, the command is marked as failed
INV-V005: Auto-shutoff rules are disabled by default and can only be enabled by the owner
INV-V006: Auto-shutoff never executes without explicit user confirmation
INV-V007: The system never reopens the valve automatically — reopening is always manual
```

---

### Entity: ValveAction *(Entrega 2)*

**Aggregate Root:** No (audit log)
**Description:** An auditable record of every open/close action on a valve. Immutable after creation.
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

### Entity: ShutoffRule *(Entrega 3)*

**Aggregate Root:** No (inside Valve aggregate)
**Description:** An automatic rule that triggers valve closure when a specific condition is met.
**Identifier:** `ruleId` (UUID)

**Attributes:**

| Attribute | Type | Description | Required | Rules |
|-----------|------|-------------|---------|-------|
| ruleId | UUID | Unique identifier | Yes | Auto-generated |
| valveId | UUID | Valve this rule belongs to | Yes | — |
| triggerType | enum | LEAK or GOAL | Yes | — |
| isEnabled | boolean | Whether the rule is active | Yes | Default: false |
| createdAt | DateTime | When the rule was created | Yes | Immutable |
| updatedAt | DateTime | Last modification | Yes | Auto-updated |

**Invariants:**

```
INV-SR001: Only one rule per trigger type per valve
INV-SR002: Rules can only be enabled if the valve state is OPEN or CLOSED (not NO_COMMUNICATION)
INV-SR003: Only the Owner can enable/disable shutoff rules
INV-SR004: When a rule triggers, the system sends a confirmation request to the Owner before closing
```

---

## BC-08: Alerts & Detection — Core

### Entity: Alert *(Entrega 2)*

**Aggregate Root:** Yes
**Description:** A system-generated fact triggered by a domain event, classified by urgency. The rule engine in this BC evaluates conditions and creates alerts; the Notification Delivery BC handles transport.
**Identifier:** `alertId` (UUID)

**Attributes:**

| Attribute | Type | Description | Required | Rules |
|-----------|------|-------------|---------|-------|
| alertId | UUID | Unique identifier | Yes | Auto-generated |
| placeId | UUID | Place this alert relates to | Yes | — |
| userId | UUID | Alert recipient | Yes | — |
| type | enum | LEAK_SUSPECTED, VALVE_CLOSED, GOAL_EXCEEDED, GOAL_WARNING, SENSOR_DISCONNECTED, BILL_MISMATCH, BILL_REMINDER, PAYMENT_REMINDER | Yes | — |
| urgency | enum | CRITICAL, IMPORTANT, INFORMATIVE | Yes | Derived from type |
| title | string | Short description | Yes | — |
| body | string | Detailed message | Yes | — |
| isRead | boolean | Whether the user has seen it | Yes | Default: false |
| channels | list | Channels used for delivery | Yes | Based on urgency + user preferences |
| createdAt | DateTime | When the alert was generated | Yes | Immutable |

**Urgency derivation:**

| Type | Urgency |
|------|---------|
| LEAK_SUSPECTED | CRITICAL |
| VALVE_CLOSED | CRITICAL |
| SENSOR_DISCONNECTED | IMPORTANT |
| GOAL_EXCEEDED | IMPORTANT |
| BILL_MISMATCH | IMPORTANT |
| GOAL_WARNING | INFORMATIVE |
| BILL_REMINDER | INFORMATIVE |
| PAYMENT_REMINDER | INFORMATIVE |

**Invariants:**

```
INV-A001: Urgency is derived from type — cannot be set independently
INV-A002: CRITICAL alerts are delivered on all enabled channels — cannot be fully silenced
INV-A003: Alerts for Members are delivered only if the Member has VIEW_ALERTS permission
```

---

## BC-09: Notification Delivery — Generic

### Entity: NotificationPreference *(Entrega 2)*

**Aggregate Root:** Yes (single-entity aggregate)
**Description:** A user's channel configuration for a specific urgency level.
**Identifier:** `preferenceId` (UUID)

**Attributes:**

| Attribute | Type | Description | Required | Rules |
|-----------|------|-------------|---------|-------|
| preferenceId | UUID | Unique identifier | Yes | Auto-generated |
| userId | UUID | User this preference belongs to | Yes | — |
| urgencyLevel | enum | CRITICAL, IMPORTANT, INFORMATIVE | Yes | — |
| inAppEnabled | boolean | In-app notifications | Yes | Default: true |
| pushEnabled | boolean | Push notifications | Yes | Default: true |
| emailEnabled | boolean | Email notifications | Yes | Default: false |
| smsEnabled | boolean | SMS/WhatsApp notifications | Yes | Default: false |
| createdAt | DateTime | When the preference was created | Yes | Immutable |
| updatedAt | DateTime | Last modification | Yes | Auto-updated |

**Invariants:**

```
INV-NP001: One preference record per user per urgency level
INV-NP002: CRITICAL urgency must have at least one channel enabled — cannot disable all channels
INV-NP003: Default preferences are created on user registration (in-app + push enabled for all levels)
```

---

## BC-11: Savings Tips — Generic

### Entity: Recommendation *(Entrega 4)*

**Aggregate Root:** Yes (single-entity aggregate)
**Description:** A generic water-saving tip or recommendation created by an Admin. Displayed to all users in a dedicated section. Not personalized per place.
**Identifier:** `recommendationId` (UUID)

**Attributes:**

| Attribute | Type | Description | Required | Rules |
|-----------|------|-------------|---------|-------|
| recommendationId | UUID | Unique identifier | Yes | Auto-generated |
| createdBy | UUID | Admin user who created the recommendation | Yes | Must have Admin role |
| title | string | Short title | Yes | Max 200 chars |
| body | string | Full recommendation text | Yes | Max 2000 chars |
| category | enum | SAVING_TIP, MAINTENANCE, SEASONAL, GENERAL | Yes | — |
| isActive | boolean | Whether the recommendation is visible to users | Yes | Default: true |
| createdAt | DateTime | When the recommendation was created | Yes | Immutable |
| updatedAt | DateTime | Last modification | Yes | Auto-updated |

**Invariants:**

```
INV-REC001: Only users with Admin role can create, edit, or deactivate recommendations
INV-REC002: Deactivated recommendations are hidden from users but not deleted
```

---

## Value Objects (shared across BCs)

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

### Value Object: Money

| Attribute | Type | Description |
|-----------|------|-------------|
| amount | decimal | Monetary value |
| currency | string | Currency code (COP, USD) |

**Validation:** `amount` must be ≥ 0. `currency` must be a supported code.

---

### Value Object: Permissions *(Entrega 3)*

| Attribute | Type | Description |
|-----------|------|-------------|
| values | set of strings | Set of permission keys granted to a Member |

**Valid keys:** `VIEW_CONSUMPTION`, `VIEW_DEVICE_STATUS`, `CONTROL_VALVE`, `VIEW_ALERTS`, `VIEW_BILLS`

**Validation:** Must contain at least `VIEW_CONSUMPTION`. Owner role has all permissions implicitly.

---

## Aggregates Summary

| Aggregate Root | BC | Internal Entities | Value Objects | Aggregate Invariants |
|---------------|-----|-------------------|---------------|---------------------|
| User | BC-01 IAM | — | Email, Phone | Identity and credentials must be consistent as a unit |
| Place | BC-02 Place Mgmt | Member, Invitation | Address, Permissions | A place always has exactly one Owner; invitations cannot target existing members |
| Device | BC-03 Device Mgmt | — (Reading stored separately) | — | Device identity, link state, and health must be consistent |
| ConsumptionRecord | BC-04 Consumption Monitoring | — | — | One record per (place + periodType + periodStart) |
| Goal | BC-05 Goals | — | — | One goal per place per month; only Owner can set |
| Bill | BC-06 Billing | — | DateRange, Money | Billing periods must not overlap per place |
| Tariff | BC-06 Billing | — | Money | One active tariff per place at any date |
| Valve | BC-07 Valve Control | ShutoffRule | — | Valve state and shutoff rules must be consistent |
| Alert | BC-08 Alerts | — | — | Urgency derived from type; critical alerts cannot be silenced |
| NotificationPreference | BC-09 Delivery | — | — | One preference per user per urgency; critical must have ≥ 1 channel |
| Recommendation | BC-11 Tips | — | — | Only Admin can manage; deactivation ≠ deletion |

---

## Domain Services

### ConsumptionAggregator

**BC:** BC-04 Consumption Monitoring
**Description:** Aggregates raw readings into consumption records for display (by hour, day, week, month). Stateless — reads from the readings store and writes aggregated results.
**Entrega:** MVP

### GoalEvaluator *(Entrega 2)*

**BC:** BC-05 Consumption Goals
**Description:** Compares current monthly consumption against the defined goal. Emits events when thresholds are reached (80% warning, 100% exceeded).

### LeakDetector *(Entrega 2)*

**BC:** BC-08 Alerts & Detection
**Description:** Analyzes reading patterns to detect suspected leaks. Triggers when continuous flow exceeds a threshold for a sustained period (default: 30 min). Applies higher weight to nighttime consumption (11 PM – 5 AM).

### BillComparator *(Entrega 3)*

**BC:** BC-06 Billing & Tariffs
**Description:** Compares sensor-measured consumption against a registered utility bill for the same period. Calculates absolute and percentage difference. Flags if data coverage is insufficient for a reliable comparison.

### CostEstimator *(Entrega 3)*

**BC:** BC-06 Billing & Tariffs
**Description:** Calculates the estimated monetary cost of consumption for a period using the active tariff. Multiplies consumption volume by the tariff's price-per-unit and adds the fixed charge.

### PermissionChecker *(Entrega 3)*

**BC:** BC-02 Place Management
**Description:** Validates whether a Member has the required permission for a given action on a place. Used by all services that accept requests from Members (consumption, valve, notification).

---

## Entity Summary Table

| Name | Type | Bounded Context | Aggregate Root? | Entrega |
|------|------|----------------|----------------|---------|
| User | Entity | BC-01 IAM | Yes | MVP |
| Place | Entity | BC-02 Place Management | Yes | MVP |
| Member | Entity | BC-02 Place Management | No (inside Place) | 3 |
| Invitation | Entity | BC-02 Place Management | No (inside Place) | 3 |
| Device | Entity | BC-03 Device Management | Yes | MVP |
| Reading | Entity | BC-03 Device Management | No (stored separately) | MVP |
| ConsumptionRecord | Entity | BC-04 Consumption Monitoring | Yes | MVP |
| Goal | Entity | BC-05 Consumption Goals | Yes (single-entity) | 2 |
| Bill | Entity | BC-06 Billing & Tariffs | Yes (single-entity) | 3 |
| Tariff | Entity | BC-06 Billing & Tariffs | Yes (single-entity) | 3 |
| Valve | Entity | BC-07 Valve Control | Yes | 2 |
| ValveAction | Entity | BC-07 Valve Control | No (audit log) | 2 |
| ShutoffRule | Entity | BC-07 Valve Control | No (inside Valve) | 3 |
| Alert | Entity | BC-08 Alerts & Detection | Yes | 2 |
| NotificationPreference | Entity | BC-09 Notification Delivery | Yes (single-entity) | 2 |
| Recommendation | Entity | BC-11 Savings Tips | Yes (single-entity) | 4 |
| Email | Value Object | Shared | N/A | MVP |
| Phone | Value Object | Shared | N/A | MVP |
| Address | Value Object | BC-02 Place Management | N/A | MVP |
| DateRange | Value Object | Shared | N/A | MVP |
| Money | Value Object | BC-06 Billing & Tariffs | N/A | 3 |
| Permissions | Value Object | BC-02 Place Management | N/A | 3 |

---

## Correlations

- Bounded contexts → `02-domain/domain-map.md`
- Domain events per BC → `02-domain/domain-events.md`
- Data model (DDL) → `06-data/models.md`
- User stories → `04-requirements/user-stories.md`
- Architecture → `05-architecture/overview.md`
