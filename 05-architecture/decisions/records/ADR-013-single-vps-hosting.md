# ADR-013 — Hosting on One VPS with Docker Compose and Caddy

| Field | Value |
|-------|-------|
| **ID** | ADR-013 |
| **Date** | 2026-10-01 (decision, approximate) · 2026-10-28 (recorded) |
| **Status** | Accepted |
| **Authors** | Juan Esteban Ome, Juan Manuel Gómez, Diego Alejandro Alvarado |
| **Reviewers** | Juan Esteban Ome, Juan Manuel Gómez, Diego Alejandro Alvarado |

> Recorded retroactively at project close. The decision was taken and implemented during the project; this record
> describes it as built in the `main` branches on 2026-10-28. Where the alternatives were not written down at the
> time, this is stated instead of reconstructing them.

---

## Context

The project is academic, funded by the team, with a budget of about 30,000 COP per month. It must be reachable on
the Internet with HTTPS for the web app, the API and the meters.

## Evaluated alternatives

| Alternative | Pros | Cons | Reason for discarding |
|------------|------|------|-----------------------|
| **A — One Hostinger VPS, Docker Compose, Caddy (chosen)** | Fits the budget; full control; automatic HTTPS with Caddy | Single machine: no high availability | — (chosen) |
| B — Managed cloud (Azure/AWS: App Service, managed SQL, managed broker) | HA, managed backups | Monthly cost far above budget | Budget |
| C — Kubernetes (k3s) on the VPS | Rolling updates | Operational load for three people | Complexity |
| D — PaaS free tiers (Render, Railway) | No server to manage | Sleep on idle; no MQTT/SQL Server | Requirements (MQTT, SQL Server) |

## Decision

**We decided:** one Hostinger VPS runs the whole stack from the `infra` repository (`compose.yml`): SQL Server 2022,
RabbitMQ, Redis, Mosquitto, the gateway, the six services, the documentation UI and Caddy. Domain `sy-water.tech`:
`sy-water.tech` (web), `api.sy-water.tech` (gateway and `/docs`). Secrets are in a `.env` on the VPS, never in Git.

## Consequences

- Positive: one place to operate; total cost within budget.
- Negative: no redundancy; a VPS outage stops everything. Backups are taken before each release (ADR-012) but there
  is no scheduled off-site backup and no centralized logs/metrics (risk R-008, partially resolved).
- Not taken: monitoring/alerting stack (Prometheus/Grafana/Loki) — E-16.

## References

- `10-devops/environments.md`
