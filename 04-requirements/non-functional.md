# Non-Functional Requirements (NFR)

> These requirements define how well Save Your Water must work.
> Every NFR has an explicit metric and an agreed validation method.

---

## NFR-001: Performance

| Attribute | Metric | Test condition |
|-----------|--------|---------------|
| Authentication endpoint latency | p95 < 400 ms | Under normal production-like load |
| Consumption dashboard latency | p95 < 500 ms | Under 100 concurrent users |
| Historical report generation | p95 < 2 s | For a 30-day report range |
| Critical alert processing | p99 < 1 s | During anomaly evaluation |
| Service startup time | < 30 s | Cold start in staging |

**Defined critical flows:**
- `POST /auth/login`
- `GET /dashboard/consumption`
- `POST /alerts/evaluate`
- `GET /reports/monthly`

**Validation method:**
- k6 load tests in staging
- benchmark runs every sprint before release

---

## NFR-002: Availability

| Environment | SLO | Maintenance window | Maximum downtime per month |
|------------|-----|-------------------|----------------------------|
| Production | 99.9% | Sundays 02:00-04:00 | 44 minutes |
| Staging | 99.5% | No restriction | 3.5 hours |

**Error budget:**
- Production monthly budget: 44 minutes
- If the team consumes more than half of the budget in the first half of the month, feature deployments are paused until stability is restored.

**Health checks:**
- `GET /health` — liveness check
- `GET /health/ready` — readiness check with DB and dependency validation

---

## NFR-003: Scalability

| Scenario | Expected behavior |
|---------|------------------|
| Normal growth | Support incremental household onboarding without degradation |
| Peak user load | Up to 5,000 active households without major latency increase |
| Alert bursts | Process concurrent anomaly checks without queue collapse |
| Resource reduction | Scale down without interrupting active sessions |

**Strategy:** horizontal auto-scaling, stateless API services, and persisted data in the database layer.

---

## NFR-004: Security

### Authentication and authorization
- All protected endpoints require a valid JWT in the `Authorization: Bearer <token>` header
- JWT tokens expire in **1 hour**
- Refresh tokens expire in **7 days**
- RBAC is enforced by role and service capability
- Roles are defined in `00-governance/security-policy.md`

### Data protection
- HTTPS is mandatory in all non-local environments
- Sensitive data is encrypted at rest when applicable
- Passwords use bcrypt or Argon2id with strong parameters
- Secrets are stored in environment variables or a secure secret manager, never in source code

### OWASP validation
- Dependency scanning is run in CI
- SAST review and security checks are required before release
- Critical endpoints must be reviewed for input validation and authorization flaws

---

## NFR-005: Observability

| Pillar | Requirement | Tool |
|--------|------------|------|
| Logs | Structured logs with correlation ID | Logback / Winston |
| Metrics | RED metrics per endpoint | Prometheus + Grafana |
| Traces | End-to-end request trace propagation | OpenTelemetry |
| Alerts | Alert within 5 minutes when critical SLOs are violated | Grafana / Alertmanager |

**Correlation ID:** every request must generate a UUID that travels through logs and traces for the same transaction.

---

## NFR-006: Maintainability

| Metric | Target |
|--------|--------|
| Unit test coverage | ≥ 80% line coverage |
| Domain logic coverage | ≥ 90% |
| Code review approval | At least 1 reviewer per PR |
| Technical debt tickets | Resolved in the same or next sprint when practical |
| Local setup time | New developer can run the project in under 1 hour |

---

## NFR-007: Portability and deployment

- Services must be deployable as containerized artifacts
- The application should run consistently across local, staging, and production environments using environment variables
- No service must depend on host OS-specific behavior
- Deployment must be reproducible using the project automation and environment configuration

---

## NFR priority matrix

| NFR | Priority | Validation | Owner |
|-----|----------|-----------|-------|
| Performance | P1 | k6 in staging | Tech Lead |
| Availability | P1 | Monitoring + health checks | DevOps |
| Security | P1 | SAST + JWT review + OWASP checks | Tech Lead |
| Scalability | P2 | Capacity validation in staging | DevOps |
| Observability | P1 | Smoke test + metrics review | Tech Lead |
| Maintainability | P2 | Coverage and quality gates | Team |

---

## Correlations

- Product vision and problem framing → `03-product/vision.md`
- User stories that implement these constraints → `04-requirements/user-stories.md`
- Security policy → `00-governance/security-policy.md`
- Monitoring and operations → `13-operations/README.md`
