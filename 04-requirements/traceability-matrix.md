# Traceability Matrix

> Traceability connects every line of code to its business justification.
> It allows answering: "Why does this function exist?" and "Which HU covers this part of the system?"
> It also identifies: unimplemented requirements and code without a requirement (possible technical debt).

---

## How to use this matrix

```
Requirement → HU → Test Case → Implementation → Service

If a requirement has no HU: it is not planned
If a HU has no test case: it has no completeness criterion
If a test case has no implementation: there is test technical debt
If there is code without an HU: possible gold-plating or bug introduced without a story
```

---

## FR → HU → Test → Service matrix

| FR ID | FR Description | HU(s) | Tests that verify it | Service | Status |
|-------|---------------|-------|---------------------|---------|--------|
| FR01 | Authenticate a user with role-based access for the 5 roles, and register a Person linked to a unit | HU-IAM-001, HU-PPL-001 | `AuthenticationServiceTest.java`, `PersonServiceTest.java` | iam-service, people-service | 🔴 Pending |
| FR02 | Register a unit, classifying it as residential or commercial | HU-UNIT-001 | `UnitServiceTest.java` | units-service | 🔴 Pending |
| FR03 | Edit and delete an existing unit | HU-UNIT-002 | `UnitServiceTest.java` | units-service | 🔴 Pending |
| FR04 | Register commercial establishment data for a commercial unit | HU-UNIT-003 | `CommercialEstablishmentServiceTest.java` | units-service | 🔴 Pending |
| FR05 | Create a maintenance request with type, description, and priority | HU-MAINT-001 | `MaintenanceRequestServiceTest.java` | maintenance-service | 🔴 Pending |
| FR06 | Assign a maintenance request and track its status through its lifecycle | HU-MAINT-002 | `MaintenanceRequestServiceTest.java` | maintenance-service | 🔴 Pending |
| FR07 | Notify the requester whenever a maintenance request's status changes | HU-MAINT-003 | `MaintenanceNotificationServiceTest.java` | communications-service | 🔴 Pending |
| FR08 | Generate the monthly administration fee per unit | HU-BILL-001 | `FeeGenerationServiceTest.java` | billing-service | 🔴 Pending |
| FR09 | Apply a fee rate differentiated by unit type, fixed at generation time | HU-BILL-001 | `FeeGenerationServiceTest.java` | billing-service | 🔴 Pending |
| FR10 | Let a Person check their own unit's fee payment status | HU-BILL-002 | `FeeQueryServiceTest.java` | billing-service | 🔴 Pending |
| FR11 | Publish an announcement segmented by unit scope | HU-COMM-001 | `AnnouncementServiceTest.java` | communications-service | 🔴 Pending |
| FR12 | Log a visitor's entry, distinguishing personal visitor from commercial client | HU-ACC-001 | `VisitServiceTest.java` | access-control-service | 🔴 Pending |
| FR13 | Register a vehicle associated with a visit | HU-ACC-002 | `VisitServiceTest.java` | access-control-service | 🔴 Pending |
| FR14 | Log a visitor's exit | HU-ACC-003 | `VisitServiceTest.java` | access-control-service | 🔴 Pending |
| FR15 | Log incoming correspondence for a unit | HU-ACC-004 | `CorrespondenceServiceTest.java` | access-control-service | 🔴 Pending |
| FR16 | Let a Person check their own unit's pending correspondence | HU-ACC-005 | `CorrespondenceServiceTest.java` | access-control-service | 🔴 Pending |
| FR17 | Create an extraordinary expense proposal | HU-FIN-001 | `ExpenseProposalServiceTest.java` | finance-approval-service | 🔴 Pending |
| FR18 | Approve or reject an expense proposal, with permanent history | HU-FIN-002 | `ExpenseProposalServiceTest.java` | finance-approval-service | 🔴 Pending |
| FR19 | Show requests-by-status and pending-fee-arrears reports | HU-REP-001 | `ReportsServiceTest.java` | reports-service | 🔴 Pending |

---

## NFR → Validation matrix

