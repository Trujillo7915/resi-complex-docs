# Navigation Map

> Defines the screen structure of the system, how screens connect to each other, and what routes
> exist. It is the reference when frontend and backend discuss what endpoints exist
> or how to reach a feature.

---

## Frontend route structure

```
/                              → Landing — redirects to /login if not authenticated
│
├── /login                     → HU-IAM-001 — Login form (Public)
│
├── /dashboard                 → Home after login; content varies by role (All roles)
│
├── /units                     → Units Management (Administrator)
│   ├── /new                   → HU-UNIT-001 — Register a unit (residential/commercial)
│   └── /:unitId
│       ├── /                  → Unit detail, incl. commercial establishment data (HU-UNIT-003)
│       └── /edit               → HU-UNIT-002 — Edit or delete a unit
│
├── /people                    → People Management (Administrator)
│   ├── /new                   → HU-PPL-001 — Register a Person linked to a unit
│   └── /:personId             → Person detail
│
├── /requests                  → Maintenance requests
│   ├── /                      → List — scope depends on role (see Access matrix)
│   ├── /new                   → HU-MAINT-001 — Create a request (Person)
│   └── /:requestId            → Detail — assign/update status (HU-MAINT-002, HU-MAINT-003)
│
├── /fees                      → Administration fees
│   ├── /                      → List — Administrator sees all, Person sees only their own
│   └── /:feeId                → HU-BILL-002 — Fee detail and payment status
│
├── /announcements             → HU-COMM-001
│   ├── /                      → List, segmented by unit type (all authenticated roles read)
│   └── /new                   → Publish a segmented announcement (Administrator)
│
├── /front-desk                → Security Guard's operational screens
│   ├── /visits
│   │   ├── /                  → Active/recent visits log
│   │   ├── /new                → HU-ACC-001 + HU-ACC-002 — Log entry (+ optional vehicle)
│   │   └── /:visitId/exit      → HU-ACC-003 — Log a visitor's exit
│   └── /correspondence
│       ├── /                  → Pending correspondence log
│       └── /new                → HU-ACC-004 — Log incoming correspondence
│
├── /correspondence             → HU-ACC-005 — Person's own pending correspondence (read-only)
│
├── /expenses                   → Extraordinary expense proposals
│   ├── /                      → List (Administrator: all; Board of Trustees: pending review)
│   ├── /new                   → HU-FIN-001 — Create a proposal (Administrator)
│   └── /:proposalId            → HU-FIN-002 — Detail + approve/reject (Board of Trustees)
│
├── /reports                    → HU-REP-001 — Requests-by-status and pending-fee reports (Administrator)
│
└── /profile                    → Authenticated user's own profile (All roles)
```

---

## Screen map

