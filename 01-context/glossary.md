# Project Glossary

> Official dictionary of the project. If there is ambiguity about a term, this document wins.
> Add terms throughout the project, not only at the start.

---

## Domain terms

| Term | Definition | Notes / Synonyms |
|------|-----------|-----------------|
| Lugar (Place) | A physical location (home or business) where water consumption is monitored. Each place has one address, one linked device, and its own consumption history. | A user can own multiple places. Do NOT use "property" or "site" — always "place". |
| Dueño (Owner) | The user who created a place and pays the water bill. Has full control: consumption, valve, goals, bills, alerts, invitations. | One owner per place. Not to be confused with "admin". |
| Invitado (Guest) | A person invited by the owner to view a place's consumption data. Read-only access; cannot control the valve or modify settings. | Can be a family member, roommate, or building administrator. |
| Administrador (Admin) | Platform operator who manages registered users, validates device failures, and monitors the application's health. | Does NOT manage individual places or consumption data. |
| Dispositivo (Device) | The physical IoT unit installed at a place's water inlet. Composed of an ESP32 controller, a YF-S201 flow sensor, and an electrovalve. | Always refers to the full assembly, not just the ESP32. |
| Válvula (Valve) | The electrovalve that opens or closes the water supply at the place's inlet. Controlled remotely via commands sent from the backend to the ESP32. | Also called "electroválvula". States: Open / Closed / No communication. |
| Caudal (Flow rate) | The volume of water passing through the sensor per unit of time. The raw measurement from the YF-S201 sensor. | Measured in pulses, converted to liters or m³ by the system. |
| Lectura (Reading) | A single data point sent by the ESP32 to the backend, containing a flow measurement and a timestamp. | Multiple readings are aggregated to calculate consumption. |
| Consumo (Consumption) | The total volume of water used at a place during a given period, calculated by aggregating readings. | Displayed in the unit selected by the user (liters, m³, or gallons). |
| Meta (Goal) | A monthly water consumption target set by the owner for a specific place. | Expressed in the user's configured measurement unit. |
| Recibo (Utility bill) | The water bill issued by the utility company (acueducto). Users register it manually to compare against sensor data. | Contains: billed consumption, total amount, billing period, payment due date. |
| Tarifa (Tariff) | The price per unit of water that the user pays, as stated on their utility bill. Used to estimate monetary cost of consumption. | Can be entered manually or pre-loaded from a catalog (future). |
| Estrato (Stratum) | Socioeconomic classification (1–6) used in Colombia to determine utility rates. Assigned per place. | Only applicable in Colombia. Other countries use different rate structures. |
| Fuga (Leak) | A suspected water leak, detected when continuous flow exceeds a configurable threshold for a sustained period (default: 30 min). | Higher weight is given to nighttime consumption. The system cannot confirm a physical leak — it flags suspicious patterns. |
| Cierre automático (Auto shutoff) | A rule that triggers the valve to close automatically when a condition is met (suspected leak or goal exceeded). Requires owner confirmation before executing. | All auto-shutoff rules are disabled by default. The system never closes the valve without user confirmation. |
| Período de facturación (Billing period) | The date range covered by a utility bill (start date to end date). Used to align sensor data with billed consumption for comparison. | Must not overlap with another registered bill for the same place. |
| Alerta (Alert) | A notification triggered by a system event. Classified by urgency: Critical (leak, valve closed), Important (goal exceeded, sensor disconnected), Informative (near goal, bill reminder). | Channels vary by urgency level. Critical alerts cannot be fully silenced. |

---

## Technical terms of the project

| Term | Definition |
|------|-----------|
| Microservice | Independent service with a single responsibility, its own process, and its own database |
| Domain Event | A fact that occurred in the business that other services can observe. Name always in past tense |
| Bounded Context | Boundary within which a particular domain model has consistent meaning |
| API Gateway | Single entry point to the system that routes requests to the corresponding microservices |
| Circuit Breaker | Pattern that stops calls to a failing service, preventing failure cascades |
| Saga | Sequence of local transactions across different services with compensating transactions on failure |
| Dead Letter Queue | Queue where messages that could not be processed after several retries are sent |
| Idempotence | Property of an operation to produce the same result if executed multiple times |
| MQTT | Message Queuing Telemetry Transport — lightweight publish/subscribe protocol used for ESP32-to-backend communication |
| Topic (MQTT) | A named channel in the MQTT broker where devices publish readings and the backend subscribes to receive them |
| QoS (MQTT) | Quality of Service level that determines delivery guarantees: 0 (at most once), 1 (at least once), 2 (exactly once) |
| Telemetry | The automated collection and transmission of sensor data from the ESP32 to the backend |
| OTA (Over-The-Air) | Remote firmware update mechanism for ESP32 devices without physical access |

---

## Acronyms

| Acronym | Meaning |
|---------|---------|
| IAM | Identity and Access Management |
| JWT | JSON Web Token |
| API | Application Programming Interface |
| CRUD | Create, Read, Update, Delete |
| DTO | Data Transfer Object |
| FR | Functional Requirement |
| NFR | Non-Functional Requirement |
| SLO | Service Level Objective |
| SLA | Service Level Agreement |
| ADR | Architecture Decision Record |
| PR | Pull Request |
| DoD | Definition of Done |
| DoR | Definition of Ready |
| CI/CD | Continuous Integration / Continuous Delivery |
| MQTT | Message Queuing Telemetry Transport |
| QoS | Quality of Service (in MQTT context) |
| OTA | Over-The-Air (firmware updates) |
| IoT | Internet of Things |
| ESP32 | Espressif Systems 32-bit microcontroller |
| QR | Quick Response (code used for device linking) |
| PWA | Progressive Web App |
| HU | Historia de Usuario (User Story) |
| MoSCoW | Must / Should / Could / Won't — prioritization method |
