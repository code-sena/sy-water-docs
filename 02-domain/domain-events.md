# Domain Events

> Events are facts that occurred in the business, stated in past tense.
> They are the backbone of asynchronous communication between bounded contexts.

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

### IAM Context

#### Event: UserRegistered

| Field | Value |
|-------|-------|
| **Bounded Context** | IAM |
| **Aggregate** | User |
| **Trigger** | User completes registration (local or Google OAuth) |
| **Consumers** | notification-service (send verification email for local accounts) |
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
| **Bounded Context** | IAM |
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
| **Bounded Context** | IAM |
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
| **Bounded Context** | IAM |
| **Aggregate** | User |
| **Trigger** | 5 consecutive failed login attempts |
| **Consumers** | notification-service (notify user of blocked account) |
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
| **Bounded Context** | IAM |
| **Aggregate** | User |
| **Trigger** | User changes or resets their password |
| **Consumers** | auth-service (invalidate all active sessions) |
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

### Place Management Context

#### Event: PlaceCreated

| Field | Value |
|-------|-------|
| **Bounded Context** | Place Management |
| **Aggregate** | Place |
| **Trigger** | Owner registers a new place |
| **Consumers** | consumption-service (provision storage for the new place) |
| **Topic** | `place.created` |
| **Version** | v1 |

**Payload:**
```json
{
  "placeId": "uuid",
  "ownerId": "uuid",
  "name": "string",
  "country": "string",
  "city": "string",
  "stratum": "int | null"
}
```

---

#### Event: PlaceDeleted

| Field | Value |
|-------|-------|
| **Bounded Context** | Place Management |
| **Aggregate** | Place |
| **Trigger** | Owner deletes a place (after unlinking device) |
| **Consumers** | consumption-service (archive data), notification-service (cancel pending alerts) |
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

#### Event: GuestInvited *(Entrega 3)*

| Field | Value |
|-------|-------|
| **Bounded Context** | Place Management |
| **Aggregate** | Place |
| **Trigger** | Owner sends an invitation to a person |
| **Consumers** | notification-service (send invitation email) |
| **Topic** | `place.guest.invited` |
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

#### Event: GuestAccessRevoked *(Entrega 3)*

