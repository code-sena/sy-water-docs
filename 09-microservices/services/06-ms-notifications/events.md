# Events — ms-notifications

## Events this service PUBLISHES

None.

## Events this service CONSUMES

Queue `notification.device-events` (durable, dead letters to `sywater.dlx`). Handlers are idempotent:
the source event id is part of a unique index, so a redelivered event never notifies twice.

| Event | Publisher | Action | Notification |
|-------|-----------|--------|--------------|
| `device.linked` | ms-devices | Create the `place_devices` projection (owner of the place) | INFO "Dispositivo vinculado" |
| `device.unlinked` | ms-devices | Remove the projection | INFO "Dispositivo desvinculado" |
| `device.valve.reported` | ms-devices | Compare with the last known state; only a change notifies | CRITICAL "Válvula cerrada" / INFO "Válvula abierta" |
| `device.reading.received` | ms-devices | Proof of life (`last_reading_at`); re-arms the offline alert | — |

Reports or readings of a device that is no longer linked to the place are ignored.

## Internal producer

| Job | Period | Notification |
|-----|--------|--------------|
| Offline detection | 60 s | WARNING "sensor disconnected" once per outage (10 min without readings) |

## Synchronous integration

| Direction | Endpoint | Purpose |
|-----------|----------|---------|
| ms-notifications → ms-iam | `GET /internal/users/{id}/contact` + `X-Internal-Key` | E-mail address and name for the outbox |
| ms-notifications → Expo | `POST https://exp.host/--/api/v2/push/send` | Push delivery (timeout 10 s) |
| ms-notifications → SMTP | MailKit | E-mail delivery (timeout 15 s) |