| NFR ID | Description | How it is validated | Tool | Status |
|--------|-------------|-------------------|------|--------|
| NFR-001 | Performance: P95 < 300ms / P99 < 500ms on critical endpoints | Load test in staging | k6 / JMeter | 🔴 Pending |
| NFR-002 | Availability: 99.9% SLO in production | Health checks + SLO monitoring | Grafana | 🔴 Pending |
| NFR-003 | Scalability: horizontal auto-scaling above 70% CPU | Load test with gradual and spike traffic | k6 | 🔴 Pending |
| NFR-004 | Security: JWT auth (1h expiration), RBAC, bcrypt, OWASP Top 10 | Security contract test | Postman + OWASP ZAP | 🔴 Pending |
| NFR-005 | Observability: structured logs + correlation ID, RED metrics, traces | Smoke test in CI | Prometheus + Grafana + OpenTelemetry | 🔴 Pending |
| NFR-006 | Maintainability: ≥ 80% test coverage (≥ 90% in domain layer) | Coverage report in CI | JaCoCo | 🔴 Pending |
| NFR-007 | Portability: services run as Docker images, env-based config only | Container build + run in a clean environment | Docker | 🔴 Pending |
| NFR-008 | Disaster Recovery: RTO/RPO targets per failure scenario | Failover drill | Manual (not yet automated) | 🔴 Pending |
---

## Inverse traceability: HU → FR

| HU | Title | FR(s) it implements | Sprint |
|----|-------|---------------------|--------|
| HU-IAM-001 | User login with role-based access | FR01 | Sprint 1 |
| HU-PPL-001 | Register a Person linked to a unit | FR01 | Sprint 1 |
| HU-UNIT-001 | Register a unit | FR02 | Sprint 1 |
| HU-UNIT-002 | Edit and delete a unit | FR03 | Sprint 1 |
| HU-UNIT-003 | Register commercial establishment data | FR04 | Sprint 1 |
| HU-MAINT-001 | Create a maintenance request | FR05 | Sprint 3 |
| HU-MAINT-002 | Assign and track a maintenance request | FR06 | Sprint 3 |
| HU-MAINT-003 | Notify a maintenance status change | FR07 | Sprint 3 |
| HU-BILL-001 | Generate the monthly differentiated fee | FR08, FR09 | Sprint 3 |
| HU-BILL-002 | Check fee payment status | FR10 | Sprint 3 |
| HU-COMM-001 | Publish a segmented announcement | FR11 | Sprint 5 |
| HU-ACC-001 | Log a visitor's entry | FR12 | Sprint 3 |
| HU-ACC-002 | Register a visitor's vehicle | FR13 | Sprint 3 |
| HU-ACC-003 | Log a visitor's exit | FR14 | Sprint 3 |
| HU-ACC-004 | Log incoming correspondence | FR15 | Sprint 3 |
| HU-ACC-005 | Check pending correspondence | FR16 | Sprint 3 |
| HU-FIN-001 | Create an extraordinary expense proposal | FR17 | Sprint 5 |
| HU-FIN-002 | Approve or reject an expense proposal | FR18 | Sprint 5 |
| HU-REP-001 | View requests and pending-fee reports | FR19 | Sprint 5 |

---

## Status legend

| Status | Meaning |
|--------|---------|
| ✅ Done | Implemented, tested, and in production |
| 🟡 In progress | Under development in the current sprint |
| 🔴 Pending | In the backlog, not started |
| ⏸ Blocked | Has an external blocker |
| ❌ Cancelled | Removed from scope |

---

## Identified gaps (requirements without coverage)

> This section is updated automatically or manually when reviewing the matrix.
> A gap is: an FR without an HU, or an HU without a test, or a test without implementation.

| Gap type | Description | Required action | Owner | Date |
|----------|-------------|----------------|-------|------|
| FR shared by two services | FR01 covers both `iam-service` (auth) and `people-service` (Person registration) — the only FR mapped to two HUs/services | Confirm with the instructor whether FR01 should be split into two FRs | Product Owner | TBD |
| Ambiguous service ownership | `01-context/scope.md` marks Correspondence's service as "tentative", while `02-domain/domain-map.md` and `entities-and-rules.md` already assign it firmly to `access-control-service` | Remove the "tentative" note in `scope.md` once confirmed | Tech Lead | TBD |
| HU without test | All 19 HUs have no test file yet — implementation has not started (`01-context/overview.md`) | Write tests as each HU enters a sprint | QA / Dev | TBD |
---

## How to maintain this matrix

1. When an HU is created: add the row in the FR → HU → Test → Service section
2. When a test is written: note the file in the "Tests that verify it" column
3. When an HU is completed: change the status to ✅
4. At each Sprint Planning: review gaps and assign actions

---

## Correlations

- User Stories → `04-requirements/user-stories.md`
- Non-Functional Requirements → `04-requirements/non-functional.md`
- Testing strategy → `11-quality/testing-strategy.md`
- DoD that determines when an HU is Done → `00-governance/definition-of-done.md`
