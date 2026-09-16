# Problem Framing — Problem Definition

> **Why this document exists:** Before designing solutions, the team must be
> aligned on the problem it solves. This document captures that alignment.
> A well-defined problem is already halfway to a solution.

---

## 1. The problem in one sentence

> Complete this template:

**Administrators and Boards of Trustees of small-to-medium mixed residential complexes (residential and commercial units)** who **manage these processes manually and in a fragmented way (notebooks, Excel, WhatsApp)** struggle with **visitor disorganization, difficulty tracking differentiated fees, lack of traceability in maintenance requests, and no formal record of extraordinary expense approvals**
because **they lack a centralized system that integrates these processes and distinguishes residential from commercial units**, resulting in **a lack of transparency toward residents and commercial owners, and no traceability for the Board of Trustees when reporting decisions**.


---

## 2. Affected users

| Segment | Description | Estimated size | Priority |
|---------|-------------|---------------|---------|
| Administrator | Manages units, people, fees; approves/assigns requests; publishes announcements; proposes extraordinary expenses | 1 per complex (typical) | High |
| Board of Trustees | Approves/rejects expense-budget proposals; reviews financial reports | 3–7 members per complex (typical in Colombian PH) | High |
| Resident (Person subtype) | Creates maintenance requests, checks fees, sees residential announcements and correspondence | Tens to hundreds, depending on complex size | Medium |
| Commercial owner/tenant (Person subtype) | Same as resident, but for commercial units, with differentiated fees and announcements | Variable, typically smaller than residential | Medium |
| Maintenance staff | Sees assigned requests and updates their status | 1–5 per complex | Medium |
| Security guard | Registers visitor/vehicle entry-exit and incoming correspondence | 1–4 per complex (per shift) | Medium |

*Estimated sizes to be validated with real data once the role-based surveys are applied.(`03-product/*-survey.md`).*

### Jobs-to-be-done (JTBD)

> What job is the user trying to do when they "hire" our product?

- **When** I need to generate and collect differentiated fees by unit type, **I want** a system that automatically calculates and records them per unit, **so that** I don't have to do it manually in Excel and I can show transparency to residents and merchants.
- **When** I receive a maintenance request, **I want** to assign it, track it, and make its status visible to whoever created it, **so that** it doesn't get lost or forgotten, and there is traceability of what was done.
- **When** I need to approve an extraordinary expense as a Board member, **I want** to see the proposal with its justification and record my decision, **so that** there is a verifiable history of approvals/rejections.
- **When** a visitor or a package arrives, **I want** to register it and link it to the correct unit, **so that** the resident or merchant knows they have pending correspondence or who is visiting them.

---

## 3. Evidence of the problem

> The problem must be real. Document the evidence you have.

| Evidence type | Source | Date | Key finding |
|--------------|--------|------|------------|
| User interviews | Role-based interviews (designed, see `01-context/`) | Pending | **TBD** — pending application and analysis |
| Surveys | 5 role-based surveys (Admin, Board, Person, Maintenance, Security) — `03-product/*-survey.md` | Pending | **TBD** — surveys designed, no results yet (see project status, section 12 of the Single Source of Truth) |
| Benchmarking | ConjuntoApp, TUCO 360, Edifia, PH360 (Colombian market) | Technology watch | None of the 4 validated competitors natively covers residential + commercial units with differentiated rates — that is resi-complex's identified gap |
| Direct observation | Project problem justification (Single Source of Truth, section 2) | — | Small-to-medium complexes today manage these processes with notebooks, Excel, and WhatsApp, causing visitor disorganization, difficulty tracking differentiated fees, lack of maintenance traceability, and no formal record of expense approvals |

> **Pending action:** apply the 5 surveys and role-based interviews, then return to this section to replace the "TBD" entries with real findings and figures before moving into solution design.

---

## 4. Current user solution (and its problems)

> How does the user solve the problem today?

