# First-Pass Existing Platform Catalog

## Purpose

This document is the first populated pass of the descriptive catalog defined in [EXISTING_PLATFORM_CATALOG.md](../EXISTING_PLATFORM_CATALOG.md).

It is intentionally:

- source-grounded,
- descriptive rather than prescriptive,
- incomplete where evidence is still thin,
- explicit about inferred vs directly observed behavior.

This document should answer:

- what major product surfaces exist today,
- what roles and workflows are clearly present,
- what areas look operationally or algorithmically complex,
- where the best and weakest source material currently is.

This document should not decide:

- ScholarComp service ownership,
- final frontend IA,
- Release 1 architecture,
- final domain normalization.

## Source Basis

Primary evidence used in this first pass:

- [README.md](../../README.md)
- [doc/howtos/tournament-manual.pdf](../../doc/howtos/tournament-manual.pdf)
- `web/` route and directory structure
- `web/lib/Tab/*.pm` model inventory
- `web/funclib/*.mas` utility/component inventory
- public docs site at `docs.tabroom.com`

## Inventory Summary

Directly observed in the repo:

- one large legacy Perl/Mason monolith under `web/`
- major route surfaces for `setup`, `register`, `panel`, `tabbing`, `user`, `index`, and `api`
- roughly 100+ model classes under `web/lib/Tab`
- roughly 400+ `funclib` components, including dedicated clusters for `judgemath`, `nsda`, `ncfl`, `perms`, and `sweeps`
- heavy operator-facing surface area in `register`, `panel`, `setup`, and `tabbing`
- strong evidence of specialized workflows for districts, NSDA/nationals, ADA accommodations, reporting, and live round management

High-confidence observations:

- the product is a full tournament operations platform, not just registration and results
- operator tooling is dense and spread across multiple overlapping surfaces
- debate, speech, and congress formats are likely handled with materially different rules and screens
- configuration surface area is large and distributed across many settings constructs

## Capability Domains

### 1. User Accounts and Identity

- Description: Person/account creation, login/session flows, profiles, linked identities, and account-scoped access.
- Evidence:
  - `web/user/login`
  - `web/user/*`
  - model classes `Person`, `Session`, `Permission`, `Student`, `Judge`
  - manual references to logging in, linked student accounts, and profile/password flows
- Primary users: all authenticated users
- Notes:
  - clearly includes account management beyond tournament operations
  - likely includes linked student/judge identities and role-based access
  - good candidate for later reuse mapping, but should be cataloged as a real product domain first

### 2. Tournament Creation and Administration

- Description: Create tournaments, clone prior tournaments, manage tournament metadata, ownership, contacts, and broad administrative settings.
- Evidence:
  - `web/setup/tourn`
  - `web/user/tourn`
  - manual chapters on creating tournaments and tournament setup
  - model classes `Tourn`, `TournSetting`, `TournCircuit`, `TournSite`, `Weekend`
- Primary users: league/circuit admins, tournament directors, site admins
- Notes:
  - cloning and season/year reuse appear important
  - tournament administration likely overlaps with circuit and district administration

### 3. Tournament Settings and Configuration

- Description: A large configuration surface controlling registration, judging, scoring, publication, financials, and operational behavior.
- Evidence:
  - `web/setup/*`
  - `web/funclib/settings.mas`
  - model classes `Setting`, `SettingLabel`, `TabroomSetting`, plus many `*_Setting` models
  - many references to `tourn_settings`, `event_setting`, and category-specific settings in route code
- Primary users: tournament directors, advanced operators, some league/district admins
- Notes:
  - this is one of the largest product surfaces
  - configuration likely affects almost every downstream workflow
  - evidence strongly suggests a large EAV-style settings footprint

### 4. Schedule, Rounds, and Timeslots

- Description: Build the competitive schedule, create rounds, assign rounds to time blocks, and manage round lifecycle.
- Evidence:
  - `web/setup/schedule`
  - `web/funclib/round_*`
  - model classes `Round`, `RoundSetting`, `Timeslot`
  - manual chapters on schedule and creating timeslots
- Primary users: tournament directors, tab room operators
- Notes:
  - likely central to all subsequent pairing, rooming, and judging work
  - round states and timing appear operationally important

### 5. Sites and Rooms

