# Entities, Value Objects, and Business Rules

> **What to fill in here:** The building blocks of the domain following the DDD tactical model.
> This document translates domain knowledge (obtained in Event Storming) into code models.

> **Stack note:** The concepts of Entity, Value Object, and Aggregate are language-independent.
> Code examples (classes, interfaces, decorators) are written in pseudo-TypeScript
> to illustrate the idea. To see the implementation in your technology:
> [`_stacks/node-typescript.md`](../_stacks/node-typescript.md) ·
> [`_stacks/java-spring.md`](../_stacks/java-spring.md) ·
> [`_stacks/python-fastapi.md`](../_stacks/python-fastapi.md) ·
> [`_stacks/go.md`](../_stacks/go.md)
>
> resi-complex uses Java + Spring Boot (`01-context/overview.md`); for the real
> implementation of these rules, see `_stacks/java-spring.md`.

> **A note on ownership-level authorization:** `01-context/glossary.md` defines
> "Ownership-level authorization" as an architectural concept, not a domain entity —
> access control that restricts a Person to data belonging to their own Unit(s), beyond
> plain role checks. It shows up in `00-governance/security-policy.md` as the `:own` and
> `:assigned` suffixes (`fees:read:own`, `requests:read:assigned`,
> `correspondence:read:own`). It is called out below wherever it applies to an entity,
> but it is enforced at the API/service layer, not as a domain invariant on the entity
> itself.

---

## Tactical DDD concepts

### Entity
An **Entity** is an object defined by its identity, not its attributes.
Two entities are equal if they have the same ID, even if all their other attributes differ.

```
✓ Entity: User (two users with different emails are still distinct by their ID)
✓ Entity: Order (changes state but remains the same order)
✗ Not an entity: Money (10 USD == 10 USD regardless of which bill)
```

### Value Object (VO)
A **Value Object** is an object defined by its attributes; it has no identity of its own.
It is immutable — if an attribute changes, it is a new VO.

```
✓ Value Object: Address (5th Street #10-20, Neiva, Huila)
✓ Value Object: Money (USD 150.00)
✓ Value Object: Email (user@example.com)
✓ Value Object: DateRange (2024-01-01 → 2024-01-31)
```

### Aggregate
An **Aggregate** is a cluster of entities and VOs treated as a unit.
It has an **Aggregate Root** which is the entry point — internal objects can only be
accessed through the root.

```
Order (Aggregate Root)
  ├── OrderItems[] (Entities inside the aggregate)
  ├── DeliveryAddress (Value Object)
  └── OrderTotal (Calculated Value Object)
```

**Golden rule of the Aggregate:** Transactions do not cross aggregate boundaries.
If you need to modify two aggregates in one operation, use a Domain Event and a Saga.

### Business Rules
**Business Rules** (invariants) are the constraints the domain must always satisfy.
They live in the Aggregate Root and are validated on every operation.

---

## System entities

### Entity: MaintenanceRequest

**Context:** Maintenance

**Description:** A ticket created by a Person describing a repair or service need, with a type, description, and priority, moving through a defined status lifecycle (FR05–FR07). Per the glossary, avoid "ticket" or "PQR" as synonyms in code/docs.

**Attributes:**

| Attribute | Type | Description | Required | Rules |
|-----------|------|-------------|---------|-------|
| id | UUID | Unique identifier | Yes | Auto-generated on creation |
| personId | UUID | Who reported the request | Yes | Must exist in People Management |
| unitId | UUID | Where the issue occurred | Yes | Must exist in Units Management |
| type | string | Damage category (plumbing, electrical, etc.) | Yes | Controlled list, defined in `06-data/` |
| description | string | Report details | Yes | Max. 500 characters |
| priority | enum | LOW / MEDIUM / HIGH / URGENT | Yes | Set by the Person on creation |
| assignedTo | UUID | Assigned maintenance staff member | No | Assigned after creation |
| status | enum | PENDING / ASSIGNED / IN_PROGRESS / RESOLVED | Yes | See state machine |
| evidencePhotoUrls | string[] | Photos attached to the request | No | **Future candidate**, not required for the current delivery — flagged in `03-product/resi-complex-tech-watch.md` as a low-cost enhancement seen in competitor TUCO 360; confirm with the team before implementing |
| createdAt | DateTime | Creation date | Yes | Immutable, set on creation |
| updatedAt | DateTime | Last modification | Yes | Updated automatically |

