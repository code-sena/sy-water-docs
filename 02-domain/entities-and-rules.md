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
| firstName | string | First name | Yes | Min 2 chars |
| lastName | string | Last name | Yes | Min 2 chars |
| email | Email (VO) | Account email | Yes | Unique across system, immutable |
| phone | Phone (VO) | Mobile number | Yes | Valid format per country |
| phoneVerified | boolean | Phone verification status | Yes | Default: false |
| avatarUrl | string | Profile picture URL | No | — |
| emailVerified | boolean | Email verification status | Yes | Default: false (true for Google accounts) |
| accountLockedUntil | DateTime | Block expiration timestamp | No | Set after 5 failed login attempts |
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

**Supporting entities (inside security schema):**

| Entity | Description | Entrega |
|--------|-------------|---------|
| UserCredential | Stores credential per type (LOCAL with password_hash, GOOGLE with google_id). One per type per user | MVP |
| EmailVerification | Token-based email verification with expiry | MVP |
| PasswordReset | Token-based password reset with expiry and single-use tracking | MVP |
| RefreshToken | Long-lived token for session renewal, with device_info and revocation | MVP |
| LoginAttempt | Audit record of every login attempt (success or failure) with IP | MVP |
| ActivityLog | General audit trail: action, entity_type, entity_id, metadata (JSON) | MVP |

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
| placeId | UUID | Public unique identifier | Yes | Auto-generated |
| ownerId | UUID | User who created and owns this place | Yes | Denormalized from place_members for fast lookup |
| name | string | Reference name (e.g., "Mi Casa", "Oficina Centro") | Yes | — |
| placeType | enum | RESIDENTIAL, COMMERCIAL | Yes | — |
| address | string | Full street address | Yes | — |
| city | string | City | Yes | — |
| countryCode | string | Country code | Yes | CO, EC, US |
| currency | string | Currency code | Yes | Default: COP |
| measurementUnit | enum | LITERS, CUBIC_METERS, GALLONS | Yes | Default: LITERS |
| isDefault | boolean | Whether this is the user's default place | Yes | Default: false |
| createdAt | DateTime | Creation date | Yes | Immutable |
| updatedAt | DateTime | Last modification | Yes | Auto-updated |
| deletedAt | DateTime | Soft-delete timestamp | No | Only table with soft-delete |

**Lifecycle:**

```
[Active] ──(owner deletes)──▶ [Deleted]
```

**Invariants:**

