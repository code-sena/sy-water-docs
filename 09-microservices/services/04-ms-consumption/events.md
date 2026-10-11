# Events — ms-consumption

## Events this service PUBLISHES

None.

## Events this service CONSUMES

| Event | Publisher | Queue | Action |
|-------|-----------|-------|--------|
| `device.reading.received` | ms-devices | `consumption.readings` (durable, dead letters to `sywater.dlx`) | Store the reading once and add its liters to `consumption_hourly` |

Idempotency: if a reading with the same `deviceId` and `recordedAt` exists, the message is acknowledged
and ignored. A malformed message (impossible values) is rejected to `sywater.dead-letters`.

## Synchronous integration

| Direction | Endpoint | Purpose |
|-----------|----------|---------|
| ms-consumption → ms-places | `GET /api/places/{id}` (user's JWT) | Ownership check (404 for someone else's place) |
| ms-places → ms-consumption | `GET /api/places/{id}/consumption` (user's JWT) | Cost of the period (tariffs) |
