# Role Inventory

## Overview

Tabroom uses a flat permission model stored in a single `permission` table, where each row links a `person` to a scoped entity (`tourn`, `chapter`, `circuit`, `region`, `district`, `category`, `event`) with a `tag` indicating the permission level. A `site_admin` column on `person` provides superuser access. Session-level `su` (switch-user) provides impersonation capability for admins.

**Source:** `web/lib/Tab/Permission.pm`, `web/lib/Tab/Person.pm`, `web/autohandler`

---

## Public Visitor

- **Scope of authority:** Unauthenticated access to public pages
- **Key workflows:** Browse tournament listings, view published results, view schematics/postings, view paradigms, search tournaments
- **Key restrictions:** No registration, no ballot entry, no tournament management
- **Evidence sources:** `web/index/` (no auth required), public pages bypass autohandler login checks
- **Notes:** Tournament-specific public pages exist under `web/index/tourn/` and `web/index/results/`

## Student / Competitor

- **Scope of authority:** View own results, manage profile, view postings/schematics
- **Key workflows:** View published rounds and results, check tournament assignments, manage profile, NSDA account linking
- **Key restrictions:** Cannot enter ballots, cannot register (coach/chapter admin does this), cannot modify tournament data
- **Evidence sources:** `web/user/student/`, `web/user/results/`, `Tab::Student` model
- **Notes:** Students are linked to a `person` record and registered as `entry_student` records. Students don't directly interact with most tournament operations; their coaches and chapter admins act on their behalf.

## Judge

- **Scope of authority:** Enter ballots for assigned rounds, view own schedule, manage paradigm
- **Key workflows:** Online ballot entry (`web/user/enter/`), view round assignments, submit RFDs/comments, manage paradigm profile, coin flip entry
- **Key restrictions:** Can only see/enter ballots for panels they're assigned to; cannot see other judges' ballots; cannot modify pairings
- **Evidence sources:** `web/user/enter/`, `web/user/judge/`, `Tab::Judge` model, `Tab::ChapterJudge` model
- **Notes:** Judges exist at two levels: `chapter_judge` (persistent identity across tournaments) and `judge` (tournament-specific instance). A person can be both a judge and a coach simultaneously.

## Coach / School Administrator

- **Scope of authority:** Register entries and judges for their school at tournaments; manage chapter roster
- **Key workflows:** Register school at tournament, add/drop entries, add/drop judges, manage judge prefs, manage school roster, view invoices/fines, manage chapter contacts
- **Key restrictions:** Can only manage their own school's entries and judges; cannot affect other schools; cannot modify pairings or results
- **Evidence sources:** `web/user/chapter/`, `web/register/` (school-scoped access), permission tag `chapter` and `prefs`
- **Permission tags:**
  - `chapter` — Full chapter access (register entries, judges, manage roster)
  - `prefs` / `prefs_only` — Can only manage preference sheets, not full registration
- **Notes:** The `chapter` permission scopes a person to a specific chapter (school/program). Multiple people can have chapter access to the same chapter.

## Tournament Director / Owner

- **Scope of authority:** Full control over tournament configuration, registration, and operations
- **Key workflows:** Create/request tournament, configure events/settings/rules, manage registration, manage schedule, oversee tabulation, publish results
- **Key restrictions:** Scoped to tournaments they own; cannot access other tournaments' admin areas unless also granted permission
- **Evidence sources:** `web/setup/`, `web/register/` (full access), `web/panel/`, `web/tabbing/`, permission tag `owner`
- **Permission tag:** `owner` — Full tournament access including setup, registration, paneling, tabbing
- **Notes:** Tournament owners are the primary administrative role. They can grant other users `tabber`, `limited`, `checker`, or `contact` permissions.

## Tab Room Operator / Tabber

