# 05 — Architecture

> **What is this?** The system's design decisions: how it is organized, why,
> what alternatives were evaluated, and how it is deployed. ADRs are the treasure of this section.

## Why this section exists

A system's architecture is the set of decisions that are hard to change later.
Documenting them has three benefits:
1. **New team members** understand the system without having to ask everything from scratch
2. **The team** does not repeat already-resolved discussions
3. **Years later**, everyone remembers why each decision was made

For **resi-complex**, this section translates the 9 Bounded Contexts already identified in
`02-domain/domain-map.md` into 9 deployable microservices, each internally structured with
Hexagonal Architecture (Ports & Adapters) on Java 17+ / Spring Boot 3.x, per the locked stack
in `01-context/overview.md` and `01-context/_template-project-profile.md`.

---

## Status

| File | Status | Notes |
|------|--------|-------|
| `overview.md` | ✅ Completed | C4 Context/Container diagrams, 9-service catalog, architectural principles, cross-cutting concerns, technical debt |
| `hexagonal-architecture.md` | ✅ Completed | Adapted to Java + Spring Boot (the project's locked stack), using `MaintenanceRequest` as the worked example, consistent with `02-domain/entities-and-rules.md` |
| `pattern-guide.md` | ✅ Completed | Pattern catalog (GoF + microservices patterns) with the final "Patterns adopted in this project" table filled in and justified |
| `decisions/README.md` | ✅ Completed | ADR register and process |
| `decisions/records/ADR-001-idioma-documentacion.md` | ✅ Already accepted | Governs the language of this entire folder (English) |
| `decisions/records/ADR-002-language-exception-user-research.md` | ✅ Already accepted | Extends ADR-001; does not apply to this folder's files (none of them are raw research artifacts) |
| `deployment.md` | 🔴 Not created yet | Blocked by two open decisions: container orchestration platform and target environment beyond Local (`01-context/overview.md`, `01-context/scope.md`) |
| `cross-cutting.md` | 🔴 Not created yet | Content already summarized in `overview.md` §7 (Cross-cutting concerns); promote to its own file once each concern moves from "planned" to "implemented" |
| `security-threat-model.md` | 🔴 Not created yet | Should be built from `00-governance/security-policy.md` + `04-requirements/non-functional.md` NFR-004 using STRIDE; recommended before Sprint 3 (Maintenance/Billing/Access Control go live per `03-product/vision.md`) |

---

## What is here and how to fill it in

### `overview.md` ⭐ (Start here)
High-level view of the complete system.

**Contains for resi-complex:**
- Adopted architectural style: **Microservices + Event-Driven**, one service per Bounded Context
- C4 Context diagram: the 5 authentication roles (`01-context/overview.md`) interacting with the system; no external systems (`01-context/scope.md`: "no third-party integrations planned for the MVP")
- C4 Container diagram: the 9 services, their MySQL databases, and the still-undecided message broker and API Gateway
- Service catalog: all 9 services from `02-domain/domain-map.md`, with responsibility, proposed port, database, and communication type
- Architectural principles P1–P5, including a project-specific principle (**Ownership-Level Authorization by Default**) drawn directly from `01-context/glossary.md`
- Registered architectural technical debt (API Gateway, message broker, container orchestration, standard error schema — all already flagged as "pending" in earlier sections, now tracked formally)

### `deployment.md` ⭐
How the system is deployed in each environment.
**Fill in once decided:** infrastructure diagram, what goes in Docker/K8s (or the chosen orchestrator),
network configuration, hardware requirements. Currently blocked: `01-context/scope.md` confirms only the
**Local** environment for this formative delivery — Staging and Production are explicitly "not yet planned."

### `cross-cutting.md`
Concerns that apply to all 9 microservices: standard logging, distributed tracing, centralized
configuration, feature flags, error handling, retry policies. A first draft of this content
already lives in `overview.md` §7; split it out once implementation starts.

### `pattern-guide.md`
Catalog of design patterns used in the project (GoF + microservices patterns), with a final
table stating which patterns resi-complex adopts, defers, or explicitly rejects, and why.

### `security-threat-model.md`
Security threat analysis of the system using STRIDE (Spoofing, Tampering, Repudiation,
Information Disclosure, Denial of Service, Elevation of Privilege). For each threat: the
mitigation already implemented or planned (RBAC, JWT expiration, bcrypt, ownership-level
authorization — see `04-requirements/non-functional.md` NFR-004 and `01-context/glossary.md`).

### `decisions/` ⭐⭐ — Architecture Decision Records (ADRs)

#### What is an ADR?
A record of ONE important architectural decision: what was decided, why, what alternatives
were evaluated, and what the consequences are. They are **short documents** (1-2 pages).

**When to create an ADR:**
- When choosing a message broker (RabbitMQ vs Kafka vs Redis Streams) — **pending for resi-complex**
- When deciding the database strategy (one per service vs shared) — **already the de facto standard** in `02-domain/domain-map.md` (MySQL dedicated per context), but has no ADR of its own yet
- When choosing a communication pattern (REST vs gRPC vs events) — **already implicit** (REST + Domain Events), no ADR yet
- When choosing an authentication library — **already implicit** (Spring Security + JWT + BCrypt, per `01-context/overview.md`), no ADR yet
- Any decision that, if changed, requires significant refactoring

**When NOT to create an ADR:**
- Day-to-day operational decisions
- Things that can be changed easily without systemic impact

**Use `decisions/_template-adr.md`**

**resi-complex's ADR situation today:**
```
ADR-001-idioma-documentacion.md              → Accepted — English for all docs/code (governance-wide, lives here)
ADR-002-language-exception-user-research.md  → Accepted — extends ADR-001 for raw research artifacts only
ADR-003 (not yet written)                     → Candidate: "Microservices + Event-Driven architectural style"
ADR-004 (not yet written)                     → Candidate: "Database per Service (MySQL, dedicated per context)"
ADR-005 (not yet written)                     → Candidate: "Message broker selection" (Kafka vs RabbitMQ vs Redis Streams)
ADR-006 (not yet written)                     → Candidate: "JWT-based authentication strategy"
```
> ADR-001 and ADR-002 are governance/documentation decisions, not architecture-style decisions —
> they are filed here because ADR-002 explicitly references this folder's path
> (`05-architecture/decisions/records/ADR-001-idioma-documentacion.md`), and because every future
> ADR in this repo (including the architecture-style candidates above) must comply with them.
> See the open gap tracked in `decisions/README.md`.

---

## Correlations with other sections

| This section is fed by... | And feeds... |
|--------------------------|-------------|
| `02-domain/domain-map.md` → 9 bounded contexts | `09-microservices/` → one service per context (catalog still generic, needs updating with the 9 real services) |
| `04-requirements/non-functional.md` → NFRs (NFR-001 performance, NFR-004 security, NFR-005 observability) | Decisions about technology and scale in `overview.md` §5–§7 |
| ADRs chosen here | `09-microservices/` implements the decided patterns; each service's `README.md` should link back to the ADR that justifies its structure |
| `deployment.md` (pending) | `10-devops/environments.md` |
| `01-context/glossary.md` → Ownership-level authorization | `overview.md` §5 (P5) and `00-governance/security-policy.md`'s `:own` / `:assigned` permission suffixes |

---

## The 5 most common architecture mistakes

1. **Microservices too small** — If a "service" cannot exist independently, it is not a microservice.
2. **Shared database** — Destroys service independence. Each service, its own DB. (resi-complex already
   commits to this per `02-domain/domain-map.md` — MySQL dedicated per context.)
3. **Only synchronous communication** — For non-urgent operations, async events scale better.
   (resi-complex already designed 16 domain events in `02-domain/domain-events.md` for exactly this —
   the broker technology is the open item, not the design.)
4. **No API Gateway** — Exposing microservices directly to the frontend creates coupling. (Currently
   an open item for resi-complex — see `overview.md` AT-001.)
5. **No documented decisions** — In 6 months nobody remembers why X was chosen. (This is precisely
   why `decisions/` exists — and why the architecture-style ADRs above are flagged as still missing.)

---

## Questions this section must answer

- **How is the system organized into large blocks?** → 9 microservices, one per Bounded Context
  (`overview.md` §4, mirrors `02-domain/domain-map.md`)
- **Why was each key technology chosen?** → Java + Spring Boot and MySQL are locked by the SENA ADSO
  formative program (`01-context/overview.md`); Hexagonal Architecture is chosen for testability and
  framework independence (`hexagonal-architecture.md`)
- **What alternatives were evaluated and why were they discarded?** → Not yet documented for the
  architectural style itself (see the missing ADR-003/004/005/006 candidates above); already documented
  for documentation language (`decisions/records/ADR-001-idioma-documentacion.md`)
- **How is the system deployed?** → Only Local environment confirmed so far (`deployment.md` still pending)
- **What patterns does the team apply and how?** → `pattern-guide.md`, final adoption table
