# Project Glossary

> **Instructions:** Define here all technical and business terms used in the project.
> This is the official dictionary — if there is ambiguity, this document wins.
> Add terms throughout the project, not only at the start.

---

## How to use this glossary

1. Before using a technical or business term in code, docs, or conversations: look it up here.
2. If it's not there: add it with its definition.
3. If there is disagreement about the definition: discuss it as a team and update this document.

---

## Domain terms

| Term | Definition | Notes / Synonyms |
|------|-----------|-----------------|
| Person | Any individual registered in the system who lives in or occupies a unit: a Resident or a Commercial Owner/Tenant. Modeled as a single entity (`Persona`) with a `tipo_persona` field distinguishing the two. | Do NOT model "Resident" and "Owner/Tenant" as separate authentication roles — they are subtypes of Person. |
| Unit | A physical space within the complex, classified as either residential or commercial. Owns exactly one differentiated fee rate based on its type. | Synonym to avoid: "apartment" (excludes commercial units). |
| Commercial Establishment | The business operating inside a commercial Unit (name, business type, operating hours). Only exists when the associated Unit is classified as commercial. | Do not confuse with "Unit" — a commercial Unit *has* an Establishment, it is not the same record. |
| Maintenance Request | A ticket created by a Person describing a repair or service need, with a type, description, and priority. Moves through a defined status lifecycle. | Synonyms to avoid: "ticket", "PQR" (used by competitor platforms, not our domain language). |
| Administration Fee | The periodic (monthly) charge generated for a Unit, calculated using a rate that differs by Unit type (residential vs. commercial). | Do not use "quota" in English docs — always "fee" for consistency. |
| Payment | A recorded transaction that settles (fully or partially) an Administration Fee. | |
| Announcement | A message published by the Administrator, segmented to reach all Units, only residential Units, or only commercial Units. | Synonym to avoid: "notification" (reserved for system/status alerts, e.g. RF07). |
| Visit | A record of a person entering the complex, logged by the Security Guard, associated with a destination Unit and classified as either a personal visitor or a commercial client. | |
| Visitor Vehicle | A vehicle associated with a Visit, recorded with plate number and vehicle type. | |
| Correspondence | A package or piece of mail received at the complex and logged against a specific Unit, pending pickup by its occupant. | Synonym to avoid: "mail" alone (ambiguous with email). |
| Expense Proposal | A formal request created by the Administrator for an extraordinary expense or budget item, submitted for Board approval. | |
| Approval | The Board of Directors' recorded decision (approved/rejected) on an Expense Proposal, kept as permanent historical record. | Do not overwrite — approvals are append-only for traceability. |
| Board of Directors | The authenticated role responsible for approving or rejecting Expense Proposals and reviewing financial reports. | Synonym to avoid: "committee" (not used in this domain). |
| Ownership-level authorization | Access control that restricts a Person to data belonging to their own Unit(s), enforced beyond simple role-based checks. | Not a domain entity — an architectural concept specific to this project's authorization model. |

---

## Technical terms of the project

| Term | Definition |
|------|-----------|
| Microservice | Independent service with a single responsibility, its own process, and its own database |
| Domain Event | A fact that occurred in the business that other services can observe. Name always in past tense. |
| Bounded Context | Boundary within which a particular domain model has consistent meaning |
| API Gateway | Single entry point to the system that routes requests to the corresponding microservices |
| Circuit Breaker | Pattern that stops calls to a failing service, preventing failure cascades |
| Saga | Sequence of local transactions across different services with compensating transactions on failure |
| Dead Letter Queue | Queue where messages that could not be processed after several retries are sent |
| Idempotence | Property of an operation to produce the same result if executed multiple times |

---

## Acronyms

| Acronym | Meaning |
|---------|---------|
| IAM | Identity and Access Management |
| JWT | JSON Web Token |
| API | Application Programming Interface |
| CRUD | Create, Read, Update, Delete |
| DTO | Data Transfer Object |
| FR | Functional Requirement |
| NFR | Non-Functional Requirement |
| SLO | Service Level Objective |
| SLA | Service Level Agreement |
| ADR | Architecture Decision Record |
| PR | Pull Request |
| DoD | Definition of Done |
| CI/CD | Continuous Integration / Continuous Delivery |
| RBAC | Role-Based Access Control |