```
INV-P001: A place has at most one linked device at any time
INV-P002: A place cannot be deleted if it has a linked device — unlink first
INV-P003: Deletion is a soft-delete (deleted_at); consumption history is preserved
INV-P004: Only places.places uses soft delete in the entire system
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
| role | enum | OWNER or GUEST | Yes | Exactly one OWNER per place |
| revokedAt | DateTime | When the membership was revoked | No | Null while active |
| createdAt | DateTime | When the membership was created | Yes | Immutable |
| updatedAt | DateTime | Last modification | Yes | Auto-updated |

> **Nota:** El valor `GUEST` en la BD corresponde al concepto de "Miembro" en el lenguaje ubicuo del dominio. Se eligió GUEST en el modelo físico para distinguirlo del concepto genérico de "member" (la tabla misma).

**Invariants:**

```
INV-M001: A place always has exactly one member with role OWNER
INV-M002: The OWNER member cannot be removed or have their role changed
INV-M003: Only one membership per user per place (unique: placeId + userId)
INV-M004: OWNER role has all permissions implicitly
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
| deviceId | UUID | Public unique identifier | Yes | Auto-generated |
| serialNumber | string | Physical identifier (serial number on device) | Yes | Unique, immutable |
| placeId | UUID | Place this device is linked to | No | Null when available/unlinked |
| authTokenHash | string | Hashed authentication token for MQTT | Yes | — |
| status | enum | CONNECTED, DISCONNECTED, NEVER_REPORTED | Yes | Default: NEVER_REPORTED |
| lastReportAt | DateTime | Timestamp of last received reading | No | Null until first reading |
| inactivityThresholdMin | int | Minutes without reports before marking disconnected | Yes | Default: 15 |
| linkedAt | DateTime | When the device was linked to current place | No | — |
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
INV-D006: Device is considered DISCONNECTED after inactivityThresholdMin without a reading
```

**Supporting entity:** `DeviceLinkHistory` — audit trail of every link/unlink, tracking who linked/unlinked and when.

---

### Entity: Reading

**Aggregate Root:** No (stored separately for performance)
**Description:** A single flow measurement sent by an ESP32 device via MQTT. Immutable after creation.
**Identifier:** `readingId` (UUID)
**Entrega:** MVP

**Attributes:**

| Attribute | Type | Description | Required | Rules |
|-----------|------|-------------|---------|-------|
| readingId | BIGINT | Internal identifier | Yes | Auto-generated |
| deviceId | BIGINT | Device that sent this reading | Yes | FK cross → devices.devices |
| placeId | BIGINT | Place associated at the time of reading | Yes | FK cross → places.places (denormalized, D1) |
| flowRateLph | decimal | Instantaneous flow rate (liters/hour) | Yes | Must be ≥ 0 |
| cumulativeLiters | decimal | Cumulative volume since device install | Yes | Must be ≥ 0 |
| recordedAt | DateTime | When the measurement was taken (device clock) | Yes | — |
| receivedAt | DateTime | When the backend received the reading | Yes | Server clock, SYSUTCDATETIME() |

**Invariants:**

```
INV-R001: No two readings from the same device can have the same timestamp (deduplication)
INV-R002: flowRate must be ≥ 0 and ≤ max physical range of YF-S201 (≈ 30 L/min)
INV-R003: volume must be ≥ 0
INV-R004: timestamp cannot be in the future (tolerance: 5 minutes for clock drift)
```

---

## BC-04: Consumption Monitoring — Core

### Consumption Aggregates (3 tables)

**Aggregate Root:** Yes (each table is a materialized aggregate)
**Description:** Pre-aggregated consumption data at 3 granularities. Created by the ConsumptionAggregator domain service as readings arrive.
**Entrega:** MVP

| Table | Key | Extra columns |
|-------|-----|---------------|
| `consumption_hourly` | place_id + hour_start (UQ) | total_liters, reading_count |
| `consumption_daily` | place_id + date (UQ) | total_liters, avg_hourly, peak_hour (0–23), reading_count |
| `consumption_monthly` | place_id + month_start (UQ) | total_liters, avg_daily, peak_day, reading_count, days_with_data |

**Invariants:**

```
INV-CA001: Only one record per (placeId + period start) per granularity — no duplicate windows
INV-CA002: total_liters must be ≥ 0
INV-CA003: reading_count must be ≥ 0
INV-CA004: month_start must be the first day of the month
INV-CA005: Hourly aggregates feed daily; daily feeds monthly
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
| goalId | BIGINT | Internal identifier | Yes | Auto-generated |
| placeId | BIGINT | Place this goal applies to | Yes | FK cross → places.places |
| monthStart | Date | First day of the target month | Yes | Must be DAY = 1 |
| targetLiters | decimal | Target consumption in liters | Yes | Must be > 0 |
| source | enum | MANUAL, SUGGESTED, CARRIED_OVER | Yes | Default: MANUAL |
| createdAt | DateTime | When the goal was set | Yes | Immutable |
| updatedAt | DateTime | Last modification | Yes | Auto-updated |

**Invariants:**

```
INV-G001: Only one goal per place per month (unique: placeId + monthStart)
INV-G002: targetLiters must be > 0
INV-G003: Only the Owner can set or modify a goal
INV-G004: A goal for a past month cannot be modified
```

**Supporting entity:** `GoalSuggestion` — automatically proposed goal based on historical consumption (based_on_months ≥ 1), with accepted flag for tracking user response.

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

### Entity: ValveState *(Entrega 2)*

**Aggregate Root:** Yes
**Description:** Current state of the electrovalve at a place. One record per device/place (unique constraints).
**Table:** `valve.valve_states`

**Attributes:**

| Attribute | Type | Description | Required | Rules |
|-----------|------|-------------|---------|-------|
| deviceId | BIGINT | Device (unique) | Yes | FK cross → devices.devices |
| placeId | BIGINT | Place (unique) | Yes | FK cross → places.places |
| state | enum | OPEN, CLOSED, UNKNOWN | Yes | Default: UNKNOWN |
| lastConfirmedAt | DateTime | When the device last confirmed the state | No | — |
| updatedAt | DateTime | Last change | Yes | Auto-updated |

**Invariants:**

```
INV-V001: Only a user with the Owner role (or CONTROL_VALVE permission) can send open/close commands
INV-V002: State reflects the device's confirmed state, not the last command sent
INV-V003: The system never reopens the valve automatically — reopening is always MANUAL
```

---

### Entity: ValveCommand *(Entrega 2)*

