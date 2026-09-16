# Domain Events

> **What to fill in here:** A domain event is a fact that occurred in the business.
> They are the backbone of asynchronous communication between bounded contexts.
> The name is ALWAYS in past tense and in the ubiquitous language of the domain.

---

## What is a domain event?

A **Domain Event** communicates that something important occurred in the business.
It is an immutable message that describes the fact in past tense.

```
✓ OrderCreated
✓ PaymentRejected
✓ UserRegistered
✓ StockDepleted

✗ CreateOrder (this is a command, not an event)
✗ OrderUpdated (too generic — what changed?)
✗ OrderEvent (does not indicate what occurred)
```

### Difference between Command and Event

| Concept | Intent | Tense | Can fail? |
|---------|--------|-------|-----------|
| **Command** | Instruction to do something | Present | Yes |
| **Event** | Notification of something that occurred | Past | No (it already happened) |

```
User → [CreateOrder] → System → [OrderCreated] → Other contexts
          (Command)                  (Event)
```

---

## Event catalog

### Event: UnitRegistered

| Field | Value |
|-------|-------|
| **Name** | `UnitRegistered` |
| **Bounded Context** | Units Management |
| **Aggregate** | Unit |
| **Trigger** | The Administrator registers a new unit, classifying it as residential or commercial (FR02, FR03) |
| **Consumers** | `billing-service` (to know which fee rate applies), `access-control-service` (to validate the destination unit), `communications-service` (to segment announcements), `reports-service` |
| **Channel (topic)** | `units.unit.registered` |
| **Schema version** | `v1` |
| **Delivery guarantee** | At-least-once |

**Payload (JSON schema):**

```json
{
  "eventId": "550e8400-e29b-41d4-a716-446655440000",
  "eventType": "UnitRegistered",
  "aggregateId": "550e8400-e29b-41d4-a716-446655440001",
  "aggregateType": "Unit",
  "occurredAt": "2024-01-15T10:30:00Z",
  "version": 1,
  "payload": {
    "unitNumber": "string — visible identifier (e.g. 'Apt 302', 'Shop 5')",
    "unitType": "string enum — RESIDENTIAL | COMMERCIAL",
    "towerOrBlock": "string — optional, physical grouping",
    "area": "number — area in m², optional"
  },
  "metadata": {
    "correlationId": "550e8400-e29b-41d4-a716-446655440002",
    "causationId": "550e8400-e29b-41d4-a716-446655440003",
    "userId": "550e8400-e29b-41d4-a716-446655440004"
  }
}
```

**Real payload example:**

```json
{
  "eventId": "generated-uuid",
  "eventType": "UnitRegistered",
  "aggregateId": "unit-uuid",
  "aggregateType": "Unit",
  "occurredAt": "2024-01-15T10:30:00Z",
  "version": 1,
  "payload": {
    "unitNumber": "Shop 5",
    "unitType": "COMMERCIAL",
    "towerOrBlock": "Commercial Zone",
    "area": 45.5
  }
}
```

**What do consumers do with this event?**

| Consuming service | Action | Idempotent? |
|------------------|--------|-------------|
| `billing-service` | Creates the fee configuration (rate based on `unitType`) for that unit | Yes — uses `eventId` as the idempotency key |
| `access-control-service` | Caches the unit as a valid destination for logging visits and correspondence | Yes — upsert by `aggregateId` |
| `communications-service` | Adds the unit to the matching announcement segment (residential/commercial) | Yes — upsert by `aggregateId` |

---

### Event: MaintenanceRequestCreated

| Field | Value |
|-------|-------|
| **Name** | `MaintenanceRequestCreated` |
| **Bounded Context** | Maintenance |
| **Aggregate** | MaintenanceRequest |
| **Trigger** | A Person (resident or commercial owner/tenant) creates a maintenance request (FR05) |
| **Consumers** | `communications-service` (in-app notification — FR07), `reports-service` |
| **Channel (topic)** | `maintenance.request.created` |
| **Schema version** | `v1` |
| **Delivery guarantee** | At-least-once |

