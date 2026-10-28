# ADR-008 — Single API Gateway with Defense-in-Depth JWT Validation

| Field | Value |
|-------|-------|
| **ID** | ADR-008 |
| **Date** | 2026-10-14 (decision, approximate) · 2026-10-28 (recorded) |
| **Status** | Accepted |
| **Authors** | Juan Esteban Ome, Juan Manuel Gómez, Diego Alejandro Alvarado |
| **Reviewers** | Juan Esteban Ome, Juan Manuel Gómez, Diego Alejandro Alvarado |

> Recorded retroactively at project close. The decision was taken and implemented during the project; this record
> describes it as built in the `main` branches on 2026-10-28. Where the alternatives were not written down at the
> time, this is stated instead of reconstructing them.

---

## Context

The first plan exposed every service directly behind Caddy. `09-microservices/README.md` recorded at one point that
the gateway "was never built". It was built (`api-gateway`, Spring Cloud Gateway) in October and is the only public
entry point of the API. `service-catalog.md` referenced "ADR-008" for the security model, but the record did not exist.

## Evaluated alternatives

| Alternative | Pros | Cons | Reason for discarding |
|------------|------|------|-----------------------|
| **A — Spring Cloud Gateway + JWT also validated in each service (chosen)** | One public host and CORS policy; central rate limiting and revoked-token check; services still safe if reached directly | JWT validated twice | — (chosen) |
| B — Caddy routing per path, no gateway | No extra container | No JWT/Redis logic at the edge; rate limits per service | Revocation and guest rate limits need application logic |
| C — Gateway as the only JWT check (services trust it) | Simpler services | One misconfigured route exposes a service without auth | Rejected: defense in depth |
| D — JWKS endpoint + HS256 shared secret (key options, iam DEC-001) | — | HS256 spreads a signing secret to every service; JWKS adds an endpoint to operate | RS256 with a distributed public key chosen |

## Decision

**We decided:**

1. `api.sy-water.tech` → Caddy → **api-gateway** → services. Routes by prefix: `/iam`, `/places`, `/devices`,
   `/consumption`, `/valve`, `/notifications` (prefix stripped before forwarding).
2. The gateway **blocks** `/*/internal/**`, `/*/actuator/**` and `/*/health` from outside (route `blocked`, 404).
   Every route has a Redis rate limiter (auth, guest and general API limiters).
3. The gateway validates the JWT (RS256, public key of ms-iam) and checks the Redis denylist of revoked tokens.
   **Every service validates the JWT again** with the same public key (defense in depth).
4. Anonymous routes are explicit: `iam-auth` (login, register, recovery, Google) and the guest routes
   `/places/api/guest/{token}` and `/consumption/api/guest/{token}/consumption`, rate-limited per IP with
   `RedisRateLimiter(2, 20, 1)`.

## Consequences

**Positive:**
- One host for the clients; one CORS policy; internal routes unreachable from the Internet.

**Negative / Trade-offs:**
- The gateway is a single point of failure (one instance, ADR-013).
- The Redis denylist is checked at the gateway and in ms-iam, but **not inside ms-devices** (risk R-005, open): a
  revoked token used directly against ms-devices inside the Docker network would pass. Mitigated because ms-devices is
  not published outside the network.

## References

- `09-microservices/services/00-api-gateway/README.md`
- `09-microservices/services/01-ms-iam/decisions.md` DEC-001
