# 04 — Requirements

> **What is this?** The formal specification of what the system must do.
> Functional: what it does. Non-functional: how well it does it.

## Why this section exists

Requirements are the contract between the team and the client/stakeholder.
Without them:
- There is no way to verify whether the system is complete
- Scope changes have no baseline for comparison
- Tests have no success criterion

---

## Types of requirements

### Functional (FR)
Describe **what the system does**: functions, behaviors, data transformations.
*Example: "The system must allow the user to recover their password via email."*

### Non-functional (NFR)
Describe **how it does it**: quality, performance, availability, security.
*Example: "The system must respond in less than 200ms for 95% of requests."*

NFRs are usually harder to meet than FRs and are ignored more frequently. **They are equally important.**

---

## What is here and how to fill it in

### `functional.md` ⭐
List of all the system's functional requirements (FR01–FR19).
**Status:** ✅ Completed (19 FRs extracted from `01-context/scope.md`, mapped to the 9 bounded contexts)

**Format:**
```markdown
| ID | Module | Description | Source (HU) | Priority |
|----|--------|-------------|------------|---------|
| FR01 | iam-service, people-service | Authenticate a user with role-based access and register a Person linked to a unit | HU-IAM-001, HU-PPL-001 | High |
```

### `non-functional.md` ⭐
Quality, performance, and technical constraint requirements (NFR-001 to NFR-008).
**Status:** ✅ Completed (all 8 NFRs measurable, validated in staging pipeline, aligned to formative-project scope)

**Format:**
```markdown
## Performance
| ID | Requirement | Metric | How to verify |
|----|------------|--------|--------------|
| NFR-001 | Response time | P95 < 300ms under 50 RPS | Load test with K6 in staging |

## Availability
| ID | Requirement | Metric | How to verify |
|----|------------|--------|--------------|
| NFR-002 | Uptime SLO | 99.9% monthly | Production monitoring (TBD) |

## Security
| ID | Requirement | Description |
|----|------------|-------------|
| NFR-004 | Authentication | JWT with 1-hour expiration; Ley 1581 de 2012 compliance |
```

### `user-stories.md`
Formalized user stories (HU-IAM-001 through HU-REP-001, cut into 3 sprints).
**Status:** ✅ Completed (19 HUs with As/I want/So that + Gherkin acceptance criteria + DoD)

### `traceability-matrix.md` ⭐
Table that connects: FR → HU → Test case → Service.
**Status:** ✅ Completed (maps 19 FRs to 19 HUs, 8 NFRs to validation tools, identifies 3 gaps pending confirmation)

**Format:**
```markdown
| FR ID | FR Description | HU(s) | Tests that verify it | Service | Status |
|-------|---------------|-------|---------------------|---------|--------|
| FR01 | Authenticate a user with role-based access... | HU-IAM-001, HU-PPL-001 | AuthenticationServiceTest.java, PersonServiceTest.java | iam-service, people-service | 🔴 Pending |
```

### `_template-hu.md`
Template for a complete User Story with acceptance criteria, DoD, story points, and metadata.
**Status:** ✅ Intact (reference template; do not modify without team consensus)

### `_template-nfr.md`
Template for specifying non-functional requirements with their verification metrics.
**Status:** ⚠️ Missing (referenced in this README but does not exist yet; low priority — `non-functional.md` already carries the full specifications)

---

## Correlations with other sections

| This section feeds... | Why |
|-----------------------|-----|
| `02-domain/` | FRs trace back to domain entities and events in entities-and-rules.md; NFRs inform infrastructure constraints |
| `03-product/` | Requirements come from vision.md and problem-framing.md (problem statement); vision also defines the 3 sprints where each HU lands |
| `05-architecture/decisions/` | NFRs (especially performance, availability, scalability) drive architectural decisions in ADRs; FR distribution across services comes from domain-map.md |
| `09-microservices/` | Each service's responsibility is defined by the FRs assigned to it in traceability-matrix.md |
| `11-quality/testing-strategy.md` | Each FR must have at least one test case; traceability-matrix.md shows which tests cover which FRs/NFRs |
| `07-api/` | Functional FRs that span services → API contracts (endpoints, request/response shapes) |
| `10-devops/` | NFRs (especially performance, availability, observability) define the CI/CD pipeline requirements and monitoring strategy |

---

## Common mistakes to avoid

❌ **"The system must be fast"** → Not measurable. Better: "p95 < 200ms"

❌ **"The system must be secure"** → Not verifiable. Better: "Authentication with JWT, tokens expire in 1h"

❌ Writing requirements that describe the solution instead of the problem.

✅ A good requirement is: **specific, measurable, achievable, relevant, and verifiable**.

---

## Questions this section must answer

- **What must the system do for each type of user?** ✅ Answered in `user-stories.md` (HU-IAM-001 → HU-REP-001, one per role/feature)
- **With what speed, availability, and security?** ✅ Answered in `non-functional.md` (NFR-001 → NFR-008, with metrics and validation strategy)
- **Which requirement originates each test case?** ✅ Answered in `traceability-matrix.md` (FR/NFR → Test class names)
- **Are all requirements covered by tests?** ⚠️ In progress — test implementation pending (CI/CD pipeline in `10-devops/` not yet defined)
- **Who is responsible for each FR?** ✅ Answered by service ownership in traceability-matrix.md (each FR assigned to one or more services)
- **Are there any gaps between specification and implementation?** ✅ Identified in traceability-matrix.md "Identified gaps" section (3 items pending confirmation)
