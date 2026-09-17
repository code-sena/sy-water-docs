# User Stories — Backlog

> Complete backlog for Save Your Water organized by Entrega and Epic.
> Each story follows the format: As [role], I want [goal], so that [benefit].

---

## Backlog overview

| Entrega | Epics | User stories | Story points | Status |
|---------|-------|-------------|-------------|--------|
| Entrega 1 (MVP) | E1–E4 | HU-001 to HU-017 (17) | ~80 SP | Planned |
| Entrega 2 (Smart) | E5–E7 | HU-018 to HU-029 (12) | ~56 SP | Planned |
| Entrega 3 (Social) | E8–E12 | HU-030 to HU-044 (15) | ~65 SP | Planned |
| Entrega 4 (Admin) | E13–E16 | HU-045 to HU-058 (14) | ~56 SP | Planned |
| **Total** | **16** | **58** | **~257 SP** | |

---

## Epics

| ID | Epic | Entrega | Description |
|----|------|---------|-------------|
| E1 | Accounts & Access | 1 | Register, login, password recovery, profile, account deletion |
| E2 | Place Management | 1 | Create, edit, and delete places (homes/businesses) |
| E3 | IoT Device Linking | 1 | Link/unlink ESP32 devices to places via MQTT |
| E4 | Real-time Consumption Monitoring | 1 | Real-time and historical consumption visualization |
| E5 | Manual Valve Control | 2 | Manual remote open/close of the electrovalve |
| E6 | Consumption Goals | 2 | Set monthly goals and receive progress alerts |
| E7 | Alerts & Notifications | 2 | Leak detection, alert history, notification preferences |
| E8 | Utility Bill Management | 3 | Register bills and compare against sensor data |
| E9 | Tariffs & Measurement Units | 3 | Configure tariffs for cost estimation |
| E10 | Automatic Valve Shutoff | 3 | Rule-based automatic valve closure |
| E11 | Shared Users (Owner / Guest) | 3 | Invite guests to view place consumption data |
| E12 | Shared Users (Owner / Collaborator) | 3 | Invite collaborators with extended permissions to a place |
| E13 | Downloadable Reports | 4 | Monthly reports, savings analysis, PDF/CSV export |
| E14 | Savings Tips | 4 | Generic water-saving recommendations for users |
| E15 | Preloaded Tariffs by Country | 4 | Pre-loaded tariff catalogs for Colombia, Ecuador, and USA |
| E16 | Admin Panel | 4 | Platform-wide user/device management and generic recommendations |

---


# Entrega 1 — MVP

---

## E1 — Accounts & Access

### HU-001 — Register with email and password

> **As** an unregistered user
> **I want** to create an account with my email and password
> **so that** I can access the platform and start monitoring my water consumption

```gherkin
Scenario: successful registration
  Given a user provides a valid email, password, and personal data
  When they submit the registration form
  Then the system creates the account
  And sends a verification email

Scenario: duplicate email
  Given the email is already registered
  When they submit the registration form
  Then the system rejects the request with "email already in use"
```

| Field | Value |
|-------|-------|
| Story Points | 5 |
| Priority | Must Have |
| Service(s) | auth-service |
| Dependencies | None |

---

### HU-002 — Login with email and password

> **As** a registered user
> **I want** to log in with my email and password
> **so that** I can access my protected data and features

```gherkin
Scenario: valid credentials
  Given a verified user enters correct email and password
  When they submit the login form
  Then the system issues a JWT and creates a session

Scenario: invalid credentials
  Given the user enters a wrong password
  When they submit the login form
  Then the system rejects the request and increments login attempts
```

| Field | Value |
|-------|-------|
| Story Points | 3 |
| Priority | Must Have |
| Service(s) | auth-service |
| Dependencies | HU-001 |

---

### HU-003 — Login with Google OAuth

> **As** a user
> **I want** to log in using my Google account
> **so that** I don't have to remember another password

```gherkin
Scenario: Google sign-in (new user)
  Given the user authenticates with Google for the first time
  When the OAuth callback is received
  Then the system creates an account and issues a JWT

Scenario: Google sign-in (existing user)
  Given the user already has an account linked to their Google email
  When the OAuth callback is received
  Then the system issues a JWT without creating a new account
```

| Field | Value |
|-------|-------|
| Story Points | 5 |
| Priority | Must Have |
| Service(s) | auth-service |
| Dependencies | HU-001 |

---

### HU-004 — Recover password

> **As** a registered user
> **I want** to reset my password via email
> **so that** I can regain access if I forget my credentials

