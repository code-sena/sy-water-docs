# Agile Team Conventions

> Defines how the team works through its development cycles. This version is aligned with the current delivery model for Save Your Water and is approved by the team for the first sprint.

---

## Sprint structure

| Field | Value |
|-------|-------|
| Duration | 1 week |
| Sprint start | Monday |
| Sprint end | Friday of the same week |
| Current sprint | Sprint 1 — 31/08/2026 to 05/09/2026 |
| Estimated capacity | 12 story points per sprint |

---

## Team and delivery model

- **Project:** Save Your Water
- **Team size:** 3 people
- **Roles:** Scrum Master / Technical Lead, Backend Developer, Full-stack Developer
- **Primary stack:** Java + Spring Boot
- **Secondary stack:** C# + .NET for support integrations or parallel services

---

## Ceremonies

### Sprint Planning
- **When:** Every Monday — 2:00 p.m.
- **Duration:** Maximum 1 hour
- **Who:** Entire team
- **Goal:** Select and commit to sprint user stories, break them down into technical tasks and define acceptance criteria
- **Output artifact:** Sprint Backlog updated in Trello

### Daily Stand-up
- **When:** Every day — 1:30 p.m.
- **Duration:** Maximum 15 minutes
- **Format:**
  1. What did I do yesterday?
  2. What will I do today?
  3. Is anything blocking me?
- **Rule:** Technical discussions happen after the daily, not during it

### Sprint Review
- **When:** Every Friday — 2:00 p.m.
- **Duration:** Maximum 30 minutes
- **Who:** Team + stakeholders if required
- **Goal:** Show what was built and collect feedback

### Sprint Retrospective
- **When:** Every Friday — after the review, at 3:30 p.m.
- **Duration:** Maximum 45 minutes
- **Format:** What went well / What to improve / Action commitments
- **Rule:** Each retro produces at least 1 improvement action with an owner and due date

### Backlog Refinement
- **When:** Wednesday of each sprint — 4:00 p.m.
- **Duration:** Maximum 1 hour
- **Goal:** Detail and estimate user stories for the next sprint
- **Exit criterion:** The user story meets the Definition of Ready

---

## Estimation

### Scale
| Points | Meaning |
|--------|---------|
| 1 | Trivial — done in hours |
| 2 | Small — done in one day |
| 3 | Medium — takes 2–3 days |
| 5 | Large — takes almost a full sprint |
| 8 | Very large — should be split |
| 13 | Epic — MUST be split before the sprint |

**Technique:** T-shirt sizing
**Tool:** Trello

### Estimation rule
- If there is disagreement of 2+ levels (for example, someone says 3 and another says 8), discuss before voting again.
- If a story is estimated at 8 or 13, it must be split into smaller sub-tasks.

---

## Backlog tool

**Tool:** Trello
**Board URL:** https://trello.com/invite/b/68fc9ee61282100abf319ffa/ATTIc706e5b0f2a9f4d5ddeee7e0fbd7dc29E6C21B87/sy-water

### Board columns
| Column | Meaning |
|--------|---------|
| Backlog | Pending refinement |
| Ready | Ready to enter the sprint (meets DoR) |
| In Progress | Someone is actively working on it |
| In Review | In Pull Request / code review |
| Done | Meets DoD and is closed |

---

## Team velocity

| Sprint | Story points completed | Notes |
|--------|----------------------|-------|
| Sprint 1 | 12 | Initial sprint baseline |
| Sprint 2 | — | To be recorded |
| Sprint 3 | — | To be recorded |
| Average | 12 | First measured sprint velocity |

---

## Related documents

- Definition of Ready → `00-governance/definition-of-ready.md`
- Definition of Done → `00-governance/definition-of-done.md`
- Risk management → `15-project-control/risks.md`
- Technical debt backlog → `15-project-control/tech-backlog.md`
