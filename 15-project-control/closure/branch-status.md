# Branch Status at Project Close

> Snapshot of every branch of every repository at project close (**2026-10-28**).
> Source: the remote branches of the GitHub organization `cuida-tu-agua`, read on 2026-10-28, and the local
> clones in `C:\Users\juani\Documents\cuida-tu-agua`. No merge, branch or commit was made to produce this document.
> The code is frozen: this register describes what exists, it does not plan new integrations.

---

## How to read this document

| Term | Meaning |
|------|---------|
| **Integrated** | The branch content is already in `dev` and `main` (merged or squash-merged). |
| **Back-merge missing** | `main` has a hotfix that never returned to `dev`. `main` is complete; `dev` is behind in content. |
| **Abandoned** | Work that never reached `dev` and will not be integrated (reason given). |
| **Local only** | A branch that exists in a developer clone but not in GitHub (deleted after its PR). |
| `+N −M vs dev` | Commits the branch has that `dev` lacks (`+N`) and commits `dev` has that the branch lacks (`−M`). |

**Release rule in force at close:** `feature/*` → PR → `dev` → PR → `main` → Jenkins release (`10-devops/ci-cd.md`).
Production is built **only** from `main`. Therefore the **final state of the project is the content of `main`** in each repository.

---

## 1. Summary

| Result | Count |
|--------|-------|
| Repositories reviewed in GitHub (`cuida-tu-agua`) | 17 (16 active + 1 legacy) |
| Repositories where `main` already holds everything of `dev` | 17 of 17 |
| Repositories where `dev` lacks a hotfix of `main` (back-merge missing) | 2 (`iam-backend`, `infra`) |
| Open feature branches in GitHub with work not integrated | 8 (all in `cuida-tu-agua-frontNEW`, abandoned) |
| Legacy repository with branches not integrated | 1 (`cuida-tu-agua-frontend`, superseded) |
| Repository not accessible from this review | 1 (`code-sena/sy-water-docs`, private — local clone only) |

**Conclusion:** there is **nothing pending to integrate into `main`**. Every feature that was finished is already in `main`.
The only differences are (a) two hotfixes that live in `main` and not in `dev`, and (b) eight abandoned frontend
branches whose content is listed in the out-of-scope register.

---

## 2. Per-repository status (GitHub)

### Backend