**Ownership-level authorization:** a Person may only read their own requests
(`requests:read:own`); Maintenance Staff may only read/update requests assigned to them
(`requests:read:assigned`, `requests:update:assigned`) — enforced at the service layer,
not a constructor-level invariant.

**Lifecycle / States:**

```
PENDING ──(assign)──▶ ASSIGNED ──(start)──▶ IN_PROGRESS ──(resolve)──▶ RESOLVED
```

| State | Description | Allowed transitions |
|-------|-------------|---------------------|
| PENDING | Just created, no staff assigned yet | → ASSIGNED |
| ASSIGNED | Has assigned maintenance staff, not started yet | → IN_PROGRESS |
| IN_PROGRESS | Staff is actively working on it | → RESOLVED |
| RESOLVED | Terminal — work finished | Terminal state |

**Invariants (Business rules that MUST ALWAYS hold):**

```
INV-001: Staff cannot be assigned to a request without a valid unit or person
  - Rule: unitId and personId must exist before the request is created
  - Violation: the request is not saved if either is missing or invalid
  - Implementation: validated in the constructor / factory method

INV-002: Status can only move forward, never backward
  - Rule: PENDING → ASSIGNED → IN_PROGRESS → RESOLVED is the only valid sequence
  - Violation: DomainException if, e.g., an attempt is made to move from RESOLVED to IN_PROGRESS
  - Implementation: the status-change method validates the allowed transition

INV-003: Every request must have a priority on creation
  - Rule: priority cannot be empty or outside the enum
  - Violation: DomainException if priority is not LOW/MEDIUM/HIGH/URGENT
  - Implementation: validated in the constructor
```

**Code example (TypeScript/Java):**

```typescript
// TypeScript — Entity with invariants
class MaintenanceRequest {
  private constructor(
    private readonly id: RequestId,
    private readonly personId: PersonId,
    private readonly unitId: UnitId,
    private priority: Priority,
    private status: RequestStatus,
    private assignedTo: StaffId | null,
  ) {}

  static create(personId: PersonId, unitId: UnitId, type: string, description: string, priority: Priority): MaintenanceRequest {
    if (!personId || !unitId) {
      throw new DomainException('INV-001: personId and unitId are required');
    }
    return new MaintenanceRequest(
      RequestId.new(), personId, unitId, priority, RequestStatus.PENDING, null,
    );
  }

  assign(staffId: StaffId): void {
    if (this.status !== RequestStatus.PENDING) {
      throw new DomainException('INV-002: Only a PENDING request can be assigned');
    }
    this.assignedTo = staffId;
    this.status = RequestStatus.ASSIGNED;
    this.addEvent(new MaintenanceRequestStatusUpdatedEvent(this.id, this.status));
  }
}
```

---

### Entity: AdministrationFee

**Context:** Billing

**Description:** The periodic (monthly) charge generated for a Unit, calculated using a rate that differs by Unit type (residential vs. commercial) (FR08–FR10). Per the glossary, always "fee" in English docs — never "quota".

**Attributes:**

| Attribute | Type | Description | Required | Rules |
|-----------|------|-------------|---------|-------|
| id | UUID | Unique identifier | Yes | Auto-generated on creation |
| unitId | UUID | Billed unit | Yes | Must exist in Units Management |
| period | string | Fee month/year (e.g. `2026-09`) | Yes | Format `YYYY-MM` |
| unitTypeAtGeneration | enum | RESIDENTIAL / COMMERCIAL | Yes | Copied from the unit's type on generation; not recalculated afterward |
| amount | Money (VO) | Amount due according to the applicable rate | Yes | Must be > 0 |
| dueDate | Date | Deadline without a surcharge | Yes | After the generation date |
| status | enum | PENDING / PAID / OVERDUE | Yes | See state machine |
| createdAt | DateTime | Creation date | Yes | Immutable |
| updatedAt | DateTime | Last modification | Yes | Automatic |

