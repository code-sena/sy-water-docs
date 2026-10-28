# ADR-012 — Branching, Continuous Integration and Release Process

| Field | Value |
|-------|-------|
| **ID** | ADR-012 |
| **Date** | 2026-10-22 (decision, approximate) · 2026-10-28 (recorded) |
| **Status** | Accepted |
| **Authors** | Juan Esteban Ome, Juan Manuel Gómez, Diego Alejandro Alvarado |
| **Reviewers** | Juan Esteban Ome, Juan Manuel Gómez, Diego Alejandro Alvarado |

> Recorded retroactively at project close. The decision was taken and implemented during the project; this record
> describes it as built in the `main` branches on 2026-10-28. Where the alternatives were not written down at the
> time, this is stated instead of reconstructing them.

---

## Context

Until October deployments were made by building images on the VPS by hand. Risk R-001 (no container images) and the
need to demonstrate a controlled path to production required a defined branch model, automatic tests on every pull
request, and an approved, reversible release.

## Evaluated alternatives

| Alternative | Pros | Cons | Reason for discarding |
|------------|------|------|-----------------------|
| **A — GitHub Actions for PR checks + Jenkins for releases (chosen)** | Checks run next to the PR without exposing Jenkins; Jenkins keeps credentials for the VPS and the approval step | Two tools | — (chosen) |
| B — Jenkins polling the repositories for PRs | One tool | Jenkins would need public webhooks or polling of 16 repos | Exposure and load on the VPS |
| C — GitHub Actions for everything (deploy by SSH) | One tool | VPS SSH key stored in 16 repositories; no central approval queue | Credential spread |
| D — Staging environment before production | Safer releases | A second VPS/compose stack outside budget (ADR-013) | Budget |

## Decision

**We decided:**

1. **Branches:** `feature/*` / `fix/*` → PR → `dev` → PR → `main`. `hotfix/*` may go straight to `main` and must
   then be merged back into `dev`.
2. **Rules (GitHub ruleset `dev-main-protegidas`):** no direct pushes to `dev`/`main`, PR required, required checks
   `test` (workflow `ci.yml`) and `source-branch` (workflow `enforce-source-branch.yml`, which only allows `dev` or
   `hotfix/*` as the source of a PR to `main`).
3. **Release:** Jenkins job `release` per service: build image, push to GHCR tagged with the commit SHA, **manual
   approval by the release owner (esteban)**, then `deploy-service.sh` on the VPS: back up the current tag, pull, start,
   wait up to 5 minutes for `healthy`, and roll back to the previous tag if it does not become healthy.
4. Database releases are applied before the service release (ADR-010).

## Decisions not taken

- **No staging environment** — budget (ADR-013); the local Docker compose is the only pre-production check.
- **No backup approver** — one approver was named; if he is unavailable no release can be approved. Accepted for a
  three-person team.
- **Firmware outside the flow** — `cuida-tu-agua-device-ESP` has only `dev`, no CI and is flashed by hand (E-14).
- **No automatic back-merge** after hotfixes — done by hand; it was skipped twice (`branch-status.md` §6).

## Consequences

- Positive: every image in production is traceable to a commit; failed deploys revert by themselves.
- Negative: the release history lives in Jenkins; which SHA is running per service is only known from Jenkins or
  `docker ps` on the VPS (not recorded in this repository).

## References

- `10-devops/ci-cd.md`
- `15-project-control/closure/branch-status.md`
