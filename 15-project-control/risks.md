# Risk Register

> Unmitigated risks become problems. This register allows the team to
> anticipate, not just react.
> Review and update at each retrospective or when there is a significant project change.

---

## Probability × impact matrix

```
IMPACT
  │
  │ HIGH  │  [Mitigate]    │  [Avoid]       │
  │       │  Low           │  High          │
  │       │  Probability   │  Probability   │
  │───────────────────────────────────────
  │ MEDIUM│  [Accept]      │  [Mitigate]    │
  │       │  with monitoring│              │
  │───────────────────────────────────────
  │ LOW   │  [Accept]      │  [Accept]      │
  │       │                │               │
  └───────────────────────────────────────
              LOW              HIGH
                      PROBABILITY
```

**Response strategies:**
- **Avoid:** Change the plan so the risk cannot materialize
- **Mitigate:** Reduce the probability or impact
- **Transfer:** Pass the risk to another party (insurance, provider, contract)
- **Accept:** Acknowledge the risk and have a contingency plan

---

## Active risk register

> Identified while writing the SENA deliverables against the code of 2026-10-10.
> `ACC-NN` = corrective action in `assets/deliverables/Acciones_Mejora_CuidaTuAgua_v1.0.pdf`.

| ID | Risk | Level | Strategy |
|----|------|-------|----------|
| R-001 | No container images for the services | Critical | Mitigate |
| R-002 | Frontend and backend contracts out of sync | High | Mitigate |
| R-003 | Must stories still open | High | Mitigate |
| R-004 | Device administration API missing | High | Mitigate |
| R-005 | Revoked tokens accepted by ms-devices | Medium | Mitigate |
| R-006 | MQTT without TLS | High | Mitigate |
| R-007 | Configuration drift between services | High | Mitigate |
| R-008 | No backups and no centralized observability | High | Mitigate |
| R-009 | Gaps in automated testing | Medium | Mitigate |
| R-010 | Documentation drift | Medium | Accept |
| R-011 | Places survive a deleted account | Medium | Mitigate |

---

### R-001 — No container images for the services

| Field | Value |
|-------|-------|
| **ID** | R-001 |
| **Category** | Technical |
| **Description** | None of the six services has a `Dockerfile` and no compose file starts the APIs; every service runs with `dotnet run` / `mvnw spring-boot:run`. A deployment outside the developers' PCs is manual and error-prone. |
| **Probability** | High |
| **Impact** | High |
| **Risk level** | Critical |
| **Strategy** | Mitigate |
| **Mitigation plan** | Write one Dockerfile per service and a compose file with healthchecks and `depends_on`. |
| **Contingency plan** | Deploy on one Windows/Linux host with the documented `dotnet run` procedure (`Procedimiento_Implementacion`). |
| **Trigger** | A demo or deployment date is set before the images exist |
| **Owner** | Tech Lead |
| **Review date** | 2026-10-10 |
| **Status** | Active |

---

### R-002 — Frontend and backend contracts out of sync

| Field | Value |
|-------|-------|
| **ID** | R-002 |
| **Category** | Technical |
| **Description** | The web frontend expects `ms-recommendations` on :3007 and `/api/admin/*/summary`, while the backend serves tips from ms-places (`/api/tips`, `/api/admin/tips`) and metrics from ms-iam (`/api/admin/metrics`). |
| **Probability** | High |
| **Impact** | Medium |
| **Risk level** | High |
| **Strategy** | Mitigate |
| **Mitigation plan** | Align the frontend base URLs and routes with the backend; write the missing OpenAPI contracts in `07-api/contracts/openapi/` (API-first). |
| **Contingency plan** | Disable the affected admin screens until they are aligned. |
| **Trigger** | 404 or CORS errors in the admin panel |
| **Owner** | Frontend owner |
| **Review date** | 2026-10-10 |
| **Status** | Active |

---

### R-003 — Must stories still open

| Field | Value |
|-------|-------|
| **ID** | R-003 |
| **Category** | Business |
| **Description** | 20 of 24 Must stories are implemented (37 of 69 in total). Open Must stories: HU-004 (Google sign-in), HU-029 (suspected leak alert), HU-041 (read-only guest) and HU-045 (collaborator actions by permission). |
| **Probability** | Medium |
| **Impact** | High |
| **Risk level** | High |
| **Strategy** | Mitigate |
| **Mitigation plan** | Prioritize the four Must stories in the next sprint planning; HU-029 unlocks automatic close (HU-035–037) (ACC-08). |
| **Contingency plan** | Re-negotiate their priority with the Product Owner and document it in the backlog. |
| **Trigger** | Sprint review without progress on these stories |
| **Owner** | Product Owner |
| **Review date** | 2026-10-10 |
| **Status** | Active |

