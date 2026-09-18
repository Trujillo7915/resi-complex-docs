# 12 — UX/UI

> **What is this?** The user experience design of resi-complex: how the system looks,
> how it is navigated, and how it behaves from the end user's perspective — across all
> 5 authentication roles.

## Why design comes before code

Changing a wireframe takes 5 minutes. Changing the code takes hours.
Changing the code in production with real users can cost days and reputation.

**Design first → implement later.**

---

## Status

| File | Status | Notes |
|------|--------|-------|
| `navigation-map.md` | Draft, filled | Routes, screen map, access matrix, and 4 end-to-end flows derived from the 22 User Stories in `04-requirements/user-stories.md` |
| `design-system.md` | Draft, filled | Tokens, components, and domain-specific status/priority badges. Colors are a **proposed palette**, not an approved brand |
| `wireframes.md` | **Not created yet** | Low-fidelity screens for the flows in `navigation-map.md`. This is intentionally left for the team to draw (in Figma, Balsamiq, or ASCII) — it is a visual design task, not something to generate from text alone |

---

## What's in this folder

### `navigation-map.md` ⭐ (Start here)
The full route tree for resi-complex's frontend, a screen-by-screen map (route,
component, minimum role, backend service, related HU), the role-vs-screen access
matrix, and 4 illustrative user flows — including two that cross roles (a maintenance
request from creation to resolution, and an expense proposal from creation to Board
approval).

### `design-system.md`
resi-complex's design tokens (colors, typography, spacing), base components (buttons,
forms, feedback, data tables), and — specific to this project — a **Status Badge**
component mapped to every entity's lifecycle from `02-domain/entities-and-rules.md`
(`MaintenanceRequest`, `AdministrationFee`, `Correspondence`, `ExpenseProposal`), plus
a Priority badge and a Unit Type badge.

### `wireframes.md` (pending)
Low-fidelity layouts of the main screens listed in `navigation-map.md`. **Fill in:**
one wireframe (ASCII, Figma link, or Balsamiq export) per screen in the screen map,
focused on structure and content placement — not colors or final styling.

---

## Correlations with other sections

| This section is fed by... | And feeds into... |
|---------------------------|-------------------|
| `04-requirements/user-stories.md` — the 22 HUs across 9 epics define which screens and flows must exist | Screens implementing each HU |
| `02-domain/entities-and-rules.md` — entity attributes and status lifecycles | Form fields, status/priority badges in `design-system.md` |
| `00-governance/security-policy.md` — the 5 roles and their permissions | The access matrix in `navigation-map.md` |
| `09-microservices/service-catalog.md` — which service backs which screen *(still the generic scaffold — needs updating to the real 9 services)* | The "Backend service" column in the screen map |

---

## Questions this section must answer

- How many screens does the system have? → 22 named screens across 9 modules, listed in the screen map in `navigation-map.md`.
- How does each type of user navigate? → see the Access matrix in `navigation-map.md` — each of the 5 roles (`ADMINISTRATOR`, `BOARD`, `PERSON`, `MAINTENANCE_STAFF`, `SECURITY_GUARD`) has a distinct, non-overlapping set of accessible screens except `/dashboard`, `/announcements` (read), and `/profile`.
- What visual components are repeated? → status badges, data tables, and forms with real-time validation — see `design-system.md`.
- What is the system's visual language? → an institutional, trust-oriented palette proposed in `design-system.md`, pending final branding approval from the team.
