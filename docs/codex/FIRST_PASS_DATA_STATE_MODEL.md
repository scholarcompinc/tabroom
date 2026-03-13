# First-Pass Data and State Model Pack

## Purpose

This document captures the first-pass data model and lifecycle/state model visible in the current Tabroom platform.

Its purpose is to answer:

- what the major entity clusters are,
- how those entities relate to one another,
- where lifecycle and state are encoded explicitly,
- where lifecycle and state are encoded indirectly through flags, settings, or related rows,
- which data structures are likely to be load-bearing for a rebuild.

This is not yet a target ScholarComp schema.

## Source Basis

Primary evidence used in this pass:

- `doc/sql/current-schema.sql`
- `web/lib/Tab/Tourn.pm`
- `web/lib/Tab/Event.pm`
- `web/lib/Tab/Round.pm`
- `web/lib/Tab/Panel.pm`
- `web/lib/Tab/Ballot.pm`
- `web/lib/Tab/Entry.pm`
- `web/lib/Tab/Judge.pm`
- `web/lib/Tab/Person.pm`
- `web/lib/Tab/Permission.pm`
- `web/lib/Tab/*Setting.pm`

Important direct schema evidence:

- `entry` has state flags and trigger-derived `active`
- `round`, `panel`, and `ballot` together represent the live competition state
- `result_set`, `result`, `result_key`, and `result_value` form a flexible published-results structure
- `protocol` and `tiebreak` encode ranking logic
- many `*_setting` tables encode configuration and conditional behavior

## First-Pass Model Shape

The current platform is not a simple tournament CRUD model. It is a layered operational model:

1. Administrative context:
   - tournament
   - category
   - event
   - timeslot
   - site
   - room

2. Participation context:
   - chapter
   - school
   - person
   - student
   - judge
   - entry

3. Competition execution context:
   - round
   - panel
   - ballot
   - score
   - strike/conflict

4. Result and advancement context:
   - protocol
   - tiebreak
   - result_set
   - result
   - result_key
   - result_value
   - qualifier
   - sweep_set / sweep_rule / sweep_event / sweep_award

5. Configuration overlays:
   - global setting metadata
   - scope-specific setting tables

## Core Entity Clusters

## 1. Tournament Structure Cluster

Primary entities:

- `tourn`
- `timeslot`
- `site`
- `room`
- `tourn_site`
- `event`
- `category`
- `pattern`

Observed relationships:

- one tournament owns many events, timeslots, and tournament-site links
- one site owns many rooms
- one event belongs to one tournament and one category
- one round belongs to one event and one timeslot
- one room can be reused across many panels over time

Notes:

- event type is polymorphic at the schema level: `speech`, `congress`, `debate`, `wudc`, `wsdc`, `attendee`
- room quality, ADA, capacity, and online credentials all live on the room record

## 2. Identity and Affiliation Cluster

Primary entities:

- `person`
- `login`
- `session`
- `permission`
- `person_setting`
- `chapter`
- `school`
- `student`
- `judge`
- `chapter_judge`
- `chapter_circuit`

Observed relationships:

- `person` is the broad account/identity record
- `login` is credential material linked to `person`
- `session` carries active web session state and impersonation/su data
- `permission` grants scoped access across tournament, chapter, circuit, region, district, and category
- `student` and `judge` can link back to `person`
- `school` is tournament-local and links to a broader `chapter`

Notes:

- the product distinguishes long-lived identity (`person`, `chapter`) from tournament-local participation (`school`, `entry`, `judge`)
- permissions are multi-scope rather than purely RBAC

## 3. Registration and Participation Cluster

Primary entities:

- `school`
- `entry`
- `entry_student`
- `judge`
- `judge_hire`
- `fine`
- `invoice`
- `tourn_fee`
- `file`
- `housing`
- `housing_slots`

Observed relationships:

- one school registers many entries and judges into a tournament
- one entry belongs to one event and one school
- one entry has one or more students through `entry_student`
- one judge belongs to one tournament-local school context
- fines and invoices attach to tournament/school/judge combinations

Notes:

- registration is not just roster creation; it includes payment, fines, housing, uploaded files, and judge obligation/hire flows

## 4. Competition Execution Cluster

Primary entities:

