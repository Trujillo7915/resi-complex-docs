# Design System

> The design system is the shared visual language between design and development.
> It prevents inconsistencies, accelerates design, and reduces rework.
> **Rule:** Before creating a new component, check here if it already exists.

---

## Design tokens

Tokens are the design system's variables. Changing a token changes the entire system.

### Colors

```css
/* Base palette — proposed, pending team/branding approval */
--color-primary-50:  #E8F0FE;   /* Lightest */
--color-primary-100: #C3D9FC;
--color-primary-500: #1A5FB4;   /* Default — institutional blue */
--color-primary-900: #0B2E5C;   /* Darkest */

--color-secondary-500: #2E7D5B;  /* Muted green — complements the primary blue */
--color-neutral-50:  #F7F8FA;
--color-neutral-900: #1C1E21;

/* Semantic colors */
--color-success:  #2E7D32;      /* Green — RESOLVED, PAID, APPROVED, DELIVERED */
--color-warning:  #B98900;      /* Yellow/amber — PENDING states */
--color-error:    #C62828;      /* Red — OVERDUE, REJECTED, URGENT priority */
--color-info:     #1A5FB4;      /* Blue — ASSIGNED, IN_PROGRESS, UNDER_REVIEW */

/* Text */
--color-text-primary:   #1C1E21;
--color-text-secondary: #5F6368;
--color-text-disabled:  #9AA0A6;

/* Backgrounds */
--color-bg-page:    #F7F8FA;
--color-bg-card:    #FFFFFF;
--color-bg-overlay: rgba(28, 30, 33, 0.5);
```

### Typography

```css
/* Families */
--font-family-sans:  'Inter, sans-serif';
--font-family-mono:  'JetBrains Mono, monospace';

/* Sizes (modular scale 1.25) */
--font-size-xs:   0.75rem;   /* 12px */
--font-size-sm:   0.875rem;  /* 14px */
--font-size-base: 1rem;      /* 16px */
--font-size-lg:   1.25rem;   /* 20px */
--font-size-xl:   1.563rem;  /* 25px */
--font-size-2xl:  1.953rem;  /* 31px */
--font-size-3xl:  2.441rem;  /* 39px */

/* Weights */
--font-weight-regular: 400;
--font-weight-medium:  500;
--font-weight-bold:    700;

/* Line height */
--line-height-tight:  1.2;
--line-height-normal: 1.5;
--line-height-loose:  1.8;
```

> `Inter` is proposed for its strong legibility in data-dense tables (fees, requests,
> visit logs), which make up most of resi-complex's screens. Swap freely if the team
> prefers another font — no domain reason ties this choice down.

### Spacing

```css
/* 4px system */
--space-1:  0.25rem;   /* 4px */
--space-2:  0.5rem;    /* 8px */
--space-3:  0.75rem;   /* 12px */
--space-4:  1rem;      /* 16px */
--space-6:  1.5rem;    /* 24px */
--space-8:  2rem;      /* 32px */
--space-12: 3rem;      /* 48px */
--space-16: 4rem;      /* 64px */
```

### Borders and shadows

```css
/* Border radius */
--radius-sm: 4px;
--radius-md: 8px;
--radius-lg: 16px;
--radius-full: 9999px;  /* Pill — used for status badges */

/* Shadows */
--shadow-sm: 0 1px 2px rgba(0,0,0,0.05);
--shadow-md: 0 4px 6px rgba(0,0,0,0.1);
--shadow-lg: 0 10px 15px rgba(0,0,0,0.15);
```

---

## Components

### Buttons

| Variant | Use | Disabled state |
|---------|-----|----------------|
| Primary | Main action on the page (e.g. "Save unit", "Approve proposal") | `opacity: 0.5; cursor: not-allowed` |
| Secondary | Secondary actions (e.g. "Cancel", "Back") | same |
| Danger | Destructive actions (e.g. "Delete unit", "Reject proposal") | same |
| Ghost | Tertiary actions, links (e.g. "View details") | same |

**Usage rules:**
- Only one Primary action per view
- Danger only with modal confirmation ("Are you sure?") — e.g. deleting a unit or rejecting an expense proposal
- Buttons have a loading state for async operations (e.g. submitting a maintenance request)

### Forms

| Component | When to use | resi-complex example |
|-----------|-------------|----------------------|
| Input text | Single-line free text | Unit number, visitor name |
| Textarea | Multi-line free text | Maintenance request description, expense justification |
| Select | Fixed list of options (< 15 items) | Unit type (RESIDENTIAL / COMMERCIAL), request priority |
| Combobox | List with search (> 15 items or dynamic loading) | Selecting a Unit when registering a Person (large complexes) |
| Checkbox | Independent binary option | — |
| Radio | Select one option from a few (2-5) | Visitor type (personal visitor / commercial client) |
| Toggle | Enable/disable a feature | — |
| DatePicker | Date selection | Fee due date, expense proposal date range (if added later) |

**Error messages in forms:**
- The message appears below the field, in red
- The field border turns red
- The message says how to fix the error, not just that there is an error

```
✓ "The email must have the format user@domain.com"
✗ "Invalid email"
```

### Feedback

