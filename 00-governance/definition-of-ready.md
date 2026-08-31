# Definition of Ready (DoR)

> A User Story is **Ready** for Save Your Water when the team can start it in the next sprint without needing to resolve fundamental questions or blockers mid-delivery.
> If the story does not meet this DoR, it returns to refinement in Trello before it is accepted into the sprint.

---

## DoR checklist for Save Your Water

Before moving a User Story to "Ready for Sprint", verify:

### Clarity

- [ ] The story is written in the format: **As [role], I want [action], so that [benefit]**
- [ ] The role is specific and represents a real stakeholder or user of the product
- [ ] The business value is clear and tied to water conservation, citizen engagement, service visibility, or operational efficiency
- [ ] There are no vague statements such as "improve the experience" without a concrete outcome

### Acceptance Criteria

- [ ] There are at least 2 acceptance criteria written in **Given / When / Then** format
- [ ] The criteria cover the happy path and the main failure or validation case
- [ ] Criteria are testable and can be verified by QA, automation, or a clear manual check
- [ ] No ambiguous language such as "fast", "easy", or "good" is used without measurable conditions

### Dependencies

- [ ] All internal and external dependencies are identified: services, APIs, third-party tools, data sources, or access requirements
- [ ] Blocking dependencies are resolved, or a mitigation plan is defined before sprint start
- [ ] If a story depends on another story, that story is already Done or is clearly planned with no blocker for the current sprint

### Estimation

- [ ] The team has estimated the story using T-shirt sizing or another agreed method
- [ ] The story fits within the sprint capacity of the team
- [ ] If the story is too large, it has been split into smaller, deliverable units
- [ ] The story is prioritised and aligned with the current product goal of Save Your Water

### Technical readiness

- [ ] Required access, credentials, and environments are available for implementation and testing
- [ ] APIs or integrations involved are either already defined or clearly described before development starts
- [ ] Data model impacts are identified if the story modifies storage, validation rules, or reporting structures
- [ ] The impact on other services or modules is documented, especially for authentication, data flows, and alerts

### Non-functional requirements

- [ ] Security requirements are considered: authentication, authorization, validation, and sensitive-data handling
- [ ] Observability requirements are defined when applicable: logs, metrics, traces, and operational visibility
- [ ] Performance expectations are specified when they affect user experience or service response
- [ ] Error handling and recovery behavior are understood for critical flows

---

## Common reasons a story is NOT ready

| Problem | What to do |
|---------|-----------|
| Unclear requirements | Hold a short refinement session with the team and product owner |
| Missing acceptance criteria | Add Given / When / Then criteria before the sprint |
| Unknown dependencies | Review integration points and document the blockers |
| Too large for the sprint | Split it into smaller stories before accepting it |
| No environment access | Request credentials or setup before sprint planning ends |
| Unclear API or data contract | Define the contract before implementation begins |
| No clear verification path | Define how QA or automation will validate the outcome |

---

## DoR vs DoD

| | Definition of Ready (DoR) | Definition of Done (DoD) |
|-|--------------------------|--------------------------|
| **When** | Before starting the story | After finishing the story |
| **Who verifies** | Team during refinement and planning | Team during review and validation |
| **Purpose** | Ensure the team can start without blockers | Ensure the increment is shippable and acceptable |

---

## Correlations

- Full DoD → `00-governance/definition-of-done.md`
- User Story template → `04-requirements/_template-hu.md`
- User Stories backlog → `04-requirements/user-stories.md`
- Agile team rules → `00-governance/agile-conventions.md`
