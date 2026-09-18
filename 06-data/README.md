# 06 — Data

> **What is this?** How the system stores, structures, and migrates data.
> In microservices, data management is one of the most complex challenges.

## Fundamental principle in microservices

> **Each microservice owns its own data.**

No service should access another service's database directly. If it needs data from another
service, it requests it via API or receives it via event. This principle guarantees independence.

---

## What is here and how to fill it in

### `models.md` ⭐ — done
Data models for each of the 9 microservices: tables, columns, constraints, indexes,
migration strategy, and the cross-service ER diagram. Engine: MySQL 8 for all services
(`01-context/scope.md`).

**Format per service** (as used in `models.md`):
```markdown
## Service: `service-name`
**DB Engine:** MySQL 8 — [justification for this service]

### Table: `table_name`
| Field | Type | Nullable | Description | Constraints |
|-------|------|----------|-------------|-------------|
| id | CHAR(36) | No | Unique identifier (UUID) | PK |
| [field] | [type] | [Yes/No] | [description] | [FK/Unique/etc.] |
```

### `data-dictionary.md` — merged into `models.md`
The content this file would hold already exists: each table in `models.md` carries
its own "Data dictionary" subsection with per-column business meaning. Split it out
into its own file only if the column count grows large enough that cross-service lookup
becomes painful.

### `modeling-conventions.md` — merged into `models.md`
The content this file would hold already exists inline in `models.md` → "Data modeling
principles" (naming: `snake_case`, UUIDs as `CHAR(36)`, soft delete via `deleted_at`,
audit fields `created_at`/`updated_at`). Split out only if conventions start drifting
between services.

### `normalization-assessment.md` — merged into `models.md`
The one denormalization the project has today is already documented: `reports-service`'s
projection tables (`requests_by_status_view`, `pending_fees_view`) are intentionally
denormalized read models, justified inline in `models.md` → Service: `reports-service`.
Split this out into its own file if more denormalizations are added later and need a
dedicated record.

### `migration-strategy.md` — merged into `models.md`
The content this file would hold already exists inline in `models.md` → "Migration
strategy" (tool: Flyway, per `_stacks/java-spring.md`; naming convention; compatible
vs. incompatible change examples).

---

## Status

| File | Status | Notes |
|------|--------|-------|
| `models.md` | Filled | 9 services, 15 tables, MySQL 8, ER diagram, Flyway migration strategy |
| `data-dictionary.md`, `modeling-conventions.md`, `normalization-assessment.md`, `migration-strategy.md` | Merged into `models.md` | Content exists inline for this MVP scope (see above); split out into their own files if they outgrow it |

---

## Correlations with other sections

| This section is fed by... | And feeds into... |
|---------------------------|-------------------|
| `02-domain/entities-and-rules.md` → domain entities | DB tables |
| `05-architecture/` → DB engine decisions | Engine choice in `models.md` |
| `models.md` | `07-api/contracts/` → what data each service exposes |
| `models.md` | `08-uml/` → ER diagrams |
| `models.md` | `09-microservices/[service]/data-model.md` |

---

## Important data decisions in microservices

### SQL or NoSQL?
resi-complex uses **SQL (MySQL 8)** for all 9 services — every entity in
`02-domain/entities-and-rules.md` has a fixed, well-known schema and needs ACID
guarantees (e.g. a fee cannot be half-generated). No service in this MVP scope needs:
- **Document** (MongoDB): no entity has a variable/nested shape
- **Key-value** (Redis): sessions are stateless JWTs, not server-side state
- **Time series** (InfluxDB, TimescaleDB): no telemetry/IoT entity in scope

### How to handle consistency between services?
Without a shared database, consistency is **eventual**:
- Saga Pattern: chain of compensating transactions
- Outbox Pattern: guarantee that the event is published along with the transaction

---

## Questions this section answers

- **What data does each microservice handle?** → `models.md`, one `## Service:` section per microservice (15 tables total across the 9 services).
- **Why was that database engine chosen for each service?** → MySQL 8 for all 9, per `01-context/scope.md`; see "SQL or NoSQL?" above for why the alternatives weren't needed.
- **How is the schema updated without breaking the system?** → Flyway, versioned forward-only migrations; see `models.md` → "Migration strategy" for the compatible/incompatible change examples.
- **Who is the "owner" of each piece of data in the system?** → the service whose section it lives under in `models.md`; every cross-service reference (e.g. `maintenance_requests.person_id`) is resolved via API/event, never a real SQL foreign key across databases.