**Aggregate Root:** No (command log, immutable after terminal state)
**Description:** A command to open or close the valve, with a 6-state lifecycle.
**Table:** `valve.valve_commands`

**Attributes:**

| Attribute | Type | Description | Required | Rules |
|-----------|------|-------------|---------|-------|
| deviceId | BIGINT | Target device | Yes | FK cross → devices.devices |
| placeId | BIGINT | Target place | Yes | FK cross → places.places (D2) |
| commandedBy | BIGINT | User who initiated (null = auto) | No | FK cross → security.users |
| command | enum | OPEN, CLOSE | Yes | — |
| origin | enum | MANUAL, AUTO_LEAK, AUTO_GOAL | Yes | — |
| status | enum | (see lifecycle below) | Yes | Default: PENDING_CONFIRMATION |
| confirmationCode | string | Security code for confirmation step | No | — |
| confirmedAt | DateTime | When user confirmed | No | — |
| sentAt | DateTime | When sent to device via MQTT | No | — |
| deviceAckAt | DateTime | When device acknowledged | No | — |
| timeoutAt | DateTime | When 30s timeout expired | No | — |
| failureReason | string | Failure details | No | — |
| createdAt | DateTime | When command was created | Yes | Immutable |

**Command lifecycle:**

```
PENDING_CONFIRMATION ──(user confirms code)──▶ CONFIRMED
                                                   │
                                             (sent via MQTT)
                                                   ▼
                                                 SENT
                                                   │
                                     ┌─────────────┼────────────┐
                                     ▼             ▼            ▼
                               ACK_SUCCESS    ACK_TIMEOUT    FAILED
                                              (30s no ack)
```

**Invariants:**

```
INV-VC001: A close command requires a confirmation code sent to the owner
INV-VC002: If the device does not ACK within 30 seconds → ACK_TIMEOUT
INV-VC003: Auto-shutoff never executes without explicit user confirmation (PENDING_CONFIRMATION)
INV-VC004: Reopening is always MANUAL origin
```

---

### Entity: ValveRule *(Entrega 2)*

**Aggregate Root:** No (inside Valve aggregate)
**Description:** An automatic rule that triggers valve closure when a specific condition is met.
**Table:** `valve.valve_rules`

**Attributes:**

| Attribute | Type | Description | Required | Rules |
|-----------|------|-------------|---------|-------|
| placeId | BIGINT | Place this rule applies to | Yes | FK cross → places.places |
| ruleType | enum | LEAK_DETECTION (only) | Yes | — |
| enabled | boolean | Whether the rule is active | Yes | Default: false |
| requiresConfirmation | boolean | Whether to require user confirmation | Yes | Default: true |
| confirmationTimeoutMin | int | Minutes before auto-expiry of confirmation | Yes | Default: 10, must be > 0 |
| createdBy | BIGINT | User who created this rule | Yes | FK cross → security.users |
| createdAt | DateTime | When the rule was created | Yes | Immutable |
| updatedAt | DateTime | Last modification | Yes | Auto-updated |

**Invariants:**

```
INV-VR001: Only one rule per place (unique: placeId); only LEAK_DETECTION trigger
INV-VR002: Only the Owner can enable/disable valve rules
INV-VR003: When a rule triggers, the system sends a confirmation request to the Owner before closing
```

---

## BC-08: Alerts & Detection — Core

### Entity: AlertTypeConfig *(Entrega 2 — seed data)*

**Aggregate Root:** Yes (reference data)
**Description:** Configuration for each alert type: severity, description, deduplication rules. Seed data — not user-modifiable.
**Table:** `alerts.alert_type_config`

**Alert types and severity:**

| alert_type | severity | max_per_period | period_unit |
|-----------|----------|---------------|-------------|
| SUSPECTED_LEAK | CRITICAL | 1 | HOUR |
| VALVE_CLOSED | CRITICAL | — | — |
| GOAL_EXCEEDED | IMPORTANT | 1 | MONTH |
| SENSOR_DISCONNECTED | IMPORTANT | 1 | (until reconnect) |
| GOAL_80_PERCENT | INFORMATIVE | 1 | MONTH |
| BILL_MISALIGNED | IMPORTANT | 1 | MONTH |
| BILL_REGISTER_REMINDER | INFORMATIVE | — | — |
| BILL_PAYMENT_REMINDER | INFORMATIVE | — | — |

---

### Entity: AlertEvent *(Entrega 2)*