**Payload (JSON schema):**

```json
{
  "eventId": "uuid",
  "eventType": "MaintenanceRequestCreated",
  "aggregateId": "uuid — request id",
  "aggregateType": "MaintenanceRequest",
  "occurredAt": "ISO 8601",
  "version": 1,
  "payload": {
    "personId": "uuid — who reported it",
    "unitId": "uuid — where it happened",
    "type": "string — e.g. plumbing, electrical, structural",
    "description": "string",
    "priority": "string enum — LOW | MEDIUM | HIGH | URGENT",
    "status": "string — PENDING (initial status)",
    "evidencePhotoUrls": "string[] — optional, future candidate per tech watch (not required for MVP)"
  },
  "metadata": { "correlationId": "uuid", "causationId": "uuid", "userId": "uuid" }
}
```

**Real payload example:**

```json
{
  "eventId": "generated-uuid",
  "eventType": "MaintenanceRequestCreated",
  "aggregateId": "request-uuid",
  "aggregateType": "MaintenanceRequest",
  "occurredAt": "2024-01-15T10:30:00Z",
  "version": 1,
  "payload": {
    "personId": "person-uuid",
    "unitId": "unit-uuid",
    "type": "plumbing",
    "description": "Water leak in the main bathroom",
    "priority": "URGENT",
    "status": "PENDING"
  }
}
```

**What do consumers do with this event?**

| Consuming service | Action | Idempotent? |
|------------------|--------|-------------|
| `communications-service` | Sends an in-app notification; if `priority = URGENT`, flags it (see Policy below) | Yes — uses `eventId` to avoid duplicate notifications |
| `reports-service` | Updates the count of requests by status (FR19) | Yes — idempotent projection by `aggregateId` |

> `evidencePhotoUrls` is listed as optional because `resi-complex-tech-watch.md`
> flags photo evidence (seen in competitor TUCO 360) as a low-cost enhancement worth
> evaluating for this delivery — confirm with the team before treating it as required.

---

### Event: FeeGenerated

| Field | Value |
|-------|-------|
| **Name** | `FeeGenerated` |
| **Bounded Context** | Billing |
| **Aggregate** | AdministrationFee |
| **Trigger** | The Administrator's monthly process that generates fees per unit, with a rate differentiated by type (FR08, FR09) |
| **Consumers** | `communications-service` (notifies the person), `reports-service` (pending arrears — FR19) |
| **Channel (topic)** | `billing.fee.generated` |
| **Schema version** | `v1` |
| **Delivery guarantee** | At-least-once |

**Payload (JSON schema):**

```json
{
  "eventId": "uuid",
  "eventType": "FeeGenerated",
  "aggregateId": "uuid — fee id",
  "aggregateType": "AdministrationFee",
  "occurredAt": "ISO 8601",
  "version": 1,
  "payload": {
    "unitId": "uuid",
    "period": "string — e.g. '2026-09'",
    "unitTypeAtGeneration": "string enum — RESIDENTIAL | COMMERCIAL",
    "amount": "number — amount in COP based on the rate in effect",
    "dueDate": "date",
    "status": "string — PENDING (initial status)"
  },
  "metadata": { "correlationId": "uuid", "causationId": "uuid", "userId": "uuid" }
}
```

**Real payload example:**

```json
{
  "eventId": "generated-uuid",
  "eventType": "FeeGenerated",
  "aggregateId": "fee-uuid",
  "aggregateType": "AdministrationFee",
  "occurredAt": "2026-09-01T05:00:00Z",
  "version": 1,
  "payload": {
    "unitId": "unit-uuid",
    "period": "2026-09",
    "unitTypeAtGeneration": "COMMERCIAL",
    "amount": 350000.00,
    "dueDate": "2026-09-15",
    "status": "PENDING"
  }
}
```

