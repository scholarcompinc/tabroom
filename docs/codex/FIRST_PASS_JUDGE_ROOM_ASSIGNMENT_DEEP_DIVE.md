# First-Pass Judge and Room Assignment Deep Dive

## Purpose

This document captures a deeper first-pass analysis of how the current Tabroom platform assigns judges and rooms to panels and how operators review and override those assignments.

Its purpose is to make explicit:

- what makes a judge or room eligible,
- what makes an assignment desirable versus merely legal,
- how paneling interacts with prefs, strikes, pools, and supply constraints,
- where operator override workflows are integral to the product.

This is a descriptive source-analysis artifact, not a target implementation design.

## Source Basis

Primary evidence used:

- `web/funclib/clean_to_judge.mas`
- `web/funclib/clean_judges.mas`
- `web/funclib/round_available_judges.mas`
- `web/funclib/clean_rooms.mas`
- `web/funclib/judges_by_pref.mas`
- `web/panel/schemat/judge_add.mhtml`
- `web/panel/schemat/panel_room_save.mhtml`
- `web/panel/schemat/seating_assign.mhtml`
- `web/panel/schemat/prefs_report.mhtml`
- `web/panel/schemat/disaster_check.mhtml`
- `doc/sql/current-schema.sql`

Key supporting schema:

- `judge`
- `judge_setting`
- `panel`
- `ballot`
- `room`
- `room_strike`
- `strike`
- `rating`
- `rating_tier`
- `jpool*`
- `rpool*`

## First-Pass Conclusions

High-confidence observations:

- judge assignment is a constraint-and-quality problem, not a simple “pick an available judge” routine
- room assignment is similarly constrained by time overlap, strikes, pools, site choice, room quality, and ADA/online considerations
- the system assumes operator review and intervention are normal parts of the workflow
- the distinction between “eligible,” “preferred,” and “operationally convenient” is central

## Core Assignment Model

At a high level, the assignment domain combines:

1. Structural context:
   - round
   - panel
   - event
   - timeslot
   - site
   - flight

2. Resource inventories:
   - judges
   - rooms
   - judge pools
   - room pools

3. Constraint systems:
   - strikes
   - conflicts
   - same-school rules
   - region/district rules
   - time overlap rules
   - room strikes

4. Quality systems:
   - preference scores
   - judge ratings
   - diversity flags
   - tab ratings
   - room quality

5. Recovery systems:
   - steal/swap judge
   - move panel
   - reassign room
   - disaster check

## Judge Eligibility Baseline

Primary source:

- `clean_to_judge.mas`

Observed baseline disqualifiers:

- same-school judge-entry pairing unless allowed
- NCFL same-region restrictions
- repeat judging restrictions, with special elim handling
- explicit strike ratings
- explicit entry/school/event/region strikes
- conflict-style strikes

Observed nuance:

- some restrictions return hard disqualification
- others return a softer signal like `"elim"` indicating conditional unfitness or reuse only in certain contexts

Implication:

- the assignment model is not binary; some prior-judging scenarios are context-sensitive

## Judge Availability At Round Level

Primary source:

- `round_available_judges.mas`

Observed round-level filters:

- active judge only
- judge pool restriction if the round has a jpool
- otherwise category matching
- no overlapping time strike
- no event-specific strike
- no simultaneous judging in overlapping timeslots across other events

Implication:

- a judge can be generally active but unavailable for a particular round/time/context

## Judge Candidate Scoring and Ranking

Primary sources:

- `clean_judges.mas`
- `judges_by_pref.mas`
- `prefs_report.mhtml`

Observed quality inputs:

- preference style by category (`ordinals`, `tiered`, `tiered_round`, `ndt`-derived variants)
- average and standard deviation of preference/rating data
- obligation/hired round counts
- tab rating
- diversity flag
- parli flag
- online hybrid flag
- standby status

Observed structural inputs:

- category and alt-category fit
- jpool scope
- school and region relationships
- per-panel entry makeup
- bracket context
- whether the round is blind or pre-paneled

Implication:

- the system distinguishes legal judges from better judges for a given panel

## Preference System Coupling

Observed from `judges_by_pref.mas` and `prefs_report.mhtml`:

- category preference style determines how judge quality is summarized
- prelim versus elim rounds can use different preference styles in some modes
- pref data can be aggregated into average and standard deviation metrics
- operator tools surface this data when reviewing panel quality

Implication:

- a future product must preserve the ability to reason about judge quality, not just judge legality

## Pooling Behavior

Observed pool mechanisms:

- jpool can narrow judge eligibility to a subset
- override settings can permit category overrides within pool use
- rpool can narrow available rooms to a subset

