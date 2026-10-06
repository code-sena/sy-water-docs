# ms-iam

> **Identity authority** of Cuida Tu Agua. Owns users, credentials, roles and sessions, signs the
> JWT that every other service trusts, and issues one-time codes (e-mail verification, password
> recovery and confirmation of sensitive actions such as closing the valve).

---

## Location in the architecture

| Field | Value |
|-------|-------|
| Number in catalog | 01 |
| Bounded context | BC-01 IAM (Supporting) |
| Repository | `sy-water-backend/ms-iam` |
| Database repository | `sy-water-db/ms-iam-db` (Liquibase; also creates the `sy-water-db` database) |
| Local port | **3001** (`server.port` in `application.yml`) |
| Stack | Java 21 · Spring Boot 4.1.1 (Web MVC, Security, OAuth2 Resource Server, Data JPA, Data Redis, Mail, Actuator, Validation) |
| Storage | SQL Server, schema `security` (login `security_app`) · Redis (revoked access tokens) · local folder `uploads/avatars` |
| Communicates with | SMTP (Mailpit in development) · Redis |
| Consumed by | App, admin panel, ms-valve (verifies close-valve codes); every service validates its JWT |

---

## Responsibilities (what this service DOES)

| HU | Capability |
|----|------------|
| HU-001 | Register with first name, last name, e-mail, optional phone and a strong password (BCrypt) |
| HU-002 | Verify the e-mail with a 6-digit code (24 h, 5 attempts, resend after 60 s) |
| HU-003 | Login with differentiated errors; 5 failures → 15-minute lockout; access token 1 h, refresh token 7 days with rotation |
| HU-005 | Password recovery by e-mail or phone with a 6-digit code (15 min); closes every session |
| HU-006 | Logout: revokes the refresh token and puts the access token `jti` in the Redis denylist |
| HU-007 | Profile: names, phone, avatar (JPG/PNG ≤ 2 MB) and password change (e-mail notice) |
| HU-008 | Account deletion (soft delete, data anonymized, sessions revoked) |
| HU-020 | Action codes: `POST /api/users/me/action-codes` sends a 6-digit code (5 min) to confirm closing the valve; `…/verify` is called by ms-valve |
| HU-061 | Roles `USER` and `ADMIN` in the `roles` claim (ADMIN is granted only by SQL script) |

## Out of scope (what it does NOT do)

- Places, devices, consumption or valve data — they belong to their own services.
- Resource authorization: each service checks that the place belongs to the user.
- Google sign-in (HU-004) — postponed; the `user_credentials` table already accepts `GOOGLE`.
- Unlinking devices when an account is deleted — today `NoDeviceCleanup` is a no-op (pending, see `decisions.md`).

---

## How to run it locally

```powershell
cd sy-water-backend\ms-iam
docker compose up -d                                   # Redis + Mailpit
# keys/private.pem and keys/public.pem must exist (see runbook.md §4)
$env:SECURITY_APP_PASSWORD="********"
.\mvnw spring-boot:run                                 # "Tomcat started on port 3001"
curl http://localhost:3001/actuator/health             # {"status":"UP"}
```

Requires `ms-iam-db` applied (release `v1.3-iam-action-codes`). Full procedure:
`10-devops/local-setup.md`.

---

## Emitted JWT

```json
{
  "iss": "https://iam.cuidatuagua.local",
  "sub": "c0a8012e-…",
  "jti": "5b1f…",
  "roles": ["USER"],
  "iat": 1791182400,
  "exp": 1791186000
}
```

- Algorithm **RS256**, header `kid = iam-key-1`. Private key: `keys/private.pem` (never in Git).
- The other services validate it with a copy of `keys/public.pem` saved as `keys/iam-public.pem`.

---

## Configuration

| Key | Purpose |
|-----|---------|
| `SECURITY_APP_PASSWORD` (env) | Password of the `security_app` login |
| `spring.data.redis.*` | Redis (denylist) |
| `spring.mail.*`, `iam.mail.from` | SMTP server and sender |
| `iam.jwt.*` | Issuer, key id, TTLs, key paths |
| `iam.codes.*` | TTLs (24 h / 15 min / 5 min), max attempts (5), resend cooldown (60 s) |
| `iam.lockout.*` | 5 attempts, 15 minutes |
| `iam.cors-allowed-origins` | Web origins (Expo web) |

---

## Related documents

- [data-model.md](./data-model.md) · [events.md](./events.md) · [decisions.md](./decisions.md) · [runbook.md](./runbook.md)
- API contract: `07-api/contracts/openapi/ms-iam.yaml`
