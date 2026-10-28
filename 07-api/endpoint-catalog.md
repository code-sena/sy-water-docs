# Endpoint Catalog (as built)

> Inventory of every HTTP endpoint in the `main` branch of each service at project close (**2026-10-28**), taken from
> the controller route attributes (`[Route]`/`[Http*]` in .NET, `@*Mapping` in ms-iam).
> This is an index, not a contract: request/response schemas are in each service's runtime OpenAPI document
> (see [README.md](./README.md) § Source of truth).
>
> **Public path** = gateway prefix + service path. Example: ms-places `GET /api/places` →
> `GET https://api.sy-water.tech/places/api/places`.

**Access legend**

| Code | Meaning |
|------|---------|
| `Anon` | No token. Rate-limited where noted |
| `JWT` | Valid access token (RS256), checked at the gateway and in the service (ADR-008) |
| `Admin` | JWT with role `ADMIN` |
| `Internal` | Header `X-Internal-Key`; path under `/internal/**`, **blocked by the gateway** (ADR-006) |
| `Place` | JWT + access to the place checked against ms-places (owner, or collaborator with the needed permission — ADR-014) |

---

## Gateway (`api-gateway`)

| Prefix / route | Forwards to | Notes |
|----------------|-------------|-------|
| `/iam/**` | ms-iam | Route `iam-auth` (`/iam/api/auth/**`) is public, with its own rate limiter |
| `/places/**` | ms-places | |
| `/devices/**` | ms-devices | |
| `/consumption/**` | ms-consumption | |
| `/valve/**` | ms-valve | |
| `/notifications/**` | ms-notifications | |
| `GET /places/api/guest/{token}` | ms-places | Route `guest-places`: `Anon`, `RedisRateLimiter(2, 20, 1)` per IP |
| `GET /consumption/api/guest/{token}/consumption` | ms-consumption | Route `guest-consumption`: `Anon`, same limiter |
| `/*/internal/**`, `/*/actuator/**`, `/*/health` | — | Route `blocked`: answered with **404** from outside |

Every route has a Redis rate limiter: `authRateLimiter` (`iam-auth`), `guestRateLimiter` (guest routes),
`apiRateLimiter` (the rest).

---

## ms-iam (`/iam`)

| Method | Path | Access | Purpose (HU) |
|--------|------|--------|--------------|
| POST | `/api/auth/register` | Anon | Sign up (HU-001) |
| POST | `/api/auth/verify-email` | Anon | Confirm e-mail with a 6-digit code (HU-002, ADR-020) |
| POST | `/api/auth/verify-email/resend` | Anon | Resend the code |
| POST | `/api/auth/login` | Anon | Log in (HU-003) |
| POST | `/api/auth/refresh` | Anon (refresh token) | Rotate tokens |
| POST | `/api/auth/logout` | JWT | Revoke the session (Redis denylist) (HU-006) |
| POST | `/api/auth/password/forgot` | Anon | Request a reset code (HU-005) |
| POST | `/api/auth/password/reset` | Anon | Reset with the code |
| GET | `/api/auth/google/status` | Anon | Is Google sign-in enabled (HU-004) |
| GET | `/api/auth/google/start` | Anon | Start OAuth |
| GET | `/api/auth/google/callback` | Anon | OAuth callback |
| POST | `/api/auth/google/exchange` | Anon | Exchange the one-time result for tokens |
| GET / PATCH / DELETE | `/api/users/me` | JWT | Profile (HU-007); delete account (HU-008) |
| GET | `/api/users/me/sign-in-methods` | JWT | Password / Google |
| POST | `/api/users/me/account/google-reauth` | JWT | Re-authenticate with Google before sensitive changes |
| PUT | `/api/users/me/password` | JWT | Change password |
| PUT / DELETE | `/api/users/me/avatar` | JWT | Profile picture |
| POST | `/api/users/me/action-codes` | JWT | Send a code for a sensitive action (`VALVE_CLOSE`) (ADR-017) |
| POST | `/api/users/me/action-codes/verify` | JWT | Verify the action code |
| POST | `/api/users/me/phone-verification` | JWT | Send a phone code by SMS/WhatsApp (HU-028) |
| POST | `/api/users/me/phone-verification/confirm` | JWT | Confirm the phone |
| GET | `/api/admin/users` | Admin | List users (HU-059) |
| PUT | `/api/admin/users/{id}/block` · `/unblock` | Admin | Block / unblock (HU-060) |
| GET | `/api/admin/metrics` | Admin | Platform metrics (HU-062) |
| GET | `/internal/users/by-email` | Internal | Resolve a user for an invitation |
| GET | `/internal/users/{id}/contact` | Internal | Contact data for notifications |
| POST | `/internal/invitation-emails` | Internal | Send the collaborator invitation e-mail |

