# ADR-011 — Single Expo Codebase for Web and Android, Admin Integrated and Web-Only

| Field | Value |
|-------|-------|
| **ID** | ADR-011 |
| **Date** | 2026-10-09 / 2026-10-27 (decision, approximate) · 2026-10-28 (recorded) |
| **Status** | Accepted |
| **Authors** | Juan Esteban Ome, Juan Manuel Gómez, Diego Alejandro Alvarado |
| **Reviewers** | Juan Esteban Ome, Juan Manuel Gómez, Diego Alejandro Alvarado |

> Recorded retroactively at project close. The decision was taken and implemented during the project; this record
> describes it as built in the `main` branches on 2026-10-28. Where the alternatives were not written down at the
> time, this is stated instead of reconstructing them.

---

## Context

The product needs a mobile app (Android) and a web app, plus an administration area (users, devices, tips, metrics,
message templates). `service-catalog.md` described a separate Vite panel `sy-water-admin`. The first frontend
(`cuida-tu-agua-frontend`) was a separate mobile project.

## Evaluated alternatives

| Alternative | Pros | Cons | Reason for discarding |
|------------|------|------|-----------------------|
| **A — One Expo/React Native project for web and Android, admin inside it (chosen)** | One codebase, one design system, one i18n; shared HTTP clients | Web layout constraints of React Native Web | — (chosen) |
| B — Separate mobile app + web app | Best native/web UX each | Two codebases for three people | Capacity |
| C — Separate admin panel (`sy-water-admin`, Vite) | Admin isolated | Third frontend to build and deploy | Dropped; not part of the delivered system |

## Decision

**We decided:** `cuida-tu-agua-frontNEW` (Expo SDK 57) builds the web app (static export served by Caddy at
`sy-water.tech`) and the Android APK (EAS). The admin area is a set of screens inside the same app behind the ADMIN
role. On 2026-10-27 the admin area was made **web-only**: `canUseAdmin(roles, os)` hides it on native even for ADMIN
users, because the admin tables were designed for wide screens.

## Consequences

- Positive: one release pipeline for the client (Jenkins `web` job + EAS build); features land on both platforms.
- Negative: the legacy repository `cuida-tu-agua-frontend` remains with abandoned branches (`branch-status.md` §4).
- Not taken: iOS build — requires an Apple Developer account (paid), not obtained (E-10).
- Not taken: separate `sy-water-admin` panel (E-11).

## References

- `09-microservices/service-catalog.md` § System map
