# First-Pass Role and Permission Matrix

## Purpose

This document captures the first-pass role and permission model visible in the current Tabroom platform.

It is not yet a target-state authorization design. Its purpose is to document:

- which roles clearly exist,
- what scopes permissions appear to operate at,
- what major workflows are associated with each role,
- where the current permission model is broad, narrow, or ambiguous.

## Source Basis

Primary evidence used:

- `web/lib/Tab/Person.pm`
- `web/lib/Tab/Permission.pm`
- `web/autohandler`
- `web/funclib/perms/*`
- role-specific route surfaces under `web/user/*`
- permission tag usage across setup/register/panel/tabbing/user routes

## First-Pass Permission Model Shape

Directly observed permission dimensions:

- tournament-scoped permissions
- event-scoped permissions
- category-scoped permissions
- chapter-scoped permissions
- circuit-scoped permissions
- region-scoped permissions
- district-scoped permissions
- site admin/global privilege

Directly observed tags or role-like permission values:

- `owner`
- `tabber`
- `contact`
- `checker`
- `limited`
- `chapter`
- `prefs`
- `prefs_only`
- `circuit`
- `chair`
- `wsdc`

Important observed behavior:

- site admins effectively receive owner-level tournament access
- tournament permissions can downgrade into event/category-limited access
- chapter/circuit/region/district permissions coexist with tournament permissions
- permission tags are used both as role labels and as access-state indicators

## Scope Matrix

| Scope | Direct Evidence | Notes |
|---|---|---|
| Global / Site-wide | `Person.site_admin`, admin-host checks in `autohandler` | Highest authority level |
| Tournament | `permission.tourn`, `owner`, `tabber`, `contact`, `checker` | Central operational scope |
| Event | `permission.event` | Used for limited access |
| Category | `permission.category` | Used for limited access |
| Chapter | `permission.chapter`, `chapter`, `prefs`, `prefs_only` | School/program-level authority |
| Circuit | `permission.circuit`, `circuit` | League/circuit administration |
| Region | `permission.region` | Regional administrative scope |
| District | `permission.district`, `chair`, `wsdc` | Specialized district workflows |

## Role Matrix

## 1. Public Visitor

- Scope: none
- Main capabilities:
  - browse public tournament pages
  - view published results and paradigms
- Evidence:
  - `web/index/*`
- Notes:
  - outside the formal permission model

## 2. Student

- Scope: self-service, possibly school/tournament-limited
- Main capabilities:
  - linked account/profile actions
  - view entries/results
  - possibly enter preferences when enabled
- Evidence:
  - `web/user/student`
  - manual references to student self-pref capability
- Notes:
  - current permission mechanism for students is less explicit in first-pass evidence than coach/admin roles

## 3. Judge

- Scope: self-service plus tournament assignment visibility
- Main capabilities:
  - manage judge-facing profile/paradigm
  - view assignments/panels
  - submit ballots
- Evidence:
  - `web/user/judge`
  - `Judge`, `Ballot`, `JudgeSetting`
- Notes:
  - much of judge access may derive from assignment state, not only explicit permission rows

## 4. Coach / School User

- Scope: chapter/school and tournament participation
- Main capabilities:
  - register entries/judges
  - manage roster and changes
  - submit prefs/conflicts/strikes
  - view pairings/results/invoices
- Evidence:
  - `web/user/enter`
  - `permission.tag = 'chapter'`, `'prefs'`, `'prefs_only'`
- Notes:
  - chapter-level permissions appear to be important in the current product model

## 5. Chapter / School Admin

- Scope: chapter-level
- Main capabilities:
  - manage chapter-affiliated records and people
  - participate in tournament registration/admin workflows
- Evidence:
  - `web/user/chapter`
  - chapter permission and admin helpers
- Notes:
  - likely overlaps heavily with coach workflows

## 6. Tournament Contact

- Scope: tournament-level
- Main capabilities:
  - contact and communication responsibilities
  - appears in email/day-contact logic
- Evidence:
  - `permission.tag = 'contact'`