```gherkin
Scenario: password reset request
  Given the user provides a registered email
  When they request a password reset
  Then the system sends a reset link with a time-limited token

Scenario: password reset with valid token
  Given the user clicks a valid, non-expired reset link
  When they submit a new password
  Then the system updates the password and invalidates all existing sessions
```

| Field | Value |
|-------|-------|
| Story Points | 5 |
| Priority | Must Have |
| Service(s) | auth-service |
| Dependencies | HU-001 |

---

### HU-005 — View and edit my profile

> **As** a registered user
> **I want** to view and update my profile information
> **so that** my account data stays current

```gherkin
Scenario: edit profile
  Given the user is logged in
  When they update their name, phone, or photo
  Then the system saves the changes and confirms the update
```

| Field | Value |
|-------|-------|
| Story Points | 3 |
| Priority | Should Have |
| Service(s) | auth-service |
| Dependencies | HU-002 |

---

### HU-006 — Delete my account

> **As** a registered user
> **I want** to delete my account
> **so that** my data is removed from the platform when I no longer use it

```gherkin
Scenario: account deletion
  Given the user confirms the deletion action
  When the request is processed
  Then the system soft-deletes the account
  And unlinks all devices and revokes all sessions
```

| Field | Value |
|-------|-------|
| Story Points | 3 |
| Priority | Should Have |
| Service(s) | auth-service, device-service |
| Dependencies | HU-002 |

---

## E2 — Place Management

### HU-007 — Register a place

> **As** an Owner
> **I want** to register a place (home or business)
> **so that** I can link a device to it and start monitoring water consumption

```gherkin
Scenario: create place
  Given the user provides a name, address, and stratum
  When they submit the place registration
  Then the system creates the place linked to the user as owner

Scenario: missing required fields
  Given the user omits the address
  When they submit the form
  Then the system rejects with a validation error
```

| Field | Value |
|-------|-------|
| Story Points | 5 |
| Priority | Must Have |
| Service(s) | place-service |
| Dependencies | HU-002 |

---

### HU-008 — Edit place information

> **As** an Owner
> **I want** to edit my place's information
> **so that** I can keep the data accurate if something changes

```gherkin
Scenario: update place
  Given the user is the owner of the place
  When they update the name, address, or stratum
  Then the system saves the changes
```

| Field | Value |
|-------|-------|
| Story Points | 3 |
| Priority | Should Have |
| Service(s) | place-service |
| Dependencies | HU-007 |

---

### HU-009 — Delete a place

> **As** an Owner
> **I want** to delete a place
> **so that** I can remove locations I no longer manage

```gherkin
Scenario: delete place
  Given the user confirms the deletion
  When the request is processed
  Then the system soft-deletes the place
  And unlinks all associated devices
```

| Field | Value |
|-------|-------|
| Story Points | 3 |
| Priority | Should Have |
| Service(s) | place-service, device-service |
| Dependencies | HU-007 |

---

## E3 — IoT Device Linking

### HU-010 — Link a device to a place

> **As** an Owner
> **I want** to link my ESP32 device to a place using its serial number
> **so that** the platform starts receiving water consumption readings from that location

```gherkin
Scenario: successful linking
  Given the user enters a valid, unlinked serial number
  When they confirm the linking
  Then the system registers the device to the place
  And the device activates via MQTT and begins sending readings

Scenario: device already linked
  Given the serial number is already linked to another place
  When the user attempts to link it
  Then the system rejects with "device already linked to another place"
```

| Field | Value |
|-------|-------|
| Story Points | 8 |
| Priority | Must Have |
| Service(s) | device-service |
| Dependencies | HU-007 |

---

### HU-011 — View device status

> **As** an Owner
> **I want** to see the current status of my device
> **so that** I know if it is online, offline, or has issues

```gherkin
Scenario: device online
  Given the device has sent a reading in the last 5 minutes
  When the user views the device status
  Then the system shows "Online" with the last reading timestamp

Scenario: device offline
  Given no reading has been received in the last 5 minutes
  When the user views the device status
  Then the system shows "Offline" with the last known reading timestamp
```

| Field | Value |
|-------|-------|
| Story Points | 3 |
| Priority | Must Have |
| Service(s) | device-service |
| Dependencies | HU-010 |

---

### HU-012 — Unlink a device

> **As** an Owner
> **I want** to unlink a device from a place
> **so that** I can move it to another location or stop monitoring

