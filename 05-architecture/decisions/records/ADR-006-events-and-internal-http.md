# ADR-006 — Service Integration: RabbitMQ Events plus Internal HTTP

| Field | Value |
|-------|-------|
| **ID** | ADR-006 |
| **Date** | 2026-09-20 (decision, approximate) · 2026-10-28 (recorded) |
| **Status** | Accepted |
| **Authors** | Juan Esteban Ome, Juan Manuel Gómez, Diego Alejandro Alvarado |
| **Reviewers** | Juan Esteban Ome, Juan Manuel Gómez, Diego Alejandro Alvarado |

> Recorded retroactively at project close. The decision was taken and implemented during the project; this record
> describes it as built in the `main` branches on 2026-10-28. Where the alternatives were not written down at the
> time, this is stated instead of reconstructing them.

---

## Context

Several flows cross services: a reading arrives in ms-devices and must reach ms-consumption and ms-notifications; a
valve confirmation must update ms-valve and notify users; ms-places must answer "can this user see this place?" for
every other service; ms-notifications needs the contact data owned by ms-iam. ADR-002 forbids reading another
service's schema directly.

## Evaluated alternatives

| Alternative | Pros | Cons | Reason for discarding |
|------------|------|------|-----------------------|
| **A — RabbitMQ topic exchange for facts + synchronous HTTP for queries (chosen)** | Producers do not wait for consumers; queries get a fresh answer; both are standard | Two mechanisms to operate | — (chosen) |
| B — HTTP only | One mechanism | Producer blocked by consumer availability; fan-out by hand | Telemetry fan-out to two consumers would block ms-devices |
| C — Events only (replicate all data) | Full decoupling | Every service keeps copies of places/members/contacts; eventual consistency on permissions | Permission checks must be current (a revoked collaborator must lose access at once) |
| D — Kafka | Replay, partitions | Heavier to run on one small VPS | Not justified for the volume; the VPS budget (ADR-013) |

## Decision

**We decided:**

1. **Events** go through one RabbitMQ topic exchange, `sywater.events`, with routing keys `<domain>.<fact>`. At close
   the only publisher is ms-devices: `device.linked`, `device.unlinked`, `device.reading.received`,
   `device.valve.reported`. Consumers own durable queues — `consumption.readings` (ms-consumption),
   `valve.device-events` (ms-valve), `notification.device-events` (ms-notifications); failed messages go to the
   dead-letter exchange `sywater.dlx`. Consumers are idempotent (they ignore an event
   whose id or natural key was already processed).
2. **Queries and commands that need an answer now** use internal HTTP between services, authenticated with the
   header `X-Internal-Key` (shared secret per environment). Internal routes live under `/internal/**` and are blocked
   by the gateway (ADR-008), so they are not reachable from the Internet.
3. ms-places is the authority for place access (`GET /api/places/{id}/access`, called with the user's JWT by
   ms-devices, ms-consumption and ms-valve); ms-iam is the authority for user contact data. Sharing (ADR-014) was
   integrated by HTTP only: ms-places calls ms-iam and ms-notifications for invitations; it publishes no events.

## Consequences

**Positive:**
- A stopped consumer (e.g. ms-notifications during a deploy) loses nothing: its queue accumulates and drains on start.
- Access revocation is effective on the next request because it is asked, not replicated.

**Negative / Trade-offs:**
- Synchronous calls create runtime coupling: if ms-places is down, place-scoped endpoints of other services fail.
- One shared `X-Internal-Key` per environment, not a key per service pair; no mTLS between containers (they share a
  private Docker network).
- No outbox pattern in the producers: an event can be lost if the broker is down at the moment of publishing. Accepted
  for the project scope; ms-notifications does use an outbox for e-mail and SMS delivery.

## References

- `09-microservices/service-catalog.md` § Communication matrix
- `09-microservices/services/03-ms-devices/decisions.md` DEC-006
- ADR-008 (gateway blocks `/internal/**`)
