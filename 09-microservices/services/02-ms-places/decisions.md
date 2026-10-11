# Design Decisions — ms-places

## [DEC-001] Single `UNIQUEIDENTIFIER` id per table
`id UNIQUEIDENTIFIER DEFAULT NEWID()` is both the primary key and the public id (ADR-003).

## [DEC-002] Foreign key to `security.users` without cascade
HU-008 deletes accounts with soft delete; a physical cascade would silently delete places.

## [DEC-003] Filtered unique index for the selected place
`UX_places_owner_default ON (owner_id) WHERE is_default = 1 AND deleted_at IS NULL`.
Changing the selection clears the old one and sets the new one in the same transaction, in that order.
Deleting the selected place moves the selection to another place.

## [DEC-004] Role `places_rw` without DELETE
Soft delete only. An accidental `DELETE` fails in the database; the catalog is read-only for the app.

## [DEC-005] Location catalog with Colombia and Ecuador (ADR-004)
Free-text cities would make "Bogota" and "Bogotá D.C." different places and break tariffs per city
(HU-066). Only Colombia and Ecuador are active; adding a country is a release with seeds.

## [DEC-006] .NET 10 LTS and hexagonal architecture in four projects
`Domain` (no dependencies), `Application` (ports and use cases), `Infrastructure` (EF Core, HTTP
adapters) and `Api` (REST adapter and composition root). EF Core without migrations.

## [DEC-007] 404 instead of 403 for foreign places
Other services rely on this endpoint for ownership; answering 404 hides whether the place exists.

## [DEC-008] Deleting needs ms-devices
If ms-devices does not answer, deletion is refused with 503 (`service.unavailable`), never allowed by default.

## [DEC-009] Tariffs and tips in the `places` schema
Tariffs depend on the city and stratum of a place, and tips are a small catalog, so both live in ms-places
instead of creating the `billing` and `content` schemas planned in `06-data/models.md`. Tariffs are a
history (`valid_from`); tips are deactivated, never deleted.