**Ownership-level authorization:** a Person may only read fees for their own unit
(`fees:read:own`) — enforced at the service layer.

**Lifecycle / States:**

```
PENDING ──(register payment)──▶ PAID
    │
    └──(dueDate passes unpaid)──▶ OVERDUE ──(register payment)──▶ PAID
```

| State | Description | Allowed transitions |
|-------|-------------|---------------------|
| PENDING | Generated, within the deadline | → PAID, → OVERDUE |
| OVERDUE | Past the due date without payment | → PAID |
| PAID | Terminal — payment recorded | Terminal state |

**Invariants (Business rules that MUST ALWAYS hold):**

```
INV-001: The fee amount must always be greater than 0
  - Rule: amount > 0
  - Violation: a fee with amount <= 0 cannot be saved
  - Implementation: validated in the Money Value Object constructor

INV-002: The applied rate depends on the unit type AT THE MOMENT the fee is generated
  - Rule: unitTypeAtGeneration is copied once; if the unit later changes type,
    already-generated fees are not recalculated retroactively
  - Violation: modifying unitTypeAtGeneration after the fee is created throws an exception
  - Implementation: the field is read-only after the constructor

INV-003: A PAID fee cannot revert to PENDING or OVERDUE
  - Rule: PAID is a terminal state
  - Violation: DomainException if a revert is attempted
  - Implementation: the status-change method validates the current status
```

**Code example (TypeScript/Java):**

```typescript
class AdministrationFee {
  private constructor(
    private readonly id: FeeId,
    private readonly unitId: UnitId,
    private readonly period: string,
    private readonly unitTypeAtGeneration: UnitType,
    private readonly amount: Money,
    private status: FeeStatus,
  ) {}

  static generate(unitId: UnitId, period: string, unitType: UnitType, amount: Money): AdministrationFee {
    if (amount.isZeroOrNegative()) {
      throw new DomainException('INV-001: The fee amount must be greater than 0');
    }
    return new AdministrationFee(FeeId.new(), unitId, period, unitType, amount, FeeStatus.PENDING);
  }

  registerPayment(): void {
    if (this.status === FeeStatus.PAID) {
      throw new DomainException('INV-003: The fee has already been paid');
    }
    this.status = FeeStatus.PAID;
    this.addEvent(new FeePaidEvent(this.id));
  }
}
```

---

### Entity: Correspondence

**Context:** Access Control

**Description:** A package or piece of mail received at the complex and logged against a specific Unit, pending pickup by its occupant (FR15–FR16). Per the glossary, avoid "mail" alone (ambiguous with email); confirmed to belong to Access Control, not Communications, per `01-context/scope.md`.

**Attributes:**

| Attribute | Type | Description | Required | Rules |
|-----------|------|-------------|---------|-------|
| id | UUID | Unique identifier | Yes | Auto-generated on creation |
| unitId | UUID | Destination unit | Yes | Must exist in Units Management |
| description | string | Free-text note (e.g. courier, size) | No | Max. 200 characters |
| loggedBy | UUID | Security Guard who logged it | Yes | Must be a `SECURITY_GUARD` user |
| status | enum | PENDING / DELIVERED | Yes | See state machine |
| receivedAt | DateTime | When it was logged | Yes | Immutable, set on creation |
| deliveredAt | DateTime | When it was picked up | No | Set only on delivery |

**Ownership-level authorization:** a Person may only read correspondence for their own
unit (`correspondence:read:own`); only a Security Guard may create entries
(`correspondence:create`) — enforced at the service layer.

**Lifecycle / States:**

