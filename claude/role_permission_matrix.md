# Role and Permission Matrix

## Purpose

This document converts the Phase 1 role inventory into a formal permission matrix suitable for RBAC design in the rebuild. It maps every role to every major capability area with explicit allow/deny/conditional designations.

---

## Role Definitions

| Role ID | Role Name | Scope | Authentication | Source |
|---------|-----------|-------|---------------|--------|
| R0 | Public Visitor | None | Anonymous | No auth required |
| R1 | Student / Competitor | Person | Authenticated | `person` record |
| R2 | Judge | Person + Tournament | Authenticated | `judge` record at tournament |
| R3 | Coach / Chapter Admin | Chapter | Authenticated | `permission.tag = 'chapter'` |
| R4 | Prefs-Only Coach | Chapter (limited) | Authenticated | `permission.tag = 'prefs'` |
| R5 | Tournament Contact | Tournament (metadata) | Authenticated | `permission.tag = 'contact'` |
| R6 | Checker | Tournament (limited) | Authenticated | `permission.tag = 'checker'` |
| R7 | Limited Tabber | Tournament (event/cat scoped) | Authenticated | `permission.tag = 'limited'` (derived) |
| R8 | Tabber | Tournament (full) | Authenticated | `permission.tag = 'tabber'` |
| R9 | Tournament Owner | Tournament (full) | Authenticated | `permission.tag = 'owner'` |
| R10 | Circuit Admin | Circuit | Authenticated | `permission.tag = 'circuit'` |
| R11 | District Chair | District | Authenticated | `permission.tag = 'chair'` |
| R12 | Region Admin | Region | Authenticated | `permission.tag = 'region'` |
| R13 | Site Admin | Global | Authenticated | `person.site_admin = 1` |

---

## Permission Matrix

### Legend
- **Y** = Full access
- **O** = Own data only (scoped to own school/entries/ballots)
- **S** = Scoped to assigned events/categories
- **L** = Limited to specific whitelisted routes
- **N** = No access
- **V** = View only
- **—** = Not applicable

### Tournament Setup Capabilities

| Capability | R0 | R1 | R2 | R3 | R4 | R5 | R6 | R7 | R8 | R9 | R13 |
|-----------|----|----|----|----|----|----|----|----|----|----|-----|
| Create tournament | N | N | N | N | N | N | N | N | N | Y | Y |
| Edit tournament settings | N | N | N | N | N | N | N | S | Y | Y | Y |
| Manage tournament access | N | N | N | N | N | N | N | N | N | Y | Y |
| Configure events/categories | N | N | N | N | N | N | N | S | Y | Y | Y |
| Configure schedule/timeslots | N | N | N | N | N | N | N | S | Y | Y | Y |
| Configure sites/rooms | N | N | N | N | N | N | N | S | Y | Y | Y |
| Configure judge pools | N | N | N | N | N | N | N | S | Y | Y | Y |
| Configure room pools | N | N | N | N | N | N | N | S | Y | Y | Y |
| Configure financial settings | N | N | N | N | N | N | N | N | Y | Y | Y |
| Configure sweepstakes | N | N | N | N | N | N | N | N | Y | Y | Y |
| Manage web pages | N | N | N | N | N | N | N | N | Y | Y | Y |

### Registration Capabilities

| Capability | R0 | R1 | R2 | R3 | R4 | R5 | R6 | R7 | R8 | R9 | R13 |
|-----------|----|----|----|----|----|----|----|----|----|----|-----|
| Register school at tournament | N | N | N | Y | N | N | N | N | Y | Y | Y |
| Add/edit own school's entries | N | N | N | Y | N | N | N | N | Y | Y | Y |
| Drop own school's entries | N | N | N | Y | N | N | N | N | Y | Y | Y |
| Manage own school's judges | N | N | N | Y | N | N | N | N | Y | Y | Y |
| Manage prefs/strike cards | N | N | N | Y | Y | N | N | N | Y | Y | Y |
| View own school's invoice | N | N | N | Y | N | N | N | N | Y | Y | Y |
| Admin: manage all schools | N | N | N | N | N | N | N | S | Y | Y | Y |
| Admin: manage all entries | N | N | N | N | N | N | N | S | Y | Y | Y |
| Admin: manage all judges | N | N | N | N | N | N | N | S | Y | Y | Y |
| Admin: manage waitlist | N | N | N | N | N | N | N | S | Y | Y | Y |
| Admin: manage fines | N | N | N | N | N | N | N | S | Y | Y | Y |
| Activate/deactivate entries | N | N | N | N | N | N | L | S | Y | Y | Y |
| Activate/deactivate judges | N | N | N | N | N | N | L | S | Y | Y | Y |

### Pairing and Paneling Capabilities

| Capability | R0 | R1 | R2 | R3 | R4 | R5 | R6 | R7 | R8 | R9 | R13 |
|-----------|----|----|----|----|----|----|----|----|----|----|-----|
| Pair/panel a round | N | N | N | N | N | N | N | S | Y | Y | Y |
| Assign judges | N | N | N | N | N | N | N | S | Y | Y | Y |
| Assign rooms | N | N | N | N | N | N | N | S | Y | Y | Y |
| Publish schematics | N | N | N | N | N | N | N | S | Y | Y | Y |
| Manual panel adjustments | N | N | N | N | N | N | N | S | Y | Y | Y |
| Send blast notifications | N | N | N | N | N | N | N | S | Y | Y | Y |
| View coin flips | N | N | N | N | N | N | L | S | Y | Y | Y |
| Manage coin flips | N | N | N | N | N | N | L | S | Y | Y | Y |

