# User Stories — Backlog

> **What to fill in here:** The product's User Story backlog.
> Each HU uses the standard format with Acceptance Criteria in Given/When/Then.
> Refined (Ready) HUs go to the sprint. Unrefined ones are epics or ideas.

---

## Backlog status

| Cut | Sprint | Total HUs | Refined | In progress | Completed |
|-----|--------|-----------|---------|-------------|-----------|
| Cut 1 | Sprint 1-2 | 5 | 5 | 0 | 0 |
| Cut 2 | Sprint 3-4 | 10 | 10 | 0 | 0 |
| Cut 3 | Sprint 5+ | 4 | 4 | 0 | 0 |

---

## Epics

| ID | Epic | Description |
|----|------|-------------|
| EP-001 | IAM | Authenticate users and resolve role-based permissions for the system's 5 roles |
| EP-002 | Units Management | Register and classify units (residential/commercial) and their commercial establishment data |
| EP-003 | People Management | Register residents and commercial owners/tenants and link them to a unit |
| EP-004 | Maintenance | Manage the lifecycle of maintenance requests, from creation to resolution |
| EP-005 | Billing | Generate and track differentiated administration fees by unit type |
| EP-006 | Communications | Publish announcements segmented by unit type |
| EP-007 | Access Control | Log visitor/vehicle entry-exit and incoming correspondence at the front desk |
| EP-008 | Financial Approval | Manage extraordinary expense proposals and the Board of Trustees' approval history |
| EP-009 | Reports | Consolidate read-only views (requests by status, pending fee arrears) from other services' events |

---

## User Stories

### HU-IAM-001 — User login with role-based access {#HU-IAM-001}

**Epic:** EP-001

> **As** a user with one of the 5 authentication roles (Administrator, Board of Trustees, Person, Maintenance Staff, Security Guard)
> **I want** to log in with my email and password
> **so that** I can access only the features my role allows

**Acceptance Criteria:**

```gherkin
Scenario 1: Successful login
  Given a registered user with valid credentials
  When  they submit email and password
  Then  the system returns a JWT with their role and a 1-hour expiration
  And   a refresh token valid for 7 days is issued

Scenario 2: Invalid credentials
  Given a user submits the wrong password
  When  they attempt to log in
  Then  the system responds 401 with a generic error message, without revealing whether the email exists
```

**Definition of Done:**
- [ ] Code reviewed and approved
- [ ] Unit tests written
- [ ] Acceptance criteria verified (manual or automated)
- [ ] API contract updated if applicable
- [ ] Deployed to staging

| Field | Value |
|-------|-------|
| Story Points | 5 |
| Priority | Must Have |
| Target sprint | Sprint 1 |
| Assigned to | Pending — to be assigned by the team |
| Status | Backlog |
| Dependencies | — |
| Requisito (FR) | FR01 |
| Affected service(s) | iam-service |

---

### HU-PPL-001 — Register a Person linked to a unit {#HU-PPL-001}

**Epic:** EP-003

> **As** an Administrator
> **I want** to register a Person (Resident or Commercial Owner/Tenant) linked to a unit
> **so that** the system knows who occupies each unit and can bill and notify them correctly

**Acceptance Criteria:**

```gherkin
Scenario 1: Register a resident
  Given a residential unit already exists
  When  the Administrator registers a Person with type Resident and links them to that unit
  Then  the Person is saved and appears as an occupant of the unit

Scenario 2: Register a person against a non-existent unit
  Given the Administrator provides a unitId that does not exist
  When  they try to register the Person
  Then  the system rejects the request with a validation error
```

**Definition of Done:**
- [ ] Code reviewed and approved
- [ ] Unit tests written
- [ ] Acceptance criteria verified (manual or automated)
- [ ] API contract updated if applicable
- [ ] Deployed to staging

| Field | Value |
|-------|-------|
| Story Points | 5 |
| Priority | Must Have |
| Target sprint | Sprint 1 |
| Assigned to | Pending — to be assigned by the team |
| Status | Backlog |
| Dependencies | HU-IAM-001, HU-UNIT-001 |
| Requisito (FR) | FR01 |
| Affected service(s) | people-service |

---

### HU-UNIT-001 — Register a unit {#HU-UNIT-001}

**Epic:** EP-002