- Notes:
  - may not be a full admin role; looks communication-focused

## 7. Tournament Checker

- Scope: tournament-level, probably constrained
- Main capabilities:
  - specialized limited access
- Evidence:
  - `checker` handling in permission logic
- Notes:
  - exact workflow meaning requires more extraction

## 8. Tournament Tabber / Admin Operator

- Scope: tournament-level
- Main capabilities:
  - pair rounds
  - assign judges/rooms
  - manipulate pairings
  - enter/audit ballots
  - publish results
- Evidence:
  - `permission.tag = 'tabber'`
  - `user/circuit/tourn_admins.mhtml` labels `tabber` as “Admin”
- Notes:
  - likely the main live-operations authority short of owner

## 9. Tournament Owner / Director

- Scope: tournament-level, highest non-site-admin tournament authority
- Main capabilities:
  - all tournament setup and operations
  - administrative deadlines and configuration
  - communication and publication control
- Evidence:
  - `permission.tag = 'owner'`
  - `Person.all_permissions`
- Notes:
  - site admins are effectively treated as owners for tournaments

## 10. Circuit Admin

- Scope: circuit-level
- Main capabilities:
  - manage or oversee circuit-affiliated tournaments/chapters
  - circuit contact/admin workflows
- Evidence:
  - `permission.tag = 'circuit'`
  - `web/user/circuit/*`
- Notes:
  - likely broader than a single tournament

## 11. Region Admin

- Scope: region-level
- Main capabilities:
  - regional administration and related tournament workflows
- Evidence:
  - `permission.region`
  - `web/user/region`
- Notes:
  - needs more extraction for exact rights

## 12. District Chair / District Admin

- Scope: district-level
- Main capabilities:
  - district-specific tournament and qualification workflows
  - district registration/reports
- Evidence:
  - `permission.tag = 'chair'`
  - district registration helpers and admin screens
- Notes:
  - appears specialized but operationally important

## 13. WSDC / Specialized District Role

- Scope: district-level specialized role
- Main capabilities:
  - likely specialized access related to specific district/tournament mode
- Evidence:
  - `permission.tag = 'wsdc'`
- Notes:
  - exact semantics require later extraction

## 14. Site Admin

- Scope: global
- Main capabilities:
  - effectively full administrative access
  - admin-site access
  - impersonation/su-related workflows
- Evidence:
  - `Person.site_admin`
  - admin-host checks
  - `Session.su`
- Notes:
  - explicit top-level system authority

## Observed Permission Behaviors

### Tournament Access Resolution

Observed in `autohandler` and `tourn_checks`:

- access can be derived from:
  - site admin status
  - tournament permission
  - event-level permission
  - category-level permission
- current request context often resolves to a tournament/event/category based on ids like:
  - `panel_id`
  - `round_id`
  - `event_id`
  - `category_id`
  - `school_id`
  - `jpool_id`
  - `tourn_id`

### Limited Access Model

Observed in `Person.all_permissions`:

- event- or category-scoped permissions appear to convert overall tournament access into a `limited` state
- `owner` and `tabber` appear to short-circuit lower-level restricted logic

### Chapter / Prefs Distinction

Observed in permission tag usage:

- `chapter` appears to imply broader school/program authority
- `prefs` and `prefs_only` appear to isolate preference-entry capability

## First-Pass Open Questions

- Which coach/school workflows require explicit `chapter` permission versus implied school affiliation?
- How much judge/student access is permission-row based versus assignment/account-link based?
- What exact capabilities does `checker` grant?
- Are there more tournament permission tags beyond `owner`, `tabber`, `contact`, and `checker` in active use?
- How do district/circuit/region permissions interact with tournament access in practice?
- How often is impersonation / `su` used operationally?

## Follow-On Work

1. Extract a more complete permission-tag inventory
2. Tie permissions to actual menu/screen visibility where possible
3. Map workflows to roles in greater detail
4. Separate:
   - role labels,
   - permission tags,
   - assignment-derived access,
   - account-link-derived access
