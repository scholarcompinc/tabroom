# Results Stage and Timing Model

## Purpose

This document captures the first-pass model of when different result artifacts become available or meaningful during the lifecycle of a tournament in the current Tabroom platform.

Its purpose is to tie result artifacts to tournament timing rather than treating them as one end-of-event output.

This is a descriptive model of current behavior, not a target release sequence.

## Source Basis

Primary evidence used:

- `web/index/tourn/results/menu.mas`
- `web/index/tourn/results/round_results.mhtml`
- `web/tabbing/publish/index.mhtml`
- `web/tabbing/publish/generate_results.mhtml`
- `web/tabbing/break/*`
- prior codex workflow/results/publication docs

## First-Pass Conclusions

High-confidence observations:

- results are produced throughout tournament operation, not only at the end
- some artifacts are live transparency outputs
- some artifacts are mid-tournament operational tools
- some artifacts only become meaningful after prelim completion or elim completion

## Tournament Timing Stages

## 1. During an Active Round

Result-adjacent outputs visible here:

- round publication state
- partial round-result visibility depending on posting thresholds
- panel-level publication in some judge-publication modes

Primary purpose:

- show immediate competitive outcomes and score detail as posting rules allow

## 2. After One or More Preliminary Rounds

Artifacts that become meaningful:

- standings as of a selected round
- speaker awards as of a selected round
- CSV and printable ordered lists
- prelims-table style artifacts

Primary purpose:

- give operators and stakeholders an ordered competitive picture before elimination decisions

## 3. At Break / Advancement Time

Artifacts that become meaningful:

- break tables and advancement candidate ordering
- brackets
- breakout-limited result variants
- qualifier candidate lists in some contexts

Primary purpose:

- convert standings into advancement structure

## 4. During Elimination / Final Progression

Artifacts that become meaningful:

- round-specific elimination results
- updated brackets
- increasingly authoritative final-place calculations

Primary purpose:

- communicate live elimination progress and eventual placement

## 5. Tournament Completion / Awards

Artifacts that become meaningful:

- final places
- speaker awards
- sweeps
- qualification outputs
- downloadable published packets and reports

Primary purpose:

- support awards, publication, archival, and ecosystem reporting

## Artifact Timing Matrix

| Artifact Family | Earliest Meaningful Stage | Can Change Later? | Notes |
|---|---|---|---|
| Round results | during active/just-completed round | Yes | Detail can expand as posting thresholds rise |
| Entry standings | after at least one relevant round | Yes | Recomputed as more rounds complete or corrections occur |
| Speaker awards | after relevant speaker-score rounds | Yes | Often treated as “as of round X” |
| Prelims table | after prelim data exists | Yes | Mid-tournament and pre-break oriented |
| Break tables | at advancement point | Yes | Sensitive to corrections and unresolved tie cases |
| Brackets | once break occurs and elim structure exists | Yes | Evolves through elimination progression |
| Final places | after final or decisive elimination state | Yes until finalized | Can depend on later elim/final outcomes |
| Sweepstakes | usually late tournament / completion | Yes | Aggregates across more complete result sets |
| Qualification outputs | late tournament / qualification checkpoints | Yes | Depends on authoritative placement outputs |

## Timing Behaviors Visible In Source

Observed patterns:

- public menu may default to recent published round results if available
- operator results views let users choose “as of” a round
- final-place generation explicitly reuses prelim seeds and later rounds
- publication and regeneration can happen after corrections

Implication:

- result timing is iterative and stage-aware

## Mid-Tournament Versus End-State Distinction

Mid-tournament artifacts:

- round results
- standings as of a round
- speaker orderings
- prelims tables
- break preparation views

End-state or near-end artifacts:

- final places
- final brackets
- sweeps
- qualifier packets

Implication:

- later requirements work should distinguish operational results from ceremonial/final results

## Why Timing Matters

The current product uses result artifacts for different jobs at different moments:

- transparency during the tournament
- operator decision support before breaks
- official advancement outputs
- final public outcomes and awards

That means correctness requirements vary by stage:

- speed and publication control matter more mid-tournament
- finality and downstream reporting matter more at completion

## Important Distinctions To Preserve Later

- live round-result publication vs stable final result publication
- “as of round” ordering vs final placement
- advancement artifact vs award artifact
- operational timing dependency vs pure data dependency

## Open Questions For Later Validation

- which mid-tournament artifacts are considered indispensable by operators in real tournaments
- how often result artifacts are regenerated after formal publication at each stage
- whether some late-stage artifacts are generated once or repeatedly during awards preparation
