# Domain Events

> Events are facts that occurred in the business, stated in past tense.
> They are the backbone of asynchronous communication between bounded contexts.
> Organized by BC — see `02-domain/domain-map.md` for the context map.

---

## Standard envelope for all events

All events follow this structure:

| Field | Type | Description |
|-------|------|-------------|
| `eventId` | UUID | Unique event ID (idempotency key) |
| `eventType` | string | Event name in PascalCase |
| `aggregateId` | UUID | ID of the aggregate that generated the event |
| `aggregateType` | string | Aggregate type |
| `occurredAt` | ISO 8601 | When the business fact occurred |
| `version` | integer | Schema version (for evolution) |
| `payload` | object | Event-specific data |
| `metadata.correlationId` | UUID | For tracing a transaction across services |
| `metadata.causationId` | UUID | ID of the event or command that caused this one |
| `metadata.userId` | UUID | User who initiated the chain (if applicable) |

---

## Event Catalog

### BC-01: IAM Context

#### Event: UserRegistered

| Field | Value |
|-------|-------|
| **Bounded Context** | BC-01 IAM |
| **Aggregate** | User |
| **Trigger** | User completes registration (local or Google OAuth) |
| **Consumers** | BC-09 Notification Delivery (send verification email for local accounts) |
| **Topic** | `iam.user.registered` |
| **Version** | v1 |

**Payload:**
```json
{
  "userId": "uuid",
  "email": "string",
  "fullName": "string",
  "authProvider": "LOCAL | GOOGLE",
  "isVerified": "boolean"
}
```

---

#### Event: UserVerified

| Field | Value |
|-------|-------|
| **Bounded Context** | BC-01 IAM |
| **Aggregate** | User |
| **Trigger** | User clicks the email verification link |
| **Consumers** | — (internal state change) |
| **Topic** | `iam.user.verified` |
| **Version** | v1 |

**Payload:**
```json
{
  "userId": "uuid",
  "email": "string",
  "verifiedAt": "ISO 8601"
}
```

---

#### Event: SessionStarted

| Field | Value |
|-------|-------|
| **Bounded Context** | BC-01 IAM |
| **Aggregate** | User |
| **Trigger** | Successful login |
| **Consumers** | — (audit log) |
| **Topic** | `iam.session.started` |
| **Version** | v1 |

**Payload:**
```json
{
  "userId": "uuid",
  "loginMethod": "LOCAL | GOOGLE",
  "deviceInfo": "string"
}
```

---

#### Event: AccountBlocked

| Field | Value |
|-------|-------|
| **Bounded Context** | BC-01 IAM |
| **Aggregate** | User |
| **Trigger** | 5 consecutive failed login attempts |
| **Consumers** | BC-09 Notification Delivery (notify user of blocked account) |
| **Topic** | `iam.account.blocked` |
| **Version** | v1 |

**Payload:**
```json
{
  "userId": "uuid",
  "email": "string",
  "blockedUntil": "ISO 8601",
  "failedAttempts": 5
}
```

---

#### Event: PasswordChanged

| Field | Value |
|-------|-------|
| **Bounded Context** | BC-01 IAM |
| **Aggregate** | User |
| **Trigger** | User changes or resets their password |
| **Consumers** | BC-01 IAM (invalidate all active sessions) |
| **Topic** | `iam.password.changed` |
| **Version** | v1 |

**Payload:**
```json
{
  "userId": "uuid",
  "changedVia": "PROFILE | RECOVERY"
}
```

---

### BC-02: Place Management Context

#### Event: PlaceCreated

| Field | Value |
|-------|-------|
| **Bounded Context** | BC-02 Place Management |
| **Aggregate** | Place |
| **Trigger** | Owner registers a new place |
| **Consumers** | BC-04 Consumption Monitoring (provision storage for the new place) |
| **Topic** | `place.created` |
| **Version** | v1 |

**Payload:**
```json
{
  "placeId": "uuid",
  "ownerId": "uuid",
  "name": "string",
  "placeType": "RESIDENTIAL | COMMERCIAL",
  "countryCode": "string",
  "city": "string"
}
```

---

#### Event: PlaceDeleted

| Field | Value |
|-------|-------|
| **Bounded Context** | BC-02 Place Management |
| **Aggregate** | Place |
| **Trigger** | Owner deletes a place (after unlinking device) |
| **Consumers** | BC-04 Consumption Monitoring (archive data), BC-08 Alerts & Detection (cancel pending alerts) |
| **Topic** | `place.deleted` |
| **Version** | v1 |

