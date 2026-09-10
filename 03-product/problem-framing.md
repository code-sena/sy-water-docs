# Problem Framing

> This document defines the real problem that Save Your Water solves
> before proposing any technical solution.

---

## 1. The problem in one sentence

**Heads of household and business owners** who **pay a water bill** have **no visibility into how much water they consume** until the monthly bill arrives, making it **impossible to detect leaks in time, verify if the billed amount is accurate, or take corrective action** — resulting in **unexpected costs, undetected water waste, and no way to set or track savings goals**.

---

## 2. Affected users

| Segment | Description | Priority |
|---------|-------------|---------|
| Head of household | Person who pays the residential water bill and wants to control consumption | High |
| Business owner | Person who pays the commercial water bill, often across multiple locations | High |
| Family members / roommates | People living at the same place who want visibility but don't control the bill | Medium |

### Jobs-to-be-done (JTBD)

**When** I open the tap and wonder how much water I'm using this month,
**I want** to see my real-time consumption on my phone,
**so that** I can adjust my habits before the bill arrives.

**When** I leave for vacation and worry about a water leak at home,
**I want** to receive an alert and shut off the water remotely,
**so that** a leak doesn't run for days and cost me a fortune.

**When** I receive my monthly water bill and it seems too high,
**I want** to compare what my sensor measured against what the utility billed me,
**so that** I can determine if I'm being overcharged and file a complaint with evidence.

---

## 3. Evidence of the problem

| Evidence type | Source | Key finding |
|--------------|--------|------------|
| User experience | Team members' own households | Water bills arrive as a surprise; there's no way to monitor consumption between bills |
| Market gap | Research on available tools in Colombia, Ecuador, US | Consumer-grade real-time water monitoring with remote valve control is not widely available or affordable |
| Hardware feasibility | ESP32 + YF-S201 prototyping | A low-cost IoT device can measure residential flow rate accurately enough for consumption tracking |
| Leak cost | Utility company data (general) | Undetected leaks can waste thousands of liters per month and significantly increase bills |

---

## 4. Current user solution (and its problems)

| Current solution | Limitations | Cost / Friction |
|-----------------|------------|----------------|
| Wait for the monthly utility bill | No real-time visibility; reactive, not preventive | Surprise costs; no way to adjust mid-month |
| Manually read the water meter | Requires physical access to the meter (often outside); error-prone; no historical tracking | Time and effort; meter is often hard to reach |
| Call the utility company | Slow; limited to billing inquiries; no real-time data | Long wait times; no resolution for measurement disputes |
| Smart home devices (expensive) | Industrial-grade solutions exist but are priced for utilities, not consumers | $500+ per device; requires professional installation |

---

## 5. Solution hypothesis

**We believe that** an affordable IoT device (ESP32 + flow sensor + electrovalve) paired with a mobile/web platform
**for** heads of household and business owners,
**will achieve** real-time consumption visibility, early leak detection, remote water shutoff, and accurate bill comparison.
**We will know we succeeded when** users actively monitor their consumption, respond to alerts, and use the bill comparison feature to verify their utility charges.

---

## 6. Success metrics

| Metric | How to measure | Target |
|--------|---------------|--------|
| Users actively viewing consumption | Dashboard visits per week | At least 3 visits/week per active user |
| Alert response rate | % of critical alerts acknowledged within 1 hour | > 80% |
| Bill comparison adoption | % of users who register at least 1 bill | > 50% of users with 2+ months of data |
| Leak detection effectiveness | Leaks flagged by the system vs. confirmed by user | > 70% true positive rate |
| Device uptime | % of time the device reports without disconnection | > 95% |

> **Note:** As a university project, these metrics are aspirational targets for the product design. Actual measurement will depend on the scope of user testing.

---

## 7. Hypothesis risks

| Risk | Probability | Impact | Mitigation |
|------|------------|--------|------------|
| The YF-S201 sensor is not accurate enough for bill comparison | Medium | High | Calibrate sensor per installation; warn users that comparison is an estimate |
| Users won't install the device themselves | Medium | Medium | Provide clear installation guide; device is designed for main inlet (one point) |
| Wi-Fi signal doesn't reach the water meter | Medium | High | Document minimum signal requirements; suggest Wi-Fi extender |
| False leak alerts (continuous flow that isn't a leak) | High | Medium | Tunable thresholds; nighttime weighting; confirmation before auto-shutoff |
| Auto-shutoff leaves a family without water | Low | Critical | All auto-shutoff rules disabled by default; always requires user confirmation; reopening is always manual |

---

## 8. Out of scope (we do not solve)

- **Per-fixture breakdown** — The sensor is at the main inlet; it cannot tell which tap uses how much
- **Billing or payment processing** — We compare against the utility bill, we don't generate or collect payments
- **Physical installation service** — Users install the device themselves or hire a plumber
- **Industrial/municipal SCADA integration** — This is a consumer product, not a utility management system
- **Water quality monitoring** — We measure flow volume, not water quality (pH, turbidity, etc.)

---

## Correlations

- Product vision → `03-product/vision.md`
- Delivery roadmap → `03-product/roadmap.md`
- User stories → backlog (61 HUs across 14 epics)
- System scope → `01-context/scope.md`
- Identified risks → `15-project-control/risks.md`