| Field | Value |
|-------|-------|
| **Bounded Context** | Place Management |
| **Aggregate** | Place |
| **Trigger** | Owner removes a guest's access |
| **Consumers** | auth-service (invalidate guest's sessions for this place) |
| **Topic** | `place.guest.revoked` |
| **Version** | v1 |

**Payload:**
```json
{
  "placeId": "uuid",
  "guestUserId": "uuid",
  "revokedBy": "uuid"
}
```

---

### Device Management Context

#### Event: DeviceLinked

| Field | Value |
|-------|-------|
| **Bounded Context** | Device Management |
| **Aggregate** | Device |
| **Trigger** | Owner links a device to a place (via hardware ID or QR scan) |
| **Consumers** | consumption-service (start expecting readings for this place), valve-service (create valve record) |
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
| **Bounded Context** | Device Management |
| **Aggregate** | Device |
| **Trigger** | Owner unlinks a device from a place |
| **Consumers** | valve-service (deactivate valve record), consumption-service (stop expecting readings) |
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
| **Bounded Context** | Device Management |
| **Aggregate** | Device |
| **Trigger** | No reading received for the configured threshold (default: 15 minutes) |
| **Consumers** | notification-service (sensor disconnected alert) |
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
| **Bounded Context** | Device Management |
| **Aggregate** | Device |
| **Trigger** | A reading is received from a previously disconnected device |
| **Consumers** | notification-service (clear sensor disconnected alert) |
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
| **Bounded Context** | Device Management |
| **Aggregate** | Device |
| **Trigger** | ESP32 sends a flow measurement via MQTT |
| **Consumers** | consumption-service (aggregate into consumption), notification-service (feed leak detection) |
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

### Consumption Context

#### Event: GoalThresholdReached *(Entrega 2)*

| Field | Value |
|-------|-------|
| **Bounded Context** | Consumption |
| **Aggregate** | — (domain service: GoalEvaluator) |
| **Trigger** | Monthly consumption reaches 80% of the defined goal |
| **Consumers** | notification-service (informative alert) |
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

#### Event: GoalExceeded *(Entrega 2)*

| Field | Value |
|-------|-------|
| **Bounded Context** | Consumption |
| **Aggregate** | — (domain service: GoalEvaluator) |
| **Trigger** | Monthly consumption reaches 100% of the defined goal |
| **Consumers** | notification-service (important alert), valve-service (evaluate auto-shutoff rule) |
| **Topic** | `consumption.goal.exceeded` |
| **Version** | v1 |

**Payload:**
```json
{
  "placeId": "uuid",
  "month": "2026-09",
  "goalAmount": 15000,
  "currentAmount": 15200,
  "percentage": 101,
  "unit": "LITERS",
  "daysRemaining": 22
}
```

---

#### Event: LeakSuspected *(Entrega 2)*

| Field | Value |
|-------|-------|
| **Bounded Context** | Consumption |
| **Aggregate** | — (domain service: LeakDetector) |
| **Trigger** | Continuous flow above threshold for sustained period (default: 30 min) |
| **Consumers** | notification-service (critical alert), valve-service (evaluate auto-shutoff rule) |
| **Topic** | `consumption.leak.suspected` |
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

### Valve Control Context *(Entrega 2)*

#### Event: ValveClosed

| Field | Value |
|-------|-------|
| **Bounded Context** | Valve Control |
| **Aggregate** | Valve |
| **Trigger** | Device confirms valve closure (manual or auto) |
| **Consumers** | notification-service (critical alert — all 4 channels) |
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
| **Bounded Context** | Valve Control |
| **Aggregate** | Valve |
| **Trigger** | Device confirms valve opening (always manual) |
| **Consumers** | notification-service (confirm water restored) |
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
| **Bounded Context** | Valve Control |
| **Aggregate** | Valve |
| **Trigger** | Device does not confirm within 30 seconds |
| **Consumers** | notification-service (alert user that command failed) |
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

| Event | Origin context | Topic | Consumers | Version | Entrega |
|-------|---------------|-------|-----------|---------|---------|
| UserRegistered | IAM | `iam.user.registered` | notification-service | v1 | MVP |
| UserVerified | IAM | `iam.user.verified` | — | v1 | MVP |
| SessionStarted | IAM | `iam.session.started` | — | v1 | MVP |
| AccountBlocked | IAM | `iam.account.blocked` | notification-service | v1 | MVP |
| PasswordChanged | IAM | `iam.password.changed` | auth-service | v1 | MVP |
| PlaceCreated | Place Mgmt | `place.created` | consumption-service | v1 | MVP |
| PlaceDeleted | Place Mgmt | `place.deleted` | consumption, notification | v1 | MVP |
| GuestInvited | Place Mgmt | `place.guest.invited` | notification-service | v1 | 3 |
| GuestAccessRevoked | Place Mgmt | `place.guest.revoked` | auth-service | v1 | 3 |
| DeviceLinked | Device Mgmt | `device.linked` | consumption, valve | v1 | MVP |
| DeviceUnlinked | Device Mgmt | `device.unlinked` | valve, consumption | v1 | MVP |
| DeviceDisconnected | Device Mgmt | `device.disconnected` | notification-service | v1 | MVP |
| DeviceReconnected | Device Mgmt | `device.reconnected` | notification-service | v1 | MVP |
| ReadingReceived | Device Mgmt | `device.reading.received` | consumption, notification | v1 | MVP |
| GoalThresholdReached | Consumption | `consumption.goal.threshold` | notification-service | v1 | 2 |
| GoalExceeded | Consumption | `consumption.goal.exceeded` | notification, valve | v1 | 2 |
| LeakSuspected | Consumption | `consumption.leak.suspected` | notification, valve | v1 | 2 |
| ValveClosed | Valve Control | `valve.closed` | notification-service | v1 | 2 |
| ValveOpened | Valve Control | `valve.opened` | notification-service | v1 | 2 |
| ValveCommandFailed | Valve Control | `valve.command.failed` | notification-service | v1 | 2 |

---

## Policies — Automatic reactions to events

| Trigger event | Policy | Emitted command / action | Service |
|--------------|--------|--------------------------|---------|
| ReadingReceived | Aggregate reading into consumption by hour/day/month | Update consumption records | consumption-service |
| ReadingReceived | Check for continuous flow pattern | Evaluate leak detection rules | notification-service (LeakDetector) |
| DeviceDisconnected | If threshold exceeded, generate alert | CreateAlert (SENSOR_DISCONNECTED) | notification-service |
| GoalThresholdReached | Generate informative alert (in-app only) | CreateAlert (GOAL_WARNING) | notification-service |
| GoalExceeded | Generate important alert + evaluate auto-shutoff | CreateAlert (GOAL_EXCEEDED) + EvaluateShutoffRule | notification + valve |
| LeakSuspected | Generate critical alert + evaluate auto-shutoff | CreateAlert (LEAK_SUSPECTED) + EvaluateShutoffRule | notification + valve |
| ValveClosed | Generate critical alert on all 4 channels | CreateAlert (VALVE_CLOSED) | notification-service |
| UserRegistered (LOCAL) | Send verification email | SendEmail (verification link) | notification-service |
| PasswordChanged | Invalidate all active sessions for that user | InvalidateSessions | auth-service |
| GuestAccessRevoked | Invalidate guest's sessions for that place | InvalidateSessionsByPlace | auth-service |

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
