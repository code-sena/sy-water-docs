# Events — ms-iam

## Events this service PUBLISHES

None in the current version. ms-iam has no RabbitMQ connection: everything it offers is REST.

## Events this service CONSUMES

None.

## Planned

| Event | When | Consumer | Delivery |
|-------|------|----------|----------|
| `user.deleted` | HU-008: account deleted | ms-devices (unlink the user's meters) | Alternative to the internal REST call `DELETE /api/internal/users/{id}/devices` chosen in the 2-day plan |
| `user.blocked` | HU-060: admin blocks an account | ms-notifications (optional e-mail) | Entrega 4 |

## Synchronous integration (instead of events)

| Caller | Endpoint | Purpose |
|--------|----------|---------|
| ms-valve | `POST /api/users/me/action-codes/verify` (with the user's JWT) | Confirms the 6-digit code before closing the valve (HU-020) |
