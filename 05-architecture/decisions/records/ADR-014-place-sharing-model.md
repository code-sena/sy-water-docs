# ADR-014 — Place Sharing: Guest Links and Collaborators with Permissions

| Field | Value |
|-------|-------|
| **ID** | ADR-014 |
| **Date** | 2026-10-22 (decision, approximate) · 2026-10-28 (recorded) |
| **Status** | Accepted |
| **Authors** | Juan Esteban Ome, Juan Manuel Gómez, Diego Alejandro Alvarado |
| **Reviewers** | Juan Esteban Ome, Juan Manuel Gómez, Diego Alejandro Alvarado |

> Recorded retroactively at project close. The decision was taken and implemented during the project; this record
> describes it as built in the `main` branches on 2026-10-28. Where the alternatives were not written down at the
> time, this is stated instead of reconstructing them.

---

## Context

Owners wanted to show consumption to family members without giving them an account (epic E9, HU-039 – HU-041) and
to let trusted people operate a place (epic E10, HU-042 – HU-047). The original model had only an owner per place; `place_members` and
`invitations` were in the data design but not created (as-built note of 2026-10-10).

## Evaluated alternatives

| Alternative | Pros | Cons | Reason for discarding |
|------------|------|------|-----------------------|
| **A — Two mechanisms: anonymous guest link (read-only) + collaborators by e-mail invitation with per-place permissions (chosen)** | Guests need no account; collaborators get only what is granted; revocation is immediate | Two flows to build | — (chosen) |
| B — Only collaborators (everyone needs an account) | One flow | Friction for a relative who only wants to look | UX |
| C — Roles per place (OWNER/EDITOR/VIEWER) instead of permission flags | Simpler UI | Cannot grant "valve" without "settings" | Granularity requested |
| D — Replicate membership to each service via events | No synchronous calls | Revoked access remains until the event is processed | Revocation must be immediate (ADR-006) |

## Decision

**We decided:**

- **Guest link:** `places.guest_links` holds one active token (≥ 32 random characters) per place. Anyone with the
  link can read the place summary (`/places/api/guest/{token}`) and consumption
  (`/consumption/api/guest/{token}/consumption`) without logging in. The gateway rate-limits these routes per IP. The
  owner can regenerate or deactivate the link.
- **Collaborators:** the owner invites by e-mail (`places.invitations`, status PENDING/ACCEPTED/EXPIRED/REJECTED,
  expiry); the invitation is shown in the app and e-mailed with a button to `/invitations`. On acceptance a row in
  `places.place_members` (role COLLABORATOR, `permissions` list, `revoked_at`) is created.
- **Permissions** (flags per collaborator): `VIEW_CONSUMPTION`, `VIEW_DEVICE_STATUS`, `VIEW_ALERTS`,
  `CONTROL_VALVE`, `VIEW_BILLS` (the last one has no feature behind it: bills were not built, E-02).
- **Authority:** ms-places answers `GET /api/places/{id}/access`; ms-devices, ms-consumption and ms-valve call it with
  the user's JWT on each place-scoped request. ms-notifications asks `/internal/places/{id}/recipients?permission=VIEW_ALERTS`
  to notify the owner and the collaborators allowed to see alerts. ms-consumption resolves guest tokens with
  `/internal/guest-links/{token}`.

## Consequences

- Positive: access can be revoked at once; guests never touch authenticated endpoints.
- Negative: every place-scoped request in other services adds one internal HTTP call to ms-places.
- The guest token is a bearer secret in a URL: whoever has the link can read until it is regenerated.

## References

- `09-microservices/services/02-ms-places/README.md`
- `06-data/migrations-history.md` (ms-places-db v1.4)
