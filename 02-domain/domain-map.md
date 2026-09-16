# Domain Map — Bounded Contexts

> **What to fill in here:** The domain map is the central DDD (Domain-Driven Design) artifact.
> It defines the system's boundaries and how they relate to each other.
> Build it first with the team and domain experts in an Event Storming session.

## Before filling in this document: Event Storming

**Event Storming** is a collaborative workshop for modeling the domain before writing code.
It lasts 2–4 hours with the whole team (dev + PO + business expert).

**Materials:** Long wall, 4-color sticky notes, markers.

**Standard colors:**
| Color | Represents | Example |
|-------|-----------|---------|
| 🟠 Orange | **Domain events** (something that happened, past tense) | `AppointmentScheduled`, `PaymentReceived` |
| 🔵 Blue | **Commands** (action that triggers the event) | `ScheduleAppointment`, `ProcessPayment` |
| 🟡 Yellow | **Actors** (who executes the command) | `Patient`, `Doctor`, `Admin` |
| 🩷 Pink | **External systems** or integration points | `Payment Gateway`, `Email SMTP` |

**Session steps:**
1. (30 min) Post all events that occur in the business, in chronological order, on the wall
2. (30 min) Identify which command or actor triggers each event
3. (45 min) Group related events — each group is a candidate Bounded Context
4. (30 min) Draw relationships between Bounded Contexts (who depends on whom)
5. (30 min) Discuss the resulting map and agree on names

**Result:** The session output directly feeds the 3 documents in `02-domain/`:
- Identified events → `domain-events.md`
- Entities and their rules → `entities-and-rules.md`
- Bounded Contexts and their map → this document

---

## 1. Domain overview

```
resi-complex centrally manages a mixed residential complex — one that combines
residential housing units and commercial units under a single administration. The
system allows registering units and the people who live in or run a business in them,
calculating and tracking administration fees (which differ for residential vs.
commercial units), managing maintenance requests from the moment they are reported
until they are resolved, controlling visitor and vehicle access at the front desk
together with incoming correspondence, sending announcements segmented by unit type,
and giving formal traceability to the approval of extraordinary expenses by the Board
of Trustees. The goal is to replace today's manual process (notebooks, spreadsheets,
WhatsApp) with a single system that gives transparency to residents and commercial
owners, and control to the administration and the Board.
```

---

## 2. Identified Bounded Contexts

A **Bounded Context** is the explicit boundary within which a particular domain model
has consistent meaning. Each bounded context has its own Ubiquitous Language.

> **Signs of a good bounded context:**
> - Has a clear responsible team
> - Has its own database
> - Can be deployed independently
> - The same term in two different contexts can mean different things

The 9 bounded contexts below map 1:1 to the services already named consistently across
`01-context/scope.md` and `03-product/vision.md`: `iam-service`, `units-service`,
`people-service`, `maintenance-service`, `billing-service`, `communications-service`,
`access-control-service`, `finance-approval-service`, `reports-service`.

### Bounded Context: Identity and Access Management (IAM)

| Field | Value |
|-------|-------|
| **Name** | IAM (Identity and Authentication) |
| **Responsibility** | Authenticate users and resolve RBAC permissions for the system's 5 authentication roles (FR01) |
| **Owning team** | resi-complex team (owner to be assigned) |
| **Microservice(s)** | `iam-service` |
| **Database** | MySQL (dedicated) |
| **Ubiquitous Language** | User, credentials, role, permission, token, session |

**Context-specific terms (Ubiquitous Language):**

| Term | Meaning in THIS context | Different in another context? |
|------|------------------------|-------------------------------|
| User | An account with credentials and exactly one role: `ADMINISTRATOR`, `BOARD`, `PERSON`, `MAINTENANCE_STAFF`, or `SECURITY_GUARD` (role strings per `00-governance/security-policy.md`) | Yes — in People Management the same individual exists as a `Person`, not a `User` |
| Permission | A `resource:action` string (e.g. `fees:read:own`, `requests:update:assigned`) evaluated by each service, not centrally | No |