> **As** an Administrator
> **I want** to register a unit and classify it as residential or commercial
> **so that** the system can later apply the correct fee rate and segment announcements correctly

**Acceptance Criteria:**

```gherkin
Scenario 1: Register a residential unit
  Given I am logged in as Administrator
  When  I register a unit with type RESIDENTIAL
  Then  the unit is saved and a UnitRegistered event is published

Scenario 2: Register a commercial unit without establishment data
  Given I register a unit with type COMMERCIAL
  When  I do not provide the commercial establishment data
  Then  the system rejects the request, since a commercial unit must have an establishment (AGGR-INV-001)
```

**Definition of Done:**
- [ ] Code reviewed and approved
- [ ] Unit tests written
- [ ] Acceptance criteria verified (manual or automated)
- [ ] API contract updated if applicable
- [ ] Deployed to staging

| Field | Value |
|-------|-------|
| Story Points | 3 |
| Priority | Must Have |
| Target sprint | Sprint 1 |
| Assigned to | Pending — to be assigned by the team |
| Status | Backlog |
| Dependencies | HU-IAM-001 |
| Requisito (FR) | FR02 |
| Affected service(s) | units-service |

---

### HU-UNIT-002 — Edit and delete a unit {#HU-UNIT-002}

**Epic:** EP-002

> **As** an Administrator
> **I want** to edit or delete an existing unit
> **so that** I can correct registration mistakes or remove units that no longer exist

**Acceptance Criteria:**

```gherkin
Scenario 1: Edit a unit's data
  Given an existing unit
  When  the Administrator updates its data
  Then  the change is saved and a UnitUpdated event is published

Scenario 2: Delete a unit with active fees
  Given a unit has pending or overdue fees
  When  the Administrator tries to delete it
  Then  the system blocks the deletion until those fees are resolved
```

**Definition of Done:**
- [ ] Code reviewed and approved
- [ ] Unit tests written
- [ ] Acceptance criteria verified (manual or automated)
- [ ] API contract updated if applicable
- [ ] Deployed to staging

| Field | Value |
|-------|-------|
| Story Points | 3 |
| Priority | Should Have |
| Target sprint | Sprint 1 |
| Assigned to | Pending — to be assigned by the team |
| Status | Backlog |
| Dependencies | HU-UNIT-001 |
| Requisito (FR) | FR03 |
| Affected service(s) | units-service |

---

### HU-UNIT-003 — Register commercial establishment data {#HU-UNIT-003}

**Epic:** EP-002

> **As** an Administrator
> **I want** to register the commercial establishment data (name, business type, operating hours) for a commercial unit
> **so that** the system has the information needed to operate and display that unit correctly

**Acceptance Criteria:**

```gherkin
Scenario 1: Register establishment for a commercial unit
  Given a unit already exists with type COMMERCIAL
  When  the Administrator registers its establishment name, business type, and operating hours
  Then  a CommercialEstablishmentRegistered event is published
```

**Definition of Done:**
- [ ] Code reviewed and approved
- [ ] Unit tests written
- [ ] Acceptance criteria verified (manual or automated)
- [ ] API contract updated if applicable
- [ ] Deployed to staging

| Field | Value |
|-------|-------|
| Story Points | 2 |
| Priority | Must Have |
| Target sprint | Sprint 1 |
| Assigned to | Pending — to be assigned by the team |
| Status | Backlog |
| Dependencies | HU-UNIT-001 |
| Requisito (FR) | FR04 |
| Affected service(s) | units-service |

---

### HU-MAINT-001 — Create a maintenance request {#HU-MAINT-001}

**Epic:** EP-004

> **As** a Person (Resident or Commercial Owner/Tenant)
> **I want** to create a maintenance request describing the type, description, and priority of the issue
> **so that** the Administrator can assign it and I can track its progress

**Acceptance Criteria:**

```gherkin
Scenario 1: Successful request creation
  Given I am logged in as Person
  When  I create a request with type, description, and priority
  Then  the request is saved with status PENDING and a MaintenanceRequestCreated event is published

Scenario 2: Missing priority
  Given I do not select a priority
  When  I try to submit the request
  Then  the system rejects it (INV-003: priority is required)
```

