# First-Pass Workflow Catalog

## Purpose

This document is the first-pass workflow catalog for the current Tabroom platform.

It is designed to:

- capture the major workflow families before architecture mapping,
- keep the focus on observed product behavior,
- surface Release 1 candidate workflows,
- preserve recovery and correction workflows that are easy to overlook.

This document is intentionally high-level. It is not yet a detailed requirements specification for each workflow.

## Source Basis

Primary evidence used in this first pass:

- [doc/howtos/tournament-manual.pdf](../../doc/howtos/tournament-manual.pdf)
- `web/setup/*`
- `web/register/*`
- `web/panel/*`
- `web/tabbing/*`
- `web/user/judge/*`
- `web/user/enter/*`
- public docs on `docs.tabroom.com`

## Release 1 Candidate Workflow Families

The workflows below appear central to a first usable tournament-operations release.

## 1. Create or Request a Tournament

- Primary roles:
  - league/circuit admin
  - tournament director
- Trigger:
  - a new tournament needs to be created for a season/date
- High-level steps:
  - create tournament record
  - define dates and registration windows
  - assign ownership/contact/admin access
  - optionally clone from a prior tournament
- Outputs:
  - tournament shell ready for detailed setup
- Key evidence:
  - manual chapter “League Admins: Creating Tournaments”
  - `setup/tourn`
  - `user/tourn`
- Notes:
  - cloning appears important in current operator behavior
  - this likely depends on organization/circuit context

## 2. Configure Tournament Settings and Structure

- Primary roles:
  - tournament director
  - tab room operator
- Trigger:
  - tournament shell exists and must be configured
- High-level steps:
  - define categories/divisions/events
  - configure tournament-wide settings
  - configure event/category rules
  - configure public-facing and operational settings
- Outputs:
  - tournament rules and structure are established
- Key evidence:
  - manual chapter “Tournament Directors: Setup”
  - `setup/tourn`
  - `setup/events`
  - `setup/rules`
  - `setup/web`
- Notes:
  - this is strongly settings-driven
  - likely one of the largest workflow families in the current product

## 3. Build Schedule, Rounds, and Timeslots

- Primary roles:
  - tournament director
  - tab room operator
- Trigger:
  - event structure exists and competition schedule must be defined
- High-level steps:
  - create timeslots
  - assign events/categories/rounds to timeslots
  - create round records
  - manage round labels, flights, and schedule merges/splits
- Outputs:
  - operational schedule for registration, pairing, and live rounds
- Key evidence:
  - manual setup and paneling references
  - `setup/schedule`
  - many `round_*` and `timeslot_*` helpers
- Notes:
  - round lifecycle likely matters later for publication and results

## 4. Configure Sites and Rooms

- Primary roles:
  - tournament director
  - room manager
  - tab room operator
- Trigger:
  - physical or online spaces need to be managed for the tournament
- High-level steps:
  - define sites
  - create/import rooms
  - set room attributes such as capacity, quality, and ADA accessibility
  - create room pools where applicable
- Outputs:
  - assignable room inventory
- Key evidence:
  - `setup/rooms`
  - `panel/room`
  - room quality and ADA handling in room-related code
- Notes:
  - room characteristics appear to directly affect live assignment algorithms

## 5. Register Schools, Entries, Students, and Judges

- Primary roles:
  - coach
  - school/chapter admin
  - registration staff
- Trigger:
  - tournament registration is open or changes are being processed
- High-level steps:
  - add or locate school
  - register entries by event
  - manage student roster and linked identities
  - register judges and judge obligations
  - process data imports or administrative adds
- Outputs:
  - active tournament registration state
- Key evidence:
  - manual registration chapter
  - `register/school`
  - `register/entry`
  - `register/data`
  - `register/judge`
  - `user/enter`
- Notes:
  - appears to support both coach self-service and operator/admin intervention

## 6. Manage Waitlists, Drops, Adds, and Registration Corrections

- Primary roles:
  - coach
  - registration staff
  - tournament director
- Trigger:
  - registration changes occur before or during the event
- High-level steps:
  - add late entries or judges
  - drop entries or judges
  - handle waitlist admission/ranking
  - record changes and reasons
  - produce change logs or drop reports
