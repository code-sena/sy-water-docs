# ADR-017 — Remote Valve Safety Rules

| Field | Value |
|-------|-------|
| **ID** | ADR-017 |
| **Date** | 2026-10-05 (decision, approximate) · 2026-10-28 (recorded) |
| **Status** | Accepted |
| **Authors** | Juan Esteban Ome, Juan Manuel Gómez, Diego Alejandro Alvarado |
| **Reviewers** | Juan Esteban Ome, Juan Manuel Gómez, Diego Alejandro Alvarado |

> Recorded retroactively at project close. The decision was taken and implemented during the project; this record
> describes it as built in the `main` branches on 2026-10-28. Where the alternatives were not written down at the
> time, this is stated instead of reconstructing them.

---

## Context

Closing the water supply remotely is the most sensitive action of the system: a wrong close leaves a home without
water; an order to an offline meter must not appear as done.

## Evaluated alternatives

| Alternative | Pros | Cons | Reason for discarding |
|------------|------|------|-----------------------|
| **A — Second factor to close, state confirmed by the device, timeout (chosen)** | Prevents accidental or stolen-session closes; UI shows the real state | One extra step to close | — (chosen) |
| B — Close and open without extra step | Fastest | Accidental closes | Safety |
| C — Second factor for both close and open | Symmetric | Opening restores supply; an extra step there only delays | Opening is the safe direction |
| D — Optimistic state (assume success) | Instant UI | UI lies when the meter is offline | Rejected |

## Decision

**We decided:**

1. **Close** requires a 6-digit code e-mailed by ms-iam (`/api/users/me/action-codes`, purpose `VALVE_CLOSE`). **Open**
   does not.
2. The valve state changes only when the device confirms it on `.../valve/state`.
3. One pending command per place (`UX_vcmd_place_pending`, filtered unique index); a command not confirmed in 30 s is
   marked as timed out (sweeper every 5 s).
4. If the device is offline (ADR-015) the order is rejected (`NoCommunication`).
5. Collaborators can operate the valve only with the `CONTROL_VALVE` permission (ADR-014, HU-044, HU-045).

## Consequences

- Positive: no false "closed" state; no double orders.
- Not taken: **automatic close on leak** (E8, HU-035–037) — the enum value `AUTO_LEAK` exists in the command origin,
  but no rule produces it (E-04).

## References

- `09-microservices/services/05-ms-valve/README.md`
- `09-microservices/services/05-ms-valve/decisions.md` DEC-001, DEC-004
