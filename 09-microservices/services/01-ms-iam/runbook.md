# Runbook — ms-iam

## 1. Quick information

| Item | Value |
|------|-------|
| Port | 3001 |
| Health | `GET /actuator/health` → `{"status":"UP"}` |
| Dependencies | SQL Server (`sy_water_db_dev`), Redis (`sy_water_redis`), SMTP (Mailpit `sy_water_mailpit` in development) |
| Logs | Console of `mvnw spring-boot:run` |

## 2. Verify health

```powershell
curl http://localhost:3001/actuator/health
docker ps --filter name=sy_water            # db, redis, mailpit must be "Up"
docker exec sy_water_redis redis-cli ping   # PONG
```

## 3. Frequent problems

| Symptom | Cause | Action |
|---------|-------|--------|
| Startup fails with `FileNotFoundException: keys\private.pem` | Started from another working directory, or keys not generated | Run from the `ms-iam` folder (IntelliJ: working directory = module) or generate the keys (§4) |
| Every service answers 401 | Redis down (fail closed) or a service has an old `iam-public.pem` | `docker start sy_water_redis`; copy `public.pem` again and restart the services |
| Users do not receive codes | SMTP misconfigured | Development: open http://localhost:8025. Real SMTP: check `SMTP_*` variables |
| Many locked accounts | Brute force or a client bug | Query `login_attempts` by IP; lockouts expire alone after 15 min |

## 4. Generate or rotate the key pair

```powershell
cd sy-water-backend\ms-iam
$openssl = "C:\Program Files\Git\usr\bin\openssl.exe"
& $openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -out keys/private.pem
& $openssl pkey -in keys/private.pem -pubout -out keys/public.pem
# Copy keys/public.pem as keys/iam-public.pem to ms-places, ms-device, ms-consumption and ms-valve, then restart them.
```

All sessions become invalid after a rotation. Never commit `keys/private.pem`.

## 5. Administration

```powershell
# Grant / revoke ADMIN (the user must log in again afterwards)
Get-Content scripts\grant-admin.sql | docker exec -i sy_water_db_dev /opt/mssql-tools18/bin/sqlcmd -S localhost -U sa -P "********" -C -b -d sy-water-db -v ADMIN_EMAIL="person@example.com"
```

## 6. Post-incident

- If a private key may have leaked: rotate it (§4) and review `activity_log` and `login_attempts`.