### Ballot and Scoring Capabilities

| Capability | R0 | R1 | R2 | R3 | R4 | R5 | R6 | R7 | R8 | R9 | R13 |
|-----------|----|----|----|----|----|----|----|----|----|----|-----|
| Enter ballot (as judge) | N | N | O | N | N | N | N | N | N | N | Y |
| Enter ballot (tab room) | N | N | N | N | N | N | N | S | Y | Y | Y |
| Audit ballots | N | N | N | N | N | N | N | S | Y | Y | Y |
| Correct scores | N | N | N | N | N | N | N | S | Y | Y | Y |
| View ballot status dashboard | N | N | N | N | N | N | L | S | Y | Y | Y |

### Results and Publication Capabilities

| Capability | R0 | R1 | R2 | R3 | R4 | R5 | R6 | R7 | R8 | R9 | R13 |
|-----------|----|----|----|----|----|----|----|----|----|----|-----|
| Compute results | N | N | N | N | N | N | N | S | Y | Y | Y |
| Run breaks | N | N | N | N | N | N | N | S | Y | Y | Y |
| Publish results | N | N | N | N | N | N | N | S | Y | Y | Y |
| Calculate sweepstakes | N | N | N | N | N | N | N | N | Y | Y | Y |
| View published results | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| Export results (CSV) | N | N | N | N | N | N | N | S | Y | Y | Y |

### Public and Discovery Capabilities

| Capability | R0 | R1 | R2 | R3 | R4 | R5 | R6 | R7 | R8 | R9 | R13 |
|-----------|----|----|----|----|----|----|----|----|----|----|-----|
| Search tournaments | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| View tournament info | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| View published schematics | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| View published results | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| View paradigms | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| Follow tournament | N | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |

### User Account Capabilities

| Capability | R0 | R1 | R2 | R3 | R4 | R5 | R6 | R7 | R8 | R9 | R13 |
|-----------|----|----|----|----|----|----|----|----|----|----|-----|
| Create account | Y | — | — | — | — | — | — | — | — | — | — |
| Edit own profile | N | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| Edit own paradigm | N | N | Y | N | N | N | N | N | N | N | Y |
| View own results | N | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| Link NSDA account | N | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |

### Administrative Capabilities

| Capability | R0 | R1 | R2 | R3 | R4 | R5 | R6 | R7 | R8 | R9 | R10 | R11 | R12 | R13 |
|-----------|----|----|----|----|----|----|----|----|----|----|-----|-----|-----|-----|
| Manage circuit chapters | N | N | N | N | N | N | N | N | N | N | Y | N | N | Y |
| Manage circuit tournaments | N | N | N | N | N | N | N | N | N | N | Y | N | N | Y |
| Manage district operations | N | N | N | N | N | N | N | N | N | N | N | Y | N | Y |
| Manage region | N | N | N | N | N | N | N | N | N | N | N | N | Y | Y |
| Switch user (SU) | N | N | N | N | N | N | N | N | N | N | N | N | N | Y |
| Access admin site | N | N | N | N | N | N | N | N | N | N | N | N | N | Y |
| Manage all users | N | N | N | N | N | N | N | N | N | N | N | N | N | Y |

---

## Permission Scope Hierarchy

```
Global (site_admin)
  └── Circuit
        └── District / Region
              └── Tournament
                    ├── Category (judge pools, prefs)
                    │     └── Event
                    │           └── Round
                    └── Chapter (school at tournament)
                          └── Entry / Judge
```

**Key rules:**
1. Higher scope does NOT automatically grant lower scope (circuit admin cannot tab a tournament)
2. `site_admin` is the only true hierarchy bypass
3. `owner` and `tabber` are tournament-scoped and functionally identical
4. `limited` is derived — a person with event/category permissions but not tournament-wide
5. Permissions are stored as flat rows, not hierarchical

---

## Rebuild Recommendations

### 1. Consolidate owner/tabber
Since `owner` and `tabber` have identical access in all autohandler checks, consider merging into a single `admin` role with a separate `creator` flag for audit.

### 2. Explicit role hierarchy
Replace flat permission tags with a proper role hierarchy:
```
site_admin > tournament_admin > event_tabber > checker > viewer
```

### 3. Decompose the limited role
Replace the derived `limited` tag with explicit event/category role assignments that compose naturally.

### 4. Separate contact from permissions
`contact` is metadata, not access control. Move to a separate `tournament_contact` entity.

### 5. Add audit context
Every permission change should be logged with `created_by`, `created_at`, and optionally `expires_at`.

### 6. Map to ScholarComp RBAC
The ScholarComp RBAC service likely supports role-based access with scope qualifiers. Map tournament roles to ScholarComp role definitions with tournament/event/category scope bindings.

---

## Cross-Tournament / Cross-Organization Access Questions

1. **Can a person be owner of multiple tournaments?** Yes — common for tournament directors.
2. **Can a person have different roles at different tournaments?** Yes — owner at one, tabber at another, judge at a third.
3. **Can a person be both a judge and a coach at the same tournament?** Yes — the system handles this via separate judge and chapter permission paths.
4. **Can a circuit admin override tournament-level decisions?** No — circuit admin has no tournament-level access.
5. **Can a chapter admin manage registrations at any tournament their school enters?** Yes — `chapter` permission grants registration access at all tournaments where the school is registered.
6. **Can multiple people have `owner` permissions on the same tournament?** Yes — common for co-directors.
7. **What happens when a person loses chapter permission?** They lose registration access to all tournaments for that school. Existing entries/judges are not affected.
