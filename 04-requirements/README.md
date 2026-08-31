# 04 — Requirements

> This section formalizes what Save Your Water must do and how well it must do it.
> The project is a water consumption and service management platform for citizens, operators, and administrators.

## Why this section exists

Requirements are the agreed contract between the product, engineering, and stakeholders.
Without them:
- the team cannot validate whether the platform is complete,
- scope changes have no measurable baseline,
- tests cannot prove that the expected value was delivered.

---

## Types of requirements

### Functional (FR)
Describe what the system must do.
Examples for this project:
- a user must register and log in,
- a household must record daily consumption,
- the platform must notify when an abnormal water pattern is detected,
- an operator must review and close a reported service issue.

### Non-functional (NFR)
Describe how well the system must perform.
Examples for this project:
- dashboard requests must respond in under 500 ms for 95% of calls,
- the platform must be available 99.9% of the month,
- all protected endpoints must enforce authentication and authorization.

---

## Requirement structure for Save Your Water

### `functional.md` ⭐
List of the system's functional requirements.
Each requirement is tied to a user story and a service owner.

### `non-functional.md` ⭐
Quality, performance, security, and availability requirements.
Each NFR includes a metric and an explicit verification method.

### `user-stories.md`
User stories in the format “As [role], I want [goal], so that [benefit]” with acceptance criteria.

### `traceability-matrix.md` ⭐
Maps each requirement to the user story and test that validates it.

---

## Functional requirements by domain

| ID | Module | Description | Source (HU) | Priority |
|----|--------|-------------|-------------|----------|
| FR-001 | Auth & profile | The system must allow the user to register, log in, and recover access securely. | HU-001 | High |
| FR-002 | Consumption | The system must allow a household to record and view water consumption by date and period. | HU-002, HU-003 | High |
| FR-003 | Alerts | The system must detect consumption anomalies and notify the user or operator when thresholds are exceeded. | HU-004 | High |
| FR-004 | Reports | The system must display monthly and historical consumption summaries to support decision-making. | HU-003 | High |
| FR-005 | Service requests | The system must allow users to report possible water leaks, service failures, or maintenance issues. | HU-005 | Medium |
| FR-006 | Administration | The system must allow operators and admins to manage users, service areas, and alert rules. | HU-006 | Medium |

---

## Non-functional priorities for this project

| Area | Requirement |
|------|-------------|
| Performance | p95 response time under 500 ms for critical pages and APIs |
| Availability | 99.9% production availability |
| Security | JWT, role-based authorization, HTTPS, secure secret handling |
| Scalability | Support up to 5,000 active households without service degradation |
| Observability | Logs, metrics, traces, and alerts for critical services |

---

## Correlations with other sections

| This section feeds... | Why |
|-----------------------|-----|
| `05-architecture/` | NFRs and critical flows drive architectural decisions |
| `07-api/` | Functional requirements define most endpoints and contracts |
| `09-microservices/` | Requirements are grouped by service ownership |
| `11-quality/testing-strategy.md` | Every FR must have verification coverage |
| `15-project-control/risks.md` | Ambitious NFRs often create delivery risks |

---

## Common mistakes to avoid

❌ “The system must be fast” → not measurable.
✅ “95% of dashboard requests must be below 500 ms.”

❌ “The system must be secure” → not testable.
✅ “Protected endpoints require valid JWT with 1-hour expiration.”

❌ Writing requirements that describe the solution instead of the problem.
✅ The requirement says what value is needed, not how it must be implemented.

---

## Questions this section must answer

- What must the system do for each type of user?
- Which business value is delivered by each feature?
- Which constraints define performance, security, and availability?
- Which test covers each requirement?
- Which requirements are blocked or missing from the current backlog?
