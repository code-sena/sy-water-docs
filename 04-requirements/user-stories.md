# User Stories — Backlog

> This backlog defines the initial product scope for Save Your Water.
> Each user story is written as a business capability that creates value for citizens, operators, or administrators.

---

## Backlog status

| Cut | Sprint | Total HUs | Refined | In progress | Completed |
|-----|--------|-----------|---------|-------------|-----------|
| Cut 1 | Sprint 1-2 | 6 | 6 | 0 | 0 |
| Cut 2 | Sprint 3-4 | 4 | 0 | 0 | 0 |

---

## Epics

| ID | Epic | Description |
|----|------|-------------|
| EP-001 | User access and onboarding | Allow users to register, access the platform securely, and manage their account data. |
| EP-002 | Water consumption monitoring | Let users record and review their household water consumption over time. |
| EP-003 | Alerts and service visibility | Detect abnormal patterns and notify users or operators about potential issues. |
| EP-004 | Service management and administration | Support issue reporting, operational review, and admin configuration. |

---

## User Stories

### HU-001 — User registration and secure login {#HU-001}

**Epic:** EP-001

> **As** a registered user or household member
> **I want** to register in the platform and log in securely
> **so that** I can access my water consumption data and protected services safely

**Acceptance Criteria:**

```gherkin
Scenario 1: successful registration
  Given a new user with valid personal data and a unique email
  When the user creates an account
  Then the system creates the account and sends a confirmation message
  And the user can log in with the created credentials

Scenario 2: invalid login
  Given a user enters an invalid email or password
  When the user submits the form
  Then the system rejects the request
  And it shows a clear error message indicating invalid credentials
```

| Field | Value |
|-------|-------|
| Story Points | 3 |
| Priority | Must Have |
| Target sprint | Sprint 1 |
| Assigned to | Team |
| Status | Ready |
| Dependencies | None |
| Affected service(s) | auth-service |

---

### HU-002 — Record household water consumption {#HU-002}

**Epic:** EP-002

> **As** a household owner or responsible user
> **I want** to record and save my water consumption readings
> **so that** I can monitor my household usage and detect abnormal patterns early

**Acceptance Criteria:**

```gherkin
Scenario 1: valid consumption entry
  Given a logged-in user with an assigned household
  When the user submits a valid reading for a date and consumption value
  Then the system stores the record
  And it appears in the household consumption history

Scenario 2: invalid reading
  Given a logged-in user enters a negative or empty consumption value
  When the user submits the form
  Then the system rejects the record
  And it shows the validation message for the invalid input
```

| Field | Value |
|-------|-------|
| Story Points | 5 |
| Priority | Must Have |
| Target sprint | Sprint 1 |
| Assigned to | Team |
| Status | Ready |
| Dependencies | HU-001 |
| Affected service(s) | api-gateway, auth-service |

---

### HU-003 — View consumption dashboard and trends {#HU-003}

**Epic:** EP-002

> **As** a registered user
> **I want** to see my water consumption dashboard and historical trend
> **so that** I can understand my use patterns and reduce unnecessary consumption

**Acceptance Criteria:**

```gherkin
Scenario 1: show dashboard data
  Given the user has existing readings on the platform
  When the user opens the dashboard
  Then the system displays the current month consumption and trend comparison
  And it shows a clear summary for the selected period

Scenario 2: no data yet
  Given a user has not yet recorded any readings
  When the user opens the dashboard
  Then the system shows a friendly empty state
  And it explains the next step to add readings
```

| Field | Value |
|-------|-------|
| Story Points | 5 |
| Priority | Must Have |
| Target sprint | Sprint 2 |
| Assigned to | Team |
| Status | Ready |
| Dependencies | HU-002 |
| Affected service(s) | analytics-service |

---

### HU-004 — Receive alerts for abnormal consumption {#HU-004}

**Epic:** EP-003

> **As** a household user or operator
> **I want** to receive alerts when water consumption is abnormal or a possible leak is detected
> **so that** I can act quickly and avoid waste or service impact

**Acceptance Criteria:**

```gherkin
Scenario 1: anomaly detected
  Given a household has a consumption value above the configured threshold
  When the system evaluates the reading
  Then it creates an alert
  And it sends the notification to the user or assigned operator

Scenario 2: threshold not exceeded
  Given the reading stays within the expected range
  When the system evaluates the reading
  Then no alert is created
  And no notification is sent
```

| Field | Value |
|-------|-------|
| Story Points | 5 |
| Priority | Must Have |
| Target sprint | Sprint 2 |
| Assigned to | Team |
| Status | Ready |
| Dependencies | HU-002, HU-003 |
| Affected service(s) | notifications-service |

---

### HU-005 — Report a service issue or possible leak {#HU-005}

**Epic:** EP-004

> **As** a citizen or service user
> **I want** to report a water leak or potential service problem
> **so that** the appropriate operator can take action and reduce service disruption

**Acceptance Criteria:**

```gherkin
Scenario 1: valid issue report
  Given a logged-in user with a valid household or service location
  When the user submits a report with a description and location
  Then the platform stores the issue
  And it assigns it to a support or operational queue

Scenario 2: incomplete report
  Given the user attempts to submit a report without required information
  When the form is submitted
  Then the system rejects the request
  And it requires the missing fields to be completed
```

| Field | Value |
|-------|-------|
| Story Points | 3 |
| Priority | Should Have |
| Target sprint | Sprint 2 |
| Assigned to | Team |
| Status | Ready |
| Dependencies | HU-001 |
| Affected service(s) | support-service |

---

### HU-006 — Manage users, zones, and alerts in administration {#HU-006}

**Epic:** EP-004

> **As** an operator or administrator
> **I want** to manage users, service areas, and alert rules
> **so that** I can keep the platform operational and aligned with local water service needs

**Acceptance Criteria:**

```gherkin
Scenario 1: admin creates an alert rule
  Given an administrator is logged in with management permissions
  When the administrator configures a threshold and service zone
  Then the system saves the rule
  And future readings are evaluated against it

Scenario 2: operator without permission
  Given a non-admin user tries to access administration functions
  When the user navigates to the admin panel
  Then the system denies access
  And it redirects the user to the authorized dashboard
```

| Field | Value |
|-------|-------|
| Story Points | 5 |
| Priority | Should Have |
| Target sprint | Sprint 3 |
| Assigned to | Team |
| Status | Backlog |
| Dependencies | HU-001, HU-004 |
| Affected service(s) | admin-service |

---

## Rules for writing HUs

### 1. The role must be precise
Use specific actors such as:
```
✓ As a registered household user
✓ As an operator
✓ As an administrator
✗ As a user
✗ As a person
```

### 2. The benefit must be business value
The “so that” clause must explain why the feature matters:
```
✓ so that I can reduce waste and respond to anomalies in time
✗ so that I can see the data (this only repeats the feature)
```

### 3. Acceptance criteria must be verifiable
```
✓ Then the system shows a confirmation message
✓ Then the alert is sent to the operator email
✗ Then the experience is good
```

### 4. One HU must be reasonably sized for one sprint
If a story covers too many flows, it should be split before sprint planning.

---

## Correlations

- Full template with DoD checklist → `04-requirements/_template-hu.md`
- Non-functional requirements → `04-requirements/non-functional.md`
- Traceability matrix → `04-requirements/traceability-matrix.md`
- API contracts derived from these HUs → `07-api/contracts/openapi/`
