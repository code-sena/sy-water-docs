# ADR-015 — Device Liveness: Offline after One Minute

| Field | Value |
|-------|-------|
| **ID** | ADR-015 |
| **Date** | 2026-10-23 (decision, approximate) · 2026-10-28 (recorded) |
| **Status** | Accepted (supersedes the 10-minute default of HU-013) |
| **Authors** | Juan Esteban Ome, Juan Manuel Gómez, Diego Alejandro Alvarado |
| **Reviewers** | Juan Esteban Ome, Juan Manuel Gómez, Diego Alejandro Alvarado |

> Recorded retroactively at project close. The decision was taken and implemented during the project; this record
> describes it as built in the `main` branches on 2026-10-28. Where the alternatives were not written down at the
> time, this is stated instead of reconstructing them.

---

## Context

HU-013 and the first release of `ms-device-db` used an inactivity threshold of 10 minutes, with a status sweep every
60 s. With those values a meter that lost power could take up to about 11 minutes to show "offline" in the app and in
the notifications, which was too late for the user and for the valve safety rule (ADR-017).

## Evaluated alternatives

| Alternative | Pros | Cons | Reason for discarding |
|------------|------|------|-----------------------|
| **A — 1 minute threshold, sweep every 15 s (chosen)** | Offline visible in ≤ 75 s; heartbeat (30 s) and telemetry (10 s) both prove life, so a live meter never misses the window | A Wi-Fi hiccup of > 1 min raises an offline/online pair of notifications | — (chosen) |
| B — Keep 10 minutes | Fewer false alarms | State too late to act on | Rejected after testing |
| C — 2 minutes | Fewer false alarms than A | Still slow for the valve rule | Considered and rejected |
| D — MQTT Last Will and Testament | Instant on clean disconnects | Does not fire on power loss until keep-alive expires; needs firmware change | Not evaluated at the time; listed for completeness |

## Decision

**We decided:** default `inactivity_threshold_minutes = 1` (`ms-device-db` v1.2 changes the default and updates
existing devices), `DefaultInactivityThresholdMinutes = 1` and `StatusSweepSeconds = 15` in ms-devices,
`OfflineAfterMinutes = 1` with a 15 s worker in ms-notifications. The firmware keeps its heartbeat every 30 s and a
reading every 10 s; ms-devices records both as proof of life (`RecordHeartbeat` is called for telemetry too).

> Note: the firmware `config.py` comment still says "Desconectado after 10 min"; it is a stale comment in a frozen
> repository, the behaviour is defined by the server.

## Consequences

- Positive: the app and the alerts reflect a disconnected meter in about a minute.
- Negative: short network drops produce offline/online notification pairs.

## References

- `09-microservices/services/03-ms-devices/README.md`
- HU-013 in `04-requirements/user-stories.md` (default changed from 10 to 1 minute)
- `09-microservices/services/03-ms-devices/decisions.md` DEC-005
