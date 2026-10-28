# ADR-007 — Polyglot Backend: Java for IAM and Gateway, .NET 10 for Domain Services

| Field | Value |
|-------|-------|
| **ID** | ADR-007 |
| **Date** | 2026-09-12 (decision, approximate) · 2026-10-28 (recorded) |
| **Status** | Accepted |
| **Authors** | Juan Esteban Ome, Juan Manuel Gómez, Diego Alejandro Alvarado |
| **Reviewers** | Juan Esteban Ome, Juan Manuel Gómez, Diego Alejandro Alvarado |

> Recorded retroactively at project close. The decision was taken and implemented during the project; this record
> describes it as built in the `main` branches on 2026-10-28. Where the alternatives were not written down at the
> time, this is stated instead of reconstructing them.

---

## Context

The team had people stronger in Java/Spring and people stronger in C#/.NET. The identity service and the edge
(gateway) needed mature ready-made security components; the domain services needed fast iteration, EF Core and a
light runtime on a small VPS. `01-ms-iam/decisions.md` (DEC-007) and other documents referenced "ADR-007" for this
choice, but the record had never been written (risk R-010).

## Evaluated alternatives

| Alternative | Pros | Cons | Reason for discarding |
|------------|------|------|-----------------------|
| **A — Java/Spring for ms-iam and api-gateway, .NET 10 for the rest (chosen)** | Spring Security + Spring Cloud Gateway for security and edge; .NET minimal APIs and EF Core for the domain; each member works in the stack they know | Two toolchains, two CI templates, two dependency ecosystems | — (chosen) |
| B — All Java/Spring | One stack | Higher memory per container on the VPS; slower for the .NET-skilled members | Team skills and memory budget |
| C — All .NET | One stack | Gateway (YARP) and identity would be built from lower-level parts | Spring components were already known for IAM |
| D — Node.js for some services | Same language as the frontend | Third ecosystem; no member owned it on the back end | Not evaluated in depth |

## Decision

**We decided:** ms-iam (Java 21, Spring Boot 4) and api-gateway (Spring Cloud Gateway) are Java. ms-places,
ms-devices, ms-consumption, ms-valve and ms-notifications are .NET 10 (ASP.NET Core minimal APIs, EF Core, hexagonal).
Services interoperate only through HTTP, JWT and RabbitMQ (ADR-006), so the language is invisible across boundaries.

## Consequences

**Positive:**
- Each service builds and deploys independently as a container image (ADR-012); language does not matter at runtime.

**Negative / Trade-offs:**
- JWT validation and the internal-key filter are implemented twice (Java and C#).
- OpenAPI is exposed in two formats/paths (`/v3/api-docs` for Spring, `/openapi/v1.json` for .NET), aggregated by the
  documentation UI (`07-api/README.md`).
- Two CI pipelines (`mvn verify`, `dotnet test`).

## References

- `09-microservices/services/01-ms-iam/decisions.md` DEC-007