```gherkin
Scenario: unlink device
  Given the device is linked to the user's place
  When the user confirms the unlink
  Then the system deactivates the device
  And readings stop being associated with the place
```

| Field | Value |
|-------|-------|
| Story Points | 3 |
| Priority | Should Have |
| Service(s) | device-service |
| Dependencies | HU-010 |

---

## E4 — Real-time Consumption Monitoring

### HU-013 — View current consumption (today)

> **As** an Owner
> **I want** to see how much water I have consumed today in real time
> **so that** I can make decisions about my usage throughout the day

```gherkin
Scenario: consumption today
  Given the device is linked and online
  When the user opens the dashboard
  Then the system shows total liters consumed today with auto-refresh

Scenario: no device linked
  Given the user has no linked device
  When the user opens the dashboard
  Then the system shows an empty state prompting to link a device
```

| Field | Value |
|-------|-------|
| Story Points | 8 |
| Priority | Must Have |
| Service(s) | consumption-service, device-service |
| Dependencies | HU-010 |

---

### HU-014 — View consumption history

> **As** an Owner
> **I want** to see my consumption history by day, week, and month
> **so that** I can identify patterns and trends over time

```gherkin
Scenario: historical view
  Given the user has readings for the selected period
  When they select a date range and granularity (day/week/month)
  Then the system shows a chart with consumption data for that period
```

| Field | Value |
|-------|-------|
| Story Points | 8 |
| Priority | Must Have |
| Service(s) | consumption-service |
| Dependencies | HU-013 |

---

### HU-015 — View current flow rate

> **As** an Owner
> **I want** to see the current flow rate in liters per minute
> **so that** I can tell if water is running right now

```gherkin
Scenario: active flow
  Given the device is reporting a non-zero flow rate
  When the user views the flow rate widget
  Then the system shows the current liters/minute from the latest reading
```

| Field | Value |
|-------|-------|
| Story Points | 5 |
| Priority | Should Have |
| Service(s) | consumption-service, device-service |
| Dependencies | HU-010 |

---

### HU-016 — Export consumption data

> **As** an Owner
> **I want** to download my consumption data as a CSV file
> **so that** I can analyze it externally or share it with someone

```gherkin
Scenario: export CSV
  Given the user selects a date range
  When they click "Export"
  Then the system generates a CSV with date, liters, and flow rate columns
```

| Field | Value |
|-------|-------|
| Story Points | 5 |
| Priority | Could Have |
| Service(s) | consumption-service |
| Dependencies | HU-014 |

---

### HU-017 — View consumption summary on home screen

> **As** an Owner
> **I want** to see a quick consumption summary when I open the app
> **so that** I get a snapshot without navigating to the full dashboard

```gherkin
Scenario: home screen summary
  Given the user has an active device
  When they open the app
  Then the system shows today's consumption, this week, and this month at a glance
```

| Field | Value |
|-------|-------|
| Story Points | 5 |
| Priority | Should Have |
| Service(s) | consumption-service |
| Dependencies | HU-013 |

---

# Entrega 2 — Smart Monitoring

---

## E5 — Manual Valve Control

### HU-026 — Close the valve remotely

> **As** an Owner
> **I want** to close the water valve from my phone
> **so that** I can shut off the water supply in an emergency or when leaving home

```gherkin
Scenario: close valve
  Given the device is online and the valve is open
  When the user sends a close command
  Then the system sends a MQTT command to the device
  And the valve closes and the status updates to "closed"

Scenario: device offline
  Given the device is offline
  When the user sends a close command
  Then the system shows "device unreachable" and logs a ValveCommandFailed event
```

| Field | Value |
|-------|-------|
| Story Points | 8 |
| Priority | Must Have |
| Service(s) | valve-service, device-service |
| Dependencies | HU-010 |

---

### HU-027 — Open the valve remotely

> **As** an Owner
> **I want** to open the water valve from my phone
> **so that** I can restore water supply after a shutoff

```gherkin
Scenario: open valve
  Given the device is online and the valve is closed
  When the user sends an open command
  Then the valve opens and the status updates to "open"
```

| Field | Value |
|-------|-------|
| Story Points | 8 |
| Priority | Must Have |
| Service(s) | valve-service, device-service |
| Dependencies | HU-026 |

---

### HU-028 — View valve status

> **As** an Owner
> **I want** to see whether my valve is currently open or closed
> **so that** I know the state of my water supply

```gherkin
Scenario: valve status
  Given the user has a device with a valve
  When they view the valve widget
  Then the system shows open/closed status, last action timestamp, and who triggered it
```