**Payload:**
```json
{
  "placeId": "uuid",
  "ownerId": "uuid",
  "deletedAt": "ISO 8601"
}
```

---

#### Event: MemberInvited *(Entrega 3)*

| Field | Value |
|-------|-------|
| **Bounded Context** | BC-02 Place Management |
| **Aggregate** | Place |
| **Trigger** | Owner sends an invitation to a person |
| **Consumers** | BC-09 Notification Delivery (send invitation email) |
| **Topic** | `place.member.invited` |
| **Version** | v1 |

**Payload:**
```json
{
  "invitationId": "uuid",
  "placeId": "uuid",
  "invitedEmail": "string",
  "invitedBy": "uuid",
  "expiresAt": "ISO 8601"
}
```

---

#### Event: MemberAccessRevoked *(Entrega 3)*

| Field | Value |
|-------|-------|
| **Bounded Context** | BC-02 Place Management |
| **Aggregate** | Place |
| **Trigger** | Owner removes a member's access |
| **Consumers** | BC-01 IAM (invalidate member's sessions for this place) |
| **Topic** | `place.member.revoked` |
| **Version** | v1 |

**Payload:**
```json
{
  "placeId": "uuid",
  "memberUserId": "uuid",
  "revokedBy": "uuid"
}
```

---

### BC-03: Device Management Context

#### Event: DeviceLinked

| Field | Value |
|-------|-------|
| **Bounded Context** | BC-03 Device Management |
| **Aggregate** | Device |
| **Trigger** | Owner links a device to a place (via hardware ID or QR scan) |
| **Consumers** | BC-04 Consumption Monitoring (start expecting readings), BC-07 Valve Control (create valve record) |
| **Topic** | `device.linked` |
| **Version** | v1 |

**Payload:**
```json
{
  "deviceId": "uuid",
  "hardwareId": "string",
  "placeId": "uuid",
  "linkedBy": "uuid",
  "linkedAt": "ISO 8601"
}
```

---

#### Event: DeviceUnlinked

| Field | Value |
|-------|-------|
| **Bounded Context** | BC-03 Device Management |
| **Aggregate** | Device |
| **Trigger** | Owner unlinks a device from a place |
| **Consumers** | BC-07 Valve Control (deactivate valve record), BC-04 Consumption Monitoring (stop expecting readings) |
| **Topic** | `device.unlinked` |
| **Version** | v1 |

**Payload:**
```json
{
  "deviceId": "uuid",
  "hardwareId": "string",
  "placeId": "uuid",
  "unlinkedBy": "uuid"
}
```

---

#### Event: DeviceDisconnected

| Field | Value |
|-------|-------|
| **Bounded Context** | BC-03 Device Management |
| **Aggregate** | Device |
| **Trigger** | No reading received for the configured threshold (default: 15 minutes) |
| **Consumers** | BC-08 Alerts & Detection (create SENSOR_DISCONNECTED alert) |
| **Topic** | `device.disconnected` |
| **Version** | v1 |

**Payload:**
```json
{
  "deviceId": "uuid",
  "placeId": "uuid",
  "lastReportAt": "ISO 8601",
  "disconnectedAt": "ISO 8601"
}
```

---

#### Event: DeviceReconnected

| Field | Value |
|-------|-------|
| **Bounded Context** | BC-03 Device Management |
| **Aggregate** | Device |
| **Trigger** | A reading is received from a previously disconnected device |
| **Consumers** | BC-08 Alerts & Detection (clear sensor disconnected alert) |
| **Topic** | `device.reconnected` |
| **Version** | v1 |

**Payload:**
```json
{
  "deviceId": "uuid",
  "placeId": "uuid",
  "reconnectedAt": "ISO 8601"
}
```

---

#### Event: ReadingReceived

| Field | Value |
|-------|-------|
| **Bounded Context** | BC-03 Device Management |
| **Aggregate** | Device |
| **Trigger** | ESP32 sends a flow measurement via MQTT |
| **Consumers** | BC-04 Consumption Monitoring (aggregate into consumption), BC-08 Alerts & Detection (feed leak detection) |
| **Topic** | `device.reading.received` |
| **Delivery guarantee** | At-least-once |
| **Version** | v1 |

**Payload:**
```json
{
  "readingId": "uuid",
  "deviceId": "uuid",
  "placeId": "uuid",
  "flowRate": 2.5,
  "volume": 0.42,
  "timestamp": "ISO 8601"
}
```

**Idempotency:** Consumers use `readingId` or `(deviceId + timestamp)` as deduplication key.

---

### BC-05: Consumption Goals Context *(Entrega 2)*

#### Event: GoalThresholdReached

| Field | Value |
|-------|-------|
| **Bounded Context** | BC-05 Consumption Goals |
| **Aggregate** | — (domain service: GoalEvaluator) |
| **Trigger** | Monthly consumption reaches 80% of the defined goal |
| **Consumers** | BC-08 Alerts & Detection (create GOAL_WARNING alert) |
| **Topic** | `consumption.goal.threshold` |
| **Version** | v1 |

**Payload:**
```json
{
  "placeId": "uuid",
  "month": "2026-09",
  "goalAmount": 15000,
  "currentAmount": 12000,
  "percentage": 80,
  "unit": "LITERS"
}
```

---


### BC-06: Billing & Tariffs Context *(Entrega 3)*

#### Event: BillMismatchDetected

| Field | Value |
|-------|-------|
| **Bounded Context** | BC-06 Billing & Tariffs |
| **Aggregate** | — (domain service: BillComparator) |
| **Trigger** | Sensor-vs-bill comparison exceeds 15% deviation |
| **Consumers** | BC-08 Alerts & Detection (create BILL_MISMATCH alert) |
| **Topic** | `billing.bill.mismatch` |
| **Version** | v1 |

**Payload:**
```json
{
  "billId": "uuid",
  "placeId": "uuid",
  "billedVolume": 18000,
  "measuredVolume": 14500,
  "deviationPercent": 24.1,
  "billingPeriodStart": "2026-08-01",
  "billingPeriodEnd": "2026-08-31"
}
```

---

### BC-08: Alerts & Detection Context *(Entrega 2)*

#### Event: LeakSuspected

| Field | Value |
|-------|-------|
| **Bounded Context** | BC-08 Alerts & Detection |
| **Aggregate** | — (domain service: LeakDetector) |
| **Trigger** | Continuous flow above threshold for sustained period (default: 30 min) |
| **Consumers** | BC-07 Valve Control (evaluate auto-shutoff rule), BC-09 Notification Delivery (send critical alert) |
| **Topic** | `alerts.leak.suspected` |
| **Version** | v1 |

**Payload:**
```json
{
  "placeId": "uuid",
  "deviceId": "uuid",
  "detectedAt": "ISO 8601",
  "durationMinutes": 35,
  "volumeConsumed": 120.5,
  "isNighttime": true,
  "unit": "LITERS"
}
```

**Throttle:** No more than one event per hour for the same place and ongoing leak.

---

### BC-07: Valve Control Context *(Entrega 2)*

#### Event: ValveClosed

| Field | Value |
|-------|-------|
| **Bounded Context** | BC-07 Valve Control |
| **Aggregate** | Valve |
| **Trigger** | Device confirms valve closure (manual or auto) |
| **Consumers** | BC-08 Alerts & Detection (create VALVE_CLOSED critical alert) |
| **Topic** | `valve.closed` |
| **Version** | v1 |

**Payload:**
```json
{
  "valveId": "uuid",
  "placeId": "uuid",
  "closedBy": "uuid",
  "origin": "MANUAL | AUTO_LEAK | AUTO_GOAL",
  "confirmedAt": "ISO 8601"
}
```

---

#### Event: ValveOpened

| Field | Value |
|-------|-------|
| **Bounded Context** | BC-07 Valve Control |
| **Aggregate** | Valve |
| **Trigger** | Device confirms valve opening (always manual) |
| **Consumers** | BC-09 Notification Delivery (confirm water restored) |
| **Topic** | `valve.opened` |
| **Version** | v1 |

**Payload:**
```json
{
  "valveId": "uuid",
  "placeId": "uuid",
  "openedBy": "uuid",
  "confirmedAt": "ISO 8601"
}
```

---

#### Event: ValveCommandFailed

| Field | Value |
|-------|-------|
| **Bounded Context** | BC-07 Valve Control |
| **Aggregate** | Valve |
| **Trigger** | Device does not confirm within 30 seconds |
| **Consumers** | BC-08 Alerts & Detection (alert user that command failed) |
| **Topic** | `valve.command.failed` |
| **Version** | v1 |

**Payload:**
```json
{
  "valveId": "uuid",
  "placeId": "uuid",
  "requestedBy": "uuid",
  "requestedAction": "OPEN | CLOSE",
  "reason": "TIMEOUT | DEVICE_OFFLINE"
}
```

---

## Event Summary Table

| Event | Origin BC | Topic | Consumer BCs | Version | Entrega |
|-------|----------|-------|-------------|---------|---------|
| UserRegistered | BC-01 IAM | `iam.user.registered` | BC-09 Delivery | v1 | MVP |
| UserVerified | BC-01 IAM | `iam.user.verified` | — | v1 | MVP |
| SessionStarted | BC-01 IAM | `iam.session.started` | — | v1 | MVP |
| AccountBlocked | BC-01 IAM | `iam.account.blocked` | BC-09 Delivery | v1 | MVP |
| PasswordChanged | BC-01 IAM | `iam.password.changed` | BC-01 IAM | v1 | MVP |
| PlaceCreated | BC-02 Place | `place.created` | BC-04 Monitoring | v1 | MVP |
| PlaceDeleted | BC-02 Place | `place.deleted` | BC-04 Monitoring, BC-08 Alerts | v1 | MVP |
| MemberInvited | BC-02 Place | `place.member.invited` | BC-09 Delivery | v1 | 3 |
| MemberAccessRevoked | BC-02 Place | `place.member.revoked` | BC-01 IAM | v1 | 3 |
| DeviceLinked | BC-03 Device | `device.linked` | BC-04 Monitoring, BC-07 Valve | v1 | MVP |
| DeviceUnlinked | BC-03 Device | `device.unlinked` | BC-07 Valve, BC-04 Monitoring | v1 | MVP |
| DeviceDisconnected | BC-03 Device | `device.disconnected` | BC-08 Alerts | v1 | MVP |
| DeviceReconnected | BC-03 Device | `device.reconnected` | BC-08 Alerts | v1 | MVP |
| ReadingReceived | BC-03 Device | `device.reading.received` | BC-04 Monitoring, BC-08 Alerts | v1 | MVP |
| GoalThresholdReached | BC-05 Goals | `consumption.goal.threshold` | BC-08 Alerts | v1 | 2 |
| BillMismatchDetected | BC-06 Billing | `billing.bill.mismatch` | BC-08 Alerts | v1 | 3 |
| LeakSuspected | BC-08 Alerts | `alerts.leak.suspected` | BC-07 Valve, BC-09 Delivery | v1 | 2 |
| ValveClosed | BC-07 Valve | `valve.closed` | BC-08 Alerts | v1 | 2 |
| ValveOpened | BC-07 Valve | `valve.opened` | BC-09 Delivery | v1 | 2 |
| ValveCommandFailed | BC-07 Valve | `valve.command.failed` | BC-08 Alerts | v1 | 2 |

---

## Policies — Automatic reactions to events

| Trigger event | Policy | Emitted command / action | Responsible BC |
|--------------|--------|--------------------------|---------------|
| ReadingReceived | Aggregate reading into consumption by hour/day/month | Update consumption records | BC-04 Monitoring |
| ReadingReceived | Check for continuous flow pattern | Evaluate leak detection rules | BC-08 Alerts |
| DeviceDisconnected | If threshold exceeded, generate alert | CreateAlert (SENSOR_DISCONNECTED) | BC-08 Alerts |
| GoalThresholdReached | Generate informative alert | CreateAlert (GOAL_WARNING) | BC-08 Alerts |
| BillMismatchDetected | Generate important alert | CreateAlert (BILL_MISMATCH) | BC-08 Alerts |
| LeakSuspected | Generate critical alert + evaluate auto-shutoff | CreateAlert (LEAK_SUSPECTED) + EvaluateShutoffRule | BC-08 Alerts + BC-07 Valve |
| ValveClosed | Generate critical alert on all channels | CreateAlert (VALVE_CLOSED) | BC-08 Alerts |
| UserRegistered (LOCAL) | Send verification email | SendEmail (verification link) | BC-09 Delivery |
| PasswordChanged | Invalidate all active sessions | InvalidateSessions | BC-01 IAM |
| MemberAccessRevoked | Invalidate member's sessions for place | InvalidateSessionsByPlace | BC-01 IAM |

---

## Resilience patterns

### Delivery guarantee

All domain events use **at-least-once** delivery. Consumers must be idempotent.

### Idempotency strategy

Each consumer stores processed `eventId` values. Before processing, check if the event was already handled. Use `(eventId)` or `(aggregateId + eventType + occurredAt)` as deduplication keys.

### Dead Letter Queue (DLQ)

| Configuration | Value |
|--------------|-------|
| Retries before DLQ | 3 |
| Backoff | Exponential (1s → 2s → 4s) |
| DLQ retention | 7 days |
| Alert | When DLQ has > 0 messages |

---

## Correlations

- Bounded contexts → `02-domain/domain-map.md`
- Entities and aggregates → `02-domain/entities-and-rules.md`
- Architecture communication patterns → `05-architecture/overview.md`
- Data model → `06-data/models.md`
