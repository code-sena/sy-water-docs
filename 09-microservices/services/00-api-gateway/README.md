# 00 · api-gateway

> Repository: `cuida-tu-agua/api-gateway` · local folder `sy-water-backend/api-gateway` · Status: ✅ Built

Single entry point of Cuida Tu Agua. Spring Boot 4.1 + Spring Cloud Gateway (WebFlux), port **8080**.

## What it does
- Routes by prefix: `/iam`, `/places`, `/devices`, `/consumption`, `/valve`, `/notifications` (StripPrefix=1).
- Validates the ms-iam JWT (RS256 public key, issuer, expiry, Redis revocation list). Services validate again.
- `/…/api/admin/**` requires the ADMIN role.
- CORS for the web (the `Origin` header is removed before forwarding).
- Rate limit (Redis): auth endpoints 10/min per IP; API 20/s per user.
- `X-Correlation-Id` on every request and one access-log line per request.
- `/*/internal/**`, `/*/actuator/**`, `/*/openapi/**`, `/*/health` → 404.

## Run
```powershell
.\mvnw.cmd spring-boot:run          # needs Redis and ../ms-iam/keys/public.pem
docker compose up --build -d        # alternative
```

## Environment variables
| Variable | Default |
|---|---|
| `IAM_URL` … `NOTIFICATIONS_URL` | `http://localhost:3001` … `3006` |
| `REDIS_HOST` / `REDIS_PORT` | `localhost` / `6379` |
| `IAM_PUBLIC_KEY` | `file:../ms-iam/keys/public.pem` (public key only) |
| `IAM_JWT_ISSUER` | `https://iam.cuidatuagua.local` |
| `GATEWAY_CORS_ALLOWED_ORIGINS` | `http://localhost:8081,http://localhost:19006` |

## Tests
```powershell
.\mvnw.cmd test
```

## Route map

Rule: **service prefix + the usual path**. The gateway removes the first segment (`StripPrefix=1`) and forwards.

| Gateway prefix | Service | Target (local) | Example |
|---|---|---|---|
| `/iam/**` | ms-iam | `http://localhost:3001` | `/iam/api/auth/login` → `:3001/api/auth/login` |
| `/places/**` | ms-places | `http://localhost:3002` | `/places/api/places` → `:3002/api/places` |
| `/devices/**` | ms-device | `http://localhost:3003` | `/devices/api/places/{id}/device` → `:3003/api/places/{id}/device` |
| `/consumption/**` | ms-consumption | `http://localhost:3004` | `/consumption/api/places/{id}/consumption` |
| `/valve/**` | ms-valve | `http://localhost:3005` | `/valve/api/places/{id}/valve/close` |
| `/notifications/**` | ms-notification | `http://localhost:3006` | `/notifications/api/notifications` |

**Public routes (no token)** — same list as `SecurityConfig.PUBLIC_POSTS` in ms-iam:

| Method | Gateway path |
|---|---|
| POST | `/iam/api/auth/register`, `/iam/api/auth/verify-email`, `/iam/api/auth/verify-email/resend`, `/iam/api/auth/login`, `/iam/api/auth/refresh`, `/iam/api/auth/password/forgot`, `/iam/api/auth/password/reset` |
| GET | `/iam/api/avatars/**` (profile photos) |
| GET | `/actuator/health` (health of the gateway itself) |

**ADMIN only:** `/iam/api/admin/**`, `/places/api/admin/**`, `/devices/api/admin/**`.
**Everything else:** a valid token.

**Never exposed (404):** `/*/internal/**`, `/*/actuator/**`, `/*/openapi/**`, `/*/health`.

Why prefixes and not the same paths: several services share the start `/api/places/{id}/…` (places, device,
consumption, valve) and `/api/admin/…` (iam, places, device). With a prefix every route is unambiguous and the
front only changes its six `EXPO_PUBLIC_*` variables (no code change).
