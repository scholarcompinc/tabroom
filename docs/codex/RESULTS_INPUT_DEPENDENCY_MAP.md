# Results Input Dependency Map

## Purpose

This document captures the first-pass map of which source inputs and configuration areas each major results artifact family depends on in the current Tabroom platform.

Its purpose is to make explicit that results are not derived from scores alone.

This is a descriptive dependency map, not a future service contract.

## Source Basis

Primary evidence used:

- `web/tabbing/publish/generate_results.mhtml`
- `web/tabbing/results/order_entries.mas`
- `web/tabbing/results/order_speakers.mas`
- `web/tabbing/break/*`
- `web/tabbing/results/sweep_tourn.mas`
- prior codex results/settings/state docs

## First-Pass Conclusions

High-confidence observations:

- most result artifacts depend on round state plus protocol configuration
- many outputs also depend on event settings, breakout configuration, and exclusion rules
- some artifacts are downstream of prior result artifacts rather than directly downstream of scores

## Dependency Families

## 1. Raw Competition Inputs

Observed examples:

- ballots
- scores
- byes
- forfeits
- judge associations
- panel associations
- round identity and type

These are the lowest-level competitive inputs.

## 2. Round-State Inputs

Observed examples:

- published round state
- `post_primary`
- `post_secondary`
- `post_feedback`
- panel publication
- round labels
- round type (`prelim`, `elim`, `final`, `runoff`)
- `ignore_results`
- `use_for_breakout`

These determine both eligibility and visibility of result calculations.

## 3. Protocol / Ranking Inputs

Observed examples:

- result protocol
- speaker protocol
- tiebreak definitions
- tiebreak direction
- runoff handling
- special conditions such as `forfeits_never_break`

These determine how standings and awards are ordered.

## 4. Event-Configuration Inputs

Observed examples:

- event type
- aff/neg labels
- point increments
- judge result publication settings
- speaker award configuration
- top novice / honorable mention flags
- breakout labels and breakout student/entry filters

These shape format behavior and artifact variants.

## 5. Downstream Aggregation Inputs

Observed examples:

- breaks into elimination rounds
- final/elim progression
- qualifier configuration
- sweep rules
- affiliation/region context

These support final places, qualifiers, brackets, and award aggregation.

## Artifact Dependency Matrix

| Artifact Family | Raw Scores/Ballots | Round State | Protocol/Tiebreaks | Event Settings | Downstream Aggregations | Notes |
|---|---|---|---|---|---|---|
| Round results | Yes | Yes | Limited to display logic | Yes | No | Strongly format-sensitive |
| Entry standings / prelim seeds | Yes | Yes | Yes | Yes | Sometimes | `order_entries.mas` is central |
| Speaker awards | Yes | Yes | Yes, separate speaker protocol | Yes | Sometimes | Student-level ranking path |
| Prelims table | Yes | Yes | Yes | Yes | No | Uses cached document-style payload |
| Brackets | Indirectly yes | Yes | Yes | Yes | Yes | Depends on break and elimination structure |
| Final places | Yes | Yes | Yes | Yes | Yes | Combines prelim seeds with elim/final progression |
| Chamber results | Yes | Yes | Yes | Yes | Sometimes | Chamber/tie grouping specific |
| Sweepstakes | Indirectly via results and rounds | Yes | Rules-driven | Yes | Yes | Uses sweep rules and exclusion logic |
| Qualification outputs | Indirectly via final places / results | Yes | Yes | Yes | Yes | Often downstream of other result artifacts |

## Result Family Notes

## Round Results

Key dependencies:

- enough scoring exists to render the format
- publication thresholds permit display
- format-specific settings define what labels and detail to show

## Entry Standings

Key dependencies:

- a scoring round to order from
- valid tiebreak configuration
- included vs excluded rounds
- breakout filtering if used

## Speaker Awards

Key dependencies:

- speaker protocol exists
- student-linked score rows exist
- speaker minimum / novice settings may apply
- breakout student filters may apply

## Final Places

Key dependencies:

- prelim ordering
- elimination or final rounds when present
- place-label semantics
- special-case handling for ties and co-champions

## Sweepstakes

Key dependencies:

- prior result outputs or standings data
- sweep rule set
- exclusions such as breakouts or excluded entries
- bye/forfeit interpretation in some rule paths

## Qualification Outputs

Key dependencies:

- final-place style results
- governing-body or qualifier settings
- school/region/affiliation context

## Hidden Dependency Themes

The source corpus suggests several hidden dependency themes:

- settings often act like rule inputs rather than passive display config
- breakout logic creates alternative filtered result universes
- elimination and final rounds can override the meaning of ordinary standings
- generated artifacts may depend on earlier generated artifacts or their conventions

## Important Distinctions To Preserve Later

- raw result input vs ranking rule input
- display gating vs calculation eligibility
- event configuration vs tournament-wide rules
- direct score-derived artifact vs downstream aggregation artifact

## Open Questions For Later Validation

- which artifact families operators consider authoritative when multiple derived views coexist
- which settings are operationally required versus historical configuration residue
- where downstream artifacts use generated result sets versus recomputing from raw scores
