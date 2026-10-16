# Runbook — ms-valve

## 1. Quick information

| Item | Value |
|------|-------|
| Port | 3005 |
| Health | `GET /health` (database) |
| Dependencies | SQL Server, Redis, RabbitMQ, ms-iam, ms-places, ms-devices, public key of ms-iam |

## 2. Frequent problems

| Symptom | Cause | Action |
|---------|-------|--------|
| The place has no valve (404 `valve.not_found`) after linking | ms-valve missed `device.linked` (RabbitMQ was not configured when the meter was linked) | Configure `RabbitMq:*`; unlink and link the meter again |
| Every order ends `ACK_TIMEOUT` | The device is offline or did not receive `valve/set` | Check the device LED and Mosquitto logs; check the ACL pattern for `valve/set` |
| 409 `valve.device_offline` when closing | No valve report for 3 min | Power/connect the device; wait for its next report (60 s) |
| 409 `valve.command_in_progress` | Another order is `SENT` | Wait up to 30 s |
| 503 `service.unavailable` | ms-iam, ms-places or ms-devices down, or wrong `Internal:ApiKey` | Start the services; use the same key in ms-devices and ms-valve |
| State stays `UNKNOWN` | The device never reported its valve | Expected until the first report |

## 3. Maintenance

- Orders are never deleted; the history query defaults to the last 30 days.
- Rollback: run the previous build; the database is forward-only.
