# ADR-005 — Hexagonal Architecture in Every Service and the Frontend

| Field | Value |
|-------|-------|
| **ID** | ADR-005 |
| **Date** | 2026-09-15 (decision, approximate) · 2026-10-28 (recorded) |
| **Status** | Accepted |
| **Authors** | Juan Esteban Ome, Juan Manuel Gómez, Diego Alejandro Alvarado |
| **Reviewers** | Juan Esteban Ome, Juan Manuel Gómez, Diego Alejandro Alvarado |

> Recorded retroactively at project close. The decision was taken and implemented during the project; this record
> describes it as built in the `main` branches on 2026-10-28. Where the alternatives were not written down at the
> time, this is stated instead of reconstructing them.

---

## Context

Six back-end services and one client were going to be written by three people in two languages (Java and C#) and
in parallel. The team needed one internal structure so that any member could open any repository and find the same
pieces in the same place, and so that domain rules could be unit-tested without SQL Server, RabbitMQ or MQTT running.

## Evaluated alternatives

| Alternative | Pros | Cons | Reason for discarding |
|------------|------|------|-----------------------|
| **A — Hexagonal (ports and adapters) (chosen)** | Domain free of frameworks; adapters replaceable (e.g. Twilio ↔ Meta, SMTP ↔ log); unit tests on the domain without infrastructure | More projects/packages and mapping code per service | — (chosen) |
| B — Classic layered (controller → service → repository) | Less code; familiar | Domain coupled to EF Core / JPA; harder to swap providers | Provider swaps were expected (push, SMS, e-mail) |
| C — Vertical slices per feature | Little ceremony per use case | No common layout across Java and .NET; rules scattered | Consistency across repos was the priority |

## Decision

**We decided:** every service follows ports and adapters with the same four parts — `Domain` (entities, value
objects, rules), `Application` (use cases and ports), `Infrastructure` (adapters: SQL Server, RabbitMQ, MQTT, SMTP,
Twilio, Expo) and `Api` (HTTP entry point). In .NET they are separate projects of one solution; in ms-iam they are
packages. The frontend (`cuida-tu-agua-frontNEW`) uses the same idea: `src/domain`, `src/application`,
`src/infrastructure` (HTTP clients) and `src/presentation` (screens, components).

## Consequences

**Positive:**
- Provider changes were made by adding an adapter only: SMS/WhatsApp by Twilio or Meta (ms-notifications), log or
  SMTP e-mail senders, in-memory or Redis rate limiting.
- Domain rules (valve state machine, goal suggestion, invitation expiry, threshold validation) are covered by unit tests
  that run in CI without containers.

**Negative / Trade-offs:**
- Boilerplate: DTO ↔ domain ↔ entity mapping in every use case.
- The rule was followed by convention, not enforced by architecture tests.

## Decisions not taken

- No CQRS/event sourcing: the read models are simple queries; event sourcing was never evaluated as necessary.
- No shared "common" library between services: each service duplicates its small cross-cutting code (JWT validation,
  internal-key filter) to keep deployments independent.

## References

- `09-microservices/services/*/README.md` § Architecture