**Definition of Done:**
- [ ] Code reviewed and approved
- [ ] Unit tests written
- [ ] Acceptance criteria verified (manual or automated)
- [ ] API contract updated if applicable
- [ ] Deployed to staging

| Field | Value |
|-------|-------|
| Story Points | 3 |
| Priority | Must Have |
| Target sprint | Sprint 3 |
| Assigned to | Pending — to be assigned by the team |
| Status | Backlog |
| Dependencies | HU-PPL-001, HU-UNIT-001 |
| Requisito (FR) | FR05 |
| Affected service(s) | maintenance-service |

---

### HU-MAINT-002 — Assign and track a maintenance request {#HU-MAINT-002}

**Epic:** EP-004

> **As** an Administrator
> **I want** to assign a maintenance request to a Maintenance Staff member and track its status
> **so that** no request gets lost and there is traceability of what was done

**Acceptance Criteria:**

```gherkin
Scenario 1: Assign a pending request
  Given a request with status PENDING
  When  the Administrator assigns it to a Maintenance Staff member
  Then  the status changes to ASSIGNED

Scenario 2: Invalid status transition
  Given a request with status RESOLVED
  When  someone tries to move it back to IN_PROGRESS
  Then  the system rejects the change (INV-002: status can only move forward)
```

**Definition of Done:**
- [ ] Code reviewed and approved
- [ ] Unit tests written
- [ ] Acceptance criteria verified (manual or automated)
- [ ] API contract updated if applicable
- [ ] Deployed to staging

| Field | Value |
|-------|-------|
| Story Points | 5 |
| Priority | Must Have |
| Target sprint | Sprint 3 |
| Assigned to | Pending — to be assigned by the team |
| Status | Backlog |
| Dependencies | HU-MAINT-001 |
| Requisito (FR) | FR06 |
| Affected service(s) | maintenance-service |

---

### HU-MAINT-003 — Notify a maintenance status change {#HU-MAINT-003}

**Epic:** EP-004

> **As** a Person who created a maintenance request
> **I want** to be notified whenever its status changes
> **so that** I know the progress without having to ask

**Acceptance Criteria:**

```gherkin
Scenario 1: Status change notification
  Given my request changes from ASSIGNED to IN_PROGRESS
  When  the change is saved
  Then  I receive an in-app notification
```

**Definition of Done:**
- [ ] Code reviewed and approved
- [ ] Unit tests written
- [ ] Acceptance criteria verified (manual or automated)
- [ ] API contract updated if applicable
- [ ] Deployed to staging

| Field | Value |
|-------|-------|
| Story Points | 2 |
| Priority | Should Have |
| Target sprint | Sprint 3 |
| Assigned to | Pending — to be assigned by the team |
| Status | Backlog |
| Dependencies | HU-MAINT-002 |
| Requisito (FR) | FR07 |
| Affected service(s) | communications-service |

---

### HU-BILL-001 — Generate the monthly differentiated fee {#HU-BILL-001}

**Epic:** EP-005

> **As** an Administrator
> **I want** the system to generate the monthly administration fee per unit, with a rate that differs by unit type
> **so that** residential and commercial units are billed fairly and automatically

**Acceptance Criteria:**

```gherkin
Scenario 1: Generate a fee for a commercial unit
  Given a commercial unit and its commercial rate configured
  When  the monthly fee generation runs
  Then  a fee is created with status PENDING using the commercial rate, and a FeeGenerated event is published

Scenario 2: Unit type changes after fee generation
  Given a fee was already generated for a unit this period
  When  the unit's type is changed afterward
  Then  the already-generated fee keeps its original unitTypeAtGeneration (INV-002)
```

**Definition of Done:**
- [ ] Code reviewed and approved
- [ ] Unit tests written
- [ ] Acceptance criteria verified (manual or automated)
- [ ] API contract updated if applicable
- [ ] Deployed to staging

| Field | Value |
|-------|-------|
| Story Points | 8 |
| Priority | Must Have |
| Target sprint | Sprint 3 |
| Assigned to | Pending — to be assigned by the team |
| Status | Backlog |
| Dependencies | HU-UNIT-001 |
| Requisito (FR) | FR08, FR09 |
| Affected service(s) | billing-service |

---

### HU-BILL-002 — Check fee payment status {#HU-BILL-002}

