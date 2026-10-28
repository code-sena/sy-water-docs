# ADR-009 — MQTT with Mosquitto for Devices

| Field | Value |
|-------|-------|
| **ID** | ADR-009 |
| **Date** | 2026-09-22 (decision, approximate) · 2026-10-28 (recorded) |
| **Status** | Accepted |
| **Authors** | Juan Esteban Ome, Juan Manuel Gómez, Diego Alejandro Alvarado |
| **Reviewers** | Juan Esteban Ome, Juan Manuel Gómez, Diego Alejandro Alvarado |

> Recorded retroactively at project close. The decision was taken and implemented during the project; this record
> describes it as built in the `main` branches on 2026-10-28. Where the alternatives were not written down at the
> time, this is stated instead of reconstructing them.

---

## Context

The meter (ESP32 + YF-S201 flow sensor + solenoid valve) sends a reading every 10 s and must receive close/open
orders. It runs MicroPython with little memory and a Wi-Fi link that drops. Devices are pre-registered by an administrator
(factory registration, risk R-004) and linked to a place by a user with a link code.

## Evaluated alternatives

| Alternative | Pros | Cons | Reason for discarding |
|------------|------|------|-----------------------|
| **A — MQTT broker (Mosquitto) (chosen)** | Light client (`umqtt`); server → device push; retained/QoS; standard | One more container | — (chosen) |
| B — HTTP polling from the device | No broker | Valve orders delayed by the poll interval; more traffic | Valve commands need push |
| C — WebSocket from the device | Push | Heavier client in MicroPython | Library maturity |
| D — Managed IoT cloud (AWS IoT, Azure IoT Hub) | TLS and identity built in | Cost and vendor lock; outside the VPS budget | Budget (ADR-013) |

## Decision

**We decided:**

- Mosquitto as broker. Topics: device → server `sywater/devices/{serial}/heartbeat` (30 s), `.../telemetry` (10 s),
  `.../valve/state` (60 s and after each order); server → device `.../valve/set`.
- **One shared broker user for devices** plus an ACL by serial pattern; the device proves its identity with a
  per-device token whose **hash** is stored in `devices.devices` and checked by ms-devices on each message.
- ms-devices is the only service connected to MQTT; it translates MQTT to RabbitMQ events (ADR-006).
- Telemetry carries a device timestamp; ms-devices rejects readings more than 5 min ahead of server time or older than
  24 h. The firmware buffers up to 360 readings (1 h) while offline and resynchronizes NTP every 6 h.

## Decisions not taken

- **TLS on the broker was not enabled**: Mosquitto listens on 1883 in plain text. Reasons: certificate handling on the
  ESP32 with MicroPython (memory, CA bundle, renewal) did not fit the schedule, and the device token gives
  application-level identity. Accepted risk: traffic between meter and VPS can be read on the network (risk R-006).
- No per-device broker credentials (one broker user per meter); rejected for operational simplicity with factory
  provisioning.

## Consequences

- Positive: valve orders reach the device in under a second when it is online; offline gaps are back-filled.
- Negative: plain MQTT; a leaked shared broker password allows connecting (but not impersonating a serial without its
  token).

## References

- `09-microservices/services/03-ms-devices/README.md`
- `09-microservices/services/03-ms-devices/decisions.md` DEC-001, DEC-002
- `15-project-control/risks.md` R-006