```
PENDING ──(mark delivered)──▶ DELIVERED
```

| State | Description | Allowed transitions |
|-------|-------------|---------------------|
| PENDING | Logged, not yet picked up | → DELIVERED |
| DELIVERED | Terminal — picked up by the occupant | Terminal state |

**Invariants (Business rules that MUST ALWAYS hold):**

```
INV-001: Correspondence must reference a valid, existing unit
  - Rule: unitId must exist at the time of logging
  - Violation: the record is not saved if the unit is invalid
  - Implementation: validated in the constructor / factory method

INV-002: DELIVERED is a terminal state
  - Rule: once marked DELIVERED, it cannot revert to PENDING
  - Violation: DomainException if a revert is attempted
  - Implementation: the status-change method validates the current status
```

**Code example (TypeScript/Java):**

```typescript
class Correspondence {
  private constructor(
    private readonly id: CorrespondenceId,
    private readonly unitId: UnitId,
    private readonly loggedBy: GuardId,
    private status: CorrespondenceStatus,
  ) {}

  static log(unitId: UnitId, loggedBy: GuardId, description?: string): Correspondence {
    if (!unitId) {
      throw new DomainException('INV-001: unitId is required');
    }
    return new Correspondence(CorrespondenceId.new(), unitId, loggedBy, CorrespondenceStatus.PENDING);
  }

  markDelivered(): void {
    if (this.status === CorrespondenceStatus.DELIVERED) {
      throw new DomainException('INV-002: Already marked as delivered');
    }
    this.status = CorrespondenceStatus.DELIVERED;
    this.addEvent(new CorrespondenceDeliveredEvent(this.id));
  }
}
```

---

## System Value Objects

### Value Object: Money (Fee / expense amount)

**Description:** Represents a monetary amount in Colombian pesos (COP), used in `AdministrationFee` and `ExpenseProposal`.

**Attributes:**

| Attribute | Type | Description |
|-----------|------|-------------|
| amount | decimal | Amount, with 2 decimal places |
| currency | string | Fixed at `COP` for this project |

**Validation rules:**

```
- amount must be a number with at most 2 decimal places
- amount cannot be negative
- currency is always "COP" (no multi-currency support in the MVP scope)
```

**Example:**

```typescript
class Money {
  private readonly amount: number;
  private readonly currency: string;

  constructor(amount: number, currency: string = 'COP') {
    if (amount < 0) {
      throw new DomainException(`Invalid amount: ${amount}`);
    }
    this.amount = Math.round(amount * 100) / 100;
    this.currency = currency;
  }

  isZeroOrNegative(): boolean { return this.amount <= 0; }

  add(other: Money): Money { return new Money(this.amount + other.amount, this.currency); }
}
```

---

### Value Object: UnitType

**Description:** Classifies a unit as residential or commercial (FR03). It is the basis for the differentiated fee rate (FR09) and for segmenting announcements (FR11).

**Attributes:**

| Attribute | Type | Description |
|-----------|------|-------------|
| value | enum | `RESIDENTIAL` \| `COMMERCIAL` |

**Validation rules:**

```
- Can only take one of the two enum values
- If COMMERCIAL, the Unit must have an associated Commercial Establishment (FR04)
```

**Example:**

```typescript
enum UnitType { RESIDENTIAL = 'RESIDENTIAL', COMMERCIAL = 'COMMERCIAL' }
```

---

### Value Object: Email

**Description:** Email address used by `IAM` (credentials) and `People Management` (contact info).

**Attributes:**

| Attribute | Type | Description |
|-----------|------|-------------|
| value | string | Email address, normalized to lowercase |

**Validation rules:**

```
- The email must have a valid format: text@domain.extension
- The extension must be at least 2 characters long
```

**Example:**

```typescript
// Value Object — Immutable, validated in the constructor
class Email {
  private readonly value: string;

  constructor(email: string) {
    if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)) {
      throw new DomainException(`Invalid email: ${email}`);
    }
    this.value = email.toLowerCase();
  }

  toString(): string { return this.value; }

  equals(other: Email): boolean { return this.value === other.value; }
}
```

