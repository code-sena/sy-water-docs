# Runbook — ms-devices

## 1. Quick information

| Item | Value |
|------|-------|
| Port | 3003 |
| Health | `GET /health` — checks `database` and `mqtt` |
| Dependencies | SQL Server, Mosquitto, RabbitMQ, ms-places, public key of ms-iam |
| Broker UIs | RabbitMQ `http://localhost:15672` · Mosquitto logs `docker logs sy_water_mosquitto` |

## 2. Frequent problems

| Symptom | Cause | Action |
|---------|-------|--------|
| `/health` unhealthy on `mqtt` | Mosquitto down or wrong `Mqtt:Password` | `docker compose up -d mosquitto`; check user-secrets and `passwd` |
| Mosquitto rejects users after editing `passwd` | It reads `passwd` only at startup | `docker compose restart mosquitto` |
| Linking answers 409 `device.offline` | No authenticated heartbeat yet | Power the device / run `simulate-device.ps1`; check the device token and `MQTT_HOST` (LAN IP, never `localhost`) |
| Linking answers 429 `device.pairing_locked` | 5 wrong codes on that device | Wait 15 min |
| Linking answers 429 with `Retry-After` | 10 attempts in 15 min by the user | Wait for `Retry-After` |
| Readings do not reach ms-consumption | `RabbitMq:Host` empty (log-only publisher) or wrong password | Set `RabbitMq:*`; check the `consumption.readings` queue |
| Messages in `sywater.dead-letters` | A consumer rejected a malformed event | Inspect the message in RabbitMQ UI |
| A logged-out token still works here | ms-devices does not check the Redis denylist (R-005) | Tokens expire in 1 h; fix pending |

## 3. Maintenance

- New test devices: release in `ms-devices-db` with context `dev`; production must use `--contexts=prod`.
- Rollback: stop the service, run the previous build; the database is forward-only (fix with a new release).
- MQTT runs without TLS on 1883 in development; production must use TLS on 8883.