| Field | Value |
|-------|-------|
| Story Points | 3 |
| Priority | Must Have |
| Service(s) | valve-service |
| Dependencies | HU-026 |

---

### HU-029 — View valve action history

> **As** an Owner
> **I want** to see a log of all valve open/close actions
> **so that** I can audit who controlled the valve and when

```gherkin
Scenario: action history
  Given there have been valve actions
  When the user opens the valve history
  Then the system shows a list with action, timestamp, actor, and reason
```

| Field | Value |
|-------|-------|
| Story Points | 3 |
| Priority | Should Have |
| Service(s) | valve-service |
| Dependencies | HU-026 |

---

## E6 — Consumption Goals

### HU-018 — Set a monthly consumption goal

> **As** an Owner
> **I want** to set a monthly water consumption goal in liters
> **so that** I can track my usage against a target and save water

```gherkin
Scenario: set goal
  Given the user selects a place
  When they set a monthly goal (e.g., 15,000 liters)
  Then the system saves the goal for the current billing period
```

| Field | Value |
|-------|-------|
| Story Points | 5 |
| Priority | Must Have |
| Service(s) | consumption-service |
| Dependencies | HU-013 |

---

### HU-019 — View goal progress

> **As** an Owner
> **I want** to see how much of my monthly goal I have consumed
> **so that** I know if I need to reduce my usage

```gherkin
Scenario: goal progress
  Given the user has an active goal
  When they view the goal widget
  Then the system shows a progress bar with percentage consumed vs. goal
```

| Field | Value |
|-------|-------|
| Story Points | 5 |
| Priority | Must Have |
| Service(s) | consumption-service |
| Dependencies | HU-018 |

---

### HU-020 — Receive alert when approaching goal

> **As** an Owner
> **I want** to be notified when I reach a configurable percentage of my goal
> **so that** I can adjust my habits before exceeding it

```gherkin
Scenario: threshold alert
  Given the user has a goal with a threshold of 80%
  When consumption reaches 80% of the goal
  Then the system sends a GoalThresholdReached alert
```

| Field | Value |
|-------|-------|
| Story Points | 3 |
| Priority | Must Have |
| Service(s) | consumption-service, notification-service |
| Dependencies | HU-018 |

---

### HU-021 — Receive alert when goal is exceeded

> **As** an Owner
> **I want** to be notified when I exceed my monthly goal
> **so that** I am aware I have passed my target

```gherkin
Scenario: goal exceeded
  Given the user has an active goal
  When consumption exceeds 100% of the goal
  Then the system sends a GoalExceeded alert
```

| Field | Value |
|-------|-------|
| Story Points | 3 |
| Priority | Must Have |
| Service(s) | consumption-service, notification-service |
| Dependencies | HU-018 |

---

## E7 — Alerts & Notifications

### HU-022 — Receive leak detection alert

> **As** an Owner
> **I want** to be alerted when a possible water leak is detected
> **so that** I can act before the leak causes significant damage or waste

```gherkin
Scenario: leak suspected
  Given continuous flow is detected for more than 30 minutes during inactivity hours
  When the leak detector evaluates the readings
  Then the system creates a LeakSuspected alert and notifies the user

Scenario: normal continuous flow
  Given continuous flow during daytime active hours
  When the leak detector evaluates the readings
  Then no alert is created
```

| Field | Value |
|-------|-------|
| Story Points | 8 |
| Priority | Must Have |
| Service(s) | consumption-service, notification-service |
| Dependencies | HU-013 |

---

### HU-023 — View alert history

> **As** an Owner
> **I want** to see a list of all alerts I have received
> **so that** I can review past events and their resolution

```gherkin
Scenario: alert list
  Given the user has received alerts
  When they open the alert history
  Then the system shows a chronological list with type, timestamp, and status
```

| Field | Value |
|-------|-------|
| Story Points | 3 |
| Priority | Should Have |
| Service(s) | notification-service |
| Dependencies | HU-022 |

---

### HU-024 — Configure notification preferences

> **As** an Owner
> **I want** to choose how I receive notifications (push, email, WhatsApp)
> **so that** I am alerted through my preferred channel

```gherkin
Scenario: configure preferences
  Given the user opens notification settings
  When they toggle push/email/WhatsApp per alert type
  Then the system saves the preferences and applies them to future alerts
```

| Field | Value |
|-------|-------|
| Story Points | 5 |
| Priority | Should Have |
| Service(s) | notification-service |
| Dependencies | HU-022 |