**What do consumers do with this event?**

| Consuming service | Action | Idempotent? |
|------------------|--------|-------------|
| `communications-service` | Notifies the person associated with the unit that a fee was generated | Yes — `eventId` as idempotency key |
| `reports-service` | Adds the fee to the period's pending arrears (FR19) | Yes — idempotent projection |

---

### Event: VisitRegistered

| Field | Value |
|-------|-------|
| **Name** | `VisitRegistered` |
| **Bounded Context** | Access Control |
| **Aggregate** | Visit |
| **Trigger** | The Security Guard logs a visitor's entry, with destination unit and visitor type (FR12–FR14), optionally including a vehicle (FR13) |
| **Consumers** | `reports-service` |
| **Channel (topic)** | `access.visit.registered` |
| **Schema version** | `v1` |
| **Delivery guarantee** | At-least-once |

**Payload (JSON schema):**

```json
{
  "eventId": "uuid",
  "eventType": "VisitRegistered",
  "aggregateId": "uuid — visit id",
  "aggregateType": "Visit",
  "occurredAt": "ISO 8601",
  "version": 1,
  "payload": {
    "unitId": "uuid — destination unit",
    "visitorType": "string enum — PERSONAL_VISITOR | COMMERCIAL_CLIENT",
    "visitorName": "string",
    "vehicle": {
      "plate": "string — optional, present only if the visitor arrived by vehicle",
      "vehicleType": "string — optional, e.g. car, motorcycle"
    },
    "loggedBy": "uuid — Security Guard user id"
  },
  "metadata": { "correlationId": "uuid", "causationId": "uuid", "userId": "uuid" }
}
```

**Real payload example:**

```json
{
  "eventId": "generated-uuid",
  "eventType": "VisitRegistered",
  "aggregateId": "visit-uuid",
  "aggregateType": "Visit",
  "occurredAt": "2026-09-11T14:05:00Z",
  "version": 1,
  "payload": {
    "unitId": "unit-uuid",
    "visitorType": "COMMERCIAL_CLIENT",
    "visitorName": "Jane Doe",
    "vehicle": { "plate": "ABC123", "vehicleType": "car" },
    "loggedBy": "guard-uuid"
  }
}
```

**What do consumers do with this event?**

| Consuming service | Action | Idempotent? |
|------------------|--------|-------------|
| `reports-service` | Updates the visitor/vehicle log for reporting purposes | Yes — idempotent projection by `aggregateId` |

> The vehicle is an optional object inside the payload, not a separate event, since per
> FR13 it is always registered together with the visit, never on its own.

---

### Event: CorrespondenceReceived

| Field | Value |
|-------|-------|
| **Name** | `CorrespondenceReceived` |
| **Bounded Context** | Access Control |
| **Aggregate** | Correspondence |
| **Trigger** | The Security Guard logs an incoming package/letter for a unit (FR15) |
| **Consumers** | `communications-service` (so the Person can be notified), `reports-service` |
| **Channel (topic)** | `access.correspondence.received` |
| **Schema version** | `v1` |
| **Delivery guarantee** | At-least-once |

**Payload (JSON schema):**

```json
{
  "eventId": "uuid",
  "eventType": "CorrespondenceReceived",
  "aggregateId": "uuid — correspondence id",
  "aggregateType": "Correspondence",
  "occurredAt": "ISO 8601",
  "version": 1,
  "payload": {
    "unitId": "uuid",
    "description": "string — e.g. 'small box, courier XYZ'",
    "loggedBy": "uuid — Security Guard user id",
    "status": "string — PENDING (initial status)"
  },
  "metadata": { "correlationId": "uuid", "causationId": "uuid", "userId": "uuid" }
}
```

**Real payload example:**

