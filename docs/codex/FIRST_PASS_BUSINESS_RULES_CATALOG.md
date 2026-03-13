# First-Pass Business Rules Catalog

## Purpose

This document captures the first-pass business-rules inventory for the current Tabroom platform.

Its purpose is to identify:

- the highest-value rule clusters,
- where those rules are encoded,
- which rules appear configurable,
- which rules vary by event type,
- which rules need domain-expert review before a rebuild.

This is not yet a complete rule specification.

## Source Basis

Primary evidence used in this pass:

- `web/panel/round/pair_powered.mas`
- `web/panel/round/pair_speech.mas`
- `web/tabbing/break/break_debate.mhtml`
- `web/tabbing/break/break_speech.mhtml`
- `web/tabbing/results/order_speakers.mas`
- `web/funclib/tiebreak_types.mas`
- `web/panel/schemat/disaster_check.mhtml`
- `web/funclib/event_speaker_order.mas`
- `doc/sql/current-schema.sql`
- `docs.tabroom.com`
- `doc/howtos/tournament-manual.pdf`

## First-Pass Conclusions

High-confidence observations:

- the hardest product logic lives in pairing, assignment, ballot interpretation, tiebreaking, breaks, and operational recovery
- many rules are configurable rather than hard-coded universally
- debate, speech, congress, WUDC, and WSDC diverge materially
- some important rules are encoded through weighted-penalty heuristics rather than simple booleans

## Rule Cluster 1: Entry Activation and Eligibility

Primary evidence:

- `entry` trigger logic in `doc/sql/current-schema.sql`
- `register` and `tabbing` surfaces referencing dropped, waitlist, unconfirmed, DQ, no-elims, and sweeps-exclusion settings

Observed rules:

- dropped, waitlisted, and unconfirmed entries are not treated as active
- entry eligibility for breaks/elims can be overridden through settings such as `no_elims`
- entries can be excluded from sweeps through settings
- ADA and preferred-flight attributes influence operational handling

Why it matters:

- active/eligible is not a single field; rebuild logic must preserve the distinction between participation state and competitive eligibility

## Rule Cluster 2: Debate Powermatching and Bracketing

Primary evidence:

- `web/panel/round/pair_powered.mas`

Observed rules:

- powered debate pairing is based on previous round standings and records
- side constraints apply based on round count parity unless explicitly disabled
- same-school hits are heavily penalized unless allowed by event setting
- repeat hits are penalized
- wrong-side assignments are penalized
- pullups are managed with configurable methods and penalties
- repeat pullup exposure can be penalized
- hybrid conflicts and regional constraints can influence pairing
- byes may be assigned automatically when judge supply is insufficient, with fairness logic based on prior bye counts

Rule shape:

- this appears to be a weighted optimization problem rather than a simple deterministic bracket algorithm

High-risk note:

- the numeric penalty model is part of the product behavior, even if the specific weights eventually change

## Rule Cluster 3: Speech Sectioning and Diversity Balancing

Primary evidence:

- `web/panel/round/pair_speech.mas`

Observed rules:

- speech rounds are grouped into sections/panels rather than paired opponents
- same-school hits are strongly penalized
- repeat encounters are penalized
- regional/state/district separation may be absolute or weighted depending on tournament mode
- previous speaker order is tracked and balanced across rounds
- titles/special attributes may be distributed across sections
- some specialized national/district modes tighten separation rules further

Rule shape:

- section generation appears to balance section size, conflict avoidance, and order fairness

High-risk note:

- speech scheduling cannot be treated as “debate pairing without sides”

## Rule Cluster 4: Speaker Order Assignment

Primary evidence:

- `web/funclib/event_speaker_order.mas`
- `web/panel/round/speaker_order.mhtml`
- `web/panel/round/speaker_order_improve.mhtml`
- `ballot.speakerorder`

Observed rules:

- speaker order is tracked across prior rounds per entry
- order fairness is a distinct optimization target
- order needs vary by panel size
- prior order exposure affects future assignment