- **Scope of authority:** Nearly equivalent to owner for day-of-tournament operations; full access to registration, paneling, tabbing
- **Key workflows:** Pair rounds, assign judges, enter/audit ballots, manage results, run breaks, publish schematics
- **Key restrictions:** May not have setup-level access depending on implementation; functionally equivalent to owner in most autohandler checks
- **Evidence sources:** Autohandler checks treat `owner` and `tabber` identically in most contexts
- **Permission tag:** `tabber` — Full tournament access (identical to owner in most access checks)
- **Notes:** The code treats `owner` and `tabber` as interchangeable for nearly all access checks. The distinction appears to be organizational rather than technical.

## Limited Event/Category Tabber

- **Scope of authority:** Access to specific events or categories within a tournament, not the full tournament
- **Key workflows:** Pair rounds for assigned events, enter ballots, manage results — but only for their assigned events/categories
- **Key restrictions:** Cannot access events/categories outside their permission scope; cannot modify tournament-wide settings
- **Evidence sources:** `web/setup/autohandler` lines 39-95, `web/funclib/perms/check.mas`, permission with `event` or `category` FK populated
- **Permission tag:** `limited` — Derived tag; set when person has event- or category-scoped permissions but not tournament-wide access
- **Notes:** This is the mechanism for large tournaments to delegate event-level tabbing to assistants. The `check.mas` funclib component validates entity-level access.

## Checker

- **Scope of authority:** Very limited tournament access — can confirm panel status, activate judges, view dashboard, manage coin flips
- **Key workflows:** Confirm panels (check-in), activate/deactivate entries and judges, view tournament dashboard/status
- **Key restrictions:** Cannot pair, cannot enter ballots, cannot modify settings; specifically whitelisted for a small number of routes
- **Evidence sources:** `web/tabbing/autohandler` lines 86-94 (explicit URI whitelist), `web/panel/autohandler` lines 23-30, `web/register/autohandler` lines 21-29
- **Permission tag:** `checker` — Limited check-in role with explicit route whitelisting
- **Whitelisted routes:**
  - `/tabbing/status/dashboard.mhtml`
  - `/tabbing/status/status.mhtml`
  - `/tabbing/status/panel_confirm.mhtml`
  - `/tabbing/entry/index.mhtml`
  - `/tabbing/entry/limit.mhtml`
  - `/panel/judge/activate.mhtml`
  - `/panel/judge/activate_judges.mhtml`
  - `/panel/schemat/flips.mhtml`
  - `/register/entry/entry_switch.mhtml`
  - `/register/judge/judge_switch.mhtml`
  - `/register/judge/activate.mhtml`
  - `/register/event/activate.mhtml`

## Tournament Contact

- **Scope of authority:** Listed as tournament contact for communication purposes
- **Key workflows:** Receives tournament-related communications; contact info displayed publicly
- **Key restrictions:** The `contact` tag does not itself grant setup/register/panel/tabbing access
- **Evidence sources:** Permission tag `contact`, used in email blast sender lookups and public display
- **Permission tag:** `contact`
- **Notes:** This appears to be primarily a metadata role rather than an access-control role. Contacts are displayed on tournament pages and used as recipients for system notifications.

## Circuit Administrator

- **Scope of authority:** Manage circuit membership, view circuit-wide results, manage circuit tournaments
- **Key workflows:** Manage chapters in circuit, view tournament results across circuit, manage circuit contacts, manage circuit-level settings
- **Key restrictions:** Scoped to their circuit; cannot directly modify individual tournament operations
- **Evidence sources:** `web/user/circuit/`, permission tag `circuit`
- **Permission tag:** `circuit` — Circuit-level administration
- **Notes:** Circuits are organizational groupings of schools/chapters (e.g., a state league, a national circuit). Circuit admins can view aggregate results and manage membership.

## District / Region Administrator

