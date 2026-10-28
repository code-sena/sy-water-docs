# 15 — Project Control

> **What is this?** Project administration: risks, dependencies, unanswered questions,
> and technical debt. The difference between a project that gets delivered and one that "keeps getting delayed".

---

## Project closure (2026-10-28)

The project is closed and its code frozen (ADR-021). The closure record is in [`closure/`](./closure/README.md):

| Document | Content |
|----------|---------|
| [closure/README.md](./closure/README.md) | Index, final state, risk status at close |
| [closure/branch-status.md](./closure/branch-status.md) | Every branch of every repository; what is integrated, abandoned or missing a back-merge |
| [closure/repository-documentation-review.md](./closure/repository-documentation-review.md) | Documentation vs real state, per repository |
| [closure/docs-changes-06-07-09.md](./closure/docs-changes-06-07-09.md) | What changed in sections 06, 07 and 09 at close |
| [closure/out-of-scope-register.md](./closure/out-of-scope-register.md) | What will not be done, and why |

---

## What is here and how to fill it in

### `risks.md` ⭐
Project risk register.
**Fill in:** from the start of the project. Update each sprint.

**Format:**
```markdown
| ID | Risk | Probability | Impact | Severity | Strategy | Owner | Status |
|----|------|------------|--------|---------|---------|-------|--------|
| R-001 | [Description] | High/Medium/Low | High/Medium/Low | High | Mitigate/Accept/Transfer/Avoid | [Name] | Open |
```

**Severity = Probability × Impact:**
- High × High = 🔴 Critical
- High × Low or Low × High = 🟡 Moderate
- Low × Low = 🟢 Low

**Strategies:**
- **Mitigate:** reduce probability or impact
- **Accept:** record and monitor, do not act
- **Transfer:** move the risk (insurance, contracts)
- **Avoid:** change the plan to eliminate the risk

### `dependencies.md`
External project dependencies.
**Fill in:** third-party services, external teams, pending decisions from stakeholders.

**Format:**
```markdown
| Dependency | Type | Required for | External owner | Expected date | Status |
|------------|------|-------------|----------------|--------------|--------|
| [Third-party API] | External | [Feature X] | [Company/Team] | [Date] | Waiting |
```

### `open-questions.md` ⭐
Unanswered questions that block or could block the project.
**Fill in:** every time the team encounters something it does not know and that requires a decision.
**Criticality:** when a question has an answer, turn it into an ADR (if architectural) or close it here.

**Format:**
```markdown
| # | Question | Context | Required for | Who answers | Deadline | Status |
|---|---------|---------|-------------|------------|---------|--------|
| Q-001 | Do we use JWT or sessions? | Multiple services need auth | Sprint 2 | Tech Lead | [Date] | 🔴 Unanswered |
```

### `technical-backlog.md`
Technical debt and technical improvements identified.
**Fill in:** when the team identifies something that "works but not as it should".
Clearly separate from product HUs.

**Format:**
```markdown
| ID | Description | Impact if not resolved | Estimated effort | Priority | Sprint |
|----|-------------|----------------------|-----------------|---------|--------|
| TD-001 | Refactor module X | Increased maintenance difficulty | 3 SP | Medium | Sprint 5 |
```

---

## Correlations with other sections

| Fed by... | Why |
|-----------|-----|
| `05-architecture/` — difficult decisions | Technical risks |
| `04-requirements/` — demanding NFRs | Performance risks |
| Incidents in `13-operations/` | Post-incident technical debt |
| `03-product/product-backlog.md` | Coordination of technical vs business priorities |

---

## Questions this section must answer

- What can go wrong and how prepared are we?
- What external things do we depend on that we do not control?
- What decisions are blocked waiting for information?
- What code needs to be improved before it becomes a bigger problem?
