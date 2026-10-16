# Runbook — ms-consumption

## 1. Quick information

| Item | Value |
|------|-------|
| Port | 3004 |
| Health | `GET /health` (database) |
| Dependencies | SQL Server, Redis, RabbitMQ, ms-places, public key of ms-iam |

## 2. Frequent problems

| Symptom | Cause | Action |
|---------|-------|--------|
| The chart stays empty while the meter sends data | Empty `RabbitMq:Host` or wrong password; ms-devices publishing only to the log | Check `RabbitMq:*` in both services and the `consumption.readings` queue in `http://localhost:15672` |
| Every request answers 401 | Old `iam-public.pem` or Redis down | Copy the key again; `docker start sy_water_redis` |
| 400 `consumption.invalid_time_zone` | `tz` is not an IANA name | Use e.g. `America/Bogota`, `America/Guayaquil` |
| Messages in `sywater.dead-letters` | Reading with impossible values | Inspect the payload; check the firmware calibration |
| Current flow is empty | Last reading older than 2 min | Expected: the device is silent or offline |

## 3. Maintenance

- The consumer reconnects to RabbitMQ every 5 s; restarting the service is safe (messages stay in the queue).
- Rollback: run the previous build; schema changes go through a new Liquibase release.