- `round`
- `panel`
- `ballot`
- `score`
- `student_ballot`
- `student_vote`
- `rpool`
- `rpool_room`
- `rpool_round`
- `jpool`
- `jpool_judge`
- `jpool_round`
- `room_strike`
- `strike`
- `conflict`

Observed relationships:

- one round contains many panels
- one panel belongs to one room and one round
- ballots tie together entry + judge + panel
- scores are attached to ballots, optionally per student
- pools constrain available judges and rooms for rounds
- strikes/conflicts apply at person, judge, room, entry, school, or hybrid relationship level

Notes:

- `ballot` is the operational center of live competition
- debate-style side and speech-style order both live on the same table
- not all ballot behavior is explicit columns; much of it is inferred from related `score` rows

## 5. Results and Advancement Cluster

Primary entities:

- `protocol`
- `protocol_setting`
- `tiebreak`
- `result_set`
- `result`
- `result_key`
- `result_value`
- `qualifier`
- `sweep_set`
- `sweep_rule`
- `sweep_event`
- `sweep_award`
- `sweep_include`

Observed relationships:

- one protocol owns many tiebreak rows
- one round points at one protocol
- one result set owns result keys and results
- one result owns many result values
- one tournament/event can have multiple result sets

Notes:

- ranking and reporting are modeled as configurable generated outputs, not a single canonical standings table
- this is powerful but pushes complexity into generation logic

## 6. Settings and Metadata Cluster

Primary entities:

- `setting`
- `setting_label`
- `tabroom_setting`
- `tourn_setting`
- `event_setting`
- `round_setting`
- `panel_setting`
- `judge_setting`
- `entry_setting`
- `student_setting`
- `person_setting`
- `school_setting`
- `chapter_setting`
- `circuit_setting`
- `region_setting`
- `protocol_setting`
- `rpool_setting`
- `jpool_setting`

Notes:

- this is effectively a distributed EAV system
- behavior is often not discoverable from table structure alone; tag usage must also be examined
- state and override behavior can be encoded through settings rather than explicit state columns

## High-Value Relationships

The most important relationship chains visible so far are:

1. Tournament execution:
   - `tourn -> event -> round -> panel -> ballot -> score`

2. Participant modeling:
   - `chapter -> school -> entry -> entry_student -> student`

3. Judging:
   - `chapter/person -> judge -> ballot -> score`

4. Published standings:
   - `protocol -> tiebreak`
   - `result_set -> result -> result_value`

5. Access:
   - `person -> permission -> {tourn|chapter|circuit|region|district|category}`

## Lifecycle and State Models

## 1. Tournament Lifecycle

Visible state signals:

- `tourn.hidden`
- `tourn.start`
- `tourn.end`
- `tourn.reg_start`
- `tourn.reg_end`
- many tournament settings controlling visibility, registration, and operations

Observed practical phases:

- draft/requested
- registration open
- registration managed/locked
- live operations
- results published
- historical/archive

Important note:

- the tournament lifecycle appears to be mostly settings- and date-driven rather than represented by one status column

## 2. Entry Lifecycle

Direct schema state:

- `entry.active`
- `entry.dropped`
- `entry.waitlist`
- `entry.unconfirmed`
- `entry.ada`

Important direct evidence:

- triggers `insert_entry_active` and `update_entry_active` derive `active`
- `active = 0` when dropped, waitlisted, or unconfirmed

First-pass lifecycle:

- created
- unconfirmed or pending
- active
- waitlisted
- dropped
- possibly disqualified or excluded through settings rather than base columns

Important note:

- entry state is split between explicit columns and `entry_setting` tags such as DQ, no-elims, sweeps exclusion, preferred flight, breakout values, ballot notes

## 3. Judge Lifecycle

Direct schema state:

- `judge.active`
- `judge.ada`
- `judge.obligation`
- `judge.hired`
- judge-hire request rows

Practical lifecycle inferred:

- available in chapter/person context
- attached or requested into tournament
- active/eligible
- assigned to pools and panels
- ballots started/submitted/audited

Important note:

- judge readiness and eligibility appear to be partly explicit and partly settings-driven

## 4. Round Lifecycle

Direct schema state:

- `round.type`
- `round.flighted`
- `round.published`
- `round.start_time`
- posting flags: `post_primary`, `post_secondary`, `post_feedback`
- `round_setting` tags such as `ignore_results`, `disaster_checked`, and other operational flags

Practical lifecycle:

- created
- configured
- paired / assigned
- published
- in progress
- ballots entering / auditing
- complete
- used as result/break basis

Important note:

- the live operational state is distributed across `round`, `panel`, `ballot`, and settings rather than normalized into a single status

## 5. Panel Lifecycle

Direct schema state:

- `panel.bye`
- `panel.started`
- `panel.publish`
- `panel.bracket`
- `panel.room`

Practical lifecycle:

- created during pairing
- assigned room/flight/bracket
- published with or without room/judge
- started
- completed once all ballots are entered and audited

Important note:

- panel is both a pairing container and a live operational unit

## 6. Ballot Lifecycle

Direct schema state:

- `ballot.bye`
- `ballot.forfeit`
- `ballot.tv`
- `ballot.audit`
- `ballot.judge_started`
- `ballot.started_by`
- `ballot.entered_by`
- `ballot.audited_by`

Practical lifecycle:

- allocated to entry/judge/panel
- judge started
- entered once
- re-entered / verified in double-entry flows
- audited
- potentially marked bye, forfeit, or tabroom-violation related

Important note:

- acceptance and completeness of a ballot are derived both from ballot columns and presence/shape of `score` rows

## 7. Result Lifecycle

Direct schema state:

- `result_set.generated`
- `result_set.published`
- `result_set.bracket`
- `result_set.tag`

Practical lifecycle:

- result set defined
- generated from a round/protocol context
- optionally published
- consumed by public pages, coach views, speaker award views, break generation, and sweepstakes

Important note:

- published results are generated artifacts, not just query views over raw ballots

## Polymorphism and Variation

The most important model variation is by event type:

## Debate

- entry-centric
- side matters
- win/loss, ballots, points, speaker scores matter
- elim bracket position matters

## Speech

- section-centric
- speaker order matters
- rank aggregation matters
- repeat hits, school avoidance, and order balancing matter

## Congress

- chamber/session-centric
- chair/non-chair distinctions matter
- student votes and PO-related scoring appear

## WUDC / WSDC

- distinct special handling is visible in ballot, tiebreak, and result code

Implication:

- a future build cannot flatten these into one generic round/ballot contract without an explicit polymorphism strategy

## Important Invariants and Constraints

High-confidence invariants from the schema:

- one `entry` belongs to one tournament-local `school` and one `event`
- one `entry_student` pair is unique
- one `ballot` for a given entry/judge/panel is unique
- one `(panel, judge, side, speakerorder)` combination is unique
- one `(round, tag)` round setting is unique
- one `(tourn, tag)` tournament setting is unique
- one `(event, tag)` event setting is indexed and effectively scoped
- one `(person, tag)` person setting is unique
- one `(school, tag)` school setting is unique

Operational invariants inferred from code and workflows:

- dropped/waitlisted/unconfirmed entries should not behave as active entries
- ignored rounds should not affect results generation
- a panel should not produce inconsistent ballots for the same judge-entry combination
- result generation is protocol-sensitive and event-type-sensitive

## Audit and Recoverability Signals

Direct evidence of recoverability/audit signals:

- `change_log`
- `campus_log`
- ballot entered/audited/by whom
- round dump / import / backup surfaces
- `disaster_checked` round setting

Implication:

- operator auditability is part of the product model, not just an implementation convenience

## High-Risk Areas For Rebuild Modeling

The highest-risk data/model areas are:

- distributed EAV settings
- permission scoping across multiple dimensions
- round/panel/ballot composite state
- polymorphic event handling
- results as generated flexible datasets rather than fixed columns
- school/chapter/person/student/judge identity boundaries
- break and qualifier modeling

## Open Questions For Later Passes

- which entities are globally durable versus tournament-local copies?
- where is the cleanest boundary between chapter/program data and tournament registration data?
- which settings represent true state transitions versus optional presentation/configuration?
- how much of results generation is ephemeral versus persisted and relied on operationally?
- which audit/history tables are critical to product trust versus nice-to-have?
- how should online/room credential material be treated in a future architecture?