| Component | When | Duration |
|-----------|------|---------|
| Toast/Snackbar | Action confirmations (e.g. "Request created", "Announcement published") | 4 seconds |
| Inline alert | Form errors | Until corrected |
| Modal | Destructive confirmations, irreversible actions (delete unit, reject proposal) | Until the user decides |
| Loading spinner | Operations > 200ms | Until finished |
| Skeleton | Loading list content / cards (e.g. fee list, requests list) | Until loaded |

### Data table

| Aspect | Behavior |
|--------|---------|
| Pagination | Maximum 20 rows per page (user-configurable) — applies to Units, People, Requests, Fees, Visits lists |
| Sorting | Click on column, toggle asc/desc |
| Filters | Side panel or filter row above the table (e.g. filter requests by status/priority, fees by period) |
| Selection | Checkbox in the first column (bulk actions, where applicable) |
| Actions | Final column with actions menu (edit, delete, view detail) |
| Empty state | Illustration + message + primary action CTA (e.g. "No maintenance requests yet" + "Create request") |

---

## Domain-specific components

resi-complex's core screens are dominated by entities with a status lifecycle
(`02-domain/entities-and-rules.md`). A consistent **Status Badge** component is used
system-wide instead of ad-hoc colored text.

### Status badge

| Entity | Status value | Badge color token | Notes |
|--------|-------------|--------------------|-------|
| Maintenance Request | `PENDING` | `--color-warning` | |
| Maintenance Request | `ASSIGNED` | `--color-info` | |
| Maintenance Request | `IN_PROGRESS` | `--color-info` | |
| Maintenance Request | `RESOLVED` | `--color-success` | Terminal state |
| Administration Fee | `PENDING` | `--color-warning` | |
| Administration Fee | `OVERDUE` | `--color-error` | |
| Administration Fee | `PAID` | `--color-success` | Terminal state |
| Correspondence | `PENDING` | `--color-warning` | |
| Correspondence | `DELIVERED` | `--color-success` | Terminal state |
| Expense Proposal | `UNDER_REVIEW` | `--color-info` | |
| Expense Proposal | `APPROVED` | `--color-success` | Terminal state |
| Expense Proposal | `REJECTED` | `--color-error` | Terminal state |

### Priority badge (Maintenance Request only)

| Priority | Badge color token | Notes |
|----------|--------------------|-------|
| `LOW` | `--color-neutral-900` on `--color-neutral-50` (no semantic color) | |
| `MEDIUM` | `--color-info` | |
| `HIGH` | `--color-warning` | |
| `URGENT` | `--color-error` | Also bolded, to match the notification policy in `02-domain/domain-events.md` that immediately alerts maintenance staff |

### Unit type badge

| Unit type | Badge color token | Notes |
|-----------|--------------------|-------|
| `RESIDENTIAL` | `--color-secondary-500` | Neither type is "good" or "bad" — use the secondary/neutral palette, not semantic colors |
| `COMMERCIAL` | `--color-primary-500` | |

---

## UX patterns

### Principles

1. **Confirm before destroying:** Any action that permanently deletes or modifies data requires a confirmation modal (e.g. deleting a unit, rejecting an expense proposal — both terminal/hard-to-reverse actions per `02-domain/entities-and-rules.md`).

2. **Immediate feedback:** Every action must have a visual response in < 100ms (even if it is just the loading state).

3. **Prevent rather than correct:** Validate in real time in the form, not only on submit (e.g. a commercial unit's form should require establishment data before allowing submission, mirroring the `AGGR-INV-001` invariant in `entities-and-rules.md`).

4. **Empty state as a feature:** The screen without data is the new user's first impression — guide them to the first action (e.g. a brand-new resident's `/requests` screen should invite them to create their first request, not just show a blank table).

5. **Ownership-aware UI:** Screens for `PERSON` and `MAINTENANCE_STAFF` never display data belonging to other units or unassigned requests — mirrors the ownership-level authorization concept in `01-context/glossary.md`.

### Error handling

| Scenario | What to show |
|----------|-------------|
| Network error | Toast "No connection. Retrying..." with automatic retry |
| 401 error | Redirect to `/login` with message "Your session expired" |
| 403 error | Screen "You do not have permission to view this" with a link back to `/dashboard` |
| 404 error | 404 screen with back navigation |
| 500 error | Error toast + "Retry" button |
| Timeout | Toast "This is taking longer than normal" with cancel option |

---

## Accessibility guide (minimums)

| Aspect | Minimum required |
|--------|-----------------|
| Text contrast | WCAG AA (4.5:1 for normal text, 3:1 for large text) — status badges must meet this against their background, not rely on color alone (pair with an icon or label) |
| Keyboard navigation | All interactive elements accessible with Tab (important for front-desk operators using the Security Guard screens quickly) |
| Form labels | All fields with associated label (`for` / `aria-label`) |
| Images | Descriptive alt text on all non-decorative images |
| Visible focus | Visible focus indicator on all interactive elements |
| Responsive layout | The Single Source of Truth's original NFR03 requires the system to be usable from a mobile phone — every screen in `navigation-map.md` must work down to a small mobile viewport, not just tablet/desktop |

---

## Correlations

- Navigation map → `12-ux-ui/navigation-map.md`
- Wireframes → `12-ux-ui/wireframes.md` *(not created yet — pending; see `12-ux-ui/README.md`)*
- Entities and their status lifecycles → `02-domain/entities-and-rules.md`
- Roles referenced in ownership-aware UI rules → `00-governance/security-policy.md`
