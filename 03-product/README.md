# 03 — Product Definition

> **What is this?** The answer to "what are we going to build?". It is not "how" — that comes in
> architecture. Here the validated problem, product vision, and build plan for **resi-complex**
> (residential complex management system) are defined.

## Why this section exists

Without a clear product definition:
- The team builds features nobody asked for
- Scope grows out of control (scope creep) — for example, building online payments or visitor
  QR pre-authorization before the MVP, when both are explicitly out of scope (see `01-context/scope.md`)
- There is no way to know whether the project was successful

This section is the contract between the team and the instructor about **what resi-complex will build and why**.

---

## What is here and how to fill it in

### `problem-framing.md` ⭐ (Start here) — ✅ Complete
Articulates the problem before proposing solutions.
**Contains:** administrators and Boards of Trustees of small-to-medium mixed residential
complexes struggling with fee tracking, maintenance traceability, and expense-approval records
because they rely on notebooks, Excel, and WhatsApp instead of a centralized system.

**Format used:**
```markdown
## The problem
**Who has it?** Administrators and Boards of Trustees of small-to-medium mixed
  residential complexes (residential + commercial units)
**What problem do they have?** Visitor disorganization, no differentiated-fee tracking,
  no maintenance traceability, no formal expense-approval record
**When does it occur?** Every day, using notebooks, Excel, and WhatsApp instead of one system
**What is the impact?** TBD — pending the role-based surveys/interviews (see below)
**How do they solve it today?** Manually and in a fragmented way (see section 4, Current Solution)

## Why it is worth solving
No validated competitor in the Colombian market (ConjuntoApp, TUCO 360, Edifia, PH360)
natively covers residential + commercial units with differentiated fees — see
`resi-complex-tech-watch.md`.
```

### `discovery-brief.md` — 🔴 Not created yet
Findings from user research.
**Fill in once the surveys/interviews below are applied:** which assumptions from
`problem-framing.md` were confirmed (e.g., "Excel doesn't scale for differentiated fees") vs.
invalidated, and any new pain points discovered that aren't yet reflected in the RF/FR list.

### `admin-survey.md`, `board-of-trustees-survey.md`, `maintenance-survey.md`, `resident-merchant-survey.md`, `security-guard-survey.md` — ✅ Designed, 🔴 pending to apply
Role-based surveys, one per resi-complex authentication role: Administrator, Board of Trustees,
Maintenance Staff, Resident/Commercial Owner-Tenant, and Security Guard (see the 5 roles in
`01-context/glossary.md`).
**Fill in later:** apply each survey to a real or simulated respondent of that role, then
summarize results in `discovery-brief.md` — these files hold the questions, not the answers.

### `interview-resi-complex.md` — ✅ Designed, 🔴 pending to apply
Interview scripts (opening, current pain, reaction to the proposed system, closing) for
Administrator and Board of Trustees.
**Fill in later:** same as the surveys — record and summarize real answers in `discovery-brief.md`.

### `resi-complex-tech-watch.md` — ✅ Complete
Technology watch for resi-complex: validated competitors (ConjuntoApp, TUCO 360, Edifia,
PH360), patent landscape (Decisión 486 CAN — protected by copyright, not patents), and
evaluated trends (blockchain and full document-AI: not adopted; online payments and visitor
QR: future candidates, out of MVP; fee-reminder automation: prioritized, RF08–RF10).

### `vision.md` ⭐ — ✅ Complete
The product's north star in 1-2 sentences.
**Format used (Geoffrey Moore template):**
```markdown
For administrators and Boards of Trustees of small-to-medium mixed residential complexes,
who manage fees, maintenance, access control, and expense approvals through fragmented
manual tools (notebooks, Excel, WhatsApp), resi-complex is a web-based residential complex
management system that centralizes those processes, giving transparency to residents/
commercial owners and traceability to the Board — unlike generic PH platforms (ConjuntoApp,
TUCO 360, Edifia, PH360), our product natively differentiates residential and commercial
units across fees, communications, and rules.
```