## ms-places (`/places`)

| Method | Path | Access | Purpose (HU) |
|--------|------|--------|--------------|
| GET / POST | `/api/places` | JWT | List (HU-010) / create (HU-009) places |
| GET / PUT / DELETE | `/api/places/{id}` | Place / owner | Detail, edit, delete (HU-011) |
| PUT | `/api/places/{id}/default` | JWT | Selected place (HU-010) |
| GET | `/api/geography/countries` | JWT | Countries (ADR-004) |
| GET | `/api/geography/countries/{code}/subdivisions` | JWT | Departments / provinces |
| GET | `/api/geography/subdivisions/{id}/cities` | JWT | Cities |
| GET | `/api/places/{id}/tariff` | Place | Current tariff |
| PUT | `/api/places/{id}/tariff/manual` | Owner | Manual tariff (HU-054) |
| PUT | `/api/places/{id}/tariff/catalog` | Owner | Tariff from the catalog (HU-066, HU-069) |
| GET | `/api/places/{id}/cost` | Place | Cost of the consumption (HU-056) |
| GET | `/api/tariffs/catalog/cities/{cityId}` | JWT | Catalog for a city |
| GET | `/api/tips` | JWT | Savings tips (HU-065) |
| GET | `/api/tips/favorites` | JWT | Favorite tips |
| PUT / DELETE | `/api/tips/{tipId}/favorite` | JWT | Mark / unmark favorite |
| GET / POST | `/api/admin/tips` | Admin | List / create tips (HU-063) |
| PUT | `/api/admin/tips/{id}` | Admin | Edit a tip (HU-064) |
| PUT | `/api/admin/tips/{id}/active` | Admin | Activate / deactivate (HU-064; there is no delete) |
| GET | `/api/places/{id}/access` | JWT | Caller's access to the place (owner / collaborator + permissions) |
| DELETE | `/api/places/{id}/membership` | JWT | Collaborator leaves a place |
| GET / POST / DELETE | `/api/places/{id}/guest-link` | Owner | Read / (re)generate / deactivate the guest link (HU-039) |
| GET / POST | `/api/places/{id}/invitations` | Owner | List / send invitations (HU-042) |
| DELETE | `/api/places/{id}/invitations/{invitationId}` | Owner | Cancel an invitation |
| GET | `/api/places/{id}/members` | Owner | Collaborators |
| PUT | `/api/places/{id}/members/{memberId}/permissions` | Owner | Change permissions (HU-044) |
| DELETE | `/api/places/{id}/members/{memberId}` | Owner | Revoke (HU-046) |
| GET | `/api/shared-places` | JWT | Places shared with me |
| GET | `/api/invitations` | JWT | My pending invitations |
| POST | `/api/invitations/{id}/accept` · `/reject` | JWT | Answer an invitation (HU-043) |
| GET | `/api/guest/{token}` | Anon (rate-limited) | Guest view of a place (HU-040, HU-041) |
| GET | `/internal/metrics` | Internal | Counts for the admin metrics |
| GET | `/internal/guest-links/{token}` | Internal | Resolve a guest token (used by ms-consumption) |
| GET | `/internal/places/{id}/recipients` | Internal | Owner + collaborators to notify (HU-047) |

## ms-devices (`/devices`)

