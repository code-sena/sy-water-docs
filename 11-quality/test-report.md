# Test Report

> Results of the automated test suites run on **2026-10-10** against a copy of `sy-water-backend`,
> `sy-water-frontend/save-your-water` and `sy-water-hardware/firmware`.
> Full evidence (per-test list, coverage files, firmware log): `assets/deliverables/Pruebas_CuidaTuAgua_v1.0.pdf`.

---

## Summary

| Suite | Tool | Tests | Passed | Failed |
|-------|------|-------|--------|--------|
| ms-iam | JUnit Jupiter 6.0.3 (Spring Boot 4.1.1) | 121 | 121 | 0 |
| .NET services (5) | xUnit 2.9.3 + coverlet 6.0.4 | 339 | 339 | 0 |
| App (Expo) | Jest 29.7 + jest-expo 57 | 138 (23 suites) | 138 | 0 |
| Firmware | PC harness against a real Mosquitto broker | 28 checks | 28 | 0 |
| **Total** | | **626** | **626** | **0** |

## Backend by service and layer

| Service | Domain | Application | Infrastructure | Api | Total |
|---------|--------|-------------|----------------|-----|-------|
| ms-iam (Java) | — | — | — | — | 121 (16 classes in application, domain, infrastructure and presentation) |
| ms-places | 24 | 86 | — | — | 110 |
| ms-devices | 35 | 58 | — | — | 93 |
| ms-consumption | 14 | 5 | — | 6 | 25 |
| ms-valve | 5 | 12 | — | 4 | 21 |
| ms-notifications | 38 | 52 | not run | not run | 90 |

**Not run:** the 42 tests of `SyWater.Notifications.Infrastructure.Tests` and `SyWater.Notifications.Api.Tests`
could not be restored offline (MailKit was missing from the local NuGet cache). Run them with network access.

## Line coverage (.NET, coverlet)

| Service | Domain | Application | Infrastructure | Api |
|---------|--------|-------------|----------------|-----|
| ms-places | 73.6 % | 94.1 % | — | — |
| ms-devices | 91.6 % | 90.5 % | — | — |
| ms-consumption | 97.6 % | 100 % | 70.2 % | 19.3 % |
| ms-valve | 83.6 % | 95.3 % | 59.6 % | 24.1 % |
| ms-notifications | 76.8 % | 95.3 % | — | — |

App (Jest, whole project): lines 73.94 %, statements 71.12 %, branches 59.55 %, functions 55.98 %.

## Firmware checks (harness scenarios)

| Scenario | What it proves |
|----------|----------------|
| `setup` | Setup portal: form with scanned networks, `/api/info`, password validation, AP named after the serial with WPA2, restart after saving |
| `bad_wifi` | A network that never connects sends the device back to setup and forgets the credentials |
| `run` | Heartbeat, telemetry in liters, valve report, CLOSE order obeyed and confirmed with the same `commandId`, duplicate order done once, foreign orders ignored, offline buffering and ordered flush, reconnect after a broker restart |
| `restart` | After a power cut the valve comes back in its saved state; BOOT held 5 s forgets the WiFi |
| `away` | BOOT held while the saved WiFi is missing still resets |
| `no_ntp` | No readings until the clock is synced; liters kept and split into valid readings afterwards |

## How to run

```powershell
cd sy-water-backend\ms-iam;   .\mvnw test
cd sy-water-backend\ms-places; dotnet test --collect:"XPlat Code Coverage"   # same for the other .NET services
cd sy-water-frontend\save-your-water; npx jest --ci --coverage
```

The firmware harness (MicroPython stubs + `run_test.py` with `SCENARIO=<setup|bad_wifi|run|restart|away|no_ntp>`) was run outside the repository against a local Mosquitto; it is not committed in `sy-water-hardware` yet, and it must not be committed with its development credentials.

## Gaps

- No automated end-to-end or contract tests between services; 20 manual system cases (CP-01 to CP-20) are pending execution.
- Api layers have low coverage (19–24 %); ms-places and ms-devices have no Infrastructure/Api test projects.
- The firmware harness is not versioned.
- Tracked as risk R-009 in `15-project-control/risks.md`.
