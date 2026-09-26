# ADR-004 — Market Scope: Colombia and Ecuador Only

| Field | Value |
|-------|-------|
| **ID** | ADR-004 |
| **Date** | 2026-09-26 |
| **Status** | Accepted |
| **Authors** | Juan Esteban Ome |
| **Reviewers** | Juan Esteban Ome, Juan Manuel Gómez, Diego Alejandro Alvarado |

---

## Context

HU-053 ("Seleccionar país y moneda") listed three countries: Colombia, Ecuador and the United States. The place model stored the location as free text (`city`) plus a `country_code` restricted to `CO | EC | US`.

When `ms-places-db` was implemented, two needs came up:

- **Location must be structured, not free text.** Tariffs are set per municipality (HU-066 tariff catalog by city), and free-text cities ("Bogota", "Bogotá D.C.", "bogota") cannot be matched against a catalog.
- **Each country has its own administrative division.** Colombia uses *departamentos* → *municipios* (DANE DIVIPOLA codes); Ecuador uses *provincias* → *cantones* (INEC DPA codes); the United States uses states → counties / water districts, with a different unit (gallons / CCF) and a fragmented tariff landscape (HU-068, already *Won't*).

Loading and maintaining a US catalog would cost far more than CO and EC together, with no user in that market for this academic project.

**Known constraints:**
- University project, team of 3, no budget for paid geographic datasets.
- DANE (CO) and INEC (EC) publish official, free code lists.
- Ecuador uses the US dollar, so USD is already required even without the US market.

---

## Decision

**We decided:** for now the product supports **only Colombia (CO) and Ecuador (EC)**. The location of a place is a reference to a **city catalog** (`places.cities`) organized as country → subdivision → city, not free-text fields.

| Level | Table | Colombia (CO) | Ecuador (EC) |
|-------|-------|---------------|--------------|
| Country | `places.countries` | `CO` — Colombia, default currency COP | `EC` — Ecuador, default currency USD |
| Subdivision | `places.subdivisions` | Departamento — 33 (DANE, 2-digit code) | Provincia — 24 (INEC, 2-digit code) |
| City | `places.cities` | Municipio — 1,122 incl. non-municipalized areas (DANE, 5-digit code) | Cantón — 221 (INEC, 4-digit code; zone 90 excluded) |

- `places.places` stores `city_id`; country and subdivision are derived through `city → subdivision → country` (no `country_code` column, no free-text `city`).
- A new place's `currency` and `measurement_unit` are initialized from `countries.default_currency` / `default_unit`; the user may change them afterwards.
- Supported currencies: `COP`, `USD`. Default unit for both countries: `LITERS`.
- Catalogs are reference data: seeded by migrations in every environment, read-only for the application role.

**Justification:** CO and EC share the same two-level structure and both publish official codes, so one model covers both with no special cases. Adding the US now would force a third level (water district), a second unit system and a catalog we can't source.

---

## Evaluated alternatives

| Alternative | Pros | Cons | Reason for discarding |
|------------|------|------|-----------------------|
| **A — CO + EC with country/subdivision/city catalog (chosen)** | Matches official codes; enables tariffs per city; one model for both countries | Catalog must be seeded and kept current | — (chosen) |
| B — CO + EC + US (HU-053 as written) | Broader market | US needs districts, gallons/CCF, and a catalog we can't source; no US users | Cost without demand |
| C — Keep free-text `city` + `country_code` | No seed data to maintain | Can't join to a tariff catalog; duplicates and typos; 3NF-wise country is not tied to the city | Blocks HU-066 |
| D — Colombia only | Smallest catalog | Ecuador is part of the target market and has official data at no extra cost | Too narrow |

---

## Consequences

**Positive:**
- Tariff catalogs (HU-066) can key on `city_id`.
- Consistent, validated locations; the frontend renders cascading selects (country → subdivision → city).
- Currency and unit defaults come from data, not hard-coded rules.

**Negative / Trade-offs:**
- US users cannot register a place. HU-068 (US tariff catalog) stays *Won't*.
- Official catalogs change (new municipalities); updates require a new seed migration.
- Supporting a new country later means seeding its subdivisions and cities and, if its structure has more levels, revisiting this ADR.

**Impact on the system:**
- Affected services: place-service (owner of `places.countries`, `places.subdivisions`, `places.cities`), consumption-service (tariffs, currency).
- Documents that must be updated: `06-data/models.md`, `02-domain/entities-and-rules.md` (Place, Address), `04-requirements/user-stories.md` (HU-053, HU-068).

---

## Risks

| Risk | Probability | Impact | Mitigation |
|------|------------|--------|-----------|
| DANE/INEC publishes changes (new municipality, renamed canton) | Low | Low | New forward-only seed migration; codes are the stable key |
| Demand appears from a third country | Low | Medium | Catalog model is generic; new country = new seed rows + review of `default_unit` / currency CHECKs |
| A user's address falls outside CO/EC | Low | Low | Registration form only offers supported countries |

---

## References

- HU-009 (place registration), HU-053 (country and currency), HU-066 (tariff catalog by city), HU-068 (US catalog — Won't) → `04-requirements/user-stories.md`
- Data models → `06-data/models.md`
- Implementation: `sy-water-db/ms-places-db` v1.0-baseline (`01-ddl/03-tables/001..004`, `02-dml/00-inserts/001..004`)
- Sources: DANE — DIVIPOLA; INEC — Clasificador Geográfico Estadístico (DPA)
- Related to: ADR-003 (catalog tables use a single UUID PK)
