# Documentation Rules

> These rules define how documentation is written, maintained, and reviewed in Save Your Water.
> Documentation is part of the delivery quality and must be updated together with the code it describes.

---

## Core principle

> **Documentation is code. If it is not updated, it is broken.**

Every user story or technical change that modifies behaviors, interfaces, configurations, or operational flows must update the relevant documentation in the same sprint and same PR.

---

## Language

| Artifact | Language |
|----------|----------|
| Source code (variables, functions, classes) | English |
| Comments in code | English |
| Commit messages | English |
| Branch names | English |
| Markdown documentation | English |
| OpenAPI descriptions | English |
| User-facing or API error messages | English |
| Internal logs | English |

> **Rule:** Once a language standard is chosen for a category, it is binding for the project. Mixed-language documents are not accepted in the same area.

---

## File structure

```text
Each section includes a README.md that explains the purpose of that folder.
Content files use kebab-case.md.
Templates use a leading underscore, such as _template-hu.md or _template-adr.md.
ADRs are numbered sequentially: ADR-001-short-title.md.
```

---

## What to document and what not to document

### Document

| What | Where |
|------|-------|
| Non-obvious architectural decisions | `05-architecture/decisions/records/ADR-NNN.md` |
| Business rules and domain invariants | `02-domain/entities-and-rules.md` |
| API contracts for each service | `07-api/contracts/openapi/[service].yaml` |
| Data model changes | `06-data/models.md` |
| Operational procedures | `13-operations/` |
| Identified risks | `15-project-control/risks.md` |
| Sprint flow decisions | `00-governance/agile-conventions.md` |

### Do not document

- What the code already says clearly without adding context
- Temporary experiments that will be removed
- Long explanations of library internals, when the library has its own documentation
- Change history; git log already covers that

---

## Owners per section

| Section | Owner | Review frequency |
|---------|-------|-----------------|
| `00-governance/` | Tech Lead | Start of each sprint |
| `02-domain/` | Tech Lead + Product Owner | When the domain changes |
| `04-requirements/` | Product Owner | Every sprint |
| `05-architecture/` | Tech Lead | Each technical decision |
| `07-api/contracts/` | Service owner | Each API change |
| `09-microservices/` | Service owner | Each release |
| `13-operations/` | DevOps / on-call | After each incident |
| `15-project-control/` | Tech Lead | Weekly review |

---

## Document format

### Headings
- `# H1` — one title per file
- `## H2` — main sections
- `### H3` — subsections
- Do not use H4 unless the hierarchy is unavoidable

### Tables
Use tables for comparison, registers, and matrices. Do not use tables for simple bullet lists.

### Code examples
Always specify the language in fenced code blocks:

```java
public class Example {
    public static void main(String[] args) {
        System.out.println("Save Your Water");
    }
}
```

### Template instructions
Blocks marked with `> [!NOTE] INSTRUCTIONS` are templates and must be removed once the document is completed.

---

## Update process

1. The developer identifies which documents the change affects
2. Documentation is updated together with the implementation in the same PR
3. The reviewer verifies that the docs remain consistent with the code and sprint status
4. Any API-impacting task must update the relevant OpenAPI contract before merge
5. If the task changes operational behavior, the runbook or operational docs must also be reviewed

---

## Review and acceptance criteria for documentation

A PR is not considered complete if:
- the docs are not updated when behavior changes,
- the document is stale relative to the implementation,
- the team cannot explain which document covers the affected area,
- the documentation contains placeholders or unresolved instructions in final state.

---

## Correlations

- Git conventions → `00-governance/git-conventions.md`
- Agile conventions → `00-governance/agile-conventions.md`
- Definition of Done → `00-governance/definition-of-done.md`
- Microservice documentation standard → `09-microservices/README.md`
