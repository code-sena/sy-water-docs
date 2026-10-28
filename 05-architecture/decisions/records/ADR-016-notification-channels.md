# ADR-016 — Notification Channels and Providers

| Field | Value |
|-------|-------|
| **ID** | ADR-016 |
| **Date** | 2026-10-10 / 2026-10-26 (decision, approximate) · 2026-10-28 (recorded) |
| **Status** | Accepted |
| **Authors** | Juan Esteban Ome, Juan Manuel Gómez, Diego Alejandro Alvarado |
| **Reviewers** | Juan Esteban Ome, Juan Manuel Gómez, Diego Alejandro Alvarado |

> Recorded retroactively at project close. The decision was taken and implemented during the project; this record
> describes it as built in the `main` branches on 2026-10-28. Where the alternatives were not written down at the
> time, this is stated instead of reconstructing them.

---

## Context

ms-notifications delivers in-app notifications, push, e-mail and (HU-028) SMS/WhatsApp codes and alerts. Each external
channel has a provider with cost, approval and account limits.

## Evaluated alternatives

| Channel | Chosen | Alternatives considered | Reason |
|---------|--------|-------------------------|--------|
| Push | **Expo Push service with FCM V1 credentials** | Direct FCM and APNs (ms-notifications DEC-006) | One API for Android (and iOS if ever built); the app already uses Expo |
| E-mail | **SMTP through an outbox table** (ms-notifications DEC-003) with retries (1 min, 5 min, 15 min, 1 h; 5 attempts) | Transactional e-mail SaaS (SendGrid, Mailgun) — not evaluated in depth | Free and enough for the volume |
| SMS / WhatsApp | **Twilio**, WhatsApp with approved Content templates | Meta WhatsApp Cloud API (implemented as an alternate adapter, `WhatsApp:Provider = meta`) | Twilio is the provider configured in production; Meta is kept as a switchable adapter. The comparison criteria were not written down at the time |

## Decision

**We decided:**

- In-app notifications are the source; push, e-mail and SMS/WhatsApp are fan-out channels filtered by
  `notification_preferences` per severity.
- E-mail and SMS go through `email_outbox` and `sms_outbox`, drained by workers every 10 s.
- Message texts for SMS/WhatsApp/e-mail live in `notification.message_templates` (keys `ALERT_CRITICAL`,
  `PHONE_CODE`, `COLLABORATOR_INVITATION`, `GUEST_LINK_SHARE`), editable by ADMIN in the app.
- **In production the SMS sender is disabled** (`TWILIO_SMS_FROM` empty): only WhatsApp is used, because one SMS to
  Colombia costs about US$0.06–0.34 against a free WhatsApp template within the trial/session limits.

## Consequences

- Positive: the provider is a configuration choice (`Sms:Provider` = `twilio` | `mailpit` for development; `WhatsApp:Provider` = `twilio` | `meta`), not a code change (ADR-005).
- Negative: the Twilio account has trial-style limits (verified recipients, sandbox/template approval).
- Delivery of the WhatsApp verification code in production was **confirmed working on 2026-10-28** (E-12 closed).
- Not taken: iOS push (no iOS build, E-10); SMS in production (cost).

## References

- `09-microservices/services/06-ms-notifications/README.md`
