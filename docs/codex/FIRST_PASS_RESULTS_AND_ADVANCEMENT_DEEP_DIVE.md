# First-Pass Results and Advancement Deep Dive

## Purpose

This document captures a deeper first-pass analysis of how the current Tabroom platform computes standings, publishes results, and advances entries into subsequent rounds.

Its purpose is to make explicit:

- how raw ballots become ordered results,
- how published round results differ from overall standings,
- how advancement and break generation work,
- where event-type variation changes the result model,
- which parts of this domain are most rebuild-sensitive.

This is a descriptive source-analysis artifact, not a target implementation design.

## Source Basis

Primary evidence used:

- `web/tabbing/results/order_entries.mas`
- `web/tabbing/results/results_table.mas`
- `web/funclib/results_debate.mas`
- `web/funclib/results_speech.mas`
- `web/funclib/results_congress.mas`
- `web/tabbing/break/break_debate.mhtml`
- `web/tabbing/break/break_speech.mhtml`
- `web/tabbing/break/break_wudc.mhtml`
- `web/tabbing/results/order_speakers.mas`
- `doc/sql/current-schema.sql`

Key supporting schema:

- `protocol`
- `protocol_setting`
- `tiebreak`
- `result_set`
- `result`
- `result_key`
- `result_value`
- `qualifier`
- `sweep_*`

## First-Pass Conclusions

High-confidence observations:

- Tabroom does not treat “results” as a single flat standings table
- result generation is protocol-driven and event-type-sensitive
- published round results, full tournament standings, speaker awards, breaks, and sweepstakes are related but distinct artifacts
- advancement is not just display logic; it creates new operational rounds and new bracket result sets

## Core Result Model

At a high level, the platform appears to separate:

1. Raw competitive evidence:
   - `ballot`
   - `score`
   - `student_vote`
   - `student_ballot`

2. Ranking logic:
   - `protocol`
   - `tiebreak`
   - protocol settings

3. Persisted generated outputs:
   - `result_set`
   - `result`
   - `result_key`
   - `result_value`

This means the current product model is:

- configurable raw scoring
- configurable ranking rules
- persisted generated artifacts for downstream use

## Standing Generation Through `order_entries.mas`

Primary source:

- `web/tabbing/results/order_entries.mas`

Observed role of this file:

- this appears to be the central entry-ordering engine for event standings and break seeding

Observed important behaviors:

- it requires a round context and usually a protocol context
- it normalizes some round types into prelim equivalents for calculation
- it excludes rounds with `ignore_results`
- it can compute against composite protocols
- it blocks unsupported composite-of-composite or self-referential opponent-seed scenarios
- it loads scores, ballots, panel structure, and per-entry DQ state
- it changes behavior based on event type
- it supports special modes like NSDA or opponent-wins-only calculations

Implication:

- entry ordering is its own domain engine, not a simple SQL sort

## Protocol and Tiebreak Dependence

Observed from `order_entries.mas` and prior tiebreak analysis:

- each round points at a protocol
- protocols define prioritized tiebreak tiers
- tiebreaks can apply to all rounds, previous rounds, specific rounds, or specific round types
- tiebreaks can depend on opponent results
- tiebreaks can use truncation, high-low logic, multipliers, and chair constraints

Implication:

- “standings” cannot be modeled correctly without first modeling protocol evaluation

## Filtering and Eligibility Rules In Result Computation

Observed behaviors:

- ignored rounds are excluded from standings calculations
- waitlisted and unconfirmed entries are excluded from standard entry ordering
- dropped entries receive special handling rather than being simply filtered everywhere
- DQ status is consulted
- forfeit treatment is protocol-sensitive, including “forfeits never break”
- some special modes intentionally alter dropping/truncation behavior

Implication:

- result generation depends on participation state and not just score rows

## Published Round Results Versus Overall Standings

Primary sources:

- `results_debate.mas`
- `results_speech.mas`
- `results_congress.mas`

Observed distinction:

- round-result display files are focused on what can be shown for one round
- overall standings/order generation is handled separately

Observed round-result gating:

- round publication matters
- `post_primary`, `post_secondary`, and in congress `post_feedback` matter
- `panel.publish` can interact with `judge_publish_results`
- some score types become visible at different publication thresholds

Implication:

- “results publication” is audience- and stage-sensitive, not a simple published boolean

## Debate Result Publication

Primary source:

- `results_debate.mas`

Observed published result behaviors:

- win/loss becomes visible at primary post threshold
- points/ranks/refute/speaker values have their own post-threshold behavior
- motion visibility depends on round settings and publication state
- byes and forfeits get formatted as outcome-specific artifacts
- judge-level outcome detail can be shown

Important note:

- debate round results are panel-centric and side-aware

## Speech Result Publication

Primary source:

- `results_speech.mas`

Observed published result behaviors:

- results are section-centric
- speaking order is important in display
- ranks appear before or with points depending on publication threshold
- names and school display are conditional based on whether the code already embeds them

