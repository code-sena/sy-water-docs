# Events — ms-valve

## Events this service PUBLISHES

None. The owner is notified by ms-notifications from the device's own report.

## Events this service CONSUMES

Queue `valve.device-events` (durable, dead letters to `sywater.dlx`). Every handler is idempotent.

| Event | Publisher | Action |
|-------|-----------|--------|
| `device.linked` | ms-devices | Create the valve of the place with state `UNKNOWN` |
| `device.unlinked` | ms-devices | Remove the valve; an order in progress can no longer be confirmed |
| `device.valve.reported` | ms-devices | Save the real state and `last_confirmed_at`; if it carries the `commandId` of a `SENT` order, mark it `ACK_SUCCESS` |

## Synchronous integration

| Direction | Endpoint | Purpose |
|-----------|----------|---------|
| ms-valve → ms-places | `GET /api/places/{id}` (user's JWT) | Ownership check |
| ms-valve → ms-iam | `POST /api/users/me/action-codes/verify` (user's JWT) | Check and consume the close code (204 = valid) |
| ms-valve → ms-devices | `POST /api/places/{id}/device/valve-commands` (user's JWT + `X-Internal-Key`) | Publish the order on `sywater/devices/{serial}/valve/set` |
