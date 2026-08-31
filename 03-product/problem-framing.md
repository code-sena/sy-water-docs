# Problem Framing — Problem Definition

> This document defines the real problem that Save Your Water intends to solve before proposing any technical solution.
> The goal is to align the team, the stakeholders, and the product decisions around the actual pain and the concrete value to create.

---

## 1. The problem in one sentence

**Household users, operators, and administrators** who **need to monitor and manage water consumption and service incidents** struggle with **limited visibility, delayed detection of anomalies, and inefficient maintenance processes** because **consumption data is fragmented, alerts are not timely, and reporting is not centralized**, resulting in **water waste, delayed attention to service problems, and poor operational decision-making**.

---

## 2. Affected users

| Segment | Description | Estimated size | Priority |
|---------|-------------|---------------|---------|
| Households / citizens | People who consume water and want to understand usage patterns and avoid waste | High | High |
| Operators / field staff | Staff responsible for monitoring service health and responding to incidents | Medium | High |
| Administrators | People responsible for managing users, rules, alerts, and service configuration | Medium | Medium |

### Jobs-to-be-done (JTBD)

**When** a household wants to understand its monthly water consumption or detect an unusual spike,
**I want** clear data, alerts, and service visibility,
**so that** I can reduce waste, act on time, and avoid unnecessary costs.

**When** an operator receives a report of a possible leak or service issue,
**I want** a structured way to review, validate, and act on the problem,
**so that** I can resolve it faster and reduce service impact.

---

## 3. Evidence of the problem

| Evidence type | Source | Date | Key finding |
|--------------|--------|------|------------|
| User interviews | Households and service staff | 2026 | Users do not have a centralized view of water consumption and incident reporting |
| Operational observation | Local service workflows | 2026 | Incidents are often handled after the problem becomes visible to the user |
| Business need analysis | Team discussions and project brief | 2026 | Water waste and delayed response are recurring operational pain points |
| Benchmarking | Similar utility and monitoring tools | 2026 | Service monitoring tools exist, but they are fragmented and not focused on household engagement |

---

## 4. Current user solution (and its problems)

| Current solution | Limitations | Cost/Friction |
|-----------------|------------|--------------|
| Manual tracking of readings | Error-prone, inconsistent, limited historical visibility | Time spent checking and reconciling data |
| Phone calls or informal reports | Slow, hard to track, lack of structure | Delayed response and poor visibility |
| Dispersed service tools | Information is fragmented across channels | More effort to coordinate actions |

---

## 5. Solution hypothesis

**We believe that** a digital platform that centralizes water consumption monitoring, anomaly alerts, and service reporting
**for** households, operators, and administrators,
**will achieve** reduced waste, faster service response, and better decision-making.
**We will know we succeeded when** water usage is easier to understand, alerts are acted on in time, and operational incidents are managed with greater visibility and speed.

---

## 6. Success metrics (North Star)

| Metric | Current baseline | 6-month target | How to measure it |
|--------|----------------|---------------|-------------------|
| Household active engagement | Low / inconsistent | 60% of registered households monthly active | Platform analytics |
| Alert response time | Not measured / slow | < 24 hours for critical alerts | Alert workflow metrics |
| Reported issue resolution time | Unstructured process | Reduce by 30% | Service management data |
| User retention | Low | 70% monthly retention among active users | App analytics |

**North Star Metric:** percentage of active households that successfully use the platform to monitor consumption and respond to alerts in a given month.

---

## 7. Hypothesis risks

| Risk | Probability | Impact | Experiment to validate |
|------|------------|--------|----------------------|
| Users will not actively record consumption | Medium | High | Pilot with a limited number of households |
| Alert thresholds are not accurate enough | Medium | High | Compare with historical consumption patterns |
| Operators may not adopt the workflow | Medium | Medium | Pilot with real service personnel |

---

## 8. Out of scope (we do not solve)

> This version focuses on the core flow: consumption monitoring, anomaly alerts, and issue reporting. It does not include full billing automation, advanced SCADA integration, or a large public utility ecosystem in the first phase.

- Full billing and invoice management
- Deep industrial or municipal SCADA integration
- Advanced predictive maintenance using complex sensor ecosystems
- Large-scale ERP or finance integrations beyond the initial product scope

---

## Correlations

- Product vision → `03-product/vision.md`
- HUs that implement this solution → `04-requirements/user-stories.md`
- Detailed KPIs → `13-operations/README.md`