---

### HU-025 — Mark alert as resolved

> **As** an Owner
> **I want** to mark an alert as resolved
> **so that** I can acknowledge I have addressed the issue

```gherkin
Scenario: resolve alert
  Given the user views an active alert
  When they mark it as resolved
  Then the system updates the alert status to "resolved"
```

| Field | Value |
|-------|-------|
| Story Points | 2 |
| Priority | Should Have |
| Service(s) | notification-service |
| Dependencies | HU-023 |

---

# Entrega 3 — Social & Billing

---

## E8 — Utility Bill Management

### HU-034 — Register a water bill

> **As** an Owner
> **I want** to register the data from my water bill
> **so that** I can compare what the utility charged vs. what my sensor measured

```gherkin
Scenario: register bill
  Given the user enters amount, billing period, liters billed, and provider
  When they submit the bill
  Then the system stores the bill record linked to the place
```

| Field | Value |
|-------|-------|
| Story Points | 5 |
| Priority | Must Have |
| Service(s) | consumption-service |
| Dependencies | HU-007 |

---

### HU-035 — Compare bill vs. sensor measurement

> **As** an Owner
> **I want** to see a side-by-side comparison of my bill and sensor data
> **so that** I can determine if I am being overcharged

```gherkin
Scenario: bill comparison
  Given the user has a bill and sensor data for the same period
  When they open the comparison view
  Then the system shows billed liters vs. sensor liters, with difference highlighted
```

| Field | Value |
|-------|-------|
| Story Points | 8 |
| Priority | Must Have |
| Service(s) | consumption-service |
| Dependencies | HU-034, HU-014 |

---

### HU-036 — View bill history

> **As** an Owner
> **I want** to see a list of all bills I have registered
> **so that** I can track my water expenses over time

```gherkin
Scenario: bill list
  Given the user has registered bills
  When they open the bill history
  Then the system shows a chronological list with amount, period, and comparison status
```

| Field | Value |
|-------|-------|
| Story Points | 3 |
| Priority | Should Have |
| Service(s) | consumption-service |
| Dependencies | HU-034 |

---

### HU-037 — Receive alert for bill discrepancy

> **As** an Owner
> **I want** to be alerted when there is a significant difference between my bill and sensor
> **so that** I can file a complaint with the utility company

```gherkin
Scenario: discrepancy alert
  Given the difference between billed and measured liters exceeds the threshold
  When the comparison is calculated
  Then the system creates a discrepancy alert and notifies the user
```

| Field | Value |
|-------|-------|
| Story Points | 5 |
| Priority | Should Have |
| Service(s) | consumption-service, notification-service |
| Dependencies | HU-035 |

---

## E9 — Tariffs & Measurement Units

### HU-038 — Configure water tariff for a place

> **As** an Owner
> **I want** to configure the water tariff (rate per m³) for my place
> **so that** the system can estimate my cost from sensor data

```gherkin
Scenario: set tariff
  Given the user enters rate per m³, stratum, and provider
  When they save the tariff
  Then the system stores it linked to the place
```

| Field | Value |
|-------|-------|
| Story Points | 5 |
| Priority | Should Have |
| Service(s) | consumption-service |
| Dependencies | HU-007 |

---

### HU-039 — View estimated cost from sensor data

> **As** an Owner
> **I want** to see an estimated cost for my current consumption
> **so that** I can predict what my next bill will be

```gherkin
Scenario: cost estimate
  Given the place has a configured tariff and consumption data
  When the user views the cost estimate widget
  Then the system shows: liters consumed × tariff rate = estimated cost
```

| Field | Value |
|-------|-------|
| Story Points | 5 |
| Priority | Should Have |
| Service(s) | consumption-service |
| Dependencies | HU-038 |

---

### HU-040 — Tariff updates apply to future estimates only

> **As** an Owner
> **I want** tariff changes to only affect future estimates
> **so that** historical cost calculations remain accurate

```gherkin
Scenario: tariff update
  Given the user updates the tariff
  When they save the new rate
  Then future estimates use the new rate
  And historical estimates keep the old rate
```

| Field | Value |
|-------|-------|
| Story Points | 2 |
| Priority | Could Have |
| Service(s) | consumption-service |
| Dependencies | HU-038 |

---

## E10 — Automatic Valve Shutoff

### HU-041 — Configure automatic shutoff rule

> **As** an Owner
> **I want** to define rules that automatically close the valve
> **so that** the system can shut off water when a leak is detected or a goal is exceeded