**Epic:** EP-005

> **As** a Person
> **I want** to check the payment status of my own unit's fees
> **so that** I know what I owe without asking the Administrator

**Acceptance Criteria:**

```gherkin
Scenario 1: View own fees
  Given I have fees generated for my unit
  When  I open my fee status view
  Then  I see only the fees for my own unit(s) (fees:read:own)
```

**Definition of Done:**
- [ ] Code reviewed and approved
- [ ] Unit tests written
- [ ] Acceptance criteria verified (manual or automated)
- [ ] API contract updated if applicable
- [ ] Deployed to staging

| Field | Value |
|-------|-------|
| Story Points | 2 |
| Priority | Must Have |
| Target sprint | Sprint 3 |
| Assigned to | Pending — to be assigned by the team |
| Status | Backlog |
| Dependencies | HU-BILL-001 |
| Requisito (FR) | FR10 |
| Affected service(s) | billing-service |

---

### HU-COMM-001 — Publish a segmented announcement {#HU-COMM-001}

**Epic:** EP-006

> **As** an Administrator
> **I want** to publish an announcement to all units, only residential units, or only commercial units
> **so that** residents and merchants receive only the information relevant to them

**Acceptance Criteria:**

```gherkin
Scenario 1: Publish to commercial units only
  Given I write an announcement and select scope = COMMERCIAL
  When  I publish it
  Then  only Persons linked to commercial units see it, and an AnnouncementPublished event is published
```

**Definition of Done:**
- [ ] Code reviewed and approved
- [ ] Unit tests written
- [ ] Acceptance criteria verified (manual or automated)
- [ ] API contract updated if applicable
- [ ] Deployed to staging

| Field | Value |
|-------|-------|
| Story Points | 3 |
| Priority | Should Have |
| Target sprint | Sprint 5 |
| Assigned to | Pending — to be assigned by the team |
| Status | Backlog |
| Dependencies | HU-UNIT-001 |
| Requisito (FR) | FR11 |
| Affected service(s) | communications-service |

---

### HU-ACC-001 — Log a visitor's entry {#HU-ACC-001}

**Epic:** EP-007

> **As** a Security Guard
> **I want** to log a visitor's entry, distinguishing a personal visitor from a commercial client
> **so that** there is a traceable record of who enters and to which unit

**Acceptance Criteria:**

```gherkin
Scenario 1: Log a personal visitor
  Given a valid destination unit
  When  I log the entry with visitorType = PERSONAL_VISITOR
  Then  the visit is saved and a VisitRegistered event is published

Scenario 2: Invalid destination unit
  Given I enter a unitId that does not exist
  When  I try to save the visit
  Then  the system rejects it (AGGR-INV-001)
```

**Definition of Done:**
- [ ] Code reviewed and approved
- [ ] Unit tests written
- [ ] Acceptance criteria verified (manual or automated)
- [ ] API contract updated if applicable
- [ ] Deployed to staging

| Field | Value |
|-------|-------|
| Story Points | 3 |
| Priority | Must Have |
| Target sprint | Sprint 3 |
| Assigned to | Pending — to be assigned by the team |
| Status | Backlog |
| Dependencies | HU-UNIT-001 |
| Requisito (FR) | FR12 |
| Affected service(s) | access-control-service |

---

### HU-ACC-002 — Register a visitor's vehicle {#HU-ACC-002}

**Epic:** EP-007

> **As** a Security Guard
> **I want** to optionally register a vehicle (plate, vehicle type) with a visit
> **so that** there is a record of the vehicle associated with that visitor

**Acceptance Criteria:**

```gherkin
Scenario 1: Vehicle with complete data
  Given I am logging a visit for a visitor arriving by car
  When  I add the vehicle's plate and type
  Then  both fields are saved together with the visit (AGGR-INV-002: no partial vehicle data)
```

**Definition of Done:**
- [ ] Code reviewed and approved
- [ ] Unit tests written
- [ ] Acceptance criteria verified (manual or automated)
- [ ] API contract updated if applicable
- [ ] Deployed to staging

| Field | Value |
|-------|-------|
| Story Points | 2 |
| Priority | Could Have |
| Target sprint | Sprint 3 |
| Assigned to | Pending — to be assigned by the team |
| Status | Backlog |
| Dependencies | HU-ACC-001 |
| Requisito (FR) | FR13 |
| Affected service(s) | access-control-service |