| Current solution | Limitations | Cost/Friction |
|-----------------|------------|--------------|
| Physical notebooks (visitors, correspondence) | No traceability, not remotely consultable, can be lost | **TBD** — to quantify via Security Guard survey |
| Excel (fees, units) | Does not automatically differentiate residential/commercial rates, prone to manual errors, does not scale | **TBD** — to quantify via Administrator survey |
| WhatsApp (announcements, maintenance requests) | Doesn't segment by unit type, information gets lost, no formal status or priority | **TBD** — to quantify via Person and Maintenance surveys |
| Meetings/physical minutes or email (expense approval) | No centralized, queryable history, hinders accountability | **TBD** — to quantify via Board of Trustees survey |


---

## 5. Solution hypothesis

> This is the first draft of the solution direction. It is not a commitment.

**We believe that** a web-based microservices system (Java + Spring Boot) that centralizes the management of residential and commercial units, differentiated fees, maintenance, access control, correspondence, segmented communications, and extraordinary expense approvals
**for** administrators, Boards of Trustees, residents, commercial owners/tenants, maintenance staff, and security guards of small-to-medium residential complexes,
**will achieve** greater transparency toward residents and merchants, and full traceability for the Board of Trustees.
**We will know we succeeded when** the pilot complex reduces its manual handling (Excel/WhatsApp/notebooks) of these processes, and all fees, requests, and approvals are recorded and queryable in the system.
---

## 6. Success metrics (North Star)

| Metric | Current baseline | 6-month target | How to measure it |
|--------|----------------|---------------|-------------------|
| System response time | N/A (no system exists) | < 2 seconds in normal operations (RNF01) | Per-service performance monitoring |
| % of maintenance requests with status updated in the system | 0% (manual/verbal process) | **TBD** — define with team/instructor | Requests module reports (RF19) |
| % of fees generated automatically vs. manually | 0% | **TBD** | Fee/portfolio reports (RF08–RF10, RF19) |
| Expense proposals with recorded approval history | 0% (no formal record, section 2) | **TBD** | Expense approval module (RF17–RF18) |

**North Star Metric:** **TBD** — to be defined by the team as the single metric that best captures delivered value (candidate: % of the pilot complex's processes migrated from manual handling to the system).


---

## 7. Hypothesis risks

| Risk | Probability | Impact | Experiment to validate |
|------|------------|--------|----------------------|
| Administrators/guards don't adopt the system due to resistance to change from their current methods (notebook/Excel/WhatsApp) | Medium | High | Apply the already-designed role-based surveys and interviews; validate with real users before building every module |
| The microservices catalog defined isn't the one the instructor ultimately approves (pending confirmation, see architecture) | Medium | Medium | Architecture validation session with the instructor before starting `09-microservices/` |
| MVP scope grows due to pressure from "future candidates" (online payments, visitor QR, etc.) | Medium | High | Review scope every Sprint Planning against the agreed out-of-MVP list |
| Survey/interview data can't be collected in time, since this is a formative project with limited access to real users | High | Medium | Define a survey application plan with a hard deadline before starting `04-requirements/` |

---

## 8. Out of scope (we do not solve)

> Explicitly define which related problems you are NOT solving in this version.
> This prevents scope creep.

- **Full IFRS/NIIF accounting** — the system records fees and expenses, but does not replace a formal accounting system: it wasn't required by the RF/RNF and would add regulatory complexity outside this project's formative scope.
- **Physical access-control hardware** (turnstiles, biometrics, etc.) — the system only digitally records what the guard reports; integrating physical hardware was explicitly excluded from the MVP.
- **Online payments** — fees are generated and consulted, but not paid within the system in this MVP; it was deprioritized to keep the MVP scope achievable in the available time.
- **Digital assemblies / voting** — out of the current formative scope; no RF covers it.
- **Common-area reservations** — not covered by RF01–RF19.
- **AI over documents** — evaluated in the technology watch (only 1 of 4 competitors offers it); not adopted in this MVP.
- **External WhatsApp channel** — notifications are in-app (RF07); WhatsApp is not integrated, to avoid depending on a third-party channel outside the team's control.
- **Visitor QR/pre-authorization** — the data model is prepared for it, but it is not implemented in the MVP.
ñ
---

## Correlations

- Product vision → `03-product/vision.md`
- HUs that implement this solution → `04-requirements/user-stories.md`
- Detailed KPIs → `13-operations/README.md`