```gherkin
Scenario: create rule
  Given the user selects a trigger (e.g., LeakSuspected, GoalExceeded)
  When they enable the rule
  Then the system saves the shutoff rule for the place
```

| Field | Value |
|-------|-------|
| Story Points | 8 |
| Priority | Must Have |
| Service(s) | valve-service |
| Dependencies | HU-026, HU-022 |

---

### HU-042 — Automatic shutoff activates on rule trigger

> **As** an Owner
> **I want** the valve to close automatically when a rule is triggered
> **so that** water damage is minimized without manual intervention

```gherkin
Scenario: auto-shutoff triggered
  Given a shutoff rule is active for LeakSuspected
  When a LeakSuspected event is published
  Then the system closes the valve, sends an alert, and logs the action as "automatic"
```

| Field | Value |
|-------|-------|
| Story Points | 5 |
| Priority | Must Have |
| Service(s) | valve-service, notification-service |
| Dependencies | HU-041 |

---

### HU-043 — Disable automatic shutoff

> **As** an Owner
> **I want** to deactivate automatic shutoff rules
> **so that** I can return to manual-only valve control

```gherkin
Scenario: disable rule
  Given the user has an active shutoff rule
  When they disable it
  Then the rule is deactivated and the valve stays in its current state
```

| Field | Value |
|-------|-------|
| Story Points | 2 |
| Priority | Should Have |
| Service(s) | valve-service |
| Dependencies | HU-041 |

---

### HU-044 — Reopen valve after auto-shutoff (manual only)

> **As** an Owner
> **I want** the valve to never auto-reopen after an automatic shutoff
> **so that** I must consciously verify the situation before restoring water

```gherkin
Scenario: manual reopen after auto-shutoff
  Given the valve was closed by an automatic rule
  When the owner sends a manual open command
  Then the valve reopens and the action is logged as "manual override"

Scenario: no auto-reopen
  Given the valve was closed by an automatic rule
  Then the valve remains closed until a manual open command is issued
```

| Field | Value |
|-------|-------|
| Story Points | 3 |
| Priority | Must Have |
| Service(s) | valve-service |
| Dependencies | HU-042, HU-027 |

---

## E11 — Shared Users (Owner / Guest)

### HU-030 — Invite a member to a place

> **As** an Owner
> **I want** to invite someone to my place as a member
> **so that** family members or roommates can access consumption data

```gherkin
Scenario: invite member
  Given the owner provides a valid email
  When they send the invitation with selected permissions
  Then the system sends an email invitation and creates a pending invitation record

Scenario: member already invited
  Given the email already has a pending or active invitation for this place
  When the owner tries to invite again
  Then the system rejects with "user already invited"
```

| Field | Value |
|-------|-------|
| Story Points | 5 |
| Priority | Must Have |
| Service(s) | place-service, notification-service |
| Dependencies | HU-007 |

---

### HU-031 — Member views consumption dashboard

> **As** a Member
> **I want** to view the consumption dashboard for a place I was invited to
> **so that** I can see how much water is being used

```gherkin
Scenario: member dashboard
  Given the member has accepted the invitation
  When they open the dashboard for the place
  Then the system shows consumption data according to their owner-defined permissions
```

| Field | Value |
|-------|-------|
| Story Points | 3 |
| Priority | Must Have |
| Service(s) | consumption-service, place-service |
| Dependencies | HU-030, HU-013 |

---

### HU-032 — Revoke member access

> **As** an Owner
> **I want** to remove a member's access to my place
> **so that** they can no longer see my consumption data

```gherkin
Scenario: revoke access
  Given the member has active access to the place
  When the owner revokes access
  Then the system removes the member's access immediately
  And publishes a MemberAccessRevoked event
```

| Field | Value |
|-------|-------|
| Story Points | 3 |
| Priority | Should Have |
| Service(s) | place-service |
| Dependencies | HU-030 |

---

### HU-033 — Member receives alerts for the place

> **As** a Member
> **I want** to receive alerts for the place I have access to
> **so that** I am aware of leaks or anomalies even though I am not the owner

```gherkin
Scenario: member alert
  Given the member has active access and alerts are enabled by owner permissions
  When a LeakSuspected or GoalExceeded event occurs
  Then the member receives the alert through their preferred channel
```

| Field | Value |
|-------|-------|
| Story Points | 3 |
| Priority | Could Have |
| Service(s) | notification-service |
| Dependencies | HU-030, HU-022 |

---