```json
{
  "eventId": "generated-uuid",
  "eventType": "CorrespondenceReceived",
  "aggregateId": "correspondence-uuid",
  "aggregateType": "Correspondence",
  "occurredAt": "2026-09-11T09:15:00Z",
  "version": 1,
  "payload": {
    "unitId": "unit-uuid",
    "description": "Small box, courier Servientrega",
    "loggedBy": "guard-uuid",
    "status": "PENDING"
  }
}
```

**What do consumers do with this event?**

| Consuming service | Action | Idempotent? |
|------------------|--------|-------------|
| `communications-service` | Notifies the Person that they have pending correspondence, supporting FR16's "check pending correspondence" lookup | Yes — `eventId` as key |
| `reports-service` | Adds it to a pending-correspondence indicator | Yes — idempotent projection |

---

### Event: ExpenseProposalApproved

| Field | Value |
|-------|-------|
| **Name** | `ExpenseProposalApproved` |
| **Bounded Context** | Financial Approval |
| **Aggregate** | ExpenseProposal |
| **Trigger** | A Board of Trustees member (or the required quorum, to be defined) approves an extraordinary expense proposal (FR18) |
| **Consumers** | `communications-service` (notifies the Administrator), `reports-service` (decision history) |
| **Channel (topic)** | `finance.expense.approved` |
| **Schema version** | `v1` |
| **Delivery guarantee** | At-least-once |

**Payload (JSON schema):**

```json
{
  "eventId": "uuid",
  "eventType": "ExpenseProposalApproved",
  "aggregateId": "uuid — proposal id",
  "aggregateType": "ExpenseProposal",
  "occurredAt": "ISO 8601",
  "version": 1,
  "payload": {
    "proposalId": "uuid",
    "subject": "string",
    "requestedAmount": "number",
    "approvedBy": "uuid — Board of Trustees member",
    "justification": "string — optional"
  },
  "metadata": { "correlationId": "uuid", "causationId": "uuid", "userId": "uuid" }
}
```

**Real payload example:**

```json
{
  "eventId": "generated-uuid",
  "eventType": "ExpenseProposalApproved",
  "aggregateId": "proposal-uuid",
  "aggregateType": "ExpenseProposal",
  "occurredAt": "2026-09-05T14:00:00Z",
  "version": 1,
  "payload": {
    "proposalId": "proposal-uuid",
    "subject": "Water pump repair",
    "requestedAmount": 4200000.00,
    "approvedBy": "board-member-uuid",
    "justification": "Main pump out of warranty with an active leak"
  }
}
```

**What do consumers do with this event?**

| Consuming service | Action | Idempotent? |
|------------------|--------|-------------|
| `communications-service` | Notifies the Administrator that the expense can proceed | Yes — `eventId` as key |
| `reports-service` | Logs the decision in the approval history (FR18) | Yes — idempotent projection |

---

## Standard fields for all events

All events must include these fields in the envelope:

| Field | Type | Description |
|-------|------|-------------|
| `eventId` | UUID | Unique event ID (for idempotency) |
| `eventType` | string | Event name in PascalCase |
| `aggregateId` | UUID | ID of the aggregate that generated the event |
| `aggregateType` | string | Aggregate type |
| `occurredAt` | ISO 8601 | When the business fact occurred |
| `version` | integer | Schema version (for evolution) |
| `payload` | object | Event data (specific per type) |
| `metadata.correlationId` | UUID | For tracing a transaction across services |
| `metadata.causationId` | UUID | ID of the event or command that caused this event |
| `metadata.userId` | UUID | User who initiated the chain (if applicable) |

---

## Event flow: Urgent maintenance request

```
Person (resident or commercial owner/tenant)
  │
  │  CreateRequest (command)
  ▼
[Aggregate: MaintenanceRequest]
  │
  │  MaintenanceRequestCreated (event)
  ├────────────────────────────────────────┐
  │                                          ▼
  │                                [Service: Communications]
  │                                Policy: if priority = URGENT,
  │                                notify the maintenance team
  │                                immediately
  │
  │  (later)
  │  AssignRequest (command, Administrator)
  ▼
[Aggregate: MaintenanceRequest]
  │
  │  MaintenanceRequestStatusUpdated (event)
  └────────────────────────────────────────┐
                                             ▼
                                   [Service: Communications]
                                   Notifies the resident of the
                                   status change (FR07)
```