---

### HU-ACC-003 — Log a visitor's exit {#HU-ACC-003}

**Epic:** EP-007

> **As** a Security Guard
> **I want** to log a visitor's exit
> **so that** the front-desk record shows the complete visit, from entry to exit

**Acceptance Criteria:**

```gherkin
Scenario 1: Log exit for an open visit
  Given a visit was logged as entry only
  When  I log the exit
  Then  a VisitEnded event is published
```

**Definition of Done:**
- [ ] Code reviewed and approved
- [ ] Unit tests written
- [ ] Acceptance criteria verified (manual or automated)
- [ ] API contract updated if applicable
- [ ] Deployed to staging

| Field | Value |
|-------|-------|
| Story Points | 2 |
| Priority | Should Have |
| Target sprint | Sprint 3 |
| Assigned to | Pending — to be assigned by the team |
| Status | Backlog |
| Dependencies | HU-ACC-001 |
| Requisito (FR) | FR14 |
| Affected service(s) | access-control-service |

---

### HU-ACC-004 — Log incoming correspondence {#HU-ACC-004}

**Epic:** EP-007

> **As** a Security Guard
> **I want** to log a package or letter received for a unit
> **so that** the occupant knows they have something pending pickup

**Acceptance Criteria:**

```gherkin
Scenario 1: Log correspondence
  Given a valid destination unit
  When  I log an incoming package for it
  Then  it is saved with status PENDING and a CorrespondenceReceived event is published
```

**Definition of Done:**
- [ ] Code reviewed and approved
- [ ] Unit tests written
- [ ] Acceptance criteria verified (manual or automated)
- [ ] API contract updated if applicable
- [ ] Deployed to staging

| Field | Value |
|-------|-------|
| Story Points | 2 |
| Priority | Must Have |
| Target sprint | Sprint 3 |
| Assigned to | Pending — to be assigned by the team |
| Status | Backlog |
| Dependencies | HU-UNIT-001 |
| Requisito (FR) | FR15 |
| Affected service(s) | access-control-service |

---

### HU-ACC-005 — Check pending correspondence {#HU-ACC-005}

**Epic:** EP-007

> **As** a Person
> **I want** to check my own unit's pending correspondence
> **so that** I know what is waiting for me to pick up

**Acceptance Criteria:**

```gherkin
Scenario 1: View pending correspondence
  Given I have correspondence logged for my unit with status PENDING
  When  I open my correspondence view
  Then  I see only correspondence for my own unit (correspondence:read:own)
```

**Definition of Done:**
- [ ] Code reviewed and approved
- [ ] Unit tests written
- [ ] Acceptance criteria verified (manual or automated)
- [ ] API contract updated if applicable
- [ ] Deployed to staging

| Field | Value |
|-------|-------|
| Story Points | 2 |
| Priority | Should Have |
| Target sprint | Sprint 3 |
| Assigned to | Pending — to be assigned by the team |
| Status | Backlog |
| Dependencies | HU-ACC-004 |
| Requisito (FR) | FR16 |
| Affected service(s) | access-control-service |

---

### HU-FIN-001 — Create an extraordinary expense proposal {#HU-FIN-001}

**Epic:** EP-008

> **As** an Administrator
> **I want** to create a proposal for an extraordinary expense
> **so that** the Board of Trustees can review and decide on it

**Acceptance Criteria:**

```gherkin
Scenario 1: Create a proposal
  Given a subject, requested amount, and justification
  When  I submit the proposal
  Then  it is saved with status UNDER_REVIEW and an ExpenseProposalCreated event is published

Scenario 2: Amount is zero or negative
  Given I enter a requestedAmount of 0
  When  I submit the proposal
  Then  the system rejects it (AGGR-INV-003)
```

**Definition of Done:**
- [ ] Code reviewed and approved
- [ ] Unit tests written
- [ ] Acceptance criteria verified (manual or automated)
- [ ] API contract updated if applicable
- [ ] Deployed to staging