## E12 — Shared Users (Owner / Collaborator)

> User stories for this epic will be defined during sprint planning for Entrega 3.
> This epic covers inviting collaborators with extended permissions (beyond read-only guest access) to a place.

---


# Entrega 4 — Administration & Analytics

---

## E13 — Downloadable Reports

### HU-049 — View monthly consumption report

> **As** an Owner
> **I want** to see a detailed monthly consumption report
> **so that** I can analyze my usage with charts and comparisons to previous months

```gherkin
Scenario: monthly report
  Given the user has at least 2 months of data
  When they open the monthly report
  Then the system shows consumption summary, charts, and month-over-month comparison
```

| Field | Value |
|-------|-------|
| Story Points | 5 |
| Priority | Should Have |
| Service(s) | consumption-service |
| Dependencies | HU-014 |

---

### HU-050 — View water savings report

> **As** an Owner
> **I want** to see a savings report showing how I compare to my goals over time
> **so that** I can track my conservation progress

```gherkin
Scenario: savings report
  Given the user has active goals and historical consumption
  When they open the savings report
  Then the system shows savings vs. goal trend over the last 6 months
```

| Field | Value |
|-------|-------|
| Story Points | 5 |
| Priority | Could Have |
| Service(s) | consumption-service |
| Dependencies | HU-018, HU-014 |

---

### HU-051 — Download report as PDF

> **As** an Owner
> **I want** to download my consumption report as a PDF
> **so that** I can share it or keep a physical record

```gherkin
Scenario: PDF export
  Given the user is viewing a monthly or savings report
  When they click "Download PDF"
  Then the system generates and downloads a formatted PDF
```

| Field | Value |
|-------|-------|
| Story Points | 5 |
| Priority | Could Have |
| Service(s) | consumption-service |
| Dependencies | HU-049 |

---

### HU-052 — View consumption ranking across places

> **As** an Owner with multiple places
> **I want** to see a ranking of consumption across all my places
> **so that** I know which location consumes the most

```gherkin
Scenario: consumption ranking
  Given the user has 2+ places with active devices
  When they open the ranking view
  Then the system shows places sorted by consumption (highest first)
```

| Field | Value |
|-------|-------|
| Story Points | 3 |
| Priority | Could Have |
| Service(s) | consumption-service, place-service |
| Dependencies | HU-053 |

---

## E14 — Savings Tips

> User stories for this epic will be defined during sprint planning for Entrega 4.
> This epic covers generic water-saving recommendations displayed to users, differentiated by place type (residential / commercial).

---


## E15 — Preloaded Tariffs by Country

> User stories for this epic will be defined during sprint planning for Entrega 4.
> This epic covers pre-loaded tariff catalogs for Colombia (by city and stratum), Ecuador (by city and service category), and the United States (by state and water district).

---


## E16 — Admin Panel

### HU-045 — Admin views all registered users

> **As** an Admin
> **I want** to see a list of all registered users
> **so that** I can monitor platform usage and manage accounts

```gherkin
Scenario: user list
  Given the admin is authenticated with admin role
  When they open the user management section
  Then the system shows a paginated list with name, email, status, and registration date
```

| Field | Value |
|-------|-------|
| Story Points | 3 |
| Priority | Must Have |
| Service(s) | auth-service |
| Dependencies | HU-002 |

---

### HU-046 — Admin blocks/unblocks a user

> **As** an Admin
> **I want** to block or unblock a user account
> **so that** I can handle abuse or policy violations

```gherkin
Scenario: block user
  Given the admin selects an active user
  When they block the account
  Then all user sessions are revoked and the account status becomes "blocked"
```

| Field | Value |
|-------|-------|
| Story Points | 3 |
| Priority | Must Have |
| Service(s) | auth-service |
| Dependencies | HU-045 |

---

### HU-047 — Admin views all devices

> **As** an Admin
> **I want** to see a list of all devices on the platform
> **so that** I can monitor device health and connectivity

```gherkin
Scenario: device list
  Given the admin opens the device management section
  When the page loads
  Then the system shows all devices with serial, status, linked place, and last reading
```

| Field | Value |
|-------|-------|
| Story Points | 3 |
| Priority | Should Have |
| Service(s) | device-service |
| Dependencies | HU-010 |

---

### HU-048 — Admin views platform metrics

> **As** an Admin
> **I want** to see platform-wide metrics
> **so that** I can understand adoption and system health

```gherkin
Scenario: metrics dashboard
  Given the admin opens the metrics section
  When the page loads
  Then the system shows: total users, total devices, total places, active/inactive counts
```

