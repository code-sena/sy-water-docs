# Database Migrations History (as built)

> Every Liquibase baseline and release applied to `sy-water-db` up to project close (**2026-10-28**).
> Source: the `main` branch of the six database repositories (`06-releases/vX.Y/`). Dates are the date the release
> folder was added to the repository, not the date it was applied in production (applying is a manual step, ADR-010;
> its log is the Liquibase `DATABASECHANGELOG` table of each environment, not recorded here).

---

## Summary

| Repository | Schema(s) | Baseline | Releases | Last release | Tables at close |
|------------|-----------|----------|----------|--------------|-----------------|
| `ms-iam-db` | `security` | v1.0 (2026-09-01) | v1.1 – v1.5 | v1.5 (2026-10-23) | 10 |
| `ms-places-db` | `places` | v1.0 (2026-09-26) | v1.1 – v1.4 | v1.4 (2026-10-24) | 12 |
| `ms-device-db` | `devices` | v1.0 (2026-09-29) | v1.1 – v1.2 | v1.2 (2026-10-23) | 3 |
| `ms-consumption-db` | `monitoring`, `goals` | v1.0 (2026-10-03) | v1.1 | v1.1 (2026-10-23) | 2 + 1 |
| `ms-valve-db` | `valve` | v1.0 (2026-10-03) | — | baseline | 2 |
| `ms-notification-db` | `notification` | v1.0 (2026-10-07) | v1.1 – v1.7 | v1.7 (2026-10-26) | 8 |
| **Total** | **7 schemas** | 6 baselines | **19 releases** | | **38** |

---

## ms-iam-db (`security`)

| Release | Date | Changes |
|---------|------|---------|
| v1.0 baseline | 2026-09-01 | First model (PascalCase tables: users, roles, permissions, sessions, audit, error log, security config, password policies). **Superseded by v1.1.** |
| v1.1 | 2026-09-23 | Drops the v1.0 legacy tables. Creates the model v5 tables with a single UUID PK (ADR-003): `users`, `user_credentials`, `email_verifications`, `password_resets`, `refresh_tokens`, `login_attempts`, `activity_log`, `roles`, `user_roles`; FKs, indexes; seed roles `USER`, `ADMIN`. |
| v1.2 | 2026-09-30 | Failed-attempt counter on codes; index for active refresh tokens. |
| v1.3 | 2026-10-03 | `action_codes` (one-time codes for sensitive actions, purpose `VALVE_CLOSE`) (ADR-017, ADR-020). |
| v1.4 | 2026-10-08 | Seed of the official ADMIN accounts (iam DEC-006: ADMIN is granted only by SQL). |
| v1.5 | 2026-10-23 | `action_codes`: purpose `PHONE_VERIFY`, `target_phone`, channel `SMS`/`WHATSAPP`; `users.phone_channel`; check constraint on phone verification (HU-028). |

## ms-places-db (`places`)

| Release | Date | Changes |
|---------|------|---------|
| v1.0 baseline | 2026-09-26 | `countries`, `subdivisions`, `cities` (Colombia and Ecuador, ADR-004), `places`. |
| v1.1 | 2026-10-01 | `activity_log` (BIGINT IDENTITY PK, deviation recorded in ADR-019); DENY of updates/deletes on the log. |
| v1.2 | 2026-10-08 | `tariff_catalog`, `place_tariffs`; seed of the Colombian tariff catalog (HU-054, HU-056, HU-066, HU-069). |
| v1.3 | 2026-10-08 | `tips`, `tip_favorites`; seed of starter tips (HU-063 – HU-065). |
| v1.4 | 2026-10-24 | Sharing (ADR-014): `place_members`, `invitations`, `guest_links`; FKs and indexes. |

## ms-device-db (`devices`)

| Release | Date | Changes |
|---------|------|---------|
| v1.0 baseline | 2026-09-29 | `devices`, `device_link_history`. |
| v1.1 | 2026-10-09 | `device_admin_log` (`REGISTERED`, `CREDENTIALS_REGENERATED`, `DECOMMISSIONED`); grants for factory registration by an administrator (risk R-004; admin device list HU-061). |
| v1.2 | 2026-10-23 | Default `inactivity_threshold_minutes` = 1 and patch of existing devices to 1 minute (ADR-015). |

## ms-consumption-db (`monitoring`, `goals`)

| Release | Date | Changes |
|---------|------|---------|
| v1.0 baseline | 2026-10-03 | `monitoring.readings`, `monitoring.consumption_hourly` (ADR-018). |
| v1.1 | 2026-10-23 | Schema `goals`: `consumption_goals` (month start on day 1, `target_liters`, `source` MANUAL/SUGGESTED/CARRIED_OVER — only MANUAL used by the code); role `goals_rw` and grants (HU-023, HU-024). |

## ms-valve-db (`valve`)

| Release | Date | Changes |
|---------|------|---------|
| v1.0 baseline | 2026-10-03 | `valve_states`, `valve_commands`; filtered unique index `UX_vcmd_place_pending` (one pending order per place); index for the 30 s timeout sweep (ADR-017). No later releases. |

## ms-notification-db (`notification`)

| Release | Date | Changes |
|---------|------|---------|
| v1.0 baseline | 2026-10-07 | `notifications`, `notification_reads`. |
| v1.1 | 2026-10-07 | `place_devices` (local projection of place ↔ device ↔ owner, PK `place_id`). |
| v1.2 | 2026-10-07 | `notification_preferences` (only the changes from the defaults are stored). |
| v1.3 | 2026-10-07 | `email_outbox`. |
| v1.4 | 2026-10-07 | Liveness columns on `place_devices`; index for silent devices. |
| v1.5 | 2026-10-09 | `push_tokens` (HU-026). |
| v1.6 | 2026-10-23 | `sms_outbox` (HU-028). |
| v1.7 | 2026-10-26 | `message_templates` (keys `ALERT_CRITICAL`, `PHONE_CODE`, `COLLABORATOR_INVITATION`, `GUEST_LINK_SHARE`; FK `updated_by` → `security.users`); `CK_notif_type` extended with `COLLABORATOR_INVITATION` and `INVITATION_ANSWERED`. |

---

## Rules that applied to every release

Forward-only, backward-compatible with the running service, applied by hand before the service release and after a
backup, with a rollback per changeset in `05-rollbacks` (ADR-010).

## Correlations

- Schema set → `models.md` § As-built status, ADR-019
- Per-table detail → `09-microservices/services/0N-*/data-model.md`