---

## System Aggregates

### Aggregate: Unit

**Aggregate Root:** Unit

**Internal entities:**
- `Commercial Establishment` — only exists when the Unit is of type COMMERCIAL; has no business meaning outside its Unit (FR04)

**Value Objects:**
- `UnitType`

**Aggregate invariants:**

```
AGGR-INV-001: If unitType = COMMERCIAL, the Unit must have a Commercial Establishment
AGGR-INV-002: If unitType = RESIDENTIAL, the Unit MUST NOT have a Commercial Establishment
AGGR-INV-003: unitType cannot change if Fees or Requests already exist in another
              context for that period (resolved via event + cross-context validation,
              not inside this aggregate, since Fee and Request live in other contexts)
```

**Why do these objects form an aggregate?**
> A Commercial Establishment cannot exist without a Unit that contains it, and its data
> (name, type, opening hours) only makes sense together with the unit's classification.
> Keeping them together guarantees that a commercial unit is never created without
> establishment data, and that no establishment is left "orphaned."

---

### Aggregate: MaintenanceRequest

**Aggregate Root:** MaintenanceRequest

**Internal entities:**
- None — this is a simple, single-entity aggregate. `personId`, `unitId`, and
  `assignedTo` are ID references to other contexts (People Management, Units
  Management, and maintenance staff via IAM), not full objects, to avoid crossing
  aggregate boundaries.

**Value Objects:**
- `Priority` (enum: LOW / MEDIUM / HIGH / URGENT)
- `RequestStatus` (enum)

**Aggregate invariants:**

```
AGGR-INV-001: No status transition may skip a step (see entity INV-002)
AGGR-INV-002: assignedTo can only be set once the status leaves PENDING
```

**Why do these objects form an aggregate?**
> This is deliberately kept as a small aggregate: the request owns its own lifecycle
> (status, priority, assignment), but does not need to carry full Person or Unit data —
> that would create strong coupling between `Maintenance` and two other bounded
> contexts. Referencing by ID and synchronizing via events (`domain-events.md`) keeps
> the contexts independent.

---

### Aggregate: Visit

**Aggregate Root:** Visit

**Internal entities:**
- None — a Visit is a simple, single-entity aggregate.

**Value Objects:**
- `VisitorVehicle` (plate, vehicle type — optional, embedded)
- `VisitorType` (enum: PERSONAL_VISITOR / COMMERCIAL_CLIENT)

**Aggregate invariants:**

```
AGGR-INV-001: unitId must reference an existing Unit
AGGR-INV-002: A Visit's vehicle, if present, must have both plate and vehicleType set
              (no partial vehicle data)
```

