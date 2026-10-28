# 07 — API Contracts

> **What is this?** The formal specification of how microservices communicate with each other
> and with the outside world. The contract is the promise a service makes to its consumers.

## As-built status at project close (2026-10-28)

> Added at project close. The rest of this README is the original template guidance (2026-08-25) and is kept as
> reference; this section describes what actually exists.

### Source of truth for the contracts

The contracts were **not** written contract-first as YAML files in this folder. Each service generates its OpenAPI
document at runtime from its code, and the documentation UI aggregates them:

| Service | OpenAPI document (inside the service) | Stack |
|---------|---------------------------------------|-------|
| ms-iam | `/v3/api-docs` | springdoc (Spring Boot) |
| ms-places, ms-devices, ms-consumption, ms-valve, ms-notifications | `/openapi/v1.json` | ASP.NET Core OpenAPI |
| All, aggregated | `https://api.sy-water.tech/docs` (Swagger UI container) | — |

The runtime documents are the authoritative contract of the frozen code (ADR-021). The index of every endpoint at
close is **[endpoint-catalog.md](./endpoint-catalog.md)**.

### State of the files in this folder

| File | State | Note |
|------|-------|------|
| `contracts/openapi/auth-service.yaml` | **Historical** (template, 2026-08-25) | Old design (`/api/v1/auth/register`, port 8081, "auth-service"). Does not describe ms-iam |
| `contracts/openapi/api-gateway.yaml` | **Historical** (template, 2026-08-25) | Mentions `X-User-Role` VIEWER and headers the gateway does not add. Does not describe `api-gateway` |
| `contracts/openapi/_shared.yaml`, `_template-service.yaml` | Template | Never instantiated |
| `contracts/openapi/ms-iam.yaml`, `ms-places.yaml` | **Do not exist** | Referenced by some service documents; replaced by the runtime documents (E-23) |
| `guidelines.md`, `authentication.md` | **Do not exist** | Described below as to-do; the conventions applied are in the service READMEs and ADR-008 |

### Conventions actually applied

- No version segment in the URL: `/api/...` per service, public prefix by service at the gateway (`/iam`, `/places`,
  `/devices`, `/consumption`, `/valve`, `/notifications`).
- Authentication: JWT RS256 bearer (ADR-008); internal routes `/internal/**` with `X-Internal-Key`, not public (ADR-006).
- Errors: RFC 7807 problem details (`ProblemDetails` in .NET, `ProblemDetail` in ms-iam) with a domain error code.
- Foreign resources return 404, not 403 (ms-places DEC-007, ms-consumption DEC-005).
- Long operations return 202 + polling (valve orders, ms-valve DEC-002).

---

## Why contracts are critical in microservices

In a monolith, changing a function is easy: the compiler tells you what you break.
In microservices, a change to an API can silently break other services in production.

**Contract-first development:** design the API before implementing it. This way the team
can work in parallel (frontend, backend, other services) with an agreed contract.

---

## What is here and how to fill it in

### `guidelines.md` ⭐
The project's REST standards. **The entire team must follow this before designing an endpoint.**

**Key topics to define:**
```markdown
## Versioning
- URL: /api/v1/resources (version in the URL)
- Header: Accept: application/vnd.api+json;version=1

## Endpoint naming
- Plural for collections: GET /users
- Nouns, not verbs: GET /users/{id}/orders (not: GET /getUserOrders)
- snake_case or kebab-case for URLs

## Pagination
- ?page=1&limit=20 (offset-based)
- ?cursor=xyz (cursor-based for large volumes)

## Standard responses
| Code | When to use it |
|------|----------------|
| 200 | Success with body |
| 201 | Successful creation |
| 204 | Success without body |
| 400 | Client error (validation) |
| 401 | Not authenticated |
| 403 | Not authorized |
| 404 | Resource not found |
| 422 | Unprocessable entity |
| 500 | Server error |

## Error format
{
  "error": "VALIDATION_ERROR",
  "message": "The email field is required",
  "details": [{"field": "email", "message": "required"}]
}
```

### `authentication.md` ⭐
The system's authentication and authorization strategy.
**Fill in:** what mechanism (JWT, OAuth2, API Key), authentication flow, expiration policies,
refresh token handling, RBAC (roles and permissions).

### `contracts/openapi/` ⭐⭐
One `.yaml` file per microservice with the OpenAPI 3.0 contract.

**File name:** `service-name.yaml`

**Minimum structure for each contract:**
```yaml
openapi: 3.0.3
info:
  title: [Service Name] API
  version: 1.0.0
  description: [What this service does]

servers:
  - url: http://localhost:8080/api/v1
    description: Local

paths:
  /resources:
    get:
      summary: List resources
      tags: [Resources]
      parameters:
        - name: page
          in: query
          schema:
            type: integer
      responses:
        '200':
          description: List of resources
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ResourceList'

components:
  schemas:
    Resource:
      type: object
      required: [id, name]
      properties:
        id:
          type: string
          format: uuid
        name:
          type: string
  
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT

security:
  - bearerAuth: []
```

### `_shared.yaml`
Reusable schemas and components across all contracts (pagination, errors, etc.)

---

## Recommended tools

- **Swagger Editor** — Online editor for OpenAPI
- **Redocly** — Generates HTML documentation from the YAML (already configured in `redocly.yaml`)
- **Postman / Insomnia** — For testing the contracts
- **OpenAPI Generator** — Generates client/server code from the YAML

---

## Correlations with other sections

| This section is fed by... | And feeds into... |
|---------------------------|-------------------|
| `04-requirements/functional.md` → what it must do | Endpoints implementing each FR |
| `06-data/models.md` → what data exists | Schemas in the contracts |
| `02-domain/domain-events.md` | Event publishing endpoints |
| Contracts here | `09-microservices/[service]/` that implements them |
| `endpoint-catalog.md` | Index of the as-built endpoints (2026-10-28) |
| Contracts here | `11-quality/testing-strategy.md` → contract tests |

---

## Questions this section must answer

- What is each endpoint called and what does it do?
- What data does it receive and what does it return?
- How does the client authenticate?
- What errors can the service return?
- How do I version the API without breaking consumers?
