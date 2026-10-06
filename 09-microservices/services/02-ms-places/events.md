# Events — ms-places

## Events this service PUBLISHES

None in the current version.

## Events this service CONSUMES

None.

## Synchronous integration

| Direction | Endpoint | Purpose |
|-----------|----------|---------|
| ms-devices, ms-consumption, ms-valve → ms-places | `GET /api/places/{id}` with the user's JWT | Ownership check: 200 = owner, 404 = not found or not the owner |
| ms-places → ms-devices | `GET /api/places/{id}/device` | Refuse deleting a place with a linked meter (`place.has_active_device`, 409). If ms-devices does not answer → 503, never "assume no device" |

## Planned

| Event | Consumer |
|-------|----------|
| `place.deleted` | ms-consumption / ms-valve (cleanup), when members and invitations exist |