| Repository | `main` (last commit) | `dev` vs `main` | Other branches | Status |
|------------|---------------------|-----------------|----------------|--------|
| `api-gateway` | `865512c` 2026-10-24 — guest routes (#7) | Same content | — | ✅ Closed |
| `iam-backend` (ms-iam) | `585375e` 2026-10-24 — invitation e-mail button (#13) | **`dev` lacks the hotfix** `fix(iam): add a button to the collaborator invitation e-mail` (2 files: `SmtpNotificationSender.java`, `application.yml`) | — | ⚠️ Back-merge missing |
| `ms-places` | `57889f4` 2026-10-26 — invitation notices (#12) | Same content | — | ✅ Closed |
| `ms-devices` | `cff60ad` 2026-10-24 — collaborator access (#10) | Same content | — | ✅ Closed |
| `ms-consumption` | `ef3b048` 2026-10-24 — guest consumption (#9) | Same content | — | ✅ Closed |
| `ms-valve` | `051124d` 2026-10-24 — collaborator control (#7) | Same content | — | ✅ Closed |
| `ms-notifications` | `3b9b6c4` 2026-10-26 — WhatsApp/SMS templates (#13) | Same content | — | ✅ Closed |

### Database (Liquibase)

| Repository | `main` | Last release | `dev` vs `main` | Status |
|------------|--------|--------------|-----------------|--------|
| `ms-iam-db` | `8cf6634` 2026-10-23 | v1.5 (phone verification) | Same content | ✅ Closed |
| `ms-places-db` | `f07f481` 2026-10-24 | v1.4 (sharing) | Same content | ✅ Closed |
| `ms-device-db` | `7d5007b` 2026-10-23 | v1.2 (offline after 1 min) | `dev` has 1 extra merge commit (back-merge), **same tree** | ✅ Closed |
| `ms-consumption-db` | `611799b` 2026-10-23 | v1.1 (goals) | Same content | ✅ Closed |
| `ms-valve-db` | `513cbe6` 2026-10-22 | baseline (no releases) | Same content | ✅ Closed |
| `ms-notification-db` | `54492c2` 2026-10-26 | v1.7 (message templates) | Same content | ✅ Closed |

### Frontend, infrastructure and firmware

| Repository | `main` | `dev` vs `main` | Other branches | Status |
|------------|--------|-----------------|----------------|--------|
| `cuida-tu-agua-frontNEW` (app web + Android) | `dd4cff4` 2026-10-27 — button labels (#36) | Same content | 8 abandoned (§3) | ✅ Closed (with abandoned work) |
| `infra` | `b209fe9` 2026-10-26 (#12) | **`dev` lacks the hotfix** `IAM_MAIL_APP_URL: ${APP_PUBLIC_URL}` in `compose.yml` (ms-iam) | — | ⚠️ Back-merge missing |
| `cuida-tu-agua-device-ESP` (firmware) | — (**no `main`**) | Only `dev` (`5c79a4f`, 2026-10-19) | — | ⚠️ Outside the release flow (§5) |
| `cuida-tu-agua-frontend` (legacy app) | `main` = `dev` (2026-08-30) | Same | 3 not integrated (§4) | ⏸ Superseded |

---

## 3. Abandoned branches — `cuida-tu-agua-frontNEW`

Eight branches by Juan Manuel Gómez (2026-10-07/08). They are **stacked** (each one contains the previous ones, in this
order: `hu-025` → `hu-029` → `hu-033` → `hu-038` → `hu-017` → `hu-022` → `hu-057` → `hu-058`) and branch from commit
`912c21f` (2026-10-06), **before** the frontend was restructured as one project for web and mobile and before the
history rewrite of 2026-10-23. Against current `dev` they are 122 commits behind and differ in ~300 files.

| Branch | `+N vs dev` | Content (only in this branch) | Equivalent in `main` | Decision |
|--------|------------|-------------------------------|----------------------|----------|
| `feature/hu-025-notification-center` | +23 | Alerts module with an **in-memory** repository ("until the alerts API is defined") | Real notification center against ms-notifications (HU-025 ✅) | Superseded |
| `feature/hu-029-leak-alert` | +34 | Leak-alert details and a close-valve shortcut | None — no backend event exists | Abandoned (E-03) |
| `feature/hu-033-valve-closed-alert` | +27 | Origin text of a valve-closed alert | Valve-closed alerts from ms-notifications (HU-033 ✅) | Superseded |
| `feature/hu-038-export-csv` | +42 | CSV export of a period (client side, `expo-file-system`, `expo-sharing`) | None | Abandoned (E-05) |
| `feature/hu-017-chart-tooltip` | +46 | Exact value on tap/hover of a chart bar | Chart shows only the peak value | Abandoned (E-07) |
| `feature/hu-022-valve-history-filter` | +52 | Date-range filter of the valve history (`from`/`to`) | History of the last 30 days; ms-valve accepts `from`/`to` but the app does not send them | Abandoned (E-08) |
| `feature/hu-057-consumption-report` | +58 | Report screen: total, daily average, peak, data gaps | None | Abandoned (E-05) |
| `feature/hu-058-report-pdf` | +66 | PDF export of the report (`expo-print`) | None | Abandoned (E-05) |

**Why they were not integrated:** they were written against a frontend structure that no longer exists; merging them
means re-implementing them over the current code, which the code freeze forbids. The functional gaps they would have
covered are recorded in `out-of-scope-register.md`.

**Consolidation result if they were merged anyway:** not possible without conflicts in practically every touched file
(`MainNavigator`, `PlaceDashboardScreen`, `api.ts`, the i18n dictionaries, `package.json`), plus the in-memory alerts
module would duplicate the real notification center. No merge is recommended.

---

## 4. Legacy repository — `cuida-tu-agua-frontend`

First frontend (August–September 2026), replaced by `cuida-tu-agua-frontNEW`. Kept for history; nothing is deployed from it.

| Branch | vs `dev` | Last commit | Status |
|--------|----------|-------------|--------|
| `main` / `dev` | identical | 2026-08-30 | Frozen |
| `feature-NewScreen` | +3 | 2026-09-09 | Abandoned (superseded by frontNEW) |
| `feature-loginHomePage` | +1 −1 | 2026-08-27 | Abandoned (superseded by frontNEW) |
| `ci/enforce-source-branch` | +1 | 2026-10-22 | Not merged; CI check added when the rulesets were created, irrelevant for a frozen repo |

---

## 5. Firmware repository without `main`

`cuida-tu-agua-device-ESP` has a single branch, `dev`. It was never part of the PR/release flow (no `main`, no CI,
no README). The firmware that runs in the meter is the content of `dev` at `5c79a4f` (2026-10-19), flashed by hand.
This is recorded as a known gap of the release process (see ADR-012 and E-14).

---

## 6. Back-merges missing (`main` → `dev`)

| Repository | Commit only in `main` | Effect |
|------------|----------------------|--------|
| `iam-backend` | `585375e` fix(iam): invitation e-mail button (HU-043) | `dev` sends the invitation e-mail without the "Ver invitación" button |
| `infra` | `595857a` fix(infra): `IAM_MAIL_APP_URL` for ms-iam | `dev` compose does not pass the app URL to ms-iam |

Both hotfixes went `hotfix/*` → `main` (allowed by the `source-branch` check) and the step "PR `main` → `dev`" of
the hotfix procedure was not executed. Because the project is closed and production is built from `main`, **`main` is
the reference**; `dev` must not be used to rebuild production.

---

## 7. Local-only branches (developer clone)

Branches that exist in `C:\Users\juani\Documents\cuida-tu-agua` but were deleted in GitHub after their PR. All of them
are integrated (their content is in `main`); they are listed so nobody mistakes them for pending work.

| Local clone | Local-only branches |
|-------------|---------------------|
| `infra` | `chore/repo-names`, `feature/guest-and-collaborator-env`, `hotfix/invitation-email-link` |
| `api-gateway`, `ms-places`, `ms-valve` | `feature/ci` |
| `ms-consumption` | `feature/ci`, `feature/hu-023-024-goals` |
| `ms-iam` | `feature/ci`, `hotfix/invitation-email-link` |
| `ms-notifications` | `feature/guest-and-collaborator-access` |
| `ms-consumption-db` | `feature/hu-023-024-goals` |
| `ms-device-db` | `hotfix/device-offline-1min` |
| `save-your-water` (frontNEW) | `chore/android-push`, `chore/app-icons`, `feature/hu-028-twilio-whatsapp`, `traduccion` |
| `sy-water-docs` | `docs/api-gateway`, `docs/ci-cd-release-process` (see §8) |

---

## 8. Documentation repository — `code-sena/sy-water-docs`

**Not accessible from this review** (private repository of another organization; the review environment has no
credentials for it). Only the local clone was read, last fetched on 2026-10-22:

| Branch | Last commit | Status as seen locally |
|--------|-------------|------------------------|
| `main` | `3dfd6ea` 2026-10-16 | Base of this update |
| `docs/ci-cd-release-process` | `3065672` 2026-10-22 | CI/CD documentation; **whether it was merged into `main` in GitHub could not be verified** |
| `docs/api-gateway` | `f383836` 2026-10-16 | Merged (#5) |
| `origin/chore/sync-docs-with-code` | `83541bf` 2026-10-15 | Merged (#4) |

The documentation added at close was written on top of `docs/ci-cd-release-process` because it references
`10-devops/ci-cd.md`.

---

## Correlations

- Release flow → `10-devops/ci-cd.md`, ADR-012
- What will not be done and why → `out-of-scope-register.md`
- Documentation review per repository → `repository-documentation-review.md`
