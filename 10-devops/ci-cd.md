# CI/CD — Cuida Tu Agua

> How a change goes from a branch to production, what is checked automatically, and who approves.
> **Rule:** production only runs images built by Jenkins from `main`, and only after the team lead approves.

---

## Overview

```
feature/* ──PR──► dev ──PR──► main ──► Jenkins "release" ──► GHCR ──► [approval] ──► VPS
              │              │               │                              │
        GitHub Actions  GitHub Actions   build + push image         backup → deploy →
        test + source   test + source    (tag = commit SHA)         healthcheck → rollback if KO
```

| Piece | Where it runs | What it does |
|-------|---------------|--------------|
| GitHub Actions | GitHub | Runs the tests and the branch check on every PR into `dev` or `main` |
| Rulesets (`dev-main-protegidas`) | GitHub | Block direct pushes and merges without green checks |
| Jenkins | Team lead's PC (Docker, `http://localhost:8090`) | Builds the image, pushes it to GHCR, asks for approval, deploys |
| GHCR (`ghcr.io/cuida-tu-agua/*`) | GitHub Container Registry | Stores the production images |
| VPS | Hostinger, Docker Compose (`/opt/cuidatuagua/infra`) | Pulls the approved image and runs it |

There is no shared `dev` or `staging` server: `dev` is validated by automated tests and locally (see `environments.md`).

---

## 1. PR pipeline (GitHub Actions)

Two workflows run on every PR into `dev` or `main`:

| Workflow | File | Check name | Blocks the merge |
|----------|------|------------|------------------|
| CI | `.github/workflows/ci.yml` | `test` | Yes (code repos) |
| Enforce source branch | `.github/workflows/enforce-source-branch.yml` | `source-branch` | Yes (all repos) |

**`test` per stack:**

| Repos | Command |
|-------|---------|
| `iam-backend`, `api-gateway` | `bash mvnw -B -ntp verify` (Java 21, Temurin) |
| `ms-places`, `ms-devices`, `ms-consumption`, `ms-valve`, `ms-notifications` | `dotnet test <solution>.slnx -c Release` (SDK from `global.json`) |
| `cuida-tu-agua-frontNEW` | `npm ci` → `npx tsc --noEmit` → `npx jest --ci` → `npm run lint` |

- The tests do not need a database, Redis or RabbitMQ (SQLite in memory, mocks, test keys).
- Frontend lint **only warns** (`continue-on-error`) until the known `setState`-in-`useEffect` errors are fixed; then it becomes blocking.
- `source-branch`: a PR into `main` must come from `dev` or `hotfix/*`. PRs into `dev` always pass.

---

## 2. Branch protection (rulesets)

Ruleset **`dev-main-protegidas`** on `dev` and `main` in all 15 repos:

| Rule | Effect |
|------|--------|
| Pull request required | No direct pushes to `dev` / `main` |
| Required status checks | Code repos: `test` + `source-branch` · DB repos: `source-branch` · `infra`: none |
| Block deletion / force push | `dev` and `main` cannot be deleted or rewritten |
| Bypass | Nobody (see *Emergencies*) |

Required approvals are set to **0**: GitHub does not let the author approve their own PR. A teammate review is still expected by `00-governance/git-conventions.md`.

---

## 3. Release pipeline (Jenkins)

**Job:** `release` · **Definition:** `infra/jenkins/release.Jenkinsfile` (read from `main` of `infra`).

**Parameters:**

| Parameter | Values | Use |
|-----------|--------|-----|
| `SERVICE` | `ms-iam`, `ms-places`, `ms-device`, `ms-consumption`, `ms-valve`, `ms-notification`, `api-gateway`, `web` | Service to release (Compose service name) |
| `REF` | `main` (default) | Branch to build. Production = `main` |
| `SOLO_PUBLICAR` | true / false | Only build and push to GHCR, without approval or deploy |

**Stages:**

1. **Checkout** of the service repo at `REF`. The image tag is the **7-character commit SHA** (e.g. `ms-valve:be4e714`).
2. **Build** with the repo's `Dockerfile`. The `web` image receives the `EXPO_PUBLIC_*` URLs at build time (`https://api.sy-water.tech/<prefix>`).
3. **Push** to `ghcr.io/cuida-tu-agua/<service>:<tag>`.
4. **Approval** (`input`): only user `esteban` (team lead) can press *Desplegar*. Times out after 30 min.
5. **Deploy** over SSH to the VPS: `bash scripts/deploy-service.sh <service> <TAG_VAR> <tag>`, which:
   - makes a **mandatory backup** of `sy-water-db` (`scripts/backup.sh`, keeps the last 7 in `infra/backups/`);
   - writes the new tag in the server `.env` (`MS_VALVE_TAG=...`);
   - `docker compose pull` + `up -d --no-deps` of that service only;
   - waits up to 5 min for `healthy`; if it never gets there, **puts the previous tag back** and fails the build.

