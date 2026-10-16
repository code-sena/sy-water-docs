# Design Decisions — ms-consumption

## [DEC-001] Reading and hourly total in one transaction
The reading insert and the hourly increment (`ExecuteUpdate`) share a transaction, so the chart
never shows liters that are not in `readings` (or the opposite).

## [DEC-002] Only an hourly aggregate
Day, week and month are computed from `consumption_hourly` in the user's time zone. Daily or
monthly tables would be wrong for users outside UTC-5 and are not needed with the current volume.

## [DEC-003] Store liters, convert in the client
Every quantity is stored in liters; the display unit of the place (HU-055) is applied by the app.

## [DEC-004] Time zone as a parameter
`tz` is an IANA zone (default `America/Bogota`, ADR-004) so Ecuador and travelling users get their own day boundaries.

## [DEC-005] Same 404 for "not found" and "not yours"
The service never reveals whether another user's place exists.