> **Key design decision:** `IAM` and `People Management` are deliberately separate.
> `IAM` only knows about credentials and permissions; `People Management` knows about
> residents and commercial owners/tenants (their data, unit, type). This mirrors the
> glossary rule: "Resident" and "Commercial Owner/Tenant" are subtypes of `Person`,
> **not** separate authentication roles.

---

### Bounded Context: Units Management

| Field | Value |
|-------|-------|
| **Name** | Units Management |
| **Responsibility** | Register and classify units (residential/commercial) and the data of commercial establishments (FR02–FR04) |
| **Owning team** | resi-complex team (owner to be assigned) |
| **Microservice(s)** | `units-service` |
| **Database** | MySQL (dedicated) |
| **Ubiquitous Language** | Unit, unit type, commercial establishment |

**Context-specific terms (Ubiquitous Language):**

| Term | Meaning in THIS context | Different in another context? |
|------|------------------------|-------------------------------|
| Unit | A physical space in the complex (residential or commercial); per the glossary, avoid "apartment" as a synonym since it excludes commercial units | No — reused the same way everywhere as a reference (`unitId`) |
| Commercial Establishment | The business operating inside a commercial Unit (name, type, hours). Only exists if the Unit is commercial — do not confuse with the Unit record itself | No |

---

### Bounded Context: People Management

| Field | Value |
|-------|-------|
| **Name** | People Management |
| **Responsibility** | Register residents and commercial owners/tenants (`Person`), and their relationship to a unit (FR01) |
| **Owning team** | resi-complex team (owner to be assigned) |
| **Microservice(s)** | `people-service` |
| **Database** | MySQL (dedicated) |
| **Ubiquitous Language** | Person, person type, associated unit |

**Context-specific terms (Ubiquitous Language):**

| Term | Meaning in THIS context | Different in another context? |
|------|------------------------|-------------------------------|
| Person | A single entity modeled with a `personType` field distinguishing Resident from Commercial Owner/Tenant — never modeled as two separate entities or roles (glossary rule) | Yes — in `IAM` the same individual exists as a `User` (credentials only) |

> **NFR (data protection) note:** this context concentrates the most sensitive personal
> data in the system (Ley 1581 de 2012), so its separation from `IAM` and `Units` is
> also a data-protection decision, not only a modeling one.

---

### Bounded Context: Maintenance

| Field | Value |
|-------|-------|
| **Name** | Maintenance |
| **Responsibility** | Manage the lifecycle of maintenance requests, from being reported until resolved (FR05–FR07) |
| **Owning team** | resi-complex team (owner to be assigned) |
| **Microservice(s)** | `maintenance-service` |
| **Database** | MySQL (dedicated) |
| **Ubiquitous Language** | Maintenance request ("request" per the RBAC resource name), priority, status, assigned staff |

**Context-specific terms (Ubiquitous Language):**

| Term | Meaning in THIS context | Different in another context? |
|------|------------------------|-------------------------------|
| Request | A ticket with type, description, and priority, moving through a defined status lifecycle. Per the glossary, avoid "ticket" or "PQR" (competitor term) as synonyms | No |
| Status | The request's phase (pending, assigned, in progress, resolved) | Yes — in `Financial Approval`, "status" means whether a proposal was approved/rejected |

---

### Bounded Context: Billing

| Field | Value |
|-------|-------|
| **Name** | Billing |
| **Responsibility** | Generate monthly administration fees with rates differentiated by unit type, and track their payment status (FR08–FR10) |
| **Owning team** | resi-complex team (owner to be assigned) |
| **Microservice(s)** | `billing-service` |
| **Database** | MySQL (dedicated) |
| **Ubiquitous Language** | Administration fee ("fee" — per the glossary, never "quota" in English docs), payment, arrears |

**Context-specific terms (Ubiquitous Language):**