- Description: Manage sites, rooms, room pools, quality/capacity/accessibility, and room assignment.
- Evidence:
  - `web/setup/rooms`
  - `web/panel/room`
  - model classes `Site`, `Room`, `RoomStrike`, `RPool`, `RPoolRoom`, `RPoolRound`
  - explicit ADA and room quality handling in room-related screens and logic
- Primary users: tournament directors, room managers, tab room operators
- Notes:
  - ADA accommodation handling is directly visible
  - room quality and room strikes appear to matter during assignment

### 6. Events, Divisions, and Categories

- Description: Define events, categories/divisions, event grouping, event-specific settings, and competition-format behavior.
- Evidence:
  - `web/setup/events`
  - model classes `Event`, `Category`, `EventSetting`, `CategorySetting`
  - manual sections on events, event groups, and classes
- Primary users: tournament directors
- Notes:
  - category/division structure appears important for judging and pairing
  - event configuration likely varies heavily by format

### 7. School, Chapter, Circuit, Region, Diocese, and District Administration

- Description: Persistent organizational management above a single tournament.
- Evidence:
  - `web/user/chapter`, `web/user/circuit`, `web/user/diocese`, `web/user/region`, `web/user/nsda`
  - `web/register/region`, `web/register/district`
  - model classes `Chapter`, `Circuit`, `Region`, `District`, `School`
- Primary users: school admins, circuit admins, district/league admins, NSDA-affiliated admins
- Notes:
  - this is broader than single-tournament registration
  - district-specific functionality appears substantial, not incidental

### 8. Registration and Roster Management

- Description: Register schools, entries, students, judges, waitlists, drops/changes, data import, and related reports.
- Evidence:
  - `web/register/*`
  - especially `register/data`, `register/judge`, `register/school`, `register/event`, `register/entry`, `register/reports`
  - model classes `Entry`, `EntryStudent`, `School`, `Judge`, `Follower`, `File`
  - manual chapter on registration
- Primary users: coaches, school admins, tournament registration staff
- Notes:
  - this is one of the largest route surfaces
  - evidence suggests import/export and change-management workflows are important

### 9. Judges, Hiring, Preferences, Conflicts, and Strikes

- Description: Judge registration, qualification, pool assignment, hiring, burden, preferences, conflicts, and strikes.
- Evidence:
  - `web/setup/judges`
  - `web/register/judge`
  - `web/panel/judge`
  - `web/funclib/judgemath`
  - model classes `Judge`, `JudgeHire`, `JudgeShift`, `JPool`, `JPoolJudge`, `JPoolRound`, `Rating`, `Strike`, `Conflict`
- Primary users: coaches, judge coordinators, tournament directors, tab room operators
- Notes:
  - appears to be one of the densest parts of the domain
  - dedicated `judgemath` helpers strongly suggest non-trivial algorithmic burden and assignment logic

### 10. Pairing, Paneling, and Schematics

- Description: Create pairings/sections/chambers, assign entries, judges, and rooms, inspect and manipulate schematics, and run validation/disaster checks.
- Evidence:
  - `web/panel/schemat`
  - `web/panel/round`
  - `web/panel/manipulate`
  - `web/funclib/make_pairing_hash.mas`
  - `web/funclib/clean_judges.mas`
  - manual chapter on paneling
- Primary users: tab room operators, tournament directors
- Notes:
  - this appears to be the operational core of live tournament management
  - “disaster check” and many manual manipulation screens indicate extensive recovery/override workflows

### 11. Ballots, Scoring, and Audit

- Description: Enter and validate ballots, capture scores/ranks/decisions, monitor submission status, and audit outcomes.
- Evidence:
  - `web/tabbing/entry`, `web/tabbing/status`, `web/tabbing/report`
  - `web/user/judge`
  - model classes `Ballot`, `Score`, `Protocol`, `ProtocolSetting`
  - many ballot-related funclib helpers
- Primary users: judges, tab room staff, tournament directors
- Notes:
  - likely split between self-service judge ballot entry and tab room manual entry
  - ballot variants likely differ significantly by event format

### 12. Results, Breaks, Tiebreakers, and Publication

