# Git Conventions

> **Read this document before making your first commit on the Save Your Water project.**
> The project follows a simple Git workflow to keep the codebase clean, reviewable, and aligned with Trello sprint tasks.

## Branch strategy

```
main        ← Production. Stable branch. Only release-ready code is merged here.
  └── dev   ← Integration branch. All feature work is merged here during the sprint.
        └── feat/[description]    ← One branch per user story or feature
        └── fix/[description]     ← One branch per bug fix
        └── chore/[description]   ← Documentation, CI, dependency, and maintenance updates
        └── hotfix/[description]  ← Urgent fix for production issues
```

**Rules:**
- Nobody commits directly to `main` or `dev` (enforced by the GitHub ruleset `dev-main-protegidas`)
- `main` only receives PRs from `dev` (release) or `hotfix/*` (enforced by the `source-branch` check)
- `ci/[description]` is accepted for pipeline-only changes; existing `feature/hu-XXX-...` branches are accepted as `feat/`
- Every task must have one branch and one pull request
- One branch = one task; do not combine unrelated features or fixes
- Branches are deleted after merge to keep the repository clean
- All branches must be linked to a Trello card or sprint task

---

## Branch naming format

```
[type]/[description-in-kebab-case]
```

Examples:
```
feat/water-consumption-dashboard
fix/token-expiration-validation
chore/upgrade-spring-boot
hotfix/alert-notification-timeout
```

---

## Commit format (Conventional Commits)

```
[type]([scope]): [lowercase description in imperative mood, no trailing period]

[optional body — explain WHY, not only WHAT]

[optional footer — issue, Trello card, or user story reference]
```

**Types:**
| Type | When to use |
|------|-------------|
| `feat` | New feature or user-facing functionality |
| `fix` | Bug fix |
| `docs` | Documentation only |
| `style` | Formatting and layout adjustments without logic change |
| `refactor` | Internal code refactoring without behavior change |
| `test` | New or updated tests |
| `chore` | Tooling, CI, dependencies, environment changes |
| `perf` | Performance improvement |

**Examples:**
```bash
feat(auth): add JWT validation for protected routes

fix(alerts): correct duplicate notification trigger
Refs: TRE-134

docs(api): update consumption endpoint contract

chore(deps): upgrade Spring Boot to 3.2.x
```

---

## Pull Request policy

- **Size:** maximum 400 changed lines of production code, excluding tests. If it exceeds that size, split it.
- **Reviewers:** minimum 1 approval before merge
- **Review time:** reviewer must respond within 24 business hours
- **Template:** use the repository PR template or the standard project PR checklist
- **Green CI:** merge is allowed only when all validation checks pass. GitHub enforces it: `test` (code repos) and `source-branch` (all repos) — see `10-devops/ci-cd.md`
- **Approvals in GitHub:** the ruleset requires 0 approvals because the author cannot approve their own PR; the teammate review above is still part of the process
- **No large PRs:** every PR must map to one user story or one clear fix

---

## Merge policy

- Use **Squash and Merge** for features and bug fixes in `dev`
- Use **Merge Commit** for release merges into `main`
- **Do not** use Rebase & Merge in shared branches
- Release branches and hotfixes must be reviewed before merging into `main`
- A hotfix merged into `main` must also be merged back into `dev`

---

## Tags and versioning

**Production version = image tag = 7-character SHA of the `main` commit** built by Jenkins (e.g. `ms-valve:be4e714`). Each repo is released on its own; the running version of each service is the `*_TAG` value in the server `.env` (see `10-devops/ci-cd.md`).

Git tags with Semantic Versioning (`MAJOR.MINOR.PATCH`) are optional, to mark a milestone:

```bash
# Release to production
git tag -a v1.2.0 -m "Release v1.2.0: water usage alerts"
git push origin v1.2.0
```

**Rules:**
- `MAJOR`: breaking changes or incompatible API modifications
- `MINOR`: user-visible new features or enhancements
- `PATCH`: bug fixes and non-breaking corrections

---

## Team workflow with Trello

- Every sprint story must exist in Trello with a clear title and acceptance criteria
- The branch name should map to the related Trello card when possible
- PRs must state the related user story or card reference
- Completed work is moved to the corresponding Trello status before closing the task

---

## Correlations

- Agile team rules → `00-governance/agile-conventions.md`
- Definition of Ready → `00-governance/definition-of-ready.md`
- Definition of Done → `00-governance/definition-of-done.md`
- CI/CD and release process → `10-devops/ci-cd.md`
