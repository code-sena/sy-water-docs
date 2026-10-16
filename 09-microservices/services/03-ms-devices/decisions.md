# Design Decisions — ms-devices

## [DEC-001] Only hashes of device secrets
The factory token and the pairing code are stored as SHA-256 hashes (`auth_token_hash`,
`pairing_code_hash`). A database leak does not let anyone impersonate or claim a meter.

## [DEC-002] One shared MQTT user for the devices, identity by token
All ESP32 log in as `sywater-device`; the identity is the client id (serial) plus the token inside
each payload, checked against the hash. The ACL pattern `sywater/devices/%c/valve/set` keeps every
device reading only its own orders.

## [DEC-003] The device must be online to be linked
`Devices:RequireOnlineToLink = true`: the user proves physical possession (pairing code) and the
device proves it is real (authenticated heartbeat) before the link exists.

## [DEC-004] Two brute-force limits
Per device: 5 wrong codes lock pairing for 15 min (`device.pairing_locked`). Per user: 10 link
attempts per 15 min whatever the serial (`429` + `Retry-After`).

## [DEC-005] Status persisted by a sweeper
A background job writes `DISCONNECTED` every 60 s for devices silent longer than their threshold,
so other readers (reports, metrics) see an honest `status` column.

## [DEC-006] Events instead of direct calls to consumers
ms-devices publishes `device.*` events and does not know who consumes them. Adding ms-notifications
did not change this service.

## [DEC-007] Valve orders only through ms-valve
`valve-commands` needs the user's JWT **and** `X-Internal-Key`; the app cannot send raw orders.
