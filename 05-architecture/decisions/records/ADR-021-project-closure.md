# ADR-021 — Project Closure and Scope Freeze

| Field | Value |
|-------|-------|
| **ID** | ADR-021 |
| **Date** | 2026-10-28 |
| **Status** | Accepted |
| **Authors** | Juan Esteban Ome, Juan Manuel Gómez, Diego Alejandro Alvarado |
| **Reviewers** | Juan Esteban Ome, Juan Manuel Gómez, Diego Alejandro Alvarado |

---

## Context

The project reached its final delivery. The team decided that the code will not change any more and that the
documentation must record the final state, what was not done and why.

## Decision

**We decided:**

1. **The final state of each repository is its `main` branch** at 2026-10-28 (`15-project-control/closure/branch-status.md`).
2. No further code changes, merges, back-merges or branch clean-ups are planned. Missing back-merges (`iam-backend`,
   `infra`) and abandoned branches are documented, not fixed.
3. Every item not implemented is recorded with its reason in
   `15-project-control/closure/out-of-scope-register.md`. Open risks stay open with their last status.
4. Any future reopening must start from `main`, not from `dev`.

## Evaluated alternatives

| Alternative | Reason for discarding |
|------------|-----------------------|
| Integrate the abandoned frontend branches before closing | They are based on an obsolete structure; re-implementation is new development |
| Do the back-merges `main` → `dev` | Changes repositories; out of the closure mandate. `main` is already complete |
| Delete abandoned branches | They are history; kept as evidence of the work done |

## Consequences

- The documentation repository is the only one updated at close.
- Production versions are those deployed by Jenkins from `main`; this repository does not record the SHA running per
  service (Jenkins history is the authority).

## References

- `15-project-control/closure/README.md`
