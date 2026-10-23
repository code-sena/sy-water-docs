# Deployment Environments

> Defines the system's environments, their purposes, access, and key configurations.
> **Rule:** production only changes through the release pipeline (`10-devops/ci-cd.md`).
> Direct changes on the server are reserved for emergencies.

---

## Project environments

| Environment | Purpose | Update | Access | Data |
|-------------|---------|--------|--------|------|
| **local** | Development and validation of `dev` on each developer's machine | Manual (`docker compose up`) | Each developer | Synthetic / seed |
| **production** | Real users and the project demo | Jenkins `release` job, with team lead approval | Team lead (SSH user `deploy`) | Real |

There is **no shared dev or staging server**: `dev` is validated by the PR tests (GitHub Actions) and locally.

---

## Local

Started from the `infra` repo (`compose.yml` + `compose.override.yml`, loaded automatically). See `infra/README.md`.

| Service | URL / port |
|---------|-----------|
| Web app | http://localhost:8082 |
| API Gateway | http://localhost:8080 |
| Swagger UI | http://localhost:8083/docs |
| Services (direct) | 3001 ms-iam · 3002 ms-places · 3003 ms-devices · 3004 ms-consumption · 3005 ms-valve · 3006 ms-notifications |
| Mailpit (test inbox) | http://localhost:8025 |
| RabbitMQ panel | http://localhost:15672 |
| SQL Server | localhost:1433 |
| Mosquitto (MQTT) | 1883 on every interface (the ESP32 on the same Wi-Fi connects to the PC IP) |

Repos are cloned next to `infra` with `bash infra/scripts/clone-all.sh` (branch `dev` by default).

---

## Production

| Item | Value |
|------|-------|
| Server | Hostinger VPS, Docker Compose, folder `/opt/cuidatuagua/infra` |
| Compose files | `COMPOSE_FILE=compose.yml:compose.prod.yml` in the server `.env` |
| Web app | https://sy-water.tech |
| API | https://api.sy-water.tech (`/iam`, `/places`, `/devices`, `/consumption`, `/valve`, `/notifications`) |
| API docs | https://api.sy-water.tech/docs |
| Public ports | 80 / 443 (Caddy, automatic HTTPS) and 1883 (Mosquitto, for the meters) |
| Internal only (SSH tunnel) | SQL Server 1433 · RabbitMQ panel 15672 · Portainer 9443 |
| Images | `ghcr.io/cuida-tu-agua/<service>:<commit-sha>` — the server never builds |
| Backups | `infra/backups/` (last 7), automatic before each release |

---

## Configuration and secrets

- Every value lives in the `.env` of each environment. **`.env` is never committed**; `infra/.env.example` lists the variable names.
- Secrets (database passwords, `INTERNAL_API_KEY`, SMTP, Google OAuth, RabbitMQ, Mosquitto) exist only in the server `.env` and in the Jenkins credentials store.
- JWT keys: `KEYS_DIR` on the server (`public.pem` mounted in every service, `private.pem` only in ms-iam).
- Production version of each service: `MS_IAM_TAG`, `MS_PLACES_TAG`, `MS_DEVICE_TAG`, `MS_CONSUMPTION_TAG`, `MS_VALVE_TAG`, `MS_NOTIFICATION_TAG`, `API_GATEWAY_TAG`, `WEB_TAG` in the server `.env`. Jenkins updates them; Compose refuses to start if one is missing.

---

## Production deploy strategy

**Adopted strategy: Recreate per service** with healthcheck and automatic rollback.
Each release replaces one container (`docker compose up -d --no-deps <service>`); the other services keep running. If the new container is not `healthy` within 5 minutes, the previous tag is restored.

### Production deploy checklist

- [ ] The PR `dev` → `main` was merged with `test` and `source-branch` green
- [ ] If there are database changes: the migration was applied **before** the release and is backward compatible
- [ ] New environment variables were added to the server `.env` before the release
- [ ] The team lead approved the deploy in Jenkins
- [ ] The build finished in `SUCCESS` (service `healthy`)
- [ ] The changed feature was checked in https://sy-water.tech

### Rollback

```bash
# On the VPS: deploy the previous tag of the service (see the Jenkins build history)
cd /opt/cuidatuagua/infra
bash scripts/deploy-service.sh <service> <TAG_VAR> <previous-tag>
# e.g. bash scripts/deploy-service.sh ms-valve MS_VALVE_TAG be4e714
```

Restoring the database backup is an emergency action: consult the team lead first.

---

## Deploy log

The Jenkins `release` job history is the deploy log: each build is named `#N <service>:<tag>` and records who approved it and the result.

---

## Correlations

- Release pipeline → `10-devops/ci-cd.md`
- Local setup → `10-devops/local-setup.md`
- Post-deploy observability → `13-operations/observability.md`
- Branch policy → `00-governance/git-conventions.md`