- Outputs:
  - corrected registration state and audit trail
- Key evidence:
  - `register/changes`
  - `user/enter/dashboard_drop.mhtml`
  - `user/enter/student_save.mhtml`
  - `register/entry/drop.mhtml`
- Notes:
  - waitlist and drop behavior is explicitly present in source
  - this appears operationally important, not a minor edge case

## 7. Manage Judge Preferences, Conflicts, Strikes, and Burden

- Primary roles:
  - coach
  - tournament director
  - judge coordinator
  - tab room operator
- Trigger:
  - before judge assignment and pairing, or during live operations
- High-level steps:
  - submit and manage preferences
  - define conflicts and strikes
  - build judge pools
  - track burden, obligation, and availability
  - adjust judging resources
- Outputs:
  - constraint-rich judge assignment landscape
- Key evidence:
  - `setup/judges`
  - `panel/judge`
  - `funclib/judgemath/*`
  - many strike/conflict/pref helpers
- Notes:
  - one of the highest-value workflow families
  - likely deeply different by event type and tournament sophistication

## 8. Pair a Round / Build a Schematic

- Primary roles:
  - tab room operator
  - tournament director
- Trigger:
  - a round is ready to be built from current standings/constraints
- High-level steps:
  - determine event/round context
  - run format-specific pairing/grouping logic
  - create panels/sections/chambers
  - assign entries/opponents/order as applicable
- Outputs:
  - initial round schematic
- Key evidence:
  - manual paneling chapter
  - `panel/schemat`
  - `panel/round`
  - `funclib/make_pairing_hash.mas`
- Notes:
  - debate, speech, and congress should not be treated as one workflow internally
  - this is a crown-jewel workflow

## 9. Assign Judges and Rooms

- Primary roles:
  - tab room operator
  - tournament director
- Trigger:
  - a schematic exists and staffing/room assignment is required
- High-level steps:
  - assign judges to panels or rounds
  - assign rooms
  - respect conflicts, strikes, burden, quality, and ADA constraints
  - rebalance or manually intervene as needed
- Outputs:
  - operationally runnable round assignments
- Key evidence:
  - `panel/round/debate_judge_assign.mhtml`
  - `panel/round/rooms.mhtml`
  - `panel/judge/*`
  - `clean_judges.mas`
- Notes:
  - room and judge assignment appear distinct but tightly coupled workflows

## 10. Validate, Manipulate, and Recover Pairings

- Primary roles:
  - tab room operator
  - tournament director
- Trigger:
  - pairing/assignment errors are found, or manual adjustments are needed
- High-level steps:
  - run disaster/validation checks
  - reassign judges or rooms
  - move entries
  - rebalance sections/brackets
  - repair or rebuild pairings
- Outputs:
  - corrected schematic or assignment state
- Key evidence:
  - manual section “Checking for Disasters”
  - `panel/schemat/disaster_check.mhtml`
  - `panel/manipulate/*`
  - many manual pairing and replacement screens
- Notes:
  - this is explicitly a first-class workflow family in the legacy product
  - must not be treated as a niche admin convenience

## 11. Submit Ballots as a Judge

- Primary roles:
  - judge
- Trigger:
  - a round is live and assigned to the judge
- High-level steps:
  - access round/panel assignment
  - enter decision/ranks/scores according to event format
  - submit ballot
  - possibly manage online/hybrid or strike-card related flows
- Outputs:
  - submitted ballot for tab room review/use
- Key evidence:
  - `user/judge/*`
  - ballot and score models
  - extensive ballot-related screens and helpers
- Notes:
  - likely needs special attention for unreliable network/mobile contexts later

## 12. Enter, Correct, and Audit Ballots in the Tab Room

- Primary roles:
  - tab room operator
  - tournament director
- Trigger:
  - ballots need manual entry, correction, validation, or audit
- High-level steps:
  - enter or adjust ballot data
  - audit ballot completeness/correctness
  - monitor outstanding ballots
  - handle byes, forfeits, and no-shows
- Outputs:
  - validated round outcome data