- **Scope of authority:** Manage NSDA district operations including qualification tracking
- **Key workflows:** Manage district tournaments, track qualification progress, manage district chapters, run district-level reports
- **Key restrictions:** Scoped to their district/region
- **Evidence sources:** `web/user/region/`, `web/user/diocese/`, `web/register/district/`, `web/register/region/`, permission tags `region`, `district`, `chair`, `wsdc`
- **Permission tags:**
  - `region` — Region-level access
  - `district` — District-level access
  - `chair` — District chair (elevated district role)
  - `wsdc` — WSDC (World Schools Debating Championships) district designation
- **Notes:** Districts and regions are NSDA-specific organizational structures for qualifying students to nationals. The `diocese` path appears to be a legacy or alternate term for similar organizational units.

## Site Administrator / Superuser

- **Scope of authority:** Unrestricted access to all tournaments, all settings, all users
- **Key workflows:** System administration, user management, tournament oversight, switch-user (SU) capability, server monitoring
- **Key restrictions:** None — bypasses all permission checks
- **Evidence sources:** `person.site_admin` column, checked in `Person.pm:all_permissions()` (line 46), `web/autohandler` (admin.tabroom.com restriction at line 188)
- **Access mechanism:** `person.site_admin` flag (not a permission row), plus `session.su` for impersonation
- **Special surfaces:**
  - `admin.tabroom.com` — Restricted to site_admin users only
  - `web/user/admin/` — Admin management tools
  - `web/user/admin/nsda/` — NSDA-specific admin tools
- **Notes:** Site admins automatically get `owner` level access to any tournament. The SU feature lets admins impersonate other users for debugging. Server count monitoring is visible only to users with `system_administrator` person setting.

---

## Permission Model Summary

| Tag | Scope | Access Level |
|-----|-------|-------------|
| `site_admin` | Global (person column) | Superuser — all tournaments, all functions |
| `owner` | Tournament | Full tournament access |
| `tabber` | Tournament | Full tournament access (functionally = owner) |
| `limited` | Tournament (derived) | Event/category-scoped tournament access |
| `checker` | Tournament | Minimal check-in role, explicit route whitelist |
| `contact` | Tournament | Metadata only — listed as tournament contact |
| `chapter` | Chapter | Full school/chapter management |
| `prefs` / `prefs_only` | Chapter | Pref sheet management only |
| `circuit` | Circuit | Circuit-level administration |
| `region` | Region | Region-level administration |
| `district` | District | District-level administration |
| `chair` | District | District chair (elevated) |
| `wsdc` | District | WSDC-specific district designation |

## Access Control Architecture

The access control is implemented as a cascade of `autohandler` files in the Mason template hierarchy:

1. **`web/autohandler`** — Top-level: authenticates session, loads person/permissions/tournament context, enforces login for admin areas
2. **`web/setup/autohandler`** — Requires `owner`, `tabber`, or `limited`; entity-level checks via `funclib/perms/check.mas`
3. **`web/register/autohandler`** — Same as setup; special allowance for `checker` on 4 specific routes
4. **`web/panel/autohandler`** — Same as setup; special allowance for `checker` on 3 specific routes
5. **`web/tabbing/autohandler`** — Same as setup; special allowance for `checker` on 5 specific routes

Entity-level permission checks are centralized in `web/funclib/perms/check.mas`, which validates that a `limited` user has permission for the specific `round_id`, `event_id`, `category_id`, `jpool_id`, `rpool_id`, `ballot_id`, `panel_id`, `school_id`, `entry_id`, `judge_id`, `fine_id`, or `concession_id` being accessed.

## Open Questions

- The `prefs` vs `prefs_only` distinction is unclear — both appear in different contexts for the same purpose
- The `contact` role's relationship to actual access control needs clarification — it may be purely informational
- Whether `limited` users can be scoped to specific rounds (not just events/categories) is ambiguous from the autohandler code
- The `wsdc` tag on districts appears to serve a dual purpose (type designation + permission)
- The relationship between `diocese` routes and the standard district/region hierarchy needs clarification