Important note:

- speech round results emphasize panel/section tables rather than head-to-head outcomes

## Congress Result Publication

Primary source:

- `results_congress.mas`

Observed published result behaviors:

- congress uses chamber-centric displays
- chair and non-chair judges are distinguished in display labels
- rank, point, and speech-based content can all appear
- `post_feedback` is an additional visibility dimension

Important note:

- congress publication is not just a speech variant; it has its own score payload and visibility model

## Speaker Awards As A Parallel Ranking System

Primary source:

- `order_speakers.mas`

Observed behavior:

- speaker awards depend on a designated speaker protocol
- they operate per student, not just per entry
- they are affected by forfeits, byes, ignored rounds, rubric choices, and event-type specifics
- special values like WSDC reply/refute handling are present

Implication:

- a rebuild should not conflate overall standings and speaker awards into one generalized result routine

## Break Generation

Primary sources:

- `break_debate.mhtml`
- `break_speech.mhtml`
- `break_wudc.mhtml`

Observed common pattern:

1. validate source round and configuration
2. create or update target elimination/break round
3. create or update a bracket `result_set`
4. clear existing bracket results or round contents if necessary
5. use `order_entries.mas` output to pick advancing entries
6. generate panels and ballots for the next round

Implication:

- advancement is both a ranking action and a round-construction action

## Debate Break Behavior

Observed from `break_debate.mhtml`:

- break windows are defined by start/end seed range
- `no_elims` ineligibility is honored
- elim-to-elim advancement preserves bracket position rather than fully recomputing seeding
- bracket result sets are created or reused
- NSDA district debate has special qualifier-count handling

Important note:

- debate advancement is bracket-aware and preserves competitive tree structure

## Speech Break Behavior

Observed from `break_speech.mhtml`:

- advancement requires number of sections/panels
- target rounds may be created with round type, timeslot, protocol, and judge count
- snake distribution is used for paneling the advancing field
- multiple rounds may have to be reseeded against a prior “previous” round depending on round numbering
- NSDA/NCFL and novice modes influence behavior

Important note:

- speech advancement combines seeding, sectioning, and future panel construction

## WUDC / Special-Format Break Behavior

Observed from `break_wudc.mhtml`:

- advancing field can be padded with byes up to a bracket-compatible size
- panels can be randomized or reused
- bracket result sets are created and synchronized
- ballot structures are panel-based and four-entry aware

Implication:

- special formats need their own advancement logic, not just a different standings display

## Result Sets As Product Artifacts

Observed from schema and break code:

- `result_set` has label, published, bracket, tag, event, qualifier, and sweep references
- `result` stores rank/place-level placements
- `result_value` stores key-value metrics per result
- break generation writes bracket result rows intentionally

Implication:

- result sets are first-class outputs that downstream screens and operations depend on

## Qualifiers and Specialized Advancement

Observed evidence:

- `qualifier` table
- district/national special flows
- break generation and result pages referencing qualifiers and alternates

Observed behaviors:

- some tournaments need qualifier counts, alternates, and qualification-specific reporting
- district workflows interact with standings and break logic rather than sitting outside them

Implication:

- “qualification pipeline” is part of the result domain, not a post-processing report

## Sweepstakes and Related Award Logic

Observed evidence:

- `sweep_tourn.mas`
- `sweep_schools.mas`
- `sweep_students.mas`
- `nsda_sweepstakes.mhtml`
- `sweep_*` tables

Observed behavior:

- sweepstakes computation is separate from basic event standings
- award targets vary by entry/school/individual
- sweep inclusion rules and event matching appear configurable

Implication:

- results architecture needs a place for award aggregation distinct from event standings

## Publication Rules and Audience Sensitivity

Observed publication inputs:

- `round.published`
- `round.post_primary`
- `round.post_secondary`
- `round.post_feedback`
- `panel.publish`
- `event_setting.judge_publish_results`
- motion-specific round settings

Implication:

- a rebuild will need explicit publication rules by artifact and audience instead of one monolithic “published” state

## Highest-Risk Rebuild Requirements

The most load-bearing result/advancement requirements to preserve are:

- protocol-driven standings computation
- event-type-specific round result publication
- ignored-round handling
- forfeit/DQ/bye treatment in standings and speaker awards
- bracket result set creation and maintenance
- elim carry-forward behavior
- speaker award logic as a parallel ranking system
- district/national qualifier specialization

## Recommended Next Extraction Areas

The next source-analysis passes should focus on:

- deeper extraction of `order_entries.mas` tiebreak evaluation internals
- explicit result-set usage across public and operator screens
- sweepstakes-specific rule extraction
- qualifier and district advancement edge cases

## Open Questions For Later Passes

- which generated result sets are operationally critical versus display-only?
- how often do operators regenerate or manually correct result artifacts?
- which publication thresholds are relied on most in live operations?
- where do district/national requirements most sharply diverge from general tournament logic?
