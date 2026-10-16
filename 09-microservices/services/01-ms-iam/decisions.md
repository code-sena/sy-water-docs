# Design Decisions — ms-iam

## [DEC-001] RS256 with a local key pair instead of HS256 or a JWKS endpoint

- **Decision:** ms-iam signs with an RSA 2048 private key (`keys/private.pem`, PKCS#8). The other services receive a copy of the public key (`keys/iam-public.pem`).
- **Why:** services only need the public key to validate; a leaked public key cannot sign tokens. A JWKS endpoint is not needed while there is a single key.
- **Consequence:** rotating the key means copying the new public key to every service and restarting them; existing sessions become invalid. Each developer generates their own pair locally — the private key is never shared or committed.

## [DEC-002] Opaque refresh tokens with rotation and reuse detection

- Refresh tokens are random, stored only as SHA-256 and rotated on every `/refresh`.
- If a **revoked** refresh token is presented again, ms-iam revokes every session of that user (theft detection).

## [DEC-003] Access-token revocation in Redis

- Logout puts the `jti` in Redis until the token expires; password change/reset stores a "revoked before" timestamp per user.
- Every service checks Redis on each request and **fails closed** (401) if Redis is down.

## [DEC-004] One-time codes instead of links

- E-mail verification, password recovery and action confirmation use 6-digit codes typed in the app (works the same on mobile and web, no deep-link setup).
- Codes are stored hashed, expire, allow 5 attempts and can be resent after 60 s; a new code invalidates the previous one.

## [DEC-005] Lockout policy

- 5 consecutive failed logins lock the account for 15 minutes (HU-003). Every attempt is stored in `login_attempts`.

## [DEC-006] The ADMIN role is granted only by SQL

- `ms-iam-db/scripts/grant-admin.sql` and `revoke-admin.sql`. No endpoint can create administrators, so a stolen admin account cannot create more admins.

## [DEC-007] Java/Spring Boot for this service

- ms-iam was started in Java (Spring Security, OAuth2 Resource Server). Domain services were later built in .NET 10 (ADR-007). Both validate the same RS256 tokens.

## [DEC-008] Account deletion releases the devices first

- `HttpDeviceCleanup` calls `DELETE /internal/users/{id}/devices` on ms-devices with `X-Internal-Key`. If ms-devices is configured but does not answer, the deletion fails with 503 and nothing changes, so no device stays linked to a deleted account. Without `iam.integration.devices-url` it only logs (development).

## Pending

- **HU-008:** the user's places are not soft-deleted when the account is deleted (risk R-011).
- **HU-004:** Google sign-in.