- Description: Compute results, apply tiebreaks, advance competitors, publish standings and rounds, and surface public results.
- Evidence:
  - `web/tabbing/results`, `web/tabbing/break`, `web/tabbing/publish`
  - `web/index/results`
  - `web/funclib/results_*`
  - `web/funclib/tiebreak_types.mas`
  - manual chapters on tabulation and results
- Primary users: tab room operators, tournament directors, public viewers
- Notes:
  - another high-complexity core domain
  - public publication and internal results operations appear closely related but distinct

### 13. Districts, Qualifications, and Advancement Pipelines

- Description: District-specific tournament administration, qualification tracking, district tiebreakers, and nationals-related workflows.
- Evidence:
  - `web/register/district`
  - `web/funclib/district_*`
  - `web/user/nsda`
  - `web/setup/rules/national_bids.mhtml`
  - many `nsda_*` and district references across registration and rules surfaces
- Primary users: district admins, NSDA/nationals operators, affiliated tournament staff
- Notes:
  - likely niche relative to all tournaments, but operationally significant for affiliated ones
  - strong candidate for later release scoping, but must be cataloged

### 14. Sweepstakes and Awards

- Description: Team-level point accumulation, award rules, bids, and award output.
- Evidence:
  - `web/funclib/sweeps`
  - model classes `SweepSet`, `SweepRule`, `SweepAward`, `SweepEvent`, `SweepInclude`
  - manual references to awards and published results
- Primary users: tournament directors, awards staff, public viewers
- Notes:
  - dedicated sweeps helpers suggest real rules complexity
  - likely interacts with results and publication

### 15. Financials, Invoices, Fees, and Fines

- Description: Fee setup, school fees, invoices, fines, concessions, and payment-related administration.
- Evidence:
  - `web/setup/money`
  - `web/index/tournament_money.mhtml`
  - model classes `Invoice`, `Fine`, `TournFee`, `Concession*`
  - many registration reports
- Primary users: tournament directors, financial admins, coaches
- Notes:
  - broader than just “charge fees”; includes concessions and likely billing/reporting variants

### 16. Public Pages, Discovery, and Read-Only Tournament Access

- Description: Public tournament listings, schedules, results, paradigms, help pages, and public-facing tournament content.
- Evidence:
  - `web/index`
  - `web/index/tourn`
  - `web/index/results`
  - docs site positioning and public help/manual
- Primary users: public visitors, students, coaches, judges
- Notes:
  - public consumption surface is much smaller than operator surface, but very important for visibility and publishing

### 17. Paradigms, Profiles, and Judge Philosophy Content

- Description: Judge philosophy/paradigm content and profile-like content surfaces.
- Evidence:
  - `index/paradigm.mhtml`
  - `funclib/judge_paradigms.mas`
  - public docs and public route references
- Primary users: judges, coaches, competitors, public viewers
- Notes:
  - user-facing content feature with tournament relevance

### 18. Messaging, Notifications, and Email Blasts

- Description: Email composition, notification sending, scheduled blasts, contact management, and reminders.
- Evidence:
  - `web/register/emails`
  - `web/api/scheduled_blasts.mhtml`
  - `web/funclib/push_notifications.mas`
  - model classes `Email`, `Contact`
- Primary users: tournament admins, coaches, support/admin roles
- Notes:
  - likely includes both operational notices and mass tournament communications

### 19. Reports, Exports, and Print Outputs

- Description: Registration packets, judge reports, ADA reports, pairing outputs, result outputs, CSV/PDF/print views.
- Evidence:
  - `web/register/reports` is one of the largest route groups
  - `web/panel/report`
  - `web/tabbing/report`
  - many `csv` and print-related routes
  - manual references to printouts
- Primary users: operators, coaches, tournament staff
- Notes:
  - this appears operationally important, not just administrative convenience

### 20. Online / Hybrid Tournament Support

- Description: Online rooms, hybrid events, online mode settings, and online usage support.
- Evidence:
  - `funclib/online_room.mas`
  - `funclib/online_usage.mas`
  - public listing logic in `web/index/index.mhtml` references online/hybrid event counts
  - NSDA campus references in config and routes
- Primary users: tournament directors, judges, competitors
- Notes:
  - likely a later-era extension layered onto the legacy platform

### 21. API, Automation, and Admin Utilities

