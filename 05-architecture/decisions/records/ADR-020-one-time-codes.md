# ADR-020 — One-Time Six-Digit Codes Instead of Links

| Field | Value |
|-------|-------|
| **ID** | ADR-020 |
| **Date** | 2026-09-25 (decision, approximate) · 2026-10-28 (recorded) |
| **Status** | Accepted |
| **Authors** | Juan Esteban Ome, Juan Manuel Gómez, Diego Alejandro Alvarado |
| **Reviewers** | Juan Esteban Ome, Juan Manuel Gómez, Diego Alejandro Alvarado |

> Recorded retroactively at project close. The decision was taken and implemented during the project; this record
> describes it as built in the `main` branches on 2026-10-28. Where the alternatives were not written down at the
> time, this is stated instead of reconstructing them.

---

## Context

HU-002 (e-mail verification) and HU-005 (password recovery) were written as "send a link". The app runs on web and
Android; a link opened from a mail client on the phone does not reliably return to the installed app, and long-lived
link tokens in e-mails are bearer secrets.

## Evaluated alternatives

| Alternative | Pros | Cons | Reason for discarding |
|------------|------|------|-----------------------|
| **A — 6-digit code with expiry and attempt limit (chosen)** | Same flow on web and Android; short-lived; brute force bounded | User types the code | — (chosen) |
| B — Magic link with token | One click | Deep links to the app; token in mailbox | Mobile flow |
| C — Both | Flexible | Two flows | Effort |

## Decision

**We decided (iam DEC-004):** e-mail verification, password reset, sensitive actions (`action_codes`, e.g.
`VALVE_CLOSE`) and phone verification (`PHONE_VERIFY`, by SMS or WhatsApp, HU-028) use a 6-digit code stored as a
hash, with expiry and a limit of failed attempts. Collaborator invitations are the exception: the e-mail carries a
button to the app's `/invitations` screen, where the logged-in user accepts or rejects (ADR-014).

## Consequences

- HU-002 and HU-005 acceptance criteria say "enlace"; the implemented behaviour is a code. This deviation is accepted.

## References

- `09-microservices/services/01-ms-iam/decisions.md` DEC-004