- Key evidence:
  - manual sections “Entering Ranks” and “Auditing Ballots”
  - `tabbing/entry`
  - `tabbing/status`
  - `tabbing/report`
- Notes:
  - central live-operations workflow
  - likely one of the places where operational performance matters most

## 13. Compute Results, Tiebreaks, and Breaks

- Primary roles:
  - tab room operator
  - tournament director
- Trigger:
  - sufficient ballots are available for a round/event
- High-level steps:
  - compute standings/results
  - apply tiebreak rules
  - determine advancement/breaks
  - manage elimination rounds and follow-on judge needs
- Outputs:
  - standings, seeds, break lists, elimination structures
- Key evidence:
  - manual sections “Breaks” and “Elimination Round Judges”
  - `tabbing/results`
  - `tabbing/break`
  - `funclib/results_*`
  - `funclib/tiebreak_types.mas`
- Notes:
  - high-risk algorithmic workflow family

## 14. Publish Pairings, Results, and Public Information

- Primary roles:
  - tab room operator
  - tournament director
- Trigger:
  - pairings/results are ready for controlled release
- High-level steps:
  - publish rounds or specific information
  - expose pairings/results to coaches/judges/public viewers
  - trigger communications or updates where appropriate
- Outputs:
  - visible public or scoped information surfaces
- Key evidence:
  - `panel/publish`
  - `tabbing/publish`
  - coach dashboard pairing visibility logic
  - public `index/*` surfaces
- Notes:
  - appears to create fan-out read behavior across many viewers

## 15. Produce Reports, Printouts, and Operational Packets

- Primary roles:
  - tournament staff
  - coaches
  - tab room operators
- Trigger:
  - tournament prep, live operations, or post-round/post-event reporting needs
- High-level steps:
  - generate registration packets and coversheets
  - print ballots/postings/room reports
  - produce judge and result reports
  - export CSV or printable formats
- Outputs:
  - printable and exportable operational artifacts
- Key evidence:
  - manual section “Printouts”
  - `register/reports`
  - `panel/report`
  - `tabbing/report`
- Notes:
  - appears operationally central, not decorative

## 16. Run District / Qualification / Nationals-Specific Workflows

- Primary roles:
  - district admin
  - NSDA/nationals operator
  - affiliated tournament staff
- Trigger:
  - district or qualification event context
- High-level steps:
  - manage district-specific tournament views and reports
  - apply district/nationals settings or questions
  - work with qualification and affiliated reporting flows
- Outputs:
  - district/nationals-specific operational state and reporting
- Key evidence:
  - `register/district`
  - `user/nsda`
  - `funclib/district_*`
  - many `nsda_*` references
- Notes:
  - likely later-scope for a general Release 1, but substantial enough to catalog

## 17. Manage Public Pages and Paradigms

- Primary roles:
  - judges
  - tournament admins
  - public viewers
- Trigger:
  - public information needs to be surfaced or profile content updated
- High-level steps:
  - edit/read public page content
  - manage judge paradigm content
  - browse tournament-facing public pages
- Outputs:
  - public informational content
- Key evidence:
  - `index/paradigm.mhtml`
  - `index/tourn/*`
  - `Webpage` model
- Notes:
  - lower operational risk than pairing/tabbing, but ecosystem-important

## Mid-Tournament Recovery Workflow Cluster

The current platform clearly treats recovery and correction as first-class operational behavior.

Observed recovery patterns include:

- re-pairing or repairing a bracket/round
- replacing judges
- reassigning rooms
- moving entries between panels
- handling drops after pairing
- recovering from imbalance or section issues
- disaster-check-driven correction

Primary evidence:

- `panel/manipulate/*`
- `panel/schemat/disaster_check.mhtml`
- `register/changes/*`
- drop/waitlist/change flows in coach and registration areas

## Workflow Notes For Later Requirements Work

- Debate, speech, and congress workflows should be split into dedicated detailed specs later.
- Recovery workflows should be treated as core operational requirements.
- Publish and notification flows should be captured carefully because they connect operator state to coach/judge/public consumption.
- Reports/printouts should get a dedicated follow-on artifact because they are widely distributed through the product.