- Description: Data upload/download, maintenance jobs, environment inspection, operational API endpoints, and admin tools.
- Evidence:
  - `web/api`
  - many utility/maintenance endpoints
  - `doc/scripts`, `doc/utility`, install and ops scripts
- Primary users: admins, operators, maintainers
- Notes:
  - includes both end-user-facing API behavior and operational maintenance endpoints

## Role Inventory

### Public Visitor

- Evidence: `web/index/*`, public result and tournament pages
- Likely workflows: browse tournaments, see results, read paradigms, access public schedules
- Confidence: high

### Student

- Evidence: `web/user/student`, manual references to linked student accounts and self-entered prefs
- Likely workflows: manage own profile/account links, see entries/results, possibly enter prefs where enabled
- Confidence: medium

### Judge

- Evidence: `web/user/judge`, `register/judge`, many ballot-related screens
- Likely workflows: judge registration, paradigm/profile management, ballot submission, schedule/panel viewing
- Confidence: high

### Coach / School User

- Evidence: `web/user/enter`, `register/school`, `register/entry`, `register/judge`
- Likely workflows: register school, entries, students, judges, conflicts/prefs, review invoices/reports
- Confidence: high

### School / Chapter Admin

- Evidence: `web/user/chapter`, `Chapter*` models
- Likely workflows: manage organization-level records, coaches, students, and tournament participation
- Confidence: medium

### Tournament Director / Tournament Owner

- Evidence: `web/setup/*`, `web/panel/*`, `web/tabbing/*`, permission checks in the global autohandler
- Likely workflows: configure tournaments, pair rounds, assign judges/rooms, manage ballots/results/publication
- Confidence: high

### Tab Room Operator / Tabber

- Evidence: `panel`, `tabbing`, disaster/manipulate/report surfaces
- Likely workflows: live operations, corrections, audits, publication, recovery workflows
- Confidence: high

### Circuit / Region / Diocese / District Admin

- Evidence: `web/user/circuit`, `web/user/region`, `web/user/diocese`, `web/register/district`
- Likely workflows: regional administration, affiliated tournaments, standings/qualification workflows
- Confidence: medium

### NSDA / Nationals Affiliated Admin

- Evidence: `web/user/nsda`, `funclib/nsda`, many `nsda_*` references
- Likely workflows: qualification, district/nationals workflows, billing/reporting, compliance/status checks
- Confidence: medium

### Site Admin / System Admin

- Evidence: `web/user/admin`, admin hostname checks, server/admin routes
- Likely workflows: global administration, troubleshooting, configuration, support
- Confidence: high

## Workflow Inventory

High-confidence workflow families directly visible from the source:

1. Create or request a tournament
2. Clone/setup tournament structure from prior events
3. Configure categories, events, schedule, rooms, rules, and settings
4. Register schools, entries, students, and judges
5. Manage registration changes, drops, waitlists, and late adjustments
6. Collect or manage judge prefs, conflicts, strikes, and obligations
7. Build and assign judge pools and room pools
8. Pair rounds / create schematics
9. Assign judges and rooms to paired rounds
10. Inspect, manipulate, and validate pairings
11. Submit ballots as a judge
12. Enter or correct ballots from the tab room
13. Audit ballots and monitor outstanding ballots/status
14. Compute standings, tiebreakers, and breaks
15. Publish rounds, results, and awards
16. Produce registration, operational, and financial reports
17. Handle mid-tournament corrections and recovery
18. Run district / qualification / nationals-related workflows
19. Manage public tournament pages and paradigms

## Route and Surface Inventory

### `web/setup`

- Purpose: tournament configuration
- Visible clusters:
  - `setup/tourn`
  - `setup/events`
  - `setup/judges`
  - `setup/money`
  - `setup/rooms`
  - `setup/rules`
  - `setup/schedule`
  - `setup/web`
- Notes:
  - dense operator surface
  - strongly settings-heavy

### `web/register`

- Purpose: registration operations and related reports/data workflows
- Visible clusters:
  - `register/data`
  - `register/judge`
  - `register/school`
  - `register/event`
  - `register/entry`
  - `register/region`
  - `register/district`
  - `register/reports`
  - `register/emails`
  - `register/changes`
- Notes:
  - one of the largest route surfaces
  - appears to mix self-service, admin, and operational reporting

