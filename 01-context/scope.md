# System Scope

> Save Your Water — System Scope.
> Review this document at the start of each planning cycle.
> Change the scope only through the process described at the bottom.

---

## In Scope

### MVP Features (Entrega 1)

| # | Feature | Description | Epic |
|---|---------|-------------|------|
| 1 | User registration and login | Sign up with first name, last name, email, phone, and password; email verification, login, Google OAuth, password recovery (email or phone), logout | E1 |
| 2 | Profile management | View and edit name, photo, phone number; change password | E1 |
| 3 | Place registration | Create a place (home/business) with reference name, address, country, department/state, city, stratum, and number of inhabitants; unique name per user | E2 |
| 4 | Place management | List, switch between, edit, and delete places | E2 |
| 5 | Device linking | Link an ESP32 device to a place via ID or QR code; authenticate device | E3 |
| 6 | Telemetry ingestion | Receive and store flow readings from ESP32 via MQTT; validate and deduplicate | E3 |
| 7 | Device status | Show connected/disconnected/never reported status and last report timestamp | E3 |
| 8 | Device unlinking | Unlink a device from a place while preserving consumption history | E3 |
| 9 | Real-time consumption | Display current and accumulated consumption with < 1 min latency | E4 |
| 10 | Period selector | Switch between day, week, and month views | E4 |
| 11 | Consumption chart | Interactive chart adjusted to selected period (hours/days) | E4 |
| 12 | Auto-refresh panel | Panel updates automatically every minute; manual refresh available | E4 |

### Included integrations (MVP)

| External system | Integration type | Purpose |
|----------------|-----------------|---------|
| Google OAuth | OAuth 2.0 | Social login (registration and sign-in) |
| ESP32 devices | MQTT | Receive flow sensor readings and send valve commands |
| Email provider (TBD) | SMTP / API | Verification emails, password recovery, critical alerts |

### Environments being built

| Environment | Purpose |
|-------------|---------|
| Local | Development on developer machines (Docker) |
| Development (dev) | Integration testing on VPS |
| Production | Initial production deployment on VPS |

---

## Out of Scope (MVP)

| # | What is out of scope | Reason | Planned for |
|---|---------------------|--------|-------------|
| 1 | Manual valve control (open/close) | Requires confirmed device communication round-trip and security code flow | Entrega 2 |
| 2 | Consumption goals and progress tracking | Depends on having at least one month of historical data | Entrega 2 |
| 3 | Alerts and notifications (push, email, WhatsApp) | Complex multi-channel system; MVP focuses on real-time visibility | Entrega 2 |
| 4 | Leak detection alerts | Requires threshold tuning and alert infrastructure | Entrega 2 |
| 5 | Utility bill registration and comparison | Depends on stable consumption data to compare against | Entrega 3 |
| 6 | Tariffs and currency configuration | Depends on bill comparison feature | Entrega 3 |
| 7 | Automatic valve shutoff rules | High-risk feature; requires confirmed manual valve control first | Entrega 3 |
| 8 | Member/shared user access | Core owner features must be stable before adding shared access | Entrega 3 |
| 9 | Downloadable PDF/Excel reports | Nice-to-have; core value is real-time visibility | Entrega 4 |
| 10 | Water saving tips | Generic content; low priority vs. measurement features | Entrega 4 |
| 11 | Pre-loaded tariff catalogs (CO, EC, US) | Ongoing maintenance cost; manual tariff entry covers the need | Entrega 4 |
| 12 | Per-fixture consumption breakdown | Hardware limitation: single sensor at main inlet cannot disaggregate | Won't — hardware constraint |
| 13 | Staging environment | Team of 3; dev + prod is sufficient for now | Future |

### What another system / team handles

| Feature | Who handles it | Why not us |
|---------|---------------|-----------|
| Water billing and payment processing | Utility company (acueducto) | We compare against their bills, we don't generate or collect payments |
| Physical installation of the device | The end user or a plumber | We provide instructions, not installation service |
| ESP32 firmware development | Hardware team / firmware scope | Documented separately from this backend/frontend documentation repo |

---

## Scope assumptions

| # | Assumption | Consequence if false |
|---|-----------|---------------------|
| 1 | The ESP32 has stable Wi-Fi connectivity at the installation point | Telemetry gaps; device appears as disconnected frequently |
| 2 | One device per place is sufficient (single inlet) | Would need multi-device support per place |
| 3 | Users have a smartphone with Android or iOS | React Native web extension covers desktop, but mobile is the primary UX |
| 4 | MQTT broker can handle the expected device volume (initially < 1,000 devices) | Would need to evaluate a managed/cloud broker earlier |
| 5 | A single SQL Server instance is sufficient for all services (schema-per-service) | If volume grows beyond a single instance, would need to split into separate databases per service |
| 6 | The YF-S201 sensor provides readings accurate enough for bill comparison | If accuracy is insufficient, bill comparison feature loses value |

---

## Constraints

| Type | Description |
|------|-------------|
| **Team** | 3 developers (no dedicated QA, DevOps, or designer) |
| **Budget** | University project; no cloud budget — Docker local + VPS |
| **Technology** | React Native (mobile+web), Spring Boot (IAM), .NET (under evaluation), SQL Server, MQTT |
| **Hardware** | Single measurement point per place (YF-S201 + ESP32) |
| **Regulatory** | Password storage must be encrypted; personal data handling must follow good practices |

---

## External dependencies

| Dependency | Provider | Required for | Status |
|-----------|---------|-------------|--------|
| Google OAuth credentials | Google Cloud Console | HU-005 (Google login) | 🟡 To be configured |
| MQTT Broker | TBD (Mosquitto / HiveMQ) | HU-012 (telemetry ingestion) | 🟡 To be selected |
| Email service | TBD (SendGrid / AWS SES / SMTP) | HU-003 (email verification), HU-006 (password recovery) | 🟡 To be selected |
| ESP32 firmware | Hardware team | HU-011, HU-012 (device linking + readings) | 🟡 In development |
| WhatsApp Business API / SMS provider | TBD | HU-029 (WhatsApp/SMS alerts — Entrega 2) | 🔴 Not started |

---

## How to update the scope

1. Document the proposed change in this file
2. Evaluate the impact on schedule and effort
3. Obtain agreement from the full team (3 developers)
4. Update the roadmap and affected user stories in the backlog
5. Record the decision as an ADR in `05-architecture/decisions/records/` if it affects architecture