Implication:

- pools are assignment-scoping primitives, not merely organizational labels

## Room Eligibility Baseline

Primary source:

- `clean_rooms.mas`

Observed room-level filters:

- round must resolve to a site, or the operator is redirected to fix site assignment
- room must belong to the round’s site
- room cannot be inactive or deleted
- room cannot already be used in an overlapping timeslot
- flight-specific overlap matters when assigning by panel
- room time strikes disqualify a room
- event-specific room strikes disqualify a room
- room strikes tied to a panel’s judge or entry can disqualify a room
- room pools can restrict the usable set further

Observed ordering:

- rooms are sorted by name and then by quality/score signals

Implication:

- room assignment is partly availability and partly suitability

## Room Suitability Signals

Observed explicit room attributes:

- `quality`
- `capacity`
- `rowcount`
- `seats`
- `ada`
- notes
- URLs/passwords for online mode

Observed uses:

- quality affects ordering
- seating layout drives actual seat assignment logic
- ADA and online credentials indicate special operational suitability

Implication:

- rooms are domain resources with meaningful attributes, not just labels

## Panel-Level Judge Operations

Primary source:

- `judge_add.mhtml`

Observed operator behaviors:

- manually add judge to a panel
- steal judge from another panel in the same timeslot/flight
- assign chair status
- mirror the judge add into linked congress PO contest sections when needed
- log all operator assignment actions

Implication:

- assignment UI is explicitly built for live operator intervention

## Panel-Level Room Operations

Primary source:

- `panel_room_save.mhtml`

Observed operator behaviors:

- move a panel to a new room
- propagate room changes across same-letter congress panels for the event
- check overlapping room conflicts unless bypassed
- log operator assignment actions

Implication:

- room changes can have cross-round or cross-section coupling, especially in congress

## Seating Assignment Within Rooms

Primary source:

- `seating_assign.mhtml`

Observed behaviors:

- speaking/seat positions can be assigned within a room grid
- assignment can use random, school-aware, or inversion methods
- prior seat/row history in other panels is consulted
- room capacity, rowcount, and seats constrain valid placement

Implication:

- “room assignment” for some events extends beyond panel-to-room into participant seat placement

## Assignment Constraints Visible In `clean_judges.mas`

Observed constraint classes:

- direct judge-entry strikes
- hybrid strikes
- school-based judge restrictions
- category fit
- jpool restrictions
- standby behavior
- no first-year judge settings
- allow-repeat-elims / allow-repeat-judging / allow-repeat-prelim-side
- region constrain / region avoid modes
- blind-mode behavior

Important note:

- this is one of the densest rule areas in the product and likely one of the easiest to under-specify in a rebuild

## Operational Supply and Burden

Observed evidence:

- `judge.obligation`
- `judge.hired`
- burden-related helpers under `web/funclib/judgemath/*`
- availability counting in `round_available_judges.mas`
- autobye logic in debate pairing

Implication:

- assignment quality cannot be separated from supply adequacy and burden accounting

## Disaster Detection As Assignment Validation

Primary source:

- `disaster_check.mhtml`

Observed assignment failure classes:

- double-booked judges within a round
- judges changing rooms across flights
- double-booked rooms
- cross-timeslot judge conflicts

Implication:

- the assignment system needs explicit validation and repair flows, not just one-shot generation

## Review and Override Are First-Class Workflows

Observed from panel/schemat surfaces:

- judge add/remove/swap
- room save/move
- flight judge swap
- panel switch
- move panel / move speech
- score remove
- import/download/upload backup

Implication:

- operator override is not an edge-case admin feature; it is part of the live tab-room operating model

## Highest-Risk Rebuild Requirements

The most load-bearing assignment requirements to preserve are:

- judge eligibility filtering against strikes/conflicts/history
- category/jpool/pool-aware candidate selection
- preference-quality visibility for operators
- room eligibility against time overlap and room strikes
- resource-aware assignment under flights and overlapping timeslots
- operator steal/swap/move workflows
- disaster-detection compatibility
- seating/room-layout handling where it materially affects live rounds

## Recommended Next Extraction Areas

The next deeper source-analysis passes should focus on:

- `judge_priority.mas`
- `panel_judgeadd.mas`
- `round_pref_data.mas`
- `room_strikes.mas`
- `judge_standby.mas`
- `judgemath/*`

## Open Questions For Later Passes

- which assignment quality metrics do experienced tabbers rely on most?
- how often are blind-mode and specialty modes used in practice?
- which judge/room override actions are the most common during live operations?
- which burden and hire calculations are essential for Release 1 versus later?
