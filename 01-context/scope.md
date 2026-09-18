# System Scope

> **Why this document exists:** Scope prevents scope creep and aligns expectations.
> It is equally important to define what the system does NOT do as what it does.
> Review this document at the start of each planning cycle.

---

## In Scope

What the system **DOES build and maintain**:

### MVP Features

| # | Feature | Description | Responsible service |
|---|---------|-------------|---------------------|
| 1 | Authentication & role-based access | Login and session management for the 5 authentication roles (RF01) | iam-service |
| 2 | Units & commercial establishments management | Register/edit/delete units, classify as residential or commercial, and register establishment data (name, type, hours) when commercial (RF02–RF04) | units-service |
| 3 | People management | Manage Persons (Residents and Commercial Owners/Tenants) linked to their units (RF01) | people-service |
| 4 | Maintenance requests | Create requests, assign them, track status through their lifecycle, notify status changes (RF05–RF07) | maintenance-service |
| 5 | Differentiated administration fees | Monthly fee generation with rates that differ by unit type, and fee status lookup (RF08–RF10) | billing-service |
| 6 | Segmented announcements | Publish announcements to all units, only residential, or only commercial (RF11) | communications-service |
| 7 | Access control | Log visitor and vehicle entry/exit, distinguishing personal visitors from commercial clients (RF12–RF14) | access-control-service |
| 8 | Correspondence | Log incoming packages/mail per unit and let residents check pending correspondence (RF15–RF16) | access-control-service *(tentative — not explicit in the initial service catalog; pending confirmation with the instructor)* |
| 9 | Extraordinary expense approval | Administrator creates expense proposals; Board approves or rejects, with permanent history (RF17–RF18) | finance-approval-service |
| 10 | Reports | Requests by status and pending fee portfolio (RF19) | reports-service |

### Included integrations

| External system | Integration type | Purpose |
|----------------|-----------------|---------|
| N/A | N/A | This is a self-contained formative project with no external third-party system integrations planned for the MVP |

### Environments being built

| Environment | Purpose |
|-------------|---------|
| Local | Development on each team member's machine — the only environment confirmed so far |
| Development (dev) | Pending — to be defined in `10-devops/` |
| Staging | Pending — not yet planned for the formative scope of this project |
| Production | Pending — not yet planned for the formative scope of this project |

---

## Out of Scope

What the system **does NOT build** in this version and why:

| # | What is out of scope | Reason | Future version? |
|---|---------------------|--------|----------------|
| 1 | Full NIIF accounting | Exceeds the formative/academic scope; the project only covers expense proposal + approval, not full accounting | N/A |
| 2 | Physical access hardware control (electronic locks, RFID/turnstiles) | The access control module is a digital log only (RF12–RF14), not a physical automation mechanism | N/A |
| 3 | Online payments | Not evidenced as a requirement for this delivery; documented as a future integration | Yes — future line, no date set |
| 4 | Assemblies / electronic voting | No entities or RF defined for this in the current scope; present in 1 of the analyzed competitors (PH360) | Yes — team decision pending (see technology watch, section 3.3) |
| 5 | Common-area reservations | Not part of the defined entities/RF; appears in several competitor platforms | Yes — future line, no date set |
| 6 | AI over documents | Only 1 of 4 competitors (Edifia) offers it; insufficient market evidence to adopt now | Yes — to monitor, no date set |
| 7 | External channel (WhatsApp) | Not part of the defined RF; communication stays inside the system (in-app / Announcements module) | Pending — not evaluated |
| 8 | Visitor pre-authorization / QR | Data model is being prepared to support it later, but not implemented in this delivery | Yes — future line, no date set |

### What another system / team handles (and why not us)

| Feature | Who builds it | Why not us |
|---------|--------------|-----------|
| N/A | N/A | This is a single-team formative project — there is no other team or external system building any part of resi-complex |

---

## Scope assumptions

> These assumptions are taken to be true. If they change, the scope must be renegotiated.

| # | Assumption | Consequence if false |
|---|-----------|---------------------|
| 1 | A unit only needs two types: residential or commercial | Adding more unit types would require reworking the fee model and the announcement segmentation logic |
| 2 | The system serves a single residential complex at a time (not a multi-complex SaaS) | Multi-tenancy would require significant changes to the data model and authorization design |
| 3 | Initial data volume is small (single complex scale: units, residents, and daily visits) | The database and query strategy might need to change for a multi-complex or high-volume scenario |
| 4 | Users have access to a smartphone or computer with a modern browser (per RNF03) | The UX/UI design and the frontend technology choice might need to change |
| 5 | The instructor confirms the proposed microservices catalog without major changes | The service boundaries in this document (and the "Responsible service" column above) would need to be revised |

---

## Constraints

| Type | Description |
|------|-------------|
| **Time** | Tied to the SENA ADSO academic calendar; specific delivery dates pending confirmation by the team |
| **Budget** | N/A — academic formative project, no monetary budget; constrained by student hours only |
| **Technology** | Must use the confirmed stack: Java + Spring Boot (microservices, layered architecture), MySQL, Spring Security + BCrypt |
| **Regulatory** | Must comply with Ley 1581 de 2012 (Colombian personal data protection law) |
| **Team** | Pending — exact number of developers assigned to implementation not yet confirmed |

---

## External dependencies

| Dependency | Team / Provider | Required date | Status |
|-----------|----------------|--------------|--------|
| Confirmation of the final microservices catalog | SENA instructor | Before starting `06-data/` | 🟡 In progress |
| N/A — no third-party providers or external APIs | N/A | N/A | N/A |

---

## How to update the scope

The scope can change, but the change has a process:

1. Document the proposed change in this file
2. Evaluate the impact on schedule and effort
3. Obtain approval from the Product Owner and Tech Lead
4. Update the roadmap in `03-product/vision.md`
5. Create or update HUs in `04-requirements/user-stories.md`

---

## Correlations

- Vision and roadmap → `03-product/vision.md`
- Term glossary → `01-context/glossary.md`
- System overview → `01-context/overview.md`
- Scope-related risks → `15-project-control/risks.md`
