# First-Pass Pairing and Round Generation Deep Dive

## Purpose

This document captures a deeper first-pass analysis of how the current platform creates competitive rounds.

Its purpose is to make explicit:

- how round generation differs by event type,
- which rule inputs affect pairing/sectioning/chambering,
- where the system uses optimization heuristics versus fixed brackets,
- which parts of this domain are most rebuild-sensitive.

This is still a descriptive artifact, not a target design.

## Source Basis

Primary evidence used:

- `web/panel/round/pair_debate.mas`
- `web/panel/round/pair_powered.mas`
- `web/panel/round/pair_speech.mas`
- `web/panel/round/pair_congress.mas`
- `web/panel/round/pair_wudc.mas`
- `web/funclib/make_pairing_hash.mas`
- `web/funclib/event_speaker_order.mas`
- `web/panel/schemat/disaster_check.mhtml`
- `doc/sql/current-schema.sql`

## First-Pass Conclusions

High-confidence observations:

- there is no single generic pairing engine
- debate, speech, congress, and WUDC/WSDC each use materially different round generation logic
- pairing is not just “sort by standings and match”; it is constrained by history, fairness, conflicts, room/judge supply, and special tournament modes
- some algorithms are clearly optimization/penalty-based rather than pure deterministic tree logic

## Common Structural Pattern

Despite event-type differences, most round generation follows a shared high-level shape:

1. load event/tournament settings
2. delete or clear any existing round content
3. build an eligible participant set
4. compute historical constraints from prior rounds
5. compute fairness/avoidance scores or allowed positions
6. create `panel` rows
7. create `ballot` rows attaching entries and eventually judges
8. run cleanup/review/disaster tooling

Important note:

- the shared storage shape is `round -> panel -> ballot`
- the meaning of a `panel` changes substantially by event type

## Debate Round Generation

Primary sources:

- `pair_debate.mas`
- `pair_powered.mas`

Observed round types:

- prelim
- preset
- highhigh
- highlow
- elim
- final

Observed key inputs:

- current round number/type
- previous round standings and tiebreaks
- prior opponents
- prior sides
- prior byes
- pullup method
- powermatch method
- bracket-by-ballots mode
- no-side-constraints mode
- school/region/hybrid conflict rules
- number of available judges

Observed core behaviors:

- preliminary debate pairing may powermatch from previous standings
- side constraints can be inferred from round parity or explicitly overridden
- repeated matchups are penalized
- same-school matchups are strongly discouraged unless allowed
- pullups are used to fill brackets and may be optimized to avoid repeat burden
- auto-byes may be assigned when the number of available judges cannot cover all debates
- bye assignment considers prior bye counts
- side labels are configurable (`aff_label`, `neg_label`)

Important structural note:

- debate pairing is partly standings-driven and partly constraint-satisfaction/penalty minimization

## Debate-Specific Fairness Dimensions

First-pass fairness dimensions visible in code:

- bracket fairness
- pullup fairness
- side fairness
- opponent diversity
- school conflict avoidance
- region conflict avoidance
- hybrid conflict avoidance
- bye fairness

Implication:

- a rebuild must treat debate pairing as a multi-objective optimization problem

## Speech Round Generation

Primary source:

- `pair_speech.mas`

Observed key inputs:

- target number of sections
- active entries
- prior sections/hits
- prior speaker order
- school/region/state grouping data
- tournament specialization modes
- title/topic attributes in some modes
- seed basis in some modes

Observed core behaviors:

- active entries are distributed across sections
- section sizes are balanced by min/max/remainder logic
- same-school placements are strongly penalized
- repeat encounters are penalized
- regional/state separation can be mandatory or weighted
- title distribution may be considered
- speaker order exposure is tracked and balanced across rounds
- some special modes limit the advancing field before sectioning

Important structural note:

- speech “pairing” is really section composition plus speaking-order fairness

## Speech-Specific Fairness Dimensions

First-pass fairness dimensions visible in code:

- even section sizes
- school separation
- repeat avoidance
- region/state/district separation
- order fairness
- title/topic distribution in specialized contexts

