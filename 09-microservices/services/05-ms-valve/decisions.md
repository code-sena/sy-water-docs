# Design Decisions — ms-valve

## [DEC-001] The device is the source of truth
Sending an order never changes `valve_states`; only `device.valve.reported` does. The app shows
"in progress" until the device confirms or the order times out.

## [DEC-002] 202 Accepted and polling
`POST …/close|open` answers 202 with the order; the app polls `GET …/commands/{id}`. The query applies
the 30 s timeout immediately, without waiting for the sweeper.

## [DEC-003] Cheap checks before spending the code
Ownership, existing valve, pending order, current state and device communication are checked before
asking ms-iam to consume the e-mailed code, so a request that would fail anyway does not burn it.

## [DEC-004] One pending order per place in the database
The filtered unique index `UX_vcmd_place_pending` makes two simultaneous orders impossible even with
two app instances.

## [DEC-005] Name stored with the order
`requested_by_name` keeps the history readable without a call per line to ms-iam, and after the
user changes their name.

## [DEC-006] Orders only through ms-devices
ms-valve never talks MQTT; it calls ms-devices with `X-Internal-Key`, which owns the broker connection.