**Aggregate Root:** Yes
**Description:** A system-generated fact triggered by a domain event, classified by severity. The rule engine evaluates conditions and creates alert events; the Notification Delivery BC handles transport.
**Table:** `alerts.alert_events`

**Attributes:**

| Attribute | Type | Description | Required | Rules |
|-----------|------|-------------|---------|-------|
| placeId | BIGINT | Place this alert relates to | Yes | FK cross → places.places |
| alertType | string | One of the 8 configured alert types | Yes | — |
| severity | enum | CRITICAL, IMPORTANT, INFORMATIVE | Yes | Snapshot at creation time (D3) |
| title | string | Short description | Yes | Max 200 chars |
| message | string | Detailed message | Yes | Max 2000 chars |
| metadata | JSON | Additional context (flow rate, percentages, etc.) | No | Must be valid JSON |
| triggeredAt | DateTime | When the alert was generated | Yes | SYSUTCDATETIME() |
| readAt | DateTime | When the user marked it as read | No | — |
| deletedAt | DateTime | When the user dismissed it | No | — |

**Invariants:**

```
INV-A001: Severity is derived from alert_type via alert_type_config — snapshot at creation
INV-A002: CRITICAL alerts are delivered on all enabled channels — cannot be fully silenced
INV-A003: Deduplication enforced via max_per_period from alert_type_config
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
| userId | BIGINT | User this preference belongs to | Yes | FK cross → security.users |
| alertSeverity | enum | CRITICAL, IMPORTANT, INFORMATIVE | Yes | — |
| channelApp | boolean | In-app notifications | Yes | Default: true |
| channelPush | boolean | Push notifications | Yes | Default: true |
| channelEmail | boolean | Email notifications | Yes | Default: false |
| channelSms | boolean | SMS notifications | Yes | Default: false |
| updatedAt | DateTime | Last modification | Yes | Auto-updated |

**Invariants:**

```
INV-NP001: One preference record per user per severity level (unique: userId + alertSeverity)
INV-NP002: CRITICAL severity must have channel_app = true — cannot disable in-app for critical
INV-NP003: Default preferences are created on user registration (app + push enabled for all levels)
```

**Supporting entities:**
- `NotificationLog` — delivery record per (alert_event + user + channel) with status lifecycle: PENDING → SENT → DELIVERED | FAILED
- `PushSubscription` — device tokens for push notifications per user, with platform and active flag

---

## BC-11: Savings Tips — Generic

### Entity: SavingsTip *(Entrega 4)*

**Aggregate Root:** Yes (single-entity aggregate)
**Description:** A generic water-saving tip created by an Admin. Displayed to all users. Not personalized per place.
**Table:** `content.savings_tips`

**Attributes:**

| Attribute | Type | Description | Required | Rules |
|-----------|------|-------------|---------|-------|
| category | enum | RESIDENTIAL, COMMERCIAL, GENERAL | Yes | — |
| title | string | Short title | Yes | Max 200 chars |
| body | string | Full tip text | Yes | Max 2000 chars |
| sortOrder | int | Display order | Yes | Default: 0 |
| active | boolean | Whether the tip is visible to users | Yes | Default: true |
| createdAt | DateTime | When the tip was created | Yes | Immutable |

**Invariants:**

```
INV-TIP001: Only users with Admin role can create, edit, or deactivate tips
INV-TIP002: Deactivated tips are hidden from users but not deleted
```

**Supporting entity:** `TipFavorite` — tracks which users marked a tip as favorite (unique: userId + tipId).

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

| Aggregate Root | BC | Internal/Supporting Entities | Value Objects | Aggregate Invariants |
|---------------|-----|------------------------------|---------------|---------------------|
| User | BC-01 IAM | UserCredential, EmailVerification, PasswordReset, RefreshToken, LoginAttempt, ActivityLog | Email, Phone | Identity and credentials must be consistent as a unit |
| Place | BC-02 Place Mgmt | Member, Invitation | Address | A place always has exactly one Owner; only table with soft delete |
| Device | BC-03 Device Mgmt | DeviceLinkHistory | — | Device identity, link state, and health must be consistent |
| ConsumptionHourly/Daily/Monthly | BC-04 Monitoring | — | — | One record per (place + period start) per granularity |
| Goal | BC-05 Goals | GoalSuggestion | — | One goal per place per month; only Owner can set |
| Bill | BC-06 Billing | — | DateRange, Money | Billing periods must not overlap per place |
| Tariff | BC-06 Billing | TariffCatalog | Money | One active tariff per place at any date |
| ValveState | BC-07 Valve Control | ValveCommand, ValveRule | — | State confirmed by device; commands follow 6-state lifecycle |
| AlertTypeConfig | BC-08 Alerts | AlertEvent | — | Severity derived from type config; deduplication via max_per_period |
| NotificationPreference | BC-09 Delivery | NotificationLog, PushSubscription | — | One preference per user per severity; critical must have app enabled |
| SavingsTip | BC-11 Tips | TipFavorite | — | Only Admin can manage; deactivation ≠ deletion |

---

## Domain Services

### ConsumptionAggregator

**BC:** BC-04 Consumption Monitoring
**Description:** Aggregates raw readings into 3 tiers: consumption_hourly, consumption_daily, consumption_monthly. Stateless — reads from monitoring.readings and writes aggregated results.
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

| Name | Type | Bounded Context | Schema | Aggregate Root? | Entrega |
|------|------|----------------|--------|----------------|---------|
| User | Entity | BC-01 IAM | security | Yes | MVP |
| UserCredential | Entity | BC-01 IAM | security | No (inside User) | MVP |
| EmailVerification | Entity | BC-01 IAM | security | No | MVP |
| PasswordReset | Entity | BC-01 IAM | security | No | MVP |
| RefreshToken | Entity | BC-01 IAM | security | No | MVP |
| LoginAttempt | Entity | BC-01 IAM | security | No | MVP |
| ActivityLog | Entity | BC-01 IAM | security | No | MVP |
| Place | Entity | BC-02 Place Management | places | Yes | MVP |
| Member | Entity | BC-02 Place Management | places | No (inside Place) | 3 |
| Invitation | Entity | BC-02 Place Management | places | No (inside Place) | 3 |
| Device | Entity | BC-03 Device Management | devices | Yes | MVP |
| DeviceLinkHistory | Entity | BC-03 Device Management | devices | No (audit) | MVP |
| Reading | Entity | BC-03/04 Device → Monitoring | monitoring | No (stored separately) | MVP |
| ConsumptionHourly | Entity | BC-04 Consumption Monitoring | monitoring | Yes (aggregate) | MVP |
| ConsumptionDaily | Entity | BC-04 Consumption Monitoring | monitoring | Yes (aggregate) | MVP |
| ConsumptionMonthly | Entity | BC-04 Consumption Monitoring | monitoring | Yes (aggregate) | MVP |
| Goal | Entity | BC-05 Consumption Goals | goals | Yes (single-entity) | 2 |
| GoalSuggestion | Entity | BC-05 Consumption Goals | goals | No | 2 |
| Bill | Entity | BC-06 Billing & Tariffs | billing | Yes (single-entity) | 3 |
| Tariff | Entity | BC-06 Billing & Tariffs | billing | Yes (single-entity) | 3 |
| TariffCatalog | Entity | BC-06 Billing & Tariffs | billing | No (reference) | 4 |
| ValveState | Entity | BC-07 Valve Control | valve | Yes | 2 |
| ValveCommand | Entity | BC-07 Valve Control | valve | No (command log) | 2 |
| ValveRule | Entity | BC-07 Valve Control | valve | No (inside Valve) | 2 |
| AlertTypeConfig | Entity | BC-08 Alerts & Detection | alerts | Yes (seed data) | 2 |
| AlertEvent | Entity | BC-08 Alerts & Detection | alerts | Yes | 2 |
| NotificationPreference | Entity | BC-08 Alerts & Detection | alerts | Yes (single-entity) | 2 |
| NotificationLog | Entity | BC-09 Notification Delivery | notifications | No (delivery record) | 2 |
| PushSubscription | Entity | BC-09 Notification Delivery | notifications | No | 2 |
| SavingsTip | Entity | BC-11 Savings Tips | content | Yes (single-entity) | 4 |
| TipFavorite | Entity | BC-11 Savings Tips | content | No | 4 |
| Email | Value Object | Shared | — | N/A | MVP |
| Phone | Value Object | Shared | — | N/A | MVP |
| Address | Value Object | BC-02 Place Management | — | N/A | MVP |
| DateRange | Value Object | Shared | — | N/A | MVP |
| Money | Value Object | BC-06 Billing & Tariffs | — | N/A | 3 |

---

## Correlations

- Bounded contexts → `02-domain/domain-map.md`
- Domain events per BC → `02-domain/domain-events.md`
- Data model (DDL) → `06-data/models.md`
- User stories → `04-requirements/user-stories.md`
- Architecture → `05-architecture/overview.md`
