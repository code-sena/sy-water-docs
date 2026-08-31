# Security Policy

> Security is a mandatory system property for Save Your Water. It must be considered from the initial design stage and enforced in every sprint.
> Any exception must be explicitly approved by the Tech Lead and documented in the project risk register or backlog.

---

## Security principles

1. **Defense in Depth:** multiple security layers must be applied so that one vulnerability does not expose the whole system.
2. **Least Privilege:** each component must only have the permissions it needs to operate.
3. **Fail Secure:** when a component fails, access is denied and the system remains safe.
4. **Security by Design:** security controls must be planned before implementation.
5. **Zero Trust:** trust must never be assumed, even for internal services or validated users.

---

## Authentication and access

### JWT (JSON Web Tokens)

| Property | Required value |
|----------|---------------|
| Signing algorithm | RS256 or HS256 with at least 256-bit secret |
| Access token expiration | 1 hour |
| Refresh token expiration | 7 days |
| Required claims | `sub`, `iat`, `exp`, `jti`, and user roles |
| Client storage | `httpOnly` cookie for web clients or secure device storage for mobile apps |

**Prohibited in the token payload:**
- Passwords
- Raw card data
- Full personal data beyond a unique user identifier

### Refresh token handling

- Refresh tokens must be stored in a secure database record
- Rotation is mandatory for each use
- Refresh tokens are invalidated on logout and password change
- All active sessions must be invalidated when a revoked token is detected

---

## Authorization model

### RBAC (Role-Based Access Control)

| Role | Description | Typical permissions |
|------|-------------|--------------------|
| `ADMIN` | Platform administrator | full management of users, settings, and monitoring |
| `OPERATOR` | Water operations or service staff | manage incidents, alerts, and operational tasks |
| `VIEWER` | Read-only user | read dashboards, reports, and service information |
| `CUSTOMER` | End user or citizen | view profile, consumption, notifications, and service status |

**Permission model:**

```
Permission: [resource]:[action]

Examples:
  water-usage:read
  alerts:create
  alerts:read
  dashboard:read
  users:manage
  reports:export
```

**Validation rules:**
- The API gateway validates token signatures and expiration
- Each backend service validates permissions per operation
- Roles are included in the JWT claims and must be checked before executing protected actions

---

## Secure communication

### Transmission

- **HTTPS is mandatory** in all environments except local development
- TLS 1.2 minimum; TLS 1.3 recommended
- Certificates must come from a trusted CA or configured internal certificate authority
- HSTS must be enabled in production

### Internal service-to-service communication

- Internal services must use mTLS when available
- Services that cannot support mTLS must use a signed token or internal API key
- Services must never trust another service without validating identity and authorization

---

## Secret management

```
✗ Never commit secrets to source code
✗ Never store secrets in git-tracked .env files
✗ Never log secrets or sensitive values
✗ Never include secrets in client-visible errors
✓ Use environment variables injected by the runtime or deployment platform
✓ Use a secure secret manager such as Vault, AWS Secrets Manager, or equivalent
✓ Use encrypted Kubernetes or orchestration-managed secrets when applicable
```

**Rotation policy:**
- API keys: every 90 days
- TLS certificates: before expiration and no later than 60 days before renewal
- Database credentials: every 6 months or immediately upon suspicion of compromise

---

## Input validation and sanitization

### General rules

1. **Never trust user input.** Validate at the edge of the system before processing.
2. **Whitelist allowed values.** Prefer explicit validation over broad permissive rules.
3. **Reject early.** If input is invalid, return a 400 or equivalent and stop processing.

### SQL Injection prevention

```java
// Unsafe
String query = "SELECT * FROM users WHERE email = '" + email + "'";

// Safe
PreparedStatement stmt = connection.prepareStatement(
    "SELECT * FROM users WHERE email = ?"
);
stmt.setString(1, email);
```

### XSS prevention

```javascript
// Unsafe
element.innerHTML = userProvidedContent;

// Safe
element.textContent = userProvidedContent;
```

### Validation patterns

```java
// Example validation at the service boundary
if (email == null || !email.matches("^[\\w.%+-]+@[\\w.-]+\\.[A-Za-z]{2,6}$")) {
    throw new IllegalArgumentException("Invalid email format");
}
```

---

## OWASP review checklist

| Vulnerability | Required control |
|---------------|------------------|
| Broken Access Control | Role checks in each protected endpoint |
| Cryptographic Failures | TLS enforcement and secure secret management |
| Injection | Prepared statements and strict input validation |
| Insecure Design | Threat modeling and secure design review |
| Security Misconfiguration | Secure defaults, environment review, and minimal privileges |
| Vulnerable Components | Dependency review and update process |
| Authentication Failures | JWT validation, token rotation, and secure session handling |
| Software Integrity Failures | Verify dependency integrity and code review for sensitive changes |
| Logging Failures | No secrets or personal data in logs |
| SSRF | Restrict outbound destinations and validate external targets |

---

## Audit and security logs

### Security events that must be recorded

- login success and failure
- repeated failed login attempts
- password change events
- token revocation and refresh events
- unauthorized access attempts
- role change operations
- operational actions on sensitive dashboards or reports

**Required fields in security logs:**
- `userId` or `ANONYMOUS`
- `sourceIp`
- `action`
- `resource`
- `result` (SUCCESS / FAILURE)
- `timestamp`

---

## Vulnerability process

### When a vulnerability is found

1. Do not discuss it in public channels
2. Notify the Tech Lead immediately through a private communication channel
3. Create a restricted issue or private ticket with the details
4. Assign severity and classify the impact
5. Apply the fix in the current sprint for critical issues, or in the next sprint for high-impact items

### Severity response times

| Severity | Remediation target |
|----------|-------------------|
| Critical | Same day or within 24 hours |
| High | Within 7 days |
| Medium | Within 30 days |
| Low | Planned in the next maintenance cycle |

---

## Security review in delivery

Every sprint, the team must review:
- new authentication or authorization flows
- external API integrations
- data handling of personal or operational information
- any change that touches alerts, reports, or user access

If a change affects security, it must be included in the sprint review evidence and validated before release.

---

## Correlations

- Agile team conventions → `00-governance/agile-conventions.md`
- Definition of Done → `00-governance/definition-of-done.md`
- Risk management → `15-project-control/risks.md`
| Critical (CVSS 9-10) | 24 hours |
| High (CVSS 7-8.9) | 1 week |
| Medium (CVSS 4-6.9) | 1 month |
| Low (CVSS < 4) | Next security review |

---

## Correlations

- Security non-functional requirements → `04-requirements/non-functional.md`
- ADR on authentication → `05-architecture/decisions/`
- Security event observability → `13-operations/observability.md`
- RBAC implemented in → `09-microservices/services/XX-auth-service/`