---

### R-004 — Device administration API missing

| Field | Value |
|-------|-------|
| **ID** | R-004 |
| **Category** | Technical |
| **Description** | The factory registration and fleet API (`/api/admin/devices/*`, table `devices.device_admin_events`) described in the guides is not in `sy-water-backend`; HU-061 cannot be considered implemented. Devices are created only by Liquibase seeds (`dev`). |
| **Probability** | High |
| **Impact** | Medium |
| **Risk level** | High |
| **Strategy** | Mitigate |
| **Mitigation plan** | Recover or rewrite the controller and its Liquibase release; add it to the ms-devices docs and tests (ACC-07). |
| **Contingency plan** | Register devices with a reviewed SQL script per batch. |
| **Trigger** | A new physical meter must be registered |
| **Owner** | ms-devices owner |
| **Review date** | 2026-10-10 |
| **Status** | Active |

---

### R-005 — Revoked tokens accepted by ms-devices

| Field | Value |
|-------|-------|
| **ID** | R-005 |
| **Category** | Security |
| **Description** | ms-devices validates the JWT signature but does not check the Redis denylist, so a logged-out token keeps working there until it expires (≤ 1 h). |
| **Probability** | Medium |
| **Impact** | Medium |
| **Risk level** | Medium |
| **Strategy** | Mitigate |
| **Mitigation plan** | Reuse `RedisTokenRevocationChecker` from the other .NET services (ACC-09). |
| **Contingency plan** | Shorten the access-token TTL. |
| **Trigger** | Security review or penetration test |
| **Owner** | ms-devices owner |
| **Review date** | 2026-10-10 |
| **Status** | Active |

---

### R-006 — MQTT without TLS

| Field | Value |
|-------|-------|
| **ID** | R-006 |
| **Category** | Security |
| **Description** | Mosquitto listens on 1883 in plain text; device tokens travel unencrypted on the LAN. `mosquitto.conf` itself states that production must use TLS on 8883. |
| **Probability** | Medium |
| **Impact** | High |
| **Risk level** | High |
| **Strategy** | Mitigate |
| **Mitigation plan** | Enable TLS on 8883 with a certificate and update the firmware (`MQTT_PORT`, CA). |
| **Contingency plan** | Keep the broker reachable only inside the local network. |
| **Trigger** | Any deployment outside a private LAN |
| **Owner** | Hardware owner |
| **Review date** | 2026-10-10 |
| **Status** | Active |

---

### R-007 — Configuration drift between services

| Field | Value |
|-------|-------|
| **ID** | R-007 |
| **Category** | Technical |
| **Description** | `ms-notification` has no `keys/iam-public.pem`; the ms-device README says ms-iam runs on 8081 (it is 3001); `ms-iam/.env` defines variables that `application.yml` does not use; the shared `Internal:ApiKey` must be identical in five services. |
| **Probability** | High |
| **Impact** | Medium |
| **Risk level** | High |
| **Strategy** | Mitigate |
| **Mitigation plan** | One `.env.example` per service with every variable; a setup checklist in `10-devops/local-setup.md`. |
| **Contingency plan** | Follow the runbook of each service (`09-microservices/services/*/runbook.md`). |
| **Trigger** | 401/503 errors right after a fresh setup |
| **Owner** | Tech Lead |
| **Review date** | 2026-10-10 |
| **Status** | Active |

---

### R-008 — No backups and no centralized observability

| Field | Value |
|-------|-------|
| **ID** | R-008 |
| **Category** | Operational |
| **Description** | There are no backup scripts for SQL Server, the broker volumes or the avatars folder, and logs only go to the console (no metrics or aggregation). |
| **Probability** | Medium |
| **Impact** | High |
| **Risk level** | High |
| **Strategy** | Mitigate |
| **Mitigation plan** | Scheduled `BACKUP DATABASE` to a mounted volume; structured logs and a health dashboard. |
| **Contingency plan** | Restore from the Liquibase releases plus seeds (data loss accepted). |
| **Trigger** | First deployment with real users |
| **Owner** | DevOps |
| **Review date** | 2026-10-10 |
| **Status** | Active |