### `web/panel`

- Purpose: live pairing, assignment, manipulation, and print/report views
- Visible clusters:
  - `panel/schemat`
  - `panel/round`
  - `panel/manipulate`
  - `panel/judge`
  - `panel/room`
  - `panel/report`
  - `panel/publish`
- Notes:
  - likely the operational core for live round construction and repair

### `web/tabbing`

- Purpose: ballot entry, result computation, publication, and status/reporting
- Visible clusters:
  - `tabbing/entry`
  - `tabbing/report`
  - `tabbing/publish`
  - `tabbing/results`
  - `tabbing/break`
  - `tabbing/status`
- Notes:
  - live tournament control surface
  - closely coupled to paneling and ballots

### `web/user`

- Purpose: user-role-specific home and self-service/admin surfaces
- Visible clusters:
  - `user/enter`
  - `user/judge`
  - `user/chapter`
  - `user/circuit`
  - `user/diocese`
  - `user/student`
  - `user/results`
  - `user/admin`
  - `user/login`
  - `user/nsda`
- Notes:
  - strong evidence of role-specific navigation and responsibilities

### `web/index`

- Purpose: public-facing entry points
- Visible clusters:
  - `index/tourn`
  - `index/results`
  - general public pages like `about`, `help`, `paradigm`, `search`
- Notes:
  - read-mostly public surface

### `web/api`

- Purpose: operational endpoints, utilities, and data import/export
- Notes:
  - includes both product-facing and maintenance-style endpoints
  - should not be assumed to be a clean external API surface

## Data and Entity Inventory

High-confidence entity clusters visible from `web/lib/Tab`:

### Tournament and Event Structure

- `Tourn`, `Event`, `Category`, `Round`, `Timeslot`, `Site`, `Room`, `Panel`, `Pattern`, `Protocol`, `Topic`, `Weekend`

### People and Identity

- `Person`, `Session`, `Permission`, `Student`, `Judge`, `Contact`, `Follower`

### Organizations

- `School`, `Chapter`, `Circuit`, `Region`, `District`, `TournCircuit`, `TournSite`

### Registration and Participation

- `Entry`, `EntryStudent`, `SchoolSetting`, `EntrySetting`, `PersonSetting`, `StudentSetting`, `JudgeSetting`

### Pools, Strikes, and Conflicts

- `JPool`, `JPoolJudge`, `JPoolRound`, `RPool`, `RPoolRoom`, `RPoolRound`, `Strike`, `Conflict`, `RoomStrike`, `Rating*`

### Ballots, Scores, and Results

- `Ballot`, `Score`, `Result`, `ResultKey`, `ResultSet`, `ResultValue`, `Tiebreak`

### Financials and Concessions

- `Invoice`, `Fine`, `TournFee`, `Concession*`

### Sweepstakes and Awards

- `SweepSet`, `SweepRule`, `SweepAward`, `SweepAwardEvent`, `SweepEvent`, `SweepInclude`

### Content, Files, and Messaging

- `File`, `Webpage`, `Email`, `CampusLog`, `ChangeLog`

### Settings / EAV-Style Cluster

- strongly visible via:
  - `TournSetting`
  - `EventSetting`
  - `CategorySetting`
  - `RoundSetting`
  - `SchoolSetting`
  - `JudgeSetting`
  - `StudentSetting`
  - `PersonSetting`
  - `ProtocolSetting`
  - `RegionSetting`
  - `CircuitSetting`
  - `ChapterSetting`
  - `JPoolSetting`
  - `RPoolSetting`
  - `PanelSetting`
  - `WeekendSetting`
  - `TabroomSetting`

This settings cluster appears to represent a large share of current product behavior and should be treated as a major domain fact, not a small implementation detail.

## Settings and Configuration Inventory

First-pass observation:

- configuration is broad, distributed, and deeply embedded into the product
- settings appear to exist at many scopes:
  - site/global
  - tournament
  - category/division
  - event
  - round
  - pool/panel
  - person/school/judge/student
- docs and manual suggest a very large tournament setup surface
- route groups under `setup/*` and repeated `*_setting` models indicate configuration affects almost every core workflow

Areas where settings clearly appear important:

