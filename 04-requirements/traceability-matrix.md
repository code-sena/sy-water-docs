# Traceability Matrix

> This matrix connects each business need to a requirement, story, test, and service owner.
> For Save Your Water, it answers: why is the feature needed, what validates it, and which service owns it.

---

## How to use this matrix

```text
Requirement → HU → Test Case → Service → Status

If a requirement has no HU: it is not planned.
If a HU has no test: it has no completeness criterion.
If a test has no implementation: there is test debt.
If code exists without an HU: it may be gold-plating or unplanned work.
```

---

## FR → HU → Test → Service matrix

| FR ID | FR Description | HU(s) | Tests that verify it | Service | Status |
|-------|---------------|-------|----------------------|---------|--------|
| FR-001 | User can register and log in securely | HU-001 | `auth-registration.spec`, `auth-login.spec` | auth-service | 🟡 In progress |
| FR-002 | User can record household consumption | HU-002 | `consumption-create.spec` | api-gateway, auth-service | 🟡 In progress |
| FR-003 | User can see dashboard and historical trends | HU-003 | `dashboard-consumption.spec` | analytics-service | 🔴 Pending |
| FR-004 | User receives anomaly alerts | HU-004 | `alert-generation.spec` | notifications-service | 🔴 Pending |
| FR-005 | User can report a leak or service issue | HU-005 | `service-issue-create.spec` | support-service | 🔴 Pending |
| FR-006 | Admin can manage users, zones, and alerts | HU-006 | `admin-management.spec` | admin-service | 🔴 Pending |

---

## NFR → Validation matrix

| NFR ID | Description | Validation method | Tool | Status |
|--------|-------------|------------------|------|--------|
| NFR-001 | p95 dashboard latency < 500 ms | Load test in staging | k6 | 🟡 Monitoring |
| NFR-002 | 99.9% production availability | SLO monitoring | Grafana + Prometheus | 🟡 Monitoring |
| NFR-004 | JWT and RBAC protection | Security contract tests | Postman + OWASP ZAP | 🔴 Pending |
| NFR-005 | Logs and traces for request investigation | Smoke test + tracing review | OpenTelemetry + Grafana | 🔴 Pending |

---

## Inverse traceability: HU → FR

| HU | Title | FR(s) it implements | Sprint |
|----|-------|---------------------|--------|
| HU-001 | User registration and secure login | FR-001 | Sprint 1 |
| HU-002 | Record household water consumption | FR-002 | Sprint 1 |
| HU-003 | View dashboard and trends | FR-003 | Sprint 2 |
| HU-004 | Receive alerts for abnormal consumption | FR-004 | Sprint 2 |
| HU-005 | Report a service issue or leak | FR-005 | Sprint 2 |
| HU-006 | Manage users, zones, and alerts in administration | FR-006 | Sprint 3 |

---

## Status legend

| Status | Meaning |
|--------|---------|
| ✅ Done | Implemented, tested, and validated |
| 🟡 In progress | Under development in the current sprint |
| 🔴 Pending | In backlog and not yet started |
| ⏸ Blocked | Requires external dependency or blocker |
| ❌ Cancelled | Removed from project scope |

---

## Identified gaps

| Gap type | Description | Required action | Owner | Date |
|----------|-------------|----------------|-------|------|
| FR without HU | Any new requirement not assigned to a story | Create or refine a user story | Product Owner | [date] |
| HU without test | HU-006 has no acceptance tests yet | Add scenario tests before sprint | QA / Team | [date] |
| No validation method | NFR-005 must be validated in staging | Add observability smoke test | Tech Lead | [date] |

---

## How to maintain this matrix

1. Every new HU is added here with its associated FR(s)
2. Every new test is linked to a requirement and service owner
3. When a story is closed, its status becomes ✅
4. At each sprint review, identify gaps and assign corrective actions

---

## Correlations

- User stories → `04-requirements/user-stories.md`
- Non-functional requirements → `04-requirements/non-functional.md`
- Testing strategy → `11-quality/testing-strategy.md`
- Definition of Done → `00-governance/definition-of-done.md`