Why it matters:

- speaker order is a real fairness domain, not presentation metadata

## Rule Cluster 5: Judge Preference, Conflict, and Strike Handling

Primary evidence:

- `web/register/entry/strikes.mhtml`
- `web/funclib/round_pref_data.mas`
- `web/funclib/event_judgeprefs.mas`
- `web/funclib/event_strike_judges.mas`
- `web/funclib/chapter_conflicts.mas`
- `web/funclib/entry_conflicts.mas`
- `web/funclib/person_conflict.mas`
- `web/funclib/school_conflicts.mas`

Observed rules:

- conflicts and strikes exist at multiple levels
- school-level strikes, entry-level strikes, and explicit conflicts coexist
- preference access itself is permission-scoped
- preference sheets and deadlines are configurable
- assignment quality is influenced by prefs, strikes, experience, and eligibility

High-risk note:

- “judge assignment” is really a synthesis of eligibility, conflict avoidance, preference quality, and operational supply

## Rule Cluster 6: Room Assignment and Room Suitability

Primary evidence:

- `room.quality`
- `room.ada`
- `room_strike`
- `web/funclib/room_strikes.mas`
- `web/funclib/site_room_strikes.mas`
- panel schemat room assignment surfaces

Observed rules:

- rooms can be unavailable by time, event, entry, or judge
- room quality is a meaningful placement input
- ADA support must be respected
- online/hybrid room credentials are sometimes embedded in room records

Why it matters:

- rooming is more than “pick any empty room”

## Rule Cluster 7: Ballot Structure and Entry Validation

Primary evidence:

- `web/tabbing/entry/index.mhtml`
- `web/tabbing/entry/card_save.mhtml`
- `web/tabbing/entry/combined.mhtml`
- `web/lib/Tab/Ballot.pm`
- `score` schema

Observed rules:

- ballot structure varies heavily by event type
- ballots may include win/loss, ranks, points, per-student values, chair distinctions, or vote-style values
- byes and forfeits suppress or alter expected score entry
- some ballot entry paths use double-entry/audit verification
- combined ballots and rubric-driven speaker scoring are configurable
- ignored rounds should not contribute to downstream calculations

Why it matters:

- there is no one generic ballot payload

## Rule Cluster 8: Tiebreak Typing and Result Inputs

Primary evidence:

- `web/funclib/tiebreak_types.mas`
- `tiebreak`
- `protocol`
- `protocol_setting`

Observed rules:

- a protocol contains a prioritized list of tiebreak definitions
- tiebreak applicability depends on round type and sometimes specific round
- the system dynamically infers whether a result context needs win/loss, point, rank, TV, entry-vote, or special values
- composite/child tiebreak sets exist
- chair/non-chair distinctions matter in some formats
- truncation, multiplier, high-low, and threshold behavior are all configurable

Why it matters:

- result computation is protocol-driven, not hard-coded per event alone

## Rule Cluster 9: Speaker Awards and Per-Student Ordering

Primary evidence:

- `web/tabbing/results/order_speakers.mas`

Observed rules:

- speaker awards depend on a designated speaker protocol
- forfeits, byes, and ignored rounds affect scoring treatment
- rubric-based speaker scoring may replace default point behavior
- rounds are truncated to panel-size minima in some contexts
- novice and DQ-related handling exists
- WSDC-specific reply/refute distinctions exist

Why it matters:

- speaker awards are a parallel ranking engine, not a simple aggregate of team results

## Rule Cluster 10: Break Generation and Advancement

Primary evidence:

- `web/tabbing/break/break_debate.mhtml`
- `web/tabbing/break/break_speech.mhtml`
- `web/tabbing/break/break_congress.mhtml`

Observed rules:

- break generation creates new rounds and bracket result sets
- qualifying windows are defined by start/end seed ranges
- ineligibles can be removed via settings such as `no_elims`
- elim-to-elim seeding may preserve bracket position instead of recomputing from scratch
- speech break generation depends on number of sections and event-specific elim method
- national/district modes alter judge counts, snake behavior, region protection, and seed balancing

