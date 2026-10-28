# Data Model — ms-notifications

> Schema `notification` in `sy-water-db` (singular, unlike the `notifications` name planned in
> `06-data/models.md`). Source of truth: `ms-notification-db` (Liquibase, up to **v1.7**; this file said `v1.4-device-liveness` until 2026-10-28).
> ER diagram: `08-uml/diagrams/exports/ER-06_notification.png`.

## Database engine

| Item | Value |
|------|-------|
| Schema | `notification` |
| Runtime login | `notification_app` |
| ORM | EF Core 10 without migrations |

## Tables

| Table | Purpose | Key columns |
|-------|---------|-------------|
| `notifications` | Inbox entry for a user **or** a role | `id` PK · `user_id` FK NULL · `role` NULL · `type` (`DEVICE_LINKED`, `DEVICE_UNLINKED`, `VALVE_CHANGED`, `DEVICE_OFFLINE`, `SYSTEM_ANNOUNCEMENT`, and since v1.7 `COLLABORATOR_INVITATION`, `INVITATION_ANSWERED`) · `severity` (`INFO`/`WARNING`/`CRITICAL`) · `title` · `body` · `place_id` FK NULL · `source_event_id` · `created_at`. UQ `(source_event_id, user_id, role)` de-duplicates redelivered events |
| `notification_reads` | Read marks | PK (`notification_id`, `user_id`) · `read_at` |
| `notification_preferences` | Channels per user and level; only levels the user changed | PK (`user_id`, `severity`) · `in_app` · `push` · `email` · `sms` · `updated_at` |
| `place_devices` | Local projection place ↔ meter fed by events | PK `place_id` · `device_id` UQ · `user_id` · `serial_number` · `valve_state` · `valve_reported_at` · `last_reading_at` · `offline_alerted_at` · `created_at` |
| `email_outbox` | E-mails to send, with retries | `id` PK · `user_id` · `source_event_id` · `subject` · `text_body` · `html_body` · `status` (`PENDING`/`SENT`/`FAILED`) · `attempts` · `next_attempt_at` · `last_error` · `sent_at`. UQ `(source_event_id, user_id)` |
| `push_tokens` | Expo push tokens of each phone | `id` PK · `user_id` · `token` UQ · `platform` (`ANDROID`/`IOS`) · `created_at` · `last_seen_at` |
| `sms_outbox` *(v1.6)* | SMS/WhatsApp messages to send, with retries (HU-028) | `id` PK (set by the app) · `user_id` · `source_event_id` NULL · `body` (320) · `status` (`PENDING`/`SENT`/`FAILED`/`SKIPPED`) · `channel` (`SMS`/`WHATSAPP`, required when SENT) · `provider_message_id` · `attempts` · `next_attempt_at` · `last_error` · `created_at` · `sent_at` |
| `message_templates` *(v1.7)* | Texts edited by admins; a missing row means the built-in default | PK `template_key` (`ALERT_CRITICAL`, `PHONE_CODE`, `COLLABORATOR_INVITATION`, `GUEST_LINK_SHARE`) · `body` (320, not blank) · `updated_by` FK → `security.users` NULL · `updated_at` |

`last_reading_at` is saved at most every 30 s per device to avoid one write per reading.

## Releases *(added 2026-10-28)*

| Release | Content |
|---------|---------|
| v1.0 baseline | `notifications`, `notification_reads` |
| v1.1 | `place_devices` |
| v1.2 | `notification_preferences` |
| v1.3 | `email_outbox` |
| v1.4 | Liveness columns on `place_devices` |
| v1.5 | `push_tokens` |
| v1.6 | `sms_outbox` |
| v1.7 | `message_templates`; `CK_notif_type` with the invitation types |

PK deviations from ADR-003 (accepted, ADR-019): composite PKs in `notification_reads` and `notification_preferences`,
`place_id` in `place_devices`, `template_key` in `message_templates`.