---

### R-009 — Gaps in automated testing

| Field | Value |
|-------|-------|
| **ID** | R-009 |
| **Category** | Quality |
| **Description** | 460 backend, 138 app and 28 firmware checks pass, but 42 ms-notification Infrastructure/Api tests were not run offline (MailKit missing from the cache), the Api layers have low coverage (19–24 %) and there are no automated end-to-end tests; 20 manual system cases (CP-01 to CP-20) are pending. |
| **Probability** | Medium |
| **Impact** | Medium |
| **Risk level** | Medium |
| **Strategy** | Mitigate |
| **Mitigation plan** | Run the full suite in CI with network access; add contract tests between services; execute the manual cases (`Pruebas_CuidaTuAgua`). |
| **Contingency plan** | Manual regression of the main flows before each delivery. |
| **Trigger** | A defect found in a flow covered only by manual tests |
| **Owner** | QA / Tech Lead |
| **Review date** | 2026-10-10 |
| **Status** | Active |

---

### R-010 — Documentation drift

| Field | Value |
|-------|-------|
| **ID** | R-010 |
| **Category** | Technical |
| **Description** | ADR-007 and ADR-008 are referenced but do not exist; `ms-iam-documentation.md` describes classes that were removed; OpenAPI contracts exist only for the old auth-service template. |
| **Probability** | High |
| **Impact** | Low |
| **Risk level** | Medium |
| **Strategy** | Accept |
| **Mitigation plan** | Update docs in the same PR as the code (documentation rules); review `09-microservices/` each release (ACC-05). |
| **Contingency plan** | Treat the code and the service READMEs as the source of truth. |
| **Trigger** | A new developer follows an outdated document |
| **Owner** | Tech Lead |
| **Review date** | 2026-10-10 |
| **Status** | Active |

---

### R-011 — Places survive a deleted account

| Field | Value |
|-------|-------|
| **ID** | R-011 |
| **Category** | Data |
| **Description** | HU-008 soft-deletes the account and releases its meters, but ms-places keeps the user's places active, so orphan places stay in the database. |
| **Probability** | High |
| **Impact** | Low |
| **Risk level** | Medium |
| **Strategy** | Mitigate |
| **Mitigation plan** | Add `DELETE /internal/users/{id}/places` in ms-places and call it from ms-iam like the device cleanup (ACC-06). |
| **Contingency plan** | Periodic SQL clean-up of places whose owner has `deleted_at`. |
| **Trigger** | Admin metrics count places of deleted users |
| **Owner** | ms-places owner |
| **Review date** | 2026-10-10 |
| **Status** | Active |

---

## Common technical risks in microservices

These risks apply to almost all microservices projects. Evaluate which ones apply:

| Risk | Typical probability | Standard mitigation |
|------|--------------------|--------------------|
| Failure cascade (one service down brings everything down) | Medium | Circuit Breaker, timeout, fallback |
| Data inconsistency between services | High | Saga, Outbox, idempotent events |
| Network latency in synchronous calls | High | gRPC, cache, async where possible |
| Lost messages in the broker | Medium | At-least-once + idempotent consumers |
| Schema evolution breaks consumers | High | Version events, compatible changes first |
| Technical debt accumulation | High | DoD with minimum coverage, regular reviews |
| Premature over-engineering | Medium | Start simple, YAGNI, measure before optimizing |
| Sensitive data exposure in logs | Medium | Logging policy, PII masking, SAST |
| Configuration drift between environments | High | Infrastructure as Code, environment variables |

---

## Closed risks / Lessons learned

| ID | Risk | Outcome | Lesson |
|----|------|---------|--------|
| — | None closed yet | — | — |

---

## Template for adding a new risk

```markdown
### R-00X — [Name]

| Field | Value |
|-------|-------|
| **ID** | R-00X |
| **Category** | |
| **Description** | |
| **Probability** | |
| **Impact** | |
| **Risk level** | |
| **Strategy** | |
| **Mitigation plan** | |
| **Contingency plan** | |
| **Trigger** | |
| **Owner** | |
| **Review date** | |
| **Status** | Active |
```

---

## Correlations

- External dependencies → `15-project-control/dependencies.md`
- Related open questions → `15-project-control/open-questions.md`
- Architectural decisions that mitigated risks → `05-architecture/decisions/`
