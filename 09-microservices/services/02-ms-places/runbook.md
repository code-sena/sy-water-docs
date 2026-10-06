# Runbook — ms-places

## 1. Quick information

| Item | Value |
|------|-------|
| Port | 3002 |
| Health | `GET /health` (anonymous) |
| Dependencies | SQL Server, Redis, ms-devices (only for deletion), public key of ms-iam |

## 2. Frequent problems

| Symptom | Cause | Action |
|---------|-------|--------|
| Every request answers 401 | Old `iam-public.pem` or Redis down | Copy `ms-iam/keys/public.pem` again; `docker start sy_water_redis` |
| `Missing Redis:Configuration` at startup | Section missing in `appsettings.json` | Add `"Redis": { "Configuration": "localhost:6379" }` |
| Deleting a place answers 503 | ms-devices is down or `Services:DevicesBaseUrl` is wrong | Start ms-devices or fix the URL |
| `Invalid object name 'places.activity_log'` | Release v1.1 not applied | `docker compose up` in `ms-places-db` |
| `Cannot insert duplicate key row … UX_places_owner_default` | Wrong order when changing the selected place | Clear the old default before setting the new one |

## 3. Maintenance

- Adding cities or a country: new release in `ms-places-db/06-releases/vX.Y` with seeds; never edit applied changesets.