| Field | Value |
|-------|-------|
| Story Points | 5 |
| Priority | Should Have |
| Service(s) | auth-service, device-service, place-service |
| Dependencies | HU-045, HU-047 |

---

### HU-057 — Admin creates a recommendation

> **As** an Admin
> **I want** to create a generic water-saving recommendation
> **so that** all users can see tips and best practices in the app

```gherkin
Scenario: create recommendation
  Given the admin enters a title, body, and category
  When they submit the recommendation
  Then the system saves it and makes it visible to all users

Scenario: invalid recommendation
  Given the admin leaves the title or body empty
  When they try to submit
  Then the system rejects with validation errors
```

| Field | Value |
|-------|-------|
| Story Points | 5 |
| Priority | Should Have |
| Service(s) | notification-service |
| Dependencies | — |

---

### HU-058 — Admin edits or deactivates a recommendation

> **As** an Admin
> **I want** to edit or deactivate an existing recommendation
> **so that** outdated or incorrect tips are no longer shown to users

```gherkin
Scenario: edit recommendation
  Given the admin modifies the title, body, or category
  When they save the changes
  Then the system updates the recommendation

Scenario: deactivate recommendation
  Given the admin deactivates a recommendation
  When they confirm the action
  Then the recommendation is hidden from users but not deleted
```

| Field | Value |
|-------|-------|
| Story Points | 3 |
| Priority | Could Have |
| Service(s) | notification-service |
| Dependencies | HU-057 |

---

## Multi-place Extensions (extends E2 / E3)

### HU-053 — Register multiple places

> **As** an Owner
> **I want** to register more than one place
> **so that** I can monitor water consumption at all my locations

```gherkin
Scenario: second place
  Given the user already has one place
  When they register another place
  Then the system creates it independently with its own device slot
```

| Field | Value |
|-------|-------|
| Story Points | 5 |
| Priority | Must Have |
| Service(s) | place-service |
| Dependencies | HU-007 |

---

### HU-054 — Switch between places in the app

> **As** an Owner with multiple places
> **I want** to switch between places in the app
> **so that** I can view each location's data without logging out

```gherkin
Scenario: place switcher
  Given the user has 2+ places
  When they open the place selector
  Then the system shows all their places and switches context on selection
```

| Field | Value |
|-------|-------|
| Story Points | 3 |
| Priority | Must Have |
| Service(s) | place-service |
| Dependencies | HU-053 |

---

### HU-055 — View aggregated consumption across places

> **As** an Owner with multiple places
> **I want** to see total consumption across all my places
> **so that** I understand my overall water usage

```gherkin
Scenario: aggregated view
  Given the user has multiple places with active devices
  When they open the aggregated dashboard
  Then the system shows total consumption and a per-place breakdown
```

| Field | Value |
|-------|-------|
| Story Points | 5 |
| Priority | Should Have |
| Service(s) | consumption-service, place-service |
| Dependencies | HU-053 |

---

### HU-056 — Manage devices per place

> **As** an Owner with multiple places
> **I want** to manage devices independently for each place
> **so that** I can link, view, and unlink devices per location

```gherkin
Scenario: device per place
  Given the user has multiple places
  When they navigate to a specific place
  Then the system shows only the devices linked to that place
  And link/unlink actions apply only to the selected place
```

| Field | Value |
|-------|-------|
| Story Points | 3 |
| Priority | Should Have |
| Service(s) | device-service, place-service |
| Dependencies | HU-053, HU-010 |

---

## Rules for writing user stories

### 1. The role must be precise

```
✓ As an Owner
✓ As a Member
✓ As an Admin
✗ As a user
✗ As a person
```

### 2. The benefit must be business value

```
✓ so that I can detect leaks before they cost me money
✗ so that I can see the data (restates the feature)
```

### 3. Acceptance criteria must be verifiable

```
✓ Then the system shows total liters consumed today
✓ Then the valve closes and the status updates to "closed"
✗ Then the experience is good
```

### 4. One story ≈ one sprint (or less)

If a story covers too many flows, split it before sprint planning.

---

## Correlations

- Delivery roadmap → `03-product/roadmap.md`
- Non-functional requirements → `04-requirements/non-functional.md`
- Traceability matrix → `04-requirements/traceability-matrix.md`
- Domain events triggered by these stories → `02-domain/domain-events.md`
- API contracts derived from these stories → `07-api/contracts/`
- Database tables → `06-data/models.md`
