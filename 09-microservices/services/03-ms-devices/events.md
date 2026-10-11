# Events — ms-devices

## Events this service PUBLISHES

Exchange `sywater.events` (topic, durable), routing key = event type, publisher confirms on.
Payloads are defined in `SyWater.Devices.Application/Events/IntegrationEvents.cs`.

| Event | When | Payload |
|-------|------|---------|
| `device.linked` | A user links a meter (HU-012) | `deviceId`, `serialNumber`, `placeId`, `userId`, `linkedAt` |
| `device.unlinked` | A user unlinks a meter (HU-014) or the account is deleted (HU-008) | `deviceId`, `serialNumber`, `placeId`, `userId`, `unlinkedAt` |
| `device.reading.received` | A valid telemetry message arrives from a linked meter | `deviceId`, `placeId`, `recordedAt`, `flowLpm`, `volumeLiters`, `totalLiters`, `receivedAt` |
| `device.valve.reported` | The device reports its valve (every 60 s and after each order) | `deviceId`, `placeId`, `state` (`OPEN`/`CLOSED`), `commandId` (nullable), `reportedAt` |

Consumers: ms-consumption (`consumption.readings`), ms-valve (`valve.device-events`), ms-notifications
(`notification.device-events`). Messages that cannot be processed go to `sywater.dlx` → `sywater.dead-letters`.

## Events this service CONSUMES

None from RabbitMQ.

## MQTT contract (device ↔ ms-devices)

Client id = serial, QoS 1, keepalive 60 s, payload ≤ 512 bytes. Every device message carries the factory token.

| Direction | Topic | Payload | Frequency |
|-----------|-------|---------|-----------|
| device → | `sywater/devices/{serial}/heartbeat` | `{"token","fw"}` | 30 s |
| device → | `sywater/devices/{serial}/telemetry` | `{"token","ts","lpm","vol","total"}` | 10 s, buffered while offline |
| device → | `sywater/devices/{serial}/valve/state` | `{"token","state","commandId"}` | 60 s and after each order |
| ms-devices → | `sywater/devices/{serial}/valve/set` | `{"commandId","action":"OPEN"\|"CLOSE"}` | On demand |

Telemetry limits: flow ≤ 60 L/min, volume ≤ 100 L per reading, timestamp at most 5 min ahead of the server clock.
The Mosquitto ACL lets each device read only its own `valve/set` topic.

## Synchronous integration

| Direction | Endpoint | Purpose |
|-----------|----------|---------|
| ms-devices → ms-places | `GET /api/places/{id}` (user's JWT) | Ownership check before link/unlink/read |
| ms-places → ms-devices | `GET /api/places/{id}/device` | Refuse deleting a place with a linked meter |
| ms-valve → ms-devices | `POST /api/places/{id}/device/valve-commands` + `X-Internal-Key` | Send the order to the device |
| ms-iam → ms-devices | `DELETE /internal/users/{id}/devices`, `GET /internal/metrics` | Account deletion, admin metrics |