**Jenkins credentials (IDs only, values never in git):** `github-token` (clone + push to GHCR), `vps-ssh` (user `deploy`). The VPS has its own read-only GHCR login.

**The version running in production** is the `*_TAG` set in `/opt/cuidatuagua/infra/.env`; the history is the Jenkins build list (`#N service:tag`).

### How to release a service

1. Merge the PR `dev` → `main` of the service (both checks green).
2. Jenkins → `release` → **Build with Parameters** → choose `SERVICE`, leave `REF=main`, `SOLO_PUBLICAR` unchecked.
3. Check the tag in the build name, then press **Desplegar**.
4. The build ends in `SUCCESS` only if the service is `healthy`. Open https://sy-water.tech and check the changed feature.

---

## 4. Database migrations

Migrations are **not** part of the Jenkins release: they are applied by hand, **before** releasing the service that needs them.

1. Merge the change in the `-db` repo (`dev` → `main`).
2. On the VPS: backup, update the repo and run only that service's Liquibase:
   ```bash
   cd /opt/cuidatuagua/infra
   bash scripts/backup.sh
   git -C ../sy-water-db/<repo>-db pull --ff-only
   docker compose --profile migrate run --rm --no-deps lb-<service>   # lb-iam, lb-places, lb-device, ...
   ```
3. Release the service with Jenkins.

**Rules:** every migration must be **backward compatible** (the running version must keep working with the new schema): add columns/tables first, remove old ones in a later release. Never edit an applied changeset; add a new release in `06-releases/`.

---

## 5. Hotfix

1. `hotfix/<description>` from `main`.
2. PR → `main` (checks must pass; `source-branch` accepts `hotfix/*`).
3. Release with Jenkins as usual.
4. PR `main` → `dev` (or cherry-pick into a branch for `dev`) so the fix is not lost.

---

## 6. Rollback

- **Automatic:** `deploy-service.sh` restores the previous tag if the new container is not `healthy`.
- **Manual** (bad behavior that passes the healthcheck): on the VPS, deploy the previous tag (see Jenkins history or GHCR):
  ```bash
  cd /opt/cuidatuagua/infra
  bash scripts/deploy-service.sh <service> <TAG_VAR> <previous-tag>
  ```
- **Database:** restore the backup taken just before the deploy (`infra/backups/sy-water-db-YYYYMMDD-HHMM.bak`) only if the data was damaged; consult the team lead first.

---

## 7. Infrastructure changes (`infra` repo)

1. PR into `dev`, then `dev` → `main` (no tests; PR required).
2. On the VPS: `cd /opt/cuidatuagua/infra && git pull --ff-only`, then `docker compose up -d <service>` for the containers affected by the change.
3. New repos/servers: `bash scripts/clone-all.sh main` (production) or `bash scripts/clone-all.sh` (`dev`, developer PCs).

---

## 8. Emergencies

| Situation | What to do |
|-----------|------------|
| Jenkins / team lead's PC unavailable | Deploy an image **already published** in GHCR with `deploy-service.sh` from the VPS. The VPS cannot build images (production Compose has no `build`). |
| A rule blocks an urgent fix | Repo → **Settings → Rules → Rulesets → dev-main-protegidas → Enforcement: Disabled**, merge, and set it back to **Active** right away. Record it in the incident post-mortem. |
| GitHub Actions does not start | Check https://www.githubstatus.com. Push an empty commit or close/reopen the PR when it recovers. |

---

## Known limitations

- Jenkins runs on one PC: no releases while it is off.
- Single approver (team lead), no backup approver.
- No staging environment; `dev` is validated only by tests and locally.
- Frontend lint is not blocking yet.

---

## Correlations

- Environments and URLs → `10-devops/environments.md`
- Branches, commits and PR rules → `00-governance/git-conventions.md`
- Incident response → `13-operations/incident-management.md`
- Service list and repos → `09-microservices/service-catalog.md`
