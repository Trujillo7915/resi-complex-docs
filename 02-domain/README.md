# 02 — Problem Domain

> **What is this?** The mental model of the resi-complex business. It is not
> technology — it is the shared understanding of the problem the system solves,
> captured before writing code. This section follows Domain-Driven Design (DDD).

## Why this section exists

The most costly mistakes in software are not bugs — they are domain misunderstandings.
When developers do not deeply understand the business:
- They create incorrect abstractions that have to be rewritten
- Names in the code do not match the business's names → permanent confusion
- Microservice boundaries are drawn incorrectly

This section captures resi-complex's domain knowledge **before** the team finalizes
`05-architecture/`.

---

## Status

| File | Status | Version | Notes |
|------|--------|---------|-------|
| `domain-map.md` | Draft, filled | v0.2 | 9 bounded contexts identified; not yet validated in a real Event Storming session |
| `entities-and-rules.md` | Draft, filled | v0.2 | 10 entities, 4 Value Objects, 5 Aggregates documented |
| `domain-events.md` | Draft, filled | v0.2 | 16 domain events, 6 policies, 3 event flows documented |

---

## Key concepts you must know

**Entity:** Domain object with a unique identity (e.g. a `MaintenanceRequest`
identified by its ID, regardless of status changes).

**Value Object:** Object with no identity of its own, defined by its attributes
(e.g. `Money`, `Email`, `UnitType`).

**Aggregate:** Group of entities treated as a unit. Only the aggregate root
can be referenced from outside (e.g. `ExpenseProposal` is the root; its `Approval`
records can only be reached through it).

**Domain Event:** Something that occurred in the business that other bounded contexts
must know about (e.g. `FeeGenerated`, `MaintenanceRequestCreated`). Stated in past
tense, immutable.

**Bounded Context:** Area of the system where a particular model applies. In
resi-complex, each of the 9 bounded contexts maps 1:1 to a microservice
(`iam-service`, `units-service`, `people-service`, `maintenance-service`,
`billing-service`, `communications-service`, `access-control-service`,
`finance-approval-service`, `reports-service`).

---

## What's in this folder

### `domain-map.md` ⭐
The 9 bounded contexts of resi-complex, their Ubiquitous Language, the Context Map
(who is upstream/downstream of whom), and the Core/Supporting/Generic classification.
**Core Domain:** Billing and Units Management — this is where the tech watch found the
project's real competitive differentiator (residential + commercial units with
differentiated fee rates, which no evaluated competitor covers natively).

### `entities-and-rules.md` ⭐
The tactical DDD building blocks: `Unit`, `Person`, `MaintenanceRequest`,
`AdministrationFee`, `Correspondence`, `Visit`, `ExpenseProposal`, and their
Value Objects (`Money`, `UnitType`, `Email`, `VisitorVehicle`) and Aggregates, with
state machines and invariants for each.

### `domain-events.md` ⭐
The 16 domain events resi-complex needs (e.g. `UnitRegistered`,
`MaintenanceRequestCreated`, `FeeGenerated`, `CorrespondenceReceived`,
`ExpenseProposalApproved`), their payload schemas, consumers, the policies that react
to them, and 3 illustrative end-to-end event flows.

---

## Correlations with other sections

| This section feeds... | Why | Status in this repo |
|-----------------------|-----|----------------------|
| `05-architecture/overview.md` | Bounded contexts → microservices | Still the generic scaffold; needs updating with these 9 contexts |
| `06-data/models.md` | Entities → data tables | Still the generic scaffold |
| `07-api/` | Domain events → AsyncAPI contracts | Not started |
| `09-microservices/service-catalog.md` | All 9 services should be listed here | Still generic (`api-gateway`/`auth-service` examples, Node.js/PostgreSQL) |
| `04-requirements/user-stories.md` | Business rules → acceptance criteria | Still the generic scaffold |

---

## Recommended next step: Event Storming

**Event Storming** is a domain discovery workshop with sticky notes:
1. 🟠 Orange: Domain events (past tense)
2. 🔵 Blue: Commands (what triggers the event)
3. 🟡 Yellow: Actors (who executes the command)
4. 🟣 Purple: Policies (automatic reactions)
5. 🟦 Light blue: External systems

resi-complex has **not** run this session yet. Everything in `domain-map.md`,
`entities-and-rules.md`, and `domain-events.md` was derived by reading the existing
project docs (context, product, governance) rather than from a live workshop with the
whole team. Running one before finalizing `05-architecture/` and `06-data/` will
confirm — or correct — the bounded context boundaries and event list above.

---

## Questions this section answers

- What are resi-complex's main business entities? → `Unit`, `Person`,
  `MaintenanceRequest`, `AdministrationFee`, `Correspondence`, `Visit`,
  `ExpenseProposal`, and their supporting Value Objects.
- What rules can NEVER be violated? → see the **Invariants** in
  `entities-and-rules.md` for each entity and aggregate.
- What important events occur in the domain? → see the **Event catalog** and
  **Event summary table** in `domain-events.md`.
- Where are the natural boundaries of the system? → the 9 bounded contexts in
  `domain-map.md`, classified as Core (Billing, Units Management), Supporting
  (Maintenance, Communications, Access Control, People Management, Financial
  Approval), or Generic (IAM, Reports).