- judging and panel sizes
- registration rules
- public visibility/publication
- scoring and tiebreaking
- financial/billing settings
- online/hybrid behavior
- NSDA/district/nationals-specific behavior

## Reports and Exports Inventory

Direct evidence:

- `register/reports` is the single largest `register/*` route cluster
- `panel/report` and `tabbing/report` are dedicated report surfaces
- multiple CSV exports exist in registration and event/judge areas
- print-oriented routes and report names are visible in setup/panel/register areas
- the manual explicitly references printouts as part of tournament operation

High-confidence report/output families:

- registration packets and school reports
- judge rosters and judge reports
- ADA/accommodation reports
- pairing/schematic reports
- results and awards reports
- invoice and balance reports
- CSV exports
- print/PDF outputs

## Integrations Inventory

Directly visible or strongly implied integrations:

- email sending and reminders
- push notifications
- NSDA APIs, status checks, reporting, and nationals/district workflows
- payment/billing flows
- S3/file storage
- online room / online tournament support
- GeoIP/location data
- external import/export and data upload/download workflows
- tournament cloning and copied setup patterns

## Algorithmically Complex Areas

High-confidence complex areas from source structure and explicit helper clusters:

### Pairing / Powermatching

- Evidence:
  - `funclib/make_pairing_hash.mas`
  - `panel/schemat/*`
  - `panel/manipulate/*`
- Confidence: very high

### Judge Assignment and Cleaning

- Evidence:
  - `funclib/clean_judges.mas`
  - `funclib/judgemath/*`
  - `panel/round/*`
- Confidence: very high

### Speech Paneling / Speaker Order / Double-Entry Handling

- Evidence:
  - speech-specific schematic and results helpers
  - manual sections on paneling and speaker order
- Confidence: medium-high

### Congress Grouping / Congress-Specific Results

- Evidence:
  - `results_congress.mas`
  - congress-named helpers and screens
- Confidence: medium-high

### Conflicts / Strikes / Prefs / Burden

- Evidence:
  - `Strike`, `Conflict`, `Rating*`, `judgemath`, many round/panel helpers
- Confidence: high

### Ballot Validation and Scoring

- Evidence:
  - ballot and score models
  - judge ballot screens
  - tabbing entry/status surfaces
- Confidence: high

### Tiebreaks, Breaks, and Advancement

- Evidence:
  - `tabbing/break`
  - `funclib/tiebreak_types.mas`
  - `results_*`
- Confidence: high

### Sweepstakes and Awards

- Evidence:
  - dedicated sweeps helper cluster and models
- Confidence: medium-high

### Room Quality / ADA Matching

- Evidence:
  - explicit ADA room logic in room assignment and disaster-check surfaces
  - room quality fields and ordering in room management flows
- Confidence: high

## Documentation Coverage Assessment

### Strong

- top-level tournament lifecycle and setup sequencing
- many user-facing setup/configuration topics
- high-level registration workflows

Primary sources:

- `docs.tabroom.com`
- tournament manual

### Moderate

- public-facing pages and concepts
- organizational hierarchy concepts
- general judging and event setup

### Weak / Mostly Code-Only

- pairing algorithms
- judge assignment logic
- recovery/manipulation workflows
- detailed tiebreak implementation behavior
- settings interactions and edge cases
- some district/nationals specialized behavior

## Unknowns and Open Questions

- How much of the visible route surface is still actively used versus legacy residue?
- Which settings are truly common versus rarely used special cases?
- How sharply are debate, speech, and congress separated in the underlying data model versus only in workflow logic?
- Which district/nationals workflows are still current and essential?
- Which print outputs are operationally mandatory today?
- How much import/export interoperability exists today beyond raw CSV/data upload endpoints?

## First-Pass Conclusions

Based on direct repo evidence, the platform is clearly:

- a full tournament operations system,
- heavily operator-centric,
- highly configurable,
- strongly role-differentiated,
- rich in reporting/print outputs,
- and concentrated in a few high-complexity domains:
  - pairing/paneling,
  - judging/prefs/conflicts,
  - ballots/tabulation/results,
  - configuration,
  - district/nationals specialization.

The next best follow-on documents are:

1. lightweight parity matrix
2. glossary / term map
3. configuration and settings spec
4. workflow catalog for Release 1 candidate flows