| Method | Path | Access | Purpose (HU) |
|--------|------|--------|--------------|
| GET | `/api/places/{id}/device` | Place | Device of the place and its status (HU-012, HU-013) |
| POST | `/api/places/{id}/device` | Owner | Link a device with its code (HU-012) |
| DELETE | `/api/places/{id}/device` | Owner | Unlink (HU-014) |
| POST | `/api/places/{id}/device/valve-commands` | JWT + Internal key | Publish a valve order to MQTT (only ms-valve calls it) |
| GET / POST | `/api/admin/devices` | Admin | List devices (HU-061) / factory registration (R-004) |
| GET | `/api/admin/devices/{id}` | Admin | Detail |
| POST | `/api/admin/devices/{id}/credentials` | Admin | Regenerate credentials |
| POST | `/api/admin/devices/{id}/decommission` | Admin | Decommission |
| GET | `/internal/metrics` | Internal | Counts for the admin metrics |
| DELETE | `/internal/users/{id}/devices` | Internal | Release devices of a deleted account (iam DEC-008) |

MQTT (not HTTP): `sywater/devices/{serial}/heartbeat`, `.../telemetry`, `.../valve/state` (device → server),
`.../valve/set` (server → device). See ADR-009.

## ms-consumption (`/consumption`)

| Method | Path | Access | Purpose (HU) |
|--------|------|--------|--------------|
| GET | `/api/places/{id}/consumption` | Place | Consumption by period with `tz` (HU-015 – HU-018, ADR-018) |
| GET / PUT | `/api/places/{id}/goals/current` | Place / owner | Monthly goal and progress (HU-023, HU-024) |
| GET | `/api/guest/{token}/consumption` | Anon (rate-limited) | Guest consumption (HU-040) |

## ms-valve (`/valve`)

| Method | Path | Access | Purpose (HU) |
|--------|------|--------|--------------|
| GET | `/api/places/{id}/valve` | Place | Valve state (HU-019) |
| POST | `/api/places/{id}/valve/close` | Place (`CONTROL_VALVE`) + action code | Close (HU-020, ADR-017) |
| POST | `/api/places/{id}/valve/open` | Place (`CONTROL_VALVE`) | Open (HU-021) |
| GET | `/api/places/{id}/valve/commands/{commandId}` | Place | Order status (202 + polling) |
| GET | `/api/places/{id}/valve/commands` | Place | History, `from`/`to`, default 30 days (HU-022) |

## ms-notifications (`/notifications`)

| Method | Path | Access | Purpose (HU) |
|--------|------|--------|--------------|
| GET | `/api/notifications` | JWT | Notification center (HU-025) |
| GET | `/api/notifications/unread-count` | JWT | Badge |
| POST | `/api/notifications/{id}/read` | JWT | Mark as read |
| POST | `/api/notifications/read-all` | JWT | Mark all |
| GET | `/api/notification-preferences` | JWT | Channels per severity (HU-034) |
| PUT | `/api/notification-preferences/{severity}` | JWT | Change channels for a severity |
| POST / DELETE | `/api/push-tokens` | JWT | Register / remove the Expo push token (HU-026) |
| GET | `/api/admin/message-templates` | Admin | Message templates (ADR-016) |
| PUT / DELETE | `/api/admin/message-templates/{key}` | Admin | Edit / restore default |
| POST | `/api/admin/message-templates/{key}/test` | Admin | Send a test |
| GET | `/api/admin/message-templates/providers` | Admin | Configured providers |
| GET | `/api/message-templates/{key}` | JWT | Rendered text of a template (e.g. guest link share) |
| GET | `/api/short-messages/channels` | JWT | Available channels (SMS / WhatsApp) |
| POST | `/internal/short-messages/verification-code` | Internal | Send a phone code (called by ms-iam) |
| POST | `/internal/invitations` | Internal | In-app notice of a new invitation |
| POST | `/internal/invitations/answered` | Internal | Notice to the owner of an answer |

---

## Notes

- HTTP verbs marked with `/` group several operations on the same path; the exact verb per operation is in the
  OpenAPI document of the service.
- HU numbers are the main story served; some endpoints serve several.
- Health and OpenAPI endpoints of each service are not listed (blocked or documentation only).

## Correlations

- Data behind each endpoint → `06-data/models.md`, `09-microservices/services/0N-*/data-model.md`
- Security model → ADR-008
