# ADR-022 — Valve Orders for an Offline Device: Rejected, Not Queued (Saga Pattern Discarded)

| Field | Value |
|-------|-------|
| **ID** | ADR-022 |
| **Date** | 2026-10-05 (decision, approximate) · 2026-10-28 (recorded) |
| **Status** | Accepted — **risk not resolved** (recorded at project close) |
| **Authors** | Juan Esteban Ome, Juan Manuel Gómez, Diego Alejandro Alvarado |
| **Reviewers** | Juan Esteban Ome, Juan Manuel Gómez, Diego Alejandro Alvarado |

> Recorded at project close after the question raised in the technical defense (sustentación): *"where does the
> close-valve order stay when the device is disconnected?"*. The jury suggested a Saga so the order is delivered when
> the device reconnects. This record states what the system does, the option considered and why it was not built.

---

## Context

A valve order crosses three services and one device: app → ms-valve (order, `VALVE_CLOSE` code, ADR-017) →
ms-devices (`POST /api/places/{id}/device/valve-commands`) → Mosquitto `sywater/devices/{serial}/valve/set` → ESP32 →
`valve/state` → `device.valve.reported` (RabbitMQ) → ms-valve confirms the order.

The device may be offline (power or Wi-Fi loss). The behaviour as built:

| Moment of the disconnection | What happens |
|-----------------------------|--------------|
| **Before** the order (no confirmed report within the communication window, or ms-devices status `DISCONNECTED`, ADR-015) | ms-valve refuses the order (`NoCommunication`) or ms-devices answers 409 `device.offline`. **Nothing is stored or queued**; the user sees the error and must retry later |
| **After** the order was published, before the device received it | The order is published with MQTT QoS 1, but the device connects with a clean session (`umqtt.simple` default) and the server also uses a clean session, so the broker **does not keep** the message for an absent client. ms-valve marks the order `ACK_TIMEOUT` after 30 s |
| The device answers after the 30 s | The reported state updates the valve state (the device is the source of truth), but the expired order is **not revived** (`ValveCommand.cs`) |

**Risk:** a user who needs to cut the water (for example, a burst pipe) while the meter is offline cannot leave the
order pending; the order is lost and must be repeated by hand when the meter is back.

---

## Evaluated alternatives

| Alternative | Pros | Cons | Result |
|------------|------|------|--------|
| **A — Reject when offline, 30 s timeout, no queue (as built)** | Simple; the UI never shows a false state; no stale order runs hours later | The intention of the user is lost; manual retry | **Chosen** |
| B — **Saga** (orchestrated by ms-valve): order state `PENDING_DELIVERY`; on `device.reconnected` / first heartbeat ms-devices re-sends; compensation (cancel, notify) if the order expires or the user cancels; idempotency by `commandId` | Order delivered on reconnection; explicit compensation; traceable | New states and events (`device.reconnected`, `valve.command.expired`), outbox in ms-valve and ms-devices, a policy for how long an order stays valid, UI for pending orders, tests of every path | **Discarded for time**: identified late, at the end of the schedule; it touches ms-valve, ms-devices, the database (new statuses in `valve_commands`), the firmware and the app |
| C — MQTT persistent session (`clean_session=False`) + QoS 1 so Mosquitto queues the message | Few changes; the broker delivers on reconnection | Delivery without business control: an order queued for hours would run without the user knowing; no expiry or cancellation; needs firmware and broker changes | Discarded: it hides the problem instead of modelling it |
| D — Retained message on `valve/set` | Device reads the last order on reconnection | Same as C, plus the retained order would be re-applied on every reconnection | Discarded |

## Decision

**We kept alternative A.** An order is valid only while the device is online and is confirmed or expired in 30 s.
The Saga (B) is the recommended design if the project is reopened; it was **not implemented because of time**.

## Consequences

- The **risk stays open** at close: no pending orders for offline devices.
- The user is informed: the app shows "Sin comunicación" before sending, and an expired order stays in the history as
  `ACK_TIMEOUT`, so the state shown is never false.
- Recorded in `15-project-control/closure/out-of-scope-register.md` (E-25).

## Decisions not taken

- Saga with compensation (B) — time.
- Broker-side queueing (C, D) — unsafe without expiry and user control.

## References

- ADR-006 (events and internal HTTP), ADR-009 (MQTT), ADR-015 (offline after 1 minute), ADR-017 (valve safety)
- `09-microservices/services/05-ms-valve/README.md`