**Why do these objects form an aggregate?**
> The Visitor Vehicle only has meaning tied to the Visit that produced it (FR13: "a
> vehicle associated with a Visit"). Keeping it embedded avoids creating a fourth
> cross-referenced entity for a record that never exists independently of a Visit.

---

### Aggregate: Correspondence

**Aggregate Root:** Correspondence

**Internal entities:**
- None — a simple, single-entity aggregate, sharing the Access Control bounded
  context with `Visit` but modeled as its own aggregate since it has an independent
  lifecycle (pending → delivered) unrelated to any specific Visit.

**Value Objects:**
- `CorrespondenceStatus` (enum: PENDING / DELIVERED)

**Aggregate invariants:**

```
AGGR-INV-001: unitId must reference an existing Unit
AGGR-INV-002: DELIVERED is a terminal state (see entity INV-002)
```

**Why is this a separate aggregate from Visit, even in the same context?**
> Both are logged by the Security Guard at the front desk, which is why they share a
> bounded context — but a piece of correspondence is not tied to any single Visit's
> transaction boundary, and its own lifecycle (pending/delivered) has nothing to do
> with a visitor entering or leaving. Modeling them as one aggregate would force every
> correspondence update to also lock an unrelated Visit record.

---

### Aggregate: ExpenseProposal

**Aggregate Root:** ExpenseProposal

**Internal entities:**
- `Approval[]` — each Board of Trustees member's vote/decision on the proposal; has no
  meaning outside the proposal it belongs to (FR18, permanent history)

**Value Objects:**
- `Money` (requestedAmount)
- `ProposalStatus` (enum: UNDER_REVIEW / APPROVED / REJECTED)

**Aggregate invariants:**

```
AGGR-INV-001: A proposal in APPROVED or REJECTED (terminal) status does not accept new Approvals
AGGR-INV-002: The Approval history is never deleted, only appended (append-only) —
              this is what guarantees the traceability required by FR18
AGGR-INV-003: requestedAmount must be greater than 0
```

**Why do these objects form an aggregate?**
> The approval/rejection history (FR18: "with permanent history") only makes sense as
> part of the proposal it responds to. Keeping them as a single aggregate guarantees
> that the record of who (which Board of Trustees member) decided what and when is
> always consistent with the proposal's final status, and that a past decision can
> never be edited outside the proposal itself.

---

## Summary table of tactical building blocks

| Name | Type | Bounded Context | Aggregate Root? |
|------|------|----------------|----------------|
| Unit | Entity | Units Management | Yes |
| Commercial Establishment | Entity | Units Management | No (inside Unit) |
| Person | Entity | People Management | Yes |
| Maintenance Request | Entity | Maintenance | Yes |
| Administration Fee | Entity | Billing | Yes |
| Announcement | Entity | Communications | Yes |
| Visit | Entity | Access Control | Yes |
| Correspondence | Entity | Access Control | Yes (separate aggregate from Visit) |
| Expense Proposal | Entity | Financial Approval | Yes |
| Approval | Entity | Financial Approval | No (inside Expense Proposal) |
| Money | Value Object | Shared (Billing, Financial Approval) | N/A |
| UnitType | Value Object | Units Management (shared by reference) | N/A |
| Email | Value Object | Shared (IAM, People Management) | N/A |
| Visitor Vehicle | Value Object | Access Control (inside Visit) | N/A |
| FeeCalculationService | Domain Service | Billing | N/A |

---

## Domain Services

A **Domain Service** is business logic that does not naturally belong to any entity.
Use it when:
- The operation involves multiple entities or aggregates
- It would be unnatural for the operation to belong to a single entity
- The logic does not need its own state

```typescript
// Domain Service — Stateless, orchestrates logic between entities
// resi-complex equivalent: calculates a fee's amount based on the unit's type
class FeeCalculationService {
  calculateAmount(unitType: UnitType, residentialRate: Money, commercialRate: Money): Money {
    return unitType === UnitType.COMMERCIAL ? commercialRate : residentialRate;
  }
}
```

> Note: rates (`residentialRate`, `commercialRate`) are configuration managed by the
> Administrator, not fixed values in code — their data model is defined in `06-data/`.

---

## Correlation with code

| Domain artifact | Package / folder in code | File |
|----------------|--------------------------|------|
| Aggregate Root `MaintenanceRequest` | `src/domain/maintenance/` | `MaintenanceRequest.java` |
| Aggregate Root `AdministrationFee` | `src/domain/billing/` | `AdministrationFee.java` |
| Aggregate Root `Correspondence` | `src/domain/access-control/` | `Correspondence.java` |
| Aggregate Root `ExpenseProposal` | `src/domain/finance-approval/` | `ExpenseProposal.java` |
| Value Object `Email` | `src/domain/shared/valueobjects/` | `Email.java` |
| Value Object `Money` | `src/domain/shared/valueobjects/` | `Money.java` |
| Domain Service `FeeCalculationService` | `src/domain/billing/services/` | `FeeCalculationService.java` |
| Repository `MaintenanceRequestRepository` | `src/domain/maintenance/ports/` | `MaintenanceRequestRepository.java` |

> See hexagonal structure in `05-architecture/hexagonal-architecture.md`