| Screen | Route | Component | Minimum role | Backend service | Related HU |
|--------|-------|-----------|--------------|------------------|-------------|
| Login | `/login` | `LoginPage` | Public | `iam-service` | HU-IAM-001 |
| Dashboard | `/dashboard` | `DashboardPage` | Any authenticated role | Aggregates from `reports-service` | — |
| Units list | `/units` | `UnitListPage` | `ADMINISTRATOR` | `units-service` | HU-UNIT-001, HU-UNIT-002 |
| Register unit | `/units/new` | `UnitFormPage` | `ADMINISTRATOR` | `units-service` | HU-UNIT-001 |
| Unit detail | `/units/:unitId` | `UnitDetailPage` | `ADMINISTRATOR` | `units-service` | HU-UNIT-003 |
| People list | `/people` | `PersonListPage` | `ADMINISTRATOR` | `people-service` | HU-PPL-001 |
| Register person | `/people/new` | `PersonFormPage` | `ADMINISTRATOR` | `people-service` | HU-PPL-001 |
| Requests list | `/requests` | `RequestListPage` | `PERSON` (own) / `MAINTENANCE_STAFF` (assigned) / `ADMINISTRATOR` (all) | `maintenance-service` | HU-MAINT-001, HU-MAINT-002 |
| Create request | `/requests/new` | `RequestFormPage` | `PERSON` | `maintenance-service` | HU-MAINT-001 |
| Request detail | `/requests/:requestId` | `RequestDetailPage` | `PERSON` (own) / `MAINTENANCE_STAFF` (assigned) / `ADMINISTRATOR` | `maintenance-service` | HU-MAINT-002, HU-MAINT-003 |
| Fees list | `/fees` | `FeeListPage` | `PERSON` (own) / `ADMINISTRATOR` (all) | `billing-service` | HU-BILL-001, HU-BILL-002 |
| Fee detail | `/fees/:feeId` | `FeeDetailPage` | `PERSON` (own) / `ADMINISTRATOR` | `billing-service` | HU-BILL-002 |
| Announcements list | `/announcements` | `AnnouncementListPage` | Any authenticated role | `communications-service` | HU-COMM-001 |
| Publish announcement | `/announcements/new` | `AnnouncementFormPage` | `ADMINISTRATOR` | `communications-service` | HU-COMM-001 |
| Front desk — visits | `/front-desk/visits` | `VisitLogPage` | `SECURITY_GUARD` | `access-control-service` | HU-ACC-001, HU-ACC-003 |
| Log a visit | `/front-desk/visits/new` | `VisitFormPage` | `SECURITY_GUARD` | `access-control-service` | HU-ACC-001, HU-ACC-002 |
| Front desk — correspondence | `/front-desk/correspondence` | `CorrespondenceLogPage` | `SECURITY_GUARD` | `access-control-service` | HU-ACC-004 |
| My correspondence | `/correspondence` | `MyCorrespondencePage` | `PERSON` (own) | `access-control-service` | HU-ACC-005 |
| Expenses list | `/expenses` | `ExpenseListPage` | `ADMINISTRATOR` / `BOARD` | `finance-approval-service` | HU-FIN-001, HU-FIN-002 |
| Create expense proposal | `/expenses/new` | `ExpenseFormPage` | `ADMINISTRATOR` | `finance-approval-service` | HU-FIN-001 |
| Expense proposal detail | `/expenses/:proposalId` | `ExpenseDetailPage` | `ADMINISTRATOR` (read) / `BOARD` (approve/reject) | `finance-approval-service` | HU-FIN-002 |
| Reports | `/reports` | `ReportsPage` | `ADMINISTRATOR` | `reports-service` | HU-REP-001 |
| Profile | `/profile` | `ProfilePage` | Any authenticated role | `iam-service` / `people-service` | — |

---

## Access matrix

| Screen | `ADMINISTRATOR` | `BOARD` | `PERSON` | `MAINTENANCE_STAFF` | `SECURITY_GUARD` |
|--------|:---:|:---:|:---:|:---:|:---:|
| `/dashboard` | ✅ | ✅ | ✅ | ✅ | ✅ |
| `/units*` | ✅ | ❌ | ❌ | ❌ | ❌ |
| `/people*` | ✅ | ❌ | ❌ | ❌ | ❌ |
| `/requests` (list) | ✅ all | ❌ | ✅ own only | ✅ assigned only | ❌ |
| `/requests/new` | ❌ | ❌ | ✅ | ❌ | ❌ |
| `/requests/:id` | ✅ | ❌ | ✅ own only | ✅ assigned only (update status) | ❌ |
| `/fees` | ✅ all | ❌ | ✅ own only | ❌ | ❌ |
| `/announcements` (read) | ✅ | ✅ | ✅ | ✅ | ✅ |
| `/announcements/new` | ✅ | ❌ | ❌ | ❌ | ❌ |
| `/front-desk/*` | ❌ | ❌ | ❌ | ❌ | ✅ |
| `/correspondence` | ❌ | ❌ | ✅ own only | ❌ | ❌ |
| `/expenses` (read) | ✅ | ✅ | ❌ | ❌ | ❌ |
| `/expenses/new` | ✅ | ❌ | ❌ | ❌ | ❌ |
| `/expenses/:id` (approve/reject) | ❌ | ✅ | ❌ | ❌ | ❌ |
| `/reports` | ✅ | ❌ | ❌ | ❌ | ❌ |
| `/profile` | ✅ | ✅ | ✅ | ✅ | ✅ |