## Event flow: Front-desk correspondence

```
Security Guard
  │
  │  LogCorrespondence (command)
  ▼
[Aggregate: Correspondence]
  │
  │  CorrespondenceReceived (event)
  ├────────────────────────────────────────┐
  │                                          ▼
  │                                [Service: Communications]
  │                                Notifies the Person that they
  │                                have pending correspondence
  │
  │  (later, Person picks it up in person — logged by the Guard)
  │  MarkCorrespondenceDelivered (command)
  ▼
[Aggregate: Correspondence]
  │
  │  CorrespondenceDelivered (event)
  └────────────────────────────────────────┐
                                             ▼
                                   [Service: Reports]
                                   Updates the pending-correspondence
                                   indicator (FR16)
```

## Event flow: Extraordinary expense approval

```
Administrator
  │
  │  CreateExpenseProposal (command)
  ▼
[Aggregate: ExpenseProposal]
  │
  │  ExpenseProposalCreated (event)
  ├────────────────────────────────────────┐
  │                                          ▼
  │                                [Service: Communications]
  │                                Notifies the Board of Trustees
  │
  │  (Board decides)
  │  ApproveProposal / RejectProposal (command)
  ▼
[Aggregate: ExpenseProposal]
  │
  │  ExpenseProposalApproved / ExpenseProposalRejected (event)
  └────────────────────────────────────────┐
                                             ▼
                                   [Service: Communications]
                                   Notifies the Administrator
                                   [Service: Reports]
                                   Logs the decision history (FR18)
```

---

## Schema evolution strategy

Events are contracts. Changing them in an incompatible way breaks consumers.

### What is a compatible change (does not break)?

```
✓ Add a new optional field to the payload
✓ Add a new event type
✓ Change a required field → optional
```

### What is an incompatible change (breaks)?

```
✗ Remove a field from the payload
✗ Change the type of a field (string → number)
✗ Change an optional field → required
✗ Change the event name
```

### How to evolve a schema without breaking consumers

**Strategy: Version the event**

```
Step 1: Publish EventV2 (new type with incompatible changes)
Step 2: Publish both EventV1 and EventV2 during the migration period
Step 3: Migrate consumers to V2 one by one
Step 4: Deprecate EventV1 (announce 1 sprint in advance)
Step 5: Stop publishing EventV1
```

---

## Event summary table

| Event | Origin context | Topic | Consumers | Version |
|-------|---------------|-------|-----------|---------|
| UnitRegistered | Units Management | `units.unit.registered` | billing, access-control, communications, reports | v1 |
| UnitUpdated | Units Management | `units.unit.updated` | billing, access-control, communications, reports | v1 |
| CommercialEstablishmentRegistered | Units Management | `units.establishment.registered` | communications, reports | v1 |
| PersonRegistered | People Management | `people.person.registered` | billing, maintenance, access-control, communications | v1 |
| MaintenanceRequestCreated | Maintenance | `maintenance.request.created` | communications, reports | v1 |
| MaintenanceRequestStatusUpdated | Maintenance | `maintenance.request.status-updated` | communications, reports | v1 |
| FeeGenerated | Billing | `billing.fee.generated` | communications, reports | v1 |
| FeePaid | Billing | `billing.fee.paid` | reports | v1 |
| AnnouncementPublished | Communications | `communications.announcement.published` | reports | v1 |
| VisitRegistered | Access Control | `access.visit.registered` | reports | v1 |
| VisitEnded | Access Control | `access.visit.ended` | reports | v1 |
| CorrespondenceReceived | Access Control | `access.correspondence.received` | communications, reports | v1 |
| CorrespondenceDelivered | Access Control | `access.correspondence.delivered` | reports | v1 |
| ExpenseProposalCreated | Financial Approval | `finance.expense.created` | communications, reports | v1 |
| ExpenseProposalApproved | Financial Approval | `finance.expense.approved` | communications, reports | v1 |
| ExpenseProposalRejected | Financial Approval | `finance.expense.rejected` | communications, reports | v1 |

