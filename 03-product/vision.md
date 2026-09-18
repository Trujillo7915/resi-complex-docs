# Product Vision

> The vision is the team's north star. All sprints, design decisions,
> and trade-offs are evaluated against this vision.
> It must be ambitious yet achievable, inspiring but specific.

---

## Vision statement

> Use the Geoffrey Moore template:

**For** administrators and Boards of Trustees of small-to-medium mixed residential complexes (residential and commercial units) in Colombia,
**who** manage fees, maintenance, access control, correspondence, communications, and extraordinary expense approvals through fragmented manual tools (notebooks, Excel, WhatsApp),
**the** resi-complex
**is a** web-based residential complex management system,
**that** centralizes units, people, differentiated fees, maintenance requests, access control, correspondence, segmented communications, and expense approvals in one place, giving transparency to residents and commercial owners/tenants and full traceability to the Board of Trustees,
**unlike** generic PH (property horizontal) administration platforms such as ConjuntoApp, TUCO 360, Edifia, or PH360,
**our product** natively treats residential and commercial units as first-class, differentiated entities across fees, communications, and rules, instead of bolting commercial support onto a residential-only model.

---

## Team mission

To bring transparency and traceability to the day-to-day administration of small-to-medium mixed residential/commercial complexes, replacing fragmented manual tools (notebooks, spreadsheets, WhatsApp) with a centralized, secure, microservices-based system — built as a formative software engineering project for the ADSO program at SENA.

---

## Strategic pillars

Pillars are the focus areas that take us from mission to vision.
They should be few (3-5) and consistent over time.

| Pillar | Description | Success metrics |
|--------|-------------|----------------|
| Residential/commercial differentiation | Every core process (fees, communications, access) natively distinguishes residential units from commercial establishments, which competitors don't cover natively (see market research, `03-product/market-research.md`) | % of fee/communication flows with residential vs. commercial logic implemented |
| Traceability | Every critical action (expense approval, maintenance request status, visitor/correspondence log) leaves an auditable record | % of RF01–RF19 with full history/audit trail; % of expense proposals with recorded approval history |
| Security by design | RBAC by role, JWT with short expiration and refresh rotation, bcrypt-hashed passwords, parameterized queries — non-negotiable per team governance (`00-governance/`) | 0 critical security findings in code review/PR checks |
| Usability & mobility | The system must be usable from a phone, since guards and maintenance staff work on the move (RNF03) | % of core flows validated as usable on mobile |

---

## High-level roadmap

> The roadmap shows how the product evolves over time.
> Horizon 1 (0-3 months): high certainty, detail in HUs
> Horizon 2 (3-6 months): medium certainty, epics
> Horizon 3 (6-12 months): low certainty, focus areas

```
Q1 2024 ──── Q2 2024 ──── Q3 2024 ──── Q4 2024
     │                │                │                │
   [MVP core]   [Operations]      [Governance]      [Polish & docs]
   Validate       Expand            Deepen            Consolidate
   hypothesis     the flows         the value         for delivery
```

| Horizon | Period | Objective | Epics / Features | Uncertainty |
|---------|--------|----------|----------------|-------------|
| H1 (Now) | Sprint 1–2 | Stand up identity, units, and people management | IAM (RF01), Units & commercial establishments (RF02–RF04), People | Low |
| H2 (Next) | Sprint 3–4 | Enable day-to-day operations residents/staff interact with | Maintenance requests (RF05–RF07), Differentiated fees (RF08–RF10), Access control & correspondence (RF12–RF16) | Medium |
| H3 (Later) | Sprint 5+ | Close the governance loop and give visibility | Communications (RF11), Extraordinary expense approval (RF17–RF18), Reports (RF19) | High |

---

## Product principles

These principles guide design and prioritization decisions when there are trade-offs.

1. **Residential ≠ Commercial:** every feature that touches units, fees, or communications must explicitly define its behavior for both residential and commercial units — never assume one and patch the other later.

2. **Traceability first:** if an action affects money, access, or a Board decision, it must be logged and queryable later. When in doubt, log it.

3. **Security is not optional:** RBAC, hashed passwords, parameterized queries, and generic client-facing errors apply to every service from day one, per `00-governance/security.md` — no service ships without them.

4. **Mobile-usable, not mobile-only:** guards, maintenance staff, and residents often act from a phone; every core flow must work responsively (RNF03), without requiring a desktop-only experience.

---

## Product Definition of Done

> The product is "done" when it achieves these OKRs:

**Objective:** Deliver a functional MVP that covers RF01–RF19, validated with a real or simulated pilot residential complex, with governance and documentation standards fully applied.

| Key Result | Baseline | Target | Date |
|------------|---------|--------|------|
| KR1: % of RF01–RF19 implemented and passing acceptance criteria | 0% | 100% | **TBD** — end of academic term |
| KR2: Role-based surveys/interviews applied and analyzed (5 roles) | 0 / 5 applied | 5 / 5 applied and analyzed | **TBD** |
| KR3: Microservices catalog confirmed with instructor | Proposed, not confirmed | Confirmed | **TBD** |
| KR4: Response time in normal operations (RNF01) | N/A (no system yet) | < 2 seconds | **TBD** |

---

## Correlations

- Problem framing (the why) → `03-product/problem-framing.md`
- Backlog that implements the vision → `04-requirements/user-stories.md`
- KPIs in operations → `13-operations/README.md`