| Field | Value |
|-------|-------|
| Story Points | 3 |
| Priority | Must Have |
| Target sprint | Sprint 5 |
| Assigned to | Pending — to be assigned by the team |
| Status | Backlog |
| Dependencies | HU-BILL-001 |
| Requisito (FR) | FR17 |
| Affected service(s) | finance-approval-service |

---

### HU-FIN-002 — Approve or reject an expense proposal {#HU-FIN-002}

**Epic:** EP-008

> **As** a Board of Trustees member
> **I want** to approve or reject an extraordinary expense proposal
> **so that** there is a permanent, auditable record of the decision

**Acceptance Criteria:**

```gherkin
Scenario 1: Approve a proposal
  Given a proposal with status UNDER_REVIEW
  When  I approve it
  Then  its status changes to APPROVED, an ExpenseProposalApproved event is published, and the decision is appended to its history

Scenario 2: Decide on a closed proposal
  Given a proposal already APPROVED or REJECTED
  When  someone tries to record a new decision on it
  Then  the system rejects it (AGGR-INV-001: terminal status)
```

**Definition of Done:**
- [ ] Code reviewed and approved
- [ ] Unit tests written
- [ ] Acceptance criteria verified (manual or automated)
- [ ] API contract updated if applicable
- [ ] Deployed to staging

| Field | Value |
|-------|-------|
| Story Points | 5 |
| Priority | Must Have |
| Target sprint | Sprint 5 |
| Assigned to | Pending — to be assigned by the team |
| Status | Backlog |
| Dependencies | HU-FIN-001 |
| Requisito (FR) | FR18 |
| Affected service(s) | finance-approval-service |

---

### HU-REP-001 — View requests and pending-fee reports {#HU-REP-001}

**Epic:** EP-009

> **As** an Administrator
> **I want** to see a report of requests by status and pending fee arrears
> **so that** I have real-time information to support my decisions

**Acceptance Criteria:**

```gherkin
Scenario 1: View reports
  Given maintenance requests and fees exist in the system
  When  I open the reports view
  Then  I see requests grouped by status and the pending-fee arrears total, built from other services' events
```

**Definition of Done:**
- [ ] Code reviewed and approved
- [ ] Unit tests written
- [ ] Acceptance criteria verified (manual or automated)
- [ ] API contract updated if applicable
- [ ] Deployed to staging

| Field | Value |
|-------|-------|
| Story Points | 5 |
| Priority | Should Have |
| Target sprint | Sprint 5 |
| Assigned to | Pending — to be assigned by the team |
| Status | Backlog |
| Dependencies | HU-MAINT-001, HU-BILL-001 |
| Requisito (FR) | FR19 |
| Affected service(s) | reports-service |

---

## Rules for writing HUs

### 1. The role matters
Do not write "As a user" — that says nothing. Use the specific role:
```
✓ As a system administrator
✓ As a registered customer
✓ As an inventory operator
✗ As a user
✗ As a person
```

### 2. The benefit justifies the work
The "so that" must describe a business benefit, not redescribe the action:
```
✓ so that I can manage my orders without calling support
✗ so that I can see my orders (this only describes the feature)
```

### 3. ACs are verifiable
Each AC must be verifiable manually or automatable as a test:
```
✓ Then the system shows a message "Order #123 confirmed"
✓ Then the confirmation email arrives in less than 30 seconds
✗ Then the system works well (not verifiable)
✗ Then the user is satisfied (not verifiable)
```

### 4. One HU = one unit of value
If the HU has 15 ACs, it is probably 3 HUs.
The team must be able to complete it in one sprint (maximum 2 weeks).

---

## Ready-to-copy HU template

```markdown
### HU-00X — [Name] {#HU-00X}

**Epic:** EP-00X

> **As** [role]
> **I want** [action]
> **so that** [benefit]

**Acceptance Criteria:**

\```gherkin
Scenario 1: [name]
  Given [context]
  When  [action]
  Then  [result]
\```

| Field | Value |
|-------|-------|
| Story Points | |
| Priority | |
| Target sprint | |
| Status | Backlog |
| Dependencies | |
```

---

## Correlations

- Full template with DoD checklist → `04-requirements/_template-hu.md`
- Non-functional requirements → `04-requirements/non-functional.md`
- Traceability matrix → `04-requirements/traceability-matrix.md`
- API contracts derived from these HUs → `07-api/contracts/openapi/`
