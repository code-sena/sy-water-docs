# Normalization Assessment — Save Your Water

**Date:** 2026-09-22  
**Database:** SQL Server (`sy_water_db`)  
**Schema approach:** Schema-per-BC (all BCs in one database, shared FK constraints)

---

## Normalization Level

**3NF with 4 documented denormalizations (D1–D4).**

All tables adhere to 3rd Normal Form by default. Denormalized columns are explicitly listed below with their trade-off rationale.

---

## 1NF Compliance

✓ All tables use atomic, single-valued attributes.

- No multi-valued columns (e.g., comma-separated lists, JSON arrays stored as strings).
- Example: `device_id` and `place_id` are separate scalar columns, not a composite structure.
- Exception: `metadata` and similar JSON columns are stored as NVARCHAR with `ISJSON()` validation to allow structured data without violating 1NF at the logical level.

---

## 2NF Compliance

✓ No non-key attributes depend on a subset of the candidate key.

All non-key columns are *fully functionally dependent* on the entire primary key:

- Example (`monitoring.readings`): `flow_rate_lph` and `cumulative_liters` depend on the entire PK `(device_id, recorded_at)`, not just `device_id` alone.
- Example (`valve.valve_commands`): `command`, `status`, and lifecycle timestamps depend on the full PK, not a partial one.

---

## 3NF Compliance

✓ No transitive dependencies.

Removed via:

- **Separate tables for dimensions:** `alert_type_config` holds alert metadata (severity, description) indexed by `alert_type` to avoid storing this repeatedly in `alert_events`.
- **Foreign keys across tables:** `device_id` in `monitoring.readings` is a FK, not a denormalized copy of device properties.
- **User credentials separated:** `user_credentials` is a separate table (avoiding storing password hashes and OAuth IDs alongside user profile data).

---

## Intentional Denormalizations (3NF Violations)

The following 4 denormalizations are deliberate trade-offs between query performance and data redundancy. Each is justified and monitored.

### D1: `place_id` in `monitoring.readings`

**Table:** `monitoring.readings`  
**Column:** `place_id`  
**Reason:** Avoid JOIN to `devices` on every reading query.

**Justification:**  
Reading ingest is the highest write-volume path in the system. For every raw sensor reading, the application needs to quickly associate it with a place (for consumption aggregation, alert triggers, billing). Rather than joining through `devices` (`device_id` → `devices.device_id` → `devices.place_id`), we store `place_id` directly.

**Integrity:** This is redundant but safe — any insert into `readings` references an existing `device_id` which has a fixed `place_id`. A check constraint or application-level guard ensures consistency if a device is moved to a different place (update both `devices.place_id` and future readings).

**Revert if:** Reading volume drops below ~10k/hour or query patterns shift to favor device-centric aggregation.

---

### D2: `place_id` in `valve.valve_commands`

**Table:** `valve.valve_commands`  
**Column:** `place_id`  
**Reason:** Command belongs to the place, not just the device.

**Justification:**  
Valve commands are issued at the place level (user closes the water supply to their home). While `device_id` identifies the physical device, the command is a place-scoped operation. Storing `place_id` enables fast indexing by place (`IX_vcmd_place_time`) to list all commands for a place without a JOIN to `devices`.

**Integrity:** Same as D1 — `device_id` always has a fixed `place_id`.

---

### D3: `severity` in `alerts.alert_events`

**Table:** `alerts.alert_events`  
**Column:** `severity`  
**Reason:** Snapshot of classification at alert creation time.

**Justification:**  
An alert's severity (CRITICAL, IMPORTANT, INFORMATIVE) is determined by `alert_type` (looked up in `alerts.alert_type_config`). However, alert classifications can change over time — e.g., a GOAL_EXCEEDED alert is today IMPORTANT, but policy might change it to CRITICAL next quarter.

By storing the severity snapshot at the time of alert creation, we preserve the exact classification the user saw, even if the config changes later. This is essential for audit trails, historical reporting, and avoiding confusion ("why did my notification history show this alert as CRITICAL when the config says IMPORTANT?").

**Integrity:** On alert creation, `severity` is read from the current config and stored as a snapshot. No subsequent updates to `alert_type_config.severity` change existing alert records.

---

### D4: `owner_id` in `places.places`

**Table:** `places.places`  
**Column:** `owner_id` (denormalized from `place_members`)  
**Reason:** List "my places" without JOIN to `place_members`.

**Justification:**  
The primary owner is redundantly stored in two places:
1. In the `place_members` table as the row with `role = 'OWNER'`.
2. Directly in `places.owner_id`.

This enables fast queries like `SELECT * FROM places WHERE owner_id = ?` to fetch all places owned by a user, without joining to `place_members`. This is a common operation (user login → list my places → render dashboard).

The alternative (pure 3NF) would require `SELECT places.* FROM places JOIN place_members USING(place_id) WHERE place_members.user_id = ? AND place_members.role = 'OWNER'`.

**Integrity:** On place creation, `owner_id` is set to the creating user. The same user is immediately inserted into `place_members` with `role = 'OWNER'`. Application logic ensures the two stay in sync (e.g., ownership transfer requires updating both rows).

**Monitoring:** If consistency is ever questioned, a query to find places where `owner_id` ≠ the user with `role = 'OWNER'` in `place_members` will detect drift.

---

## Trade-offs Summary

| Code | Denormalization | Write cost | Read gain | Monitor with |
|------|-----------------|-----------|-----------|-------------|
| D1 | `place_id` in readings | Low (one more column) | High (eliminates device JOIN) | Volume of reading ingest; query plan on consumption aggregations |
| D2 | `place_id` in commands | Low | High (enables place-scoped index) | Volume of command queries; place timeline reports |
| D3 | `severity` snapshot | Low | High (history accuracy; no retroactive changes) | Audit log matches; alert classification consistency over time |
| D4 | `owner_id` in places | Low | High (user dashboard query performance) | Consistency check: `owner_id` vs. `place_members` role = OWNER |

---

## Storage Efficiency

Using schema-per-BC with cross-schema FKs:

- **Pro:** Logical separation of concerns; easier to understand BC boundaries.
- **Pro:** SQL Server enforces referential integrity across schemas.
- **Con:** Cross-schema JOINs may be marginally slower than same-schema, but negligible in practice.

**Current:** All 10 schemas coexist in one `sy_water_db` instance. No horizontal partitioning planned unless sharding becomes a requirement.

---

## Conclusion

The current schema is **3NF-compliant with justified denormalizations**. No issues detected. Trade-offs are documented and monitored.

If a future audit finds inconsistency in D1, D2, or D4, the fix path is:
1. Identify the root cause (application bug, missing lock, etc.).
2. Reconcile the data (mass update or atomic repair).
3. Reinforce application-level or DB-level guards to prevent recurrence.

Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>
