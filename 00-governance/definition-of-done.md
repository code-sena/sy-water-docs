# Definition of Done (DoD)

> A User Story is **DONE** for Save Your Water when it meets all required quality, technical, and validation criteria for the sprint.
> If any item on this checklist is missing, the story is not done and must return to In Progress or be explicitly re-scoped.

## Mandatory checklist

### Code
- [ ] Code implements all acceptance criteria of the user story
- [ ] Code was reviewed and approved by at least 1 team member through the pull request process
- [ ] Code follows the project standards for Java + Spring Boot / .NET and is aligned with the team's conventions
- [ ] Linting and formatting checks pass in CI
- [ ] No new technical debt is introduced without being registered in the project backlog or technical debt tracking

### Tests
- [ ] Unit tests are written for new business logic or validation rules
- [ ] The tests cover the main success path and the main validation/error path
- [ ] All relevant tests pass locally and in CI
- [ ] Acceptance criteria are verified manually or through automated checks
- [ ] The story does not reduce the project baseline of quality and test coverage

### Integration and architecture
- [ ] Changes do not break other services or application modules
- [ ] If the story changes an API contract, the OpenAPI contract is updated in `07-api/contracts/`
- [ ] If the story changes data structures or persistence behavior, the relevant service model documentation is updated
- [ ] If the story modifies events, notifications, or integrations, the corresponding event or service documentation is updated
- [ ] Security and authorization rules are validated when the story affects authenticated flows or sensitive information

### Deployment
- [ ] Code is mergeable to the target branch without conflicts
- [ ] CI/CD pipeline is green for the branch
- [ ] The change is merged into `dev` with the `test` and `source-branch` checks green (there is no staging server)
- [ ] When released: the Jenkins `release` build ends in `SUCCESS` and the feature is checked in production (`10-devops/ci-cd.md`)
- [ ] Any environment variable, secret, or configuration change is documented and validated

### Documentation
- [ ] The relevant service or feature documentation is updated if public interfaces or behavior changed
- [ ] User-facing or operational impact is documented when needed for the team or stakeholders
- [ ] If the story introduces a significant technical decision, an ADR is created or updated
- [ ] The task is reflected in the current Trello board with the correct status and acceptance evidence

---

## Allowed exceptions

The following exceptions must be explicitly approved by the Tech Lead or Scrum Master:
- E2E validation is deferred due to environment limitations and the risk is documented
- Documentation is postponed for urgent delivery, but a technical debt item is created in the backlog
- A small non-blocking issue remains, but it is explicitly accepted with a defined follow-up plan

---

## What is NOT a Done criterion

- "The code is on my machine" — it must be in the repository and ready for review
- "It works on my local environment" — it must be validated in the agreed project environment
- "The PO approved it" — that helps with product validation, but it does not replace technical completeness
- "The feature is almost done" — missing checks mean it is not done yet

---

## Review sign-off for Save Your Water

A story is considered complete only when:
1. the code is reviewed and approved,
2. the tests pass,
3. the acceptance criteria are verified,
4. the deployment check is completed, and
5. the item reflects the correct state in Trello and related documentation.

## Correlations

- Definition of Ready → `00-governance/definition-of-ready.md`
- Agile team conventions → `00-governance/agile-conventions.md`
- User stories → `04-requirements/user-stories.md`
- Architecture decisions → `05-architecture/decisions/README.md`