| Term | Meaning in THIS context | Different in another context? |
|------|------------------------|-------------------------------|
| Fee | A unit's periodic payment obligation, rate based on unit type | No |
| Payment | A recorded transaction that fully or partially settles a Fee | No |

> This is one of the two contexts that carry the project's competitive differentiator
> (`03-product/resi-complex-tech-watch.md`: none of the 4 validated Colombian
> competitors — ConjuntoApp, TUCO 360, Edifia, PH360 — natively covers residential +
> commercial units with differentiated rates).

---

### Bounded Context: Communications

| Field | Value |
|-------|-------|
| **Name** | Communications |
| **Responsibility** | Publish announcements segmented by unit scope: all units, only residential, or only commercial (FR11) |
| **Owning team** | resi-complex team (owner to be assigned) |
| **Microservice(s)** | `communications-service` |
| **Database** | MySQL (dedicated) |
| **Ubiquitous Language** | Announcement, scope/segment |

**Context-specific terms (Ubiquitous Language):**

| Term | Meaning in THIS context | Different in another context? |
|------|------------------------|-------------------------------|
| Announcement | A message targeted at a segment of units. Per the glossary, avoid "notification" as a synonym — that word is reserved for system/status alerts (e.g. FR07) | No |

> **Scope correction from v0.1:** Correspondence does **not** live here — it was moved
> to `Access Control` per `01-context/scope.md`. This context now only owns
> Announcements, which keeps it small and focused on one actor (the Administrator)
> and one action (publish to a segment).

---

### Bounded Context: Access Control

| Field | Value |
|-------|-------|
| **Name** | Access Control |
| **Responsibility** | Log visitor and vehicle entry/exit at the front desk, and log incoming correspondence per unit (FR12–FR16) |
| **Owning team** | resi-complex team (owner to be assigned) |
| **Microservice(s)** | `access-control-service` |
| **Database** | MySQL (dedicated) |
| **Ubiquitous Language** | Visit, personal visitor, commercial client, visitor vehicle, correspondence |

**Context-specific terms (Ubiquitous Language):**

| Term | Meaning in THIS context | Different in another context? |
|------|------------------------|-------------------------------|
| Visit | A record of an external person's entry/exit, with a destination unit and type (personal visitor or commercial client, FR14) | No |
| Correspondence | A package or letter logged against a unit, pending pickup by its occupant. Per the glossary, avoid "mail" alone (ambiguous with email) | No |

> **Why Correspondence and Visits share a context:** both are written in real time by
> the same actor (the Security Guard) at the same physical point (the front desk), and
> both simply reference a `unitId` without needing the Unit's or Person's full data —
> exactly the kind of low-coupling, single-actor grouping a bounded context should be.

---

### Bounded Context: Financial Approval

| Field | Value |
|-------|-------|
| **Name** | Financial Approval |
| **Responsibility** | Manage the proposal, approval, or rejection of extraordinary expenses by the Board of Trustees, with permanent history (FR17–FR18) |
| **Owning team** | resi-complex team (owner to be assigned) |
| **Microservice(s)** | `finance-approval-service` |
| **Database** | MySQL (dedicated) |
| **Ubiquitous Language** | Expense proposal, approval, decision history |

**Context-specific terms (Ubiquitous Language):**

| Term | Meaning in THIS context | Different in another context? |
|------|------------------------|-------------------------------|
| Expense Proposal | A request for an extraordinary expense created by the Administrator, submitted for Board approval | No |
| Approval | The Board of Trustees' recorded decision (approved/rejected) on a proposal. Per the glossary, approvals are append-only — never overwritten, for traceability | No |

---

### Bounded Context: Reports

| Field | Value |
|-------|-------|
| **Name** | Reports |
| **Responsibility** | Consolidate requests-by-status and pending-fee-arrears views (FR19), without owning that data |
| **Owning team** | resi-complex team (owner to be assigned) |
| **Microservice(s)** | `reports-service` |
| **Database** | MySQL (dedicated — read/projection model) |
| **Ubiquitous Language** | Report, indicator, projection |