Why it matters:

- break logic creates both standings artifacts and new operational rounds

## Rule Cluster 11: Sweepstakes and Awards

Primary evidence:

- `sweep_set`
- `sweep_rule`
- `sweep_event`
- `web/funclib/sweeps/*`
- `web/tabbing/results/sweep_*`

Observed rules:

- sweepstakes are configurable by award target, event set, counting method, and protocol
- point allocation can vary by place, count, round, truncation, and included result sets
- entries, schools, and individuals can all be targets

Why it matters:

- sweepstakes is its own configurable rule engine and should not be treated as a report-only afterthought

## Rule Cluster 12: Disaster Detection and Recovery

Primary evidence:

- `web/panel/schemat/disaster_check.mhtml`
- `web/panel/schemat/move_*`
- `web/panel/schemat/flight_judge_swap.mhtml`
- `web/panel/schemat/import_round.mhtml`
- `web/panel/schemat/upload_backup.mhtml`

Observed rules:

- the system explicitly checks for double-booked rooms
- the system explicitly checks for double-booked judges
- judges moving between rooms across flights is treated as a problem class
- timeslot-overlap conflicts across rounds are checked
- operators can move entries, judges, panels, rooms, and flights after pairing

Why it matters:

- recovery operations are a first-class part of the product, not an admin edge case

## Rule Cluster 13: Publication and Visibility

Primary evidence:

- `round.published`
- `panel.publish`
- `result_set.published`
- round posting columns
- public/index/results routes

Observed rules:

- publication happens at multiple levels
- a round can be operationally present before it is publicly visible
- different audiences may see different artifacts at different times
- some result sets are coach-only or bracket-only

Why it matters:

- visibility is not binary and may need audience-aware publication control

## Rule Cluster 14: Specialized NSDA / District / National Modes

Primary evidence:

- district-related settings and routes
- `qualifier`
- NSDA-specific admin/export flows
- district/nats branches in pairing and break generation logic

Observed rules:

- district and nationals handling materially changes pairing and break rules
- qualifier counts and qualification pipelines are explicit product logic
- district-specific forms, ballot headers, orders, and exports exist

Why it matters:

- these are not cosmetic integrations; they alter competition rules and deliverables

## Rule Cluster 15: Non-Rule-Like Logic That Still Behaves As Product Logic

Observed examples:

- print/PDF generation choices affect operational workflow
- audit/double-entry behavior affects trust model
- online/hybrid settings influence rooming and publication
- imported backups and round dumps affect recoverability

Why it matters:

- some operational behavior may sit outside pure domain math but is still essential to parity

## Event-Type Variation Summary

## Debate

Dominant rule areas:

- powermatching
- side fairness
- opponent avoidance
- byes
- win/loss and ballots
- speaker awards
- elim bracket preservation

## Speech

Dominant rule areas:

- section balancing
- school/region repeat avoidance
- speaker order fairness
- rank aggregation
- multi-section elimination seeding

## Congress

Dominant rule areas:

- chamber/session assignment
- chair/non-chair scoring
- vote-style data
- special result/tiebreak handling

## WUDC / WSDC

Dominant rule areas:

- special ballot semantics
- specialized speaker metrics
- specialized side/reply handling

## Highest-Risk Rule Areas For Later Extraction

The highest-risk areas to deepen next are:

- debate powermatching heuristics
- speech sectioning/scattering
- judge assignment with prefs/conflicts/strikes
- ballot payload variants by event type
- protocol/tiebreak composition
- break generation and bracket carry-forward
- speaker awards
- disaster recovery and repanel workflows

## Open Questions For Later Passes

- which numeric penalties are product-critical versus historical tuning?
- which recovery workflows are most common in real tournaments?
- which tiebreak protocol patterns are standard versus bespoke per tournament?
- where do congress-specific rules diverge most sharply from speech and debate?
- which district/nationals rules are essential for Release 1 versus later?
- where does the docs site disagree with live code behavior?