Implication:

- speech requires both group composition logic and order-assignment logic

## Congress Round Generation

Primary source:

- `pair_congress.mas`

Observed key inputs:

- target number of chambers/panels
- prior congress ties/chained rounds
- school/region/district/state data
- author/topic metadata
- PO contest mode
- NSDA district / NSDA nationals settings
- optional preset seeding

Observed core behaviors:

- congress can clone or realign chamber structures across tied/chained rounds
- PO contest mode can copy prior chamber structure, judges, and entries, then rebalance candidates into copied chambers
- chamber assignment uses explicit penalty structures
- school, state, region, district, bloc-school, author, and autoqual considerations can all affect assignment
- ballots are created with `speakerorder` positions rather than debate sides

Important structural note:

- congress round generation blends chamber assignment, seat/order assignment, and special national/district constraints

## WUDC / Special Debate Format Round Generation

Primary source:

- `pair_wudc.mas`

Observed key inputs:

- prior rank scores
- previous speaking positions
- prior chair ballots
- bracket size of four entries
- position eligibility based on prior use

Observed core behaviors:

- entries are sorted by prior performance
- the system tries to assign entries to speaking positions they have used least
- swaps may occur within a bracket to preserve position fairness
- pullups can still occur to fill brackets of four

Important structural note:

- this is not a simple extension of two-team debate pairing; it is a distinct position-balancing problem

## Shared Historical Inputs Across Formats

Across multiple pairing engines, the system repeatedly consults:

- previous opponents or section-mates
- prior side/order/position exposure
- previous byes
- prior bracket or seed position
- prior speaker results
- prior round/chamber structure

Implication:

- a future pairing domain needs durable access to historical competition state, not just current standings snapshots

## Supply Constraints and Operational Constraints

Observed cross-cutting constraints:

- judge supply can force auto-byes in debate
- room availability and room strikes constrain panel placement
- online mode and hybrid mode alter room assumptions
- multiple flights complicate assignment and fairness
- number of judges per round is itself configurable

Implication:

- pairing cannot be isolated from assignment and resource availability

## Pairing Output Shape

First-pass output pattern by format:

## Debate

- one panel represents one debate
- ballots attach entries to judges/panel with side and sometimes speaker order

## Speech

- one panel represents one speech section
- ballots attach entries into a section and encode speaking order

## Congress

- one panel represents one chamber/session
- ballots attach entries and often multiple judges, with speaking positions and chair distinctions

## WUDC / Other special formats

- one panel represents a four-team room
- ballots encode special positional semantics

## Coupling To Downstream Work

Pairing output is not isolated. It directly feeds:

- judge assignment
- room assignment
- printed schematics
- ballot generation
- disaster checks
- result computation
- break generation

Implication:

- errors in round generation cascade operationally

## Disaster and Recovery Relevance

Primary source:

- `web/panel/schemat/disaster_check.mhtml`

Observed problem classes:

- double-booked judges within a round
- judges moving between rooms across flights
- double-booked rooms
- cross-timeslot double scheduling

Implication:

- “successful pairing” means more than producing a bracket; it means producing an operationally viable round

## Highest-Risk Rebuild Requirements

The most load-bearing pairing requirements to preserve are:

- debate side fairness and repeat avoidance
- debate pullup and bracket logic
- speech section balancing and order fairness
- congress chambering and special constraint logic
- special-format positional fairness
- resource-aware autobye and assignment interactions
- disaster-detection compatibility with generated rounds

## Recommended Next Extraction Areas

The next deeper source-analysis passes should focus on:

- `make_pairing_hash.mas`
- judge assignment/review surfaces in `web/panel/schemat/*judge*`
- room assignment/review surfaces in `web/panel/schemat/*room*`
- congress-specific result and advancement flows
- WSDC-specific special cases

## Open Questions For Later Passes

- which penalty weights are historically tuned versus logically essential?
- which pairing behaviors are relied on most by experienced tab staff?
- where do operator overrides most commonly occur after automatic pairing?
- how often do tournaments use specialized formats versus standard debate/speech paths?