**Context-specific terms (Ubiquitous Language):**

| Term | Meaning in THIS context | Different in another context? |
|------|------------------------|-------------------------------|
| Report | A read-only, aggregated view built from other contexts' events; `reports:read` is the only permission defined for it in the security policy | No |

---

## 3. Context Map

```
                              ┌───────────────────────┐
                              │        IAM             │
                              │  (auth, roles, JWT)    │
                              └──────────┬─────────────┘
                          OHS/PL (token) │  (upstream of ALL contexts)
              ┌─────────────┬────────────┼────────────┬───────────────┐
              ▼             ▼            ▼            ▼               ▼
     ┌────────────────┐ ┌─────────┐ ┌──────────┐ ┌───────────┐ ┌─────────────┐
     │ Units           │ │ People  │ │Maintenance│ │Communicat.│ │Access       │
     │ Management      │ │ Mgmt.   │ │           │ │(Announce- │ │Control      │
     │                 │ │         │ │           │ │ments only)│ │(Visits +    │
     │                 │ │         │ │           │ │           │ │Correspond.) │
     └───┬─────────────┘ └────┬────┘ └────┬─────┘ └─────┬─────┘ └──────┬──────┘
         │ U→D                │ U→D       │             │              │
         ▼                    ▼           │             │              │
     ┌─────────────────────────────┐      │             │              │
     │          Billing             │◀─────┘             │              │
     │  (fee rate by unit type,     │                    │              │
     │   person to be billed)       │                    │              │
     └───────────────┬──────────────┘                    │              │
                      │ U→D (financial context)           │              │
                      ▼                                   │              │
           ┌─────────────────────┐                        │              │
           │ Financial           │                        │              │
           │ Approval            │                        │              │
           │ (Board of Trustees) │                        │              │
           └─────────────────────┘                        │              │
                                                            │              │
              All contexts above ──OHS/PL(events)──▶ Reports (reports:read only)
```

### Context relationship types

| Type | Symbol | Description | Example |
|------|--------|-------------|---------|
| **Upstream → Downstream** | `U → D` | U provides, D consumes. D depends on U. | Auth → Orders |
| **Shared Kernel** | `SK` | Two teams share part of the model | Shared User ID |
| **Customer/Supplier** | `C/S` | Supplier (U) negotiates with Customer (D) | Inventory → Sales |
| **Conformist** | `CONF` | D adopts U's model without negotiating | Legacy integration |
| **Anti-Corruption Layer** | `ACL` | D translates U's model to protect itself | Gateway → External API |
| **Open Host Service** | `OHS` | U publishes a published protocol | Event Bus, REST API |
| **Published Language** | `PL` | Explicit shared language | OpenAPI spec, events |

### Relationships table

| Context A | Relationship | Context B | Communication channel | Contract |
|-----------|-------------|-----------|----------------------|---------|
| IAM | OHS/PL | All other contexts | REST (JWT validation) | Signed JWT + OpenAPI |
| Units Management | U → D | Billing | Async event | AsyncAPI (`domain-events.md`) |
| Units Management | U → D | Access Control | Async event | AsyncAPI |
| Units Management | U → D | Communications | Async event | AsyncAPI |
| People Management | U → D | Billing | Async event | AsyncAPI |
| People Management | U → D | Maintenance | Async event | AsyncAPI |
| People Management | U → D | Access Control | Async event | AsyncAPI |
| Billing | U → D | Financial Approval | Async event / REST | AsyncAPI (available budget) |
| All contexts | OHS/PL | Reports | Async event | AsyncAPI (read-only projection) |
| Access Control | ACL | Units Management | REST (valid-unit lookup) | OpenAPI |

---

## 4. Core Domain, Supporting, Generic

DDD classifies subdomains by their strategic value:

| Type | Description | Investment | Example |
|------|-------------|-----------|---------|
| **Core Domain** | Where the business competitive advantage lies. What differentiates us. | MAXIMUM — build, don't buy | Matching algorithm |
| **Supporting Subdomain** | Necessary for the core but not differentiating. Can be outsourced. | MEDIUM | Order management |
| **Generic Subdomain** | Commodity. Off-the-shelf solution exists. | MINIMUM — buy/use OSS | Authentication, emails |

### Classification of this project's bounded contexts

| Bounded Context | Type | Justification |
|----------------|------|---------------|
| Billing | **Core** | The tech watch confirmed no competitor natively covers differentiated residential/commercial fees — resi-complex's main differentiator |
| Units Management | **Core** | Directly enables the Billing differentiator: without the residential/commercial classification, Billing cannot apply differentiated rates |
| Financial Approval | Supporting | Needed for Board of Trustees traceability (FR17–FR18), but not what sets resi-complex apart from competitors — the tech watch found similar budget/assembly features elsewhere |
| Maintenance | Supporting | Expected in any residential-complex platform (competitors offer photo-attached work orders); tech watch flags photo evidence as a low-cost future enhancement to this context, not required for the MVP |
| Communications | Supporting | Segmentation by unit type is useful, but announcement broadcasting itself is a common market capability |
| Access Control | Supporting | Operationally important (front desk), but the MVP explicitly excludes advanced QR/pre-authorization (tech watch: "prepared in the data model, not implemented now") |
| People Management | Supporting | Needed as the master database of people, but conceptually standard |
| IAM | **Generic** | Spring Security + BCrypt + JWT is a standard, off-the-shelf approach with no domain-specific logic |
| Reports | **Generic** | Aggregation/display of data owned elsewhere; adds no new domain logic |

---

## 5. Modeling decisions

### How were these decisions made?

- **Event Storming session:** not yet held — pending scheduling with the team
- **Tool used:** to be defined (proposal: Miro)
- **Map iterations:**
  - v0.1 (Sep 11, 2026) — derived only from the Single Source of Truth summary
  - v0.2 (Sep 11, 2026, this version) — revised against the full project repo
    (glossary, scope, security policy, vision, tech watch); moved Correspondence
    from Communications to Access Control; adopted "Board of Trustees" naming

### Key decisions and discarded alternatives

| Decision | Discarded alternative | Reason |
|----------|----------------------|--------|
| Separate `IAM` from `People Management` | A single "Users" context mixing credentials and personal data | The glossary already distinguishes the authentication role from `personType`; mixing them would complicate Ley 1581 compliance |
| Separate `Units Management` from `People Management` | Merge into one "Units and Residents" context | They evolve at different rates and have different data-protection requirements (unit data vs. personal data) |
| Move `Correspondence` into `Access Control`, out of `Communications` | Keep Correspondence in Communications (v0.1 choice) | `scope.md` assigns it to `access-control-service`; both Correspondence and Visits are written in real time by the same actor (Security Guard) at the same point (front desk) |
| Classify `Billing` and `Units Management` as Core Domain | Treat the whole system with the same level of investment | The tech watch identified differentiated fees as the real market gap; design/testing effort should concentrate there |

---

## 6. How to update this map

1. Before adding a new microservice, verify whether it belongs to an existing bounded context.
2. If a context's ubiquitous language is changing, review whether the context should be split.
3. Run an Event Storming session every time the domain changes significantly.
4. The context map MUST be synchronized with the C4 system-level diagram (`05-architecture/overview.md`) — currently still the generic scaffold and needs to be filled in with this map's 9 services.

> **Important correlation:** The bounded contexts in this document →
> Microservices in `09-microservices/service-catalog.md` (currently generic, needs
> updating to the real 9-service list) →
> C4 diagrams in `08-uml/` →
> Service separation ADRs in `05-architecture/decisions/`

> **Open item for the team:** update `01-context/glossary.md` so it says "Board of
> Trustees" instead of "Board of Directors", since the glossary itself declares it is
> the tie-breaker in case of ambiguity.