### `roadmap.md` — 🔴 Not created yet
Delivery plan over time.
**Fill in, following the horizons already sketched in `vision.md`:**
```markdown
## Phase 1 — MVP core (Sprint 1-2)
- IAM: registration/login with the 5 roles (RF01)
- Units & commercial establishments (RF02–RF04)
- People management

## Phase 2 — Operations (Sprint 3-4)
- Maintenance requests, from creation to status update (RF05–RF07)
- Differentiated fees by unit type (RF08–RF10)
- Access control & correspondence (RF12–RF16)

## Phase 3 — Governance (Sprint 5+)
- Segmented communications (RF11)
- Extraordinary expense approval with history (RF17–RF18)
- Reports (RF19)
```

### `product-backlog.md` ⭐ — 🔴 Not created yet
Prioritized list of everything that must be built.
**Fill in:** using the `_template-backlog.md` template, one entry per RF01–RF19 (see
`01-context/scope.md`), ordered by the MVP horizons above — not by convenience of
implementation.

### `_template-prd.md`
Complete Product Requirements Document.
**Use when:** the instructor asks for a formal PRD as an academic deliverable, or the project
needs to be presented to a stakeholder outside the team (e.g., a real complex administrator).

### `_template-discovery-brief.md`
Template for documenting user research — use it as the format for `discovery-brief.md` once
the 5 surveys and the interviews are applied.

### `_template-problem-framing.md`
Structured template for framing the problem — already used as the base for `problem-framing.md`.

### `_template-backlog.md`
Template for initial backlog user stories — use it as the format for `product-backlog.md`.

---

## User Story format

Every HU in `04-requirements/user-stories.md` must follow this format, using resi-complex's
real roles (Administrator, Board of Trustees, Person — Resident/Commercial Owner-Tenant,
Maintenance Staff, Security Guard) and real services (`iam-service`, `units-service`,
`people-service`, `maintenance-service`, `billing-service`, `communications-service`,
`access-control-service`, `finance-approval-service`, `reports-service`):

```markdown
## HU-BILLING-001: Generate monthly fee per unit
**As** an Administrator
**I want** the system to automatically calculate the monthly fee for each unit,
  applying the residential or commercial rate
**So that** I don't have to calculate it manually in Excel and residents/merchants
  can trust the amount is correct

### Acceptance criteria
- [ ] AC1: Given a residential unit, when the monthly fee run executes, then the
      residential rate is applied
- [ ] AC2: Given a commercial unit, when the monthly fee run executes, then the
      commercial rate is applied

### Technical notes
Owned by `billing-service` (MySQL); rate source is `units-service`'s unit type field.

**Estimation:** 5 SP  **Priority:** High
```

---

## Correlations with other sections

| This section feeds... | Why |
|-----------------------|-----|
| `04-requirements/user-stories.md` | The backlog above (RF01–RF19) is formalized into HUs with Given/When/Then criteria |
| `02-domain/domain-map.md` | `problem-framing.md` and `vision.md` already informed the 9 Bounded Contexts (IAM, Units, People, Maintenance, Billing, Communications, Access Control, Financial Approval, Reports) |
| `15-project-control/technical-backlog.md` | Scope decisions here (e.g., what's out of the MVP) become technical debt entries if revisited later |

---

## Questions this section must answer

- What problem exactly are we solving? → Answered in `problem-framing.md`
- What does product success look like for resi-complex? → Answered in `vision.md`'s Product
  Definition of Done (OKRs)
- What do we build first and why? → Answered in `roadmap.md` (Phase 1: IAM, Units, People)
- What do we NOT build in this cycle? → Answered in `01-context/scope.md`: full IFRS
  accounting, physical access hardware, online payments, digital assemblies, common-area
  reservations, AI over documents, WhatsApp channel, visitor QR pre-authorization
