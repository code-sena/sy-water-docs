# Data Model — ms-iam

> Schema `security` in the shared database `sy-water-db` (SQL Server 2022).
> Source of truth: repository `ms-iam-db` (Liquibase). Full dictionary: `06-data/models.md`.

---

## Database engine

| Item | Value |
|------|-------|
| Engine | SQL Server 2022 (container `sy_water_db_dev`) |
| Schema | `security` |
| Runtime login | `security_app` (read/write on `security`) |
| Migration login | `security_migrator` |
| ORM | Spring Data JPA, `ddl-auto: validate` (the schema belongs to Liquibase) |
| Other storage | Redis 7 (revoked `jti` and "revoked before" marks per user) · `uploads/avatars` on disk |

## Releases

| Tag | Content |
|-----|---------|
| `v1.0-baseline` | Legacy model (replaced) |
| `v1.1-iam-model-v5` | Current tables: users, credentials, roles, user_roles, tokens, verifications, resets, attempts, activity log; seed roles `USER` and `ADMIN` |
| `v1.2-iam-e1` | `failed_attempts` on `email_verifications` and `password_resets`; index for active refresh tokens |
| `v1.3-iam-action-codes` | `action_codes` (HU-020) |
| `v1.4` | Seed of the official ADMIN accounts *(added 2026-10-28)* |
| `v1.5` | `action_codes`: action `PHONE_VERIFY`, `target_phone`, `channel` (`SMS`/`WHATSAPP`); `users.phone_channel`; `CK_users_phone_verified` (verified phone needs number and channel) — HU-028 *(added 2026-10-28)* |

---

## Tables

### `security.users`
| Column | Type | Notes |
|--------|------|-------|
| `id` | UNIQUEIDENTIFIER PK | `NEWID()` |
| `first_name`, `last_name` | NVARCHAR(100) | ≥ 2 characters |
| `email` | NVARCHAR(320) UQ | Login identifier, not editable |
| `phone` | NVARCHAR(20) NULL | Optional (HU-001), unique among active users |
| `phone_verified`, `email_verified` | BIT | |
| `avatar_url` | NVARCHAR(500) NULL | HU-007 |
| `account_locked_until` | DATETIME2 NULL | 15-minute lockout (HU-003) |
| `blocked_at`, `blocked_by` | DATETIME2 / FK users | Admin block (HU-060, pending UI) |
| `deleted_at` | DATETIME2 NULL | Soft delete (HU-008) |
| `created_at`, `updated_at` | DATETIME2 | UTC |

### `security.user_credentials`
`id` BIGINT · `user_id` FK · `credential_type` LOCAL\|GOOGLE · `password_hash` (BCrypt) · `google_id` · UQ (`user_id`, `credential_type`).

### `security.roles` and `security.user_roles`
`roles(id, code USER|ADMIN, description)` · `user_roles(user_id, role_id, assigned_at, assigned_by)` with PK (`user_id`, `role_id`).

### `security.refresh_tokens`
`id` · `user_id` · `token_hash` UQ (SHA-256, the token is never stored) · `device_info` · `expires_at` · `revoked_at` · `created_at`.

### One-time codes
| Table | Purpose | TTL |
|-------|---------|-----|
| `email_verifications` | HU-002 | 24 h |
| `password_resets` (`channel` EMAIL\|PHONE) | HU-005 | 15 min |
| `action_codes` (`action` VALVE_CLOSE \| PHONE_VERIFY; `target_phone`, `channel` for PHONE_VERIFY) | HU-020, HU-028 | 5 min |

Common columns: `user_id`, `token_hash` (SHA-256 of the 6 digits), `expires_at`, `used_at`/`verified_at`,
`failed_attempts` (max 5), `created_at`.

### Audit
- `login_attempts(user_id NULL, email, ip_address, success, failure_reason, attempted_at)` — HU-003.
- `activity_log(user_id, action, entity_type, entity_id, metadata JSON, ip_address, created_at)`.

---

## Modeling decisions

- Single UUID primary key for business entities (ADR-003); `BIGINT IDENTITY` for append-only logs and tokens.
- Secrets are stored only as hashes: passwords (BCrypt), refresh tokens and codes (SHA-256).
- Soft delete anonymizes the user (`Usuario Eliminado`, `deleted-<id>@deleted.invalid`) so the e-mail can register again and foreign keys stay valid.