> This matrix mirrors the `[resource]:[action]` permissions already defined in
> `00-governance/security-policy.md`. "Own only" / "assigned only" columns correspond
> to the `:own` and `:assigned` suffixes in that policy — enforced by the backend, but
> the frontend must also filter/hide data client-side to avoid a confusing UI (e.g. a
> `PERSON` should never see a list implying access to other units' fees).

---

## Main user flows

### Flow 1 — Login

```
Landing (/)
    │
    ▼ Redirect (no session)
Login (/login)
    │
    ├── Valid credentials ──► Dashboard (/dashboard), scoped to the user's role
    │
    └── Invalid credentials ► Login with generic error message (no hint on which field failed)
```

**Related HUs:** HU-IAM-001

### Flow 2 — Report and resolve a maintenance issue (cross-role)

```
Person
  │
  │  Requests list (/requests) → "New request" (/requests/new)
  ▼
Request form — type, description, priority
  │
  ▼ Submit
Request detail (/requests/:id) — status: PENDING
  │
  │  (Administrator assigns it — outside this flow's screens, via /requests/:id)
  ▼
Maintenance Staff opens Requests list (/requests) — sees it under "assigned to me"
  │
  ▼ Opens request, updates status
Request detail (/requests/:id) — status: IN_PROGRESS → RESOLVED
  │
  ▼ (async, via CommunicationsService)
Person sees an in-app notification of the status change
```

**Related HUs:** HU-MAINT-001, HU-MAINT-002, HU-MAINT-003

### Flow 3 — Log a visitor with a vehicle at the front desk

```
Security Guard
  │
  │  Front desk — visits (/front-desk/visits) → "Log a visit" (/front-desk/visits/new)
  ▼
Visit form — destination unit, visitor type (personal / commercial client)
  │
  ├── No vehicle ──────────────► Visit saved, appears in active visits list
  │
  └── Vehicle present ─────────► Same form, vehicle section (plate, type) → Visit saved
                                       │
                                       ▼ (later)
                              Front desk — visits (/front-desk/visits)
                              → select active visit → "Log exit" (/front-desk/visits/:id/exit)
```

**Related HUs:** HU-ACC-001, HU-ACC-002, HU-ACC-003

### Flow 4 — Extraordinary expense approval (cross-role)

```
Administrator
  │
  │  Expenses list (/expenses) → "New proposal" (/expenses/new)
  ▼
Expense form — subject, requested amount, justification
  │
  ▼ Submit
Expense detail (/expenses/:id) — status: UNDER_REVIEW
  │
  ▼ (async, via CommunicationsService) — Board of Trustees notified
Board of Trustees opens Expenses list (/expenses) → opens the proposal
  │
  ▼ Reviews and decides
Expense detail (/expenses/:id) — Approve / Reject buttons
  │
  ├── Approve ──► status: APPROVED, Administrator notified
  └── Reject  ──► status: REJECTED, Administrator notified
```

**Related HUs:** HU-FIN-001, HU-FIN-002

---

## Navigation rules

| Rule | Description |
|------|-------------|
| Authentication | Every route except `/login` redirects to `/login` if there is no valid session |
| Authorization | A route accessed by a role without the matching permission (see Access matrix) redirects to `/dashboard` with a "You do not have permission to view this" message, rather than a blank or broken screen |
| Ownership filtering | For `PERSON` and `MAINTENANCE_STAFF`, list screens (`/requests`, `/fees`, `/correspondence`) must only ever render "own"/"assigned" records — this is a UI rule in addition to the backend's `:own`/`:assigned` enforcement, to avoid an inconsistent-looking UI |
| 404 | Undefined routes show a 404 screen with a link back to `/dashboard` |
| Confirmation | Destructive or irreversible actions (deleting a unit, approving/rejecting an expense proposal) show a confirmation dialog before executing, per `design-system.md` |
| Session expiration | A 401 response from any backend service redirects to `/login` with the message "Your session expired" |

---

## Correlations

- Design system (visual components, status badges) → `12-ux-ui/design-system.md`
- Wireframes → `12-ux-ui/wireframes.md` *(not created yet — pending; see `12-ux-ui/README.md`)*
- User Stories this map is derived from → `04-requirements/user-stories.md`
- Roles and permissions → `00-governance/security-policy.md`
- Backend services each screen calls → `09-microservices/service-catalog.md` *(still the generic scaffold — needs updating to the 9 real services referenced above)*