> `PaymentRegistered` is not included as its own event because online payments are
> **out of the MVP scope** (`01-context/scope.md`, item 3). If payments are recorded,
> it would be done manually by the Administrator, modeled as `FeePaid` (a status
> change on the fee), not as a separate `Payment` entity with its own events — to be
> confirmed with the team.
>
> `VisitorVehicleRegistered` is not listed separately: the vehicle (plate, type) is an
> optional object inside the `VisitRegistered` payload, since per FR13 it is always
> registered together with the visit.
>
> Topic prefixes (`units.*`, `people.*`, `maintenance.*`, `billing.*`,
> `communications.*`, `access.*`, `finance.*`) intentionally mirror the RBAC resource
> names in `00-governance/security-policy.md` (`units`, `people`, `requests`, `fees`,
> `announcements`, `visits`/`vehicles`/`correspondence`, `expenses`) so the same
> vocabulary is used for permissions, REST paths, and event topics.

---

## Policies — Reactions to events

A **Policy** (or Saga step) describes what happens automatically when an event arrives.
It is the logic of "whenever X occurs, do Y".

```
Event:  OrderCreated
Policy: Whenever an OrderCreated arrives with type=URGENT,
        emit the command NotifyOperationsTeam
```

| Trigger event | Policy | Emitted command | Service |
|--------------|--------|----------------|---------|
| `MaintenanceRequestCreated` | Whenever `priority = URGENT`, notify maintenance staff immediately | `NotifyMaintenanceTeam` | `communications-service` |
| `MaintenanceRequestStatusUpdated` | Whenever the status changes, notify the Person who reported it | `NotifyRequestStatusChange` | `communications-service` |
| `FeeGenerated` | Whenever a fee is generated, notify the Person associated with the unit | `NotifyFeeGenerated` | `communications-service` |
| `CorrespondenceReceived` | Whenever correspondence is logged, notify the Person associated with the unit | `NotifyCorrespondencePending` | `communications-service` |
| `ExpenseProposalCreated` | Whenever a proposal is created, notify all Board of Trustees members | `NotifyBoardOfTrustees` | `communications-service` |
| `ExpenseProposalApproved` / `ExpenseProposalRejected` | Whenever the Board decides, notify the Administrator | `NotifyProposalDecision` | `communications-service` |

---

## Resilience patterns for events

### At-least-once delivery + Idempotency

The message broker guarantees the event is delivered **at least once** but it may be
delivered more than once (in case of retries). Consumers must be **idempotent**.

```typescript
// Idempotent consumer — stores the processed eventId
async function processOrderCreatedEvent(event: OrderCreated): Promise<void> {
  // 1. Check if already processed
  if (await isEventAlreadyProcessed(event.eventId)) {
    logger.info(`Event ${event.eventId} already processed, ignoring`);
    return;
  }

  // 2. Process the event
  await updateModel(event.payload);

  // 3. Mark as processed (in the same transaction)
  await markEventProcessed(event.eventId);
}
```

### Dead Letter Queue (DLQ)

When an event fails after N retries, it goes to the DLQ.

| Configuration | Recommended value |
|--------------|------------------|
| Retries before DLQ | 3-5 |
| Backoff | Exponential (1s → 2s → 4s → 8s) |
| DLQ retention | 7 days |
| Alert | When DLQ has > 0 messages |

> See DLQ runbook in `09-microservices/services/XX-service/runbook.md`