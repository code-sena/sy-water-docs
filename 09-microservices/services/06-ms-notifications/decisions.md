# Design Decisions — ms-notifications

## [DEC-001] React to device events, not to other services' APIs
The service builds its own `place_devices` projection from `device.*` events, so it knows the owner
of a place and the valve state without calling ms-places or ms-valve.

## [DEC-002] De-duplication by source event
RabbitMQ delivers at least once. Unique indexes on `source_event_id` make every handler idempotent; the
valve state is saved **after** notifying so a crash in between only produces a harmless redelivery.

## [DEC-003] Outbox for e-mail
The dispatcher only queues the e-mail in `email_outbox` (unique per source event and user); a worker
sends it with backoff (1 min, 5 min, 15 min, 1 h). A slow SMTP server never blocks event processing.
Push is sent right away, and only when the in-app notification is new, so a redelivered event does not
buzz the phone twice. Broadcasts to a role are stored in-app only and never mailed.

## [DEC-004] Preferences store only changes
A level the user never touched uses the backlog default (CRITICAL all, WARNING app + push, INFO app).
CRITICAL always keeps the in-app channel (`preferences.critical_requires_in_app`, 400).

## [DEC-005] One offline alert per outage
`offline_alerted_at` is set when the alert goes out and cleared by the next reading, so a dead meter
does not produce an alert every minute.

## [DEC-006] Expo Push instead of direct FCM/APNs
The app is built with Expo; Expo Push hides the platform differences and needs no Firebase server key
in the backend.
