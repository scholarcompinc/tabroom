# Result Artifact and Result-Set Matrix

## Purpose

This document captures the first-pass matrix of result-related artifacts visible in the current Tabroom platform.

Its purpose is to distinguish:

- round-result artifacts,
- standings/result-set artifacts,
- bracket artifacts,
- award/qualification artifacts,
- export/report artifacts tied to results.

This is a descriptive inventory, not a target results architecture.

## Source Basis

Primary evidence used:

- `tabbing/results/*`
- `tabbing/publish/*`
- `index/tourn/results/*`
- `funclib/results_*`
- `funclib/tourn_result_sets.mas`
- `FIRST_PASS_RESULTS_AND_ADVANCEMENT_DEEP_DIVE.md`
- `FIRST_PASS_REPORTING_PRINT_EXPORT_DEEP_DIVE.md`
- `PUBLICATION_AND_ARTIFACT_INVENTORY.md`

## Core Distinction

The source corpus strongly suggests three different layers:

1. one-round display artifacts
2. persisted generated result sets
3. downstream reports/exports derived from either raw results or result sets

This distinction should survive later mapping work.

## Matrix

| Artifact Family | Primary Sources | Backing Model / State | Audience | Notes |
|---|---|---|---|---|
| Round pairings/postings | `index/tourn/postings/*`, `panel/schemat/*` | `round.published`, `panel.publish` | Public / coach / judge / operator | Not a result set; visibility precursor to live rounds |
| Debate round results | `funclib/results_debate.mas`, `index/tourn/results/round_results.mhtml` | `round.post_primary`, `round.post_secondary`, `panel.publish`, scores/ballots | Public / coach / judge / operator | Panel- and side-aware |
| Speech round results | `funclib/results_speech.mas`, `index/tourn/results/round_results.mhtml` | Same round publication controls plus score rows | Public / coach / judge / operator | Section-centric |
| Congress round results | `funclib/results_congress.mas`, `index/tourn/results/round_results.mhtml` | Round publication + `post_feedback` + score rows | Public / coach / judge / operator | Chamber-centric, chair/non-chair aware |
| WUDC / special-format round results | `funclib/results_wudc.mas` | Round publication + format-specific scores | Public / coach / judge / operator | Special positional semantics |
| Entry standings | `tabbing/results/order_entries.mas`, `tabbing/results/results_table.mas` | Protocol + tiebreaks + ballots/scores | Operator first, later public/coach depending on publication | Core standings engine |
| Speaker standings / speaker awards | `tabbing/results/order_speakers.mas`, `tabbing/results/speakers.mhtml` | Speaker protocol + score data | Operator / public / awards | Parallel ranking system |
| Generated result sets | `result_set`, `result`, `result_key`, `result_value`, `tabbing/publish/*` | Persisted generated artifacts with `published` and `coach` flags | Public / coach / operator | Central persisted results layer |
| Bracket result sets | `tabbing/publish/generate_bracket.mas`, break flows | `result_set.bracket = 1` plus related results | Public / coach / operator | Advancement-specific persisted artifact |
| Final places / qualifier result sets | `nsda_qualifiers.mhtml`, `tabbing/publish/*` | Generated result sets plus qualifiers/affiliation logic | Operator / specialized stakeholders / public in some modes | Specialized but important |
| Sweepstakes result artifacts | `sweep_*`, `nsda_sweepstakes.mhtml`, report flows | Sweep models + generated outputs | Operator / awards / public in some modes | Separate award engine |
| Result CSV exports | `tabbing/results/csv.mhtml`, `results_csv.mas` | Result-order output or generated set output | Operator / integration | Structured export, not just display |
| Speaker CSV exports | `speakers_csv.mhtml`, `event_speakers_csv.mhtml` | Speaker ranking data | Operator / awards / integration | Parallel export lane |
| Result printouts | `tabbing/report/results_table_print.mas`, school result printouts | Result ordering or result sets | Operator / ceremony / public | Presentation layer on top of result logic |
| Qualification / award packets | `tabbing/report/nsda/*`, `user/admin/nsda/*` | Result ordering + affiliation logic + result sets | Specialized admin / governing body | Ecosystem outputs |

## Result-Set Concepts Visible In Source

Observed first-pass `result_set` concepts:

- ordinary standings-like sets
- bracket sets
- final places sets
- coach-visible-only sets
- published public sets
- sweep/award-related sets
- qualifier-related sets

Implication:

- “result set” is a general container for multiple result artifact types, not one narrowly defined leaderboard

## Result Artifact Groups By Product Purpose

## 1. Live Competition Transparency

Artifacts:

- round results
- pairings/postings
- entry records / live updates

Purpose:

- inform participants about current round state and immediate outcomes

## 2. Tournament Standings And Advancement

Artifacts:

- entry standings
- speaker standings
- bracket sets
- breaks / final places

Purpose:

- determine who advances and how the tournament resolves competitively

## 3. Award, Recognition, And Ceremony

Artifacts:

- speaker awards
- sweeps
- awards scripts/printouts

Purpose:

- support end-of-tournament recognition and reporting

## 4. External Or Specialized Reporting

Artifacts:

- result CSV
- affiliation-specific outputs
- qualification packets

Purpose:

- provide ecosystem interoperability and downstream consumption

## Result-Set Lifecycle Signals

Observed state/control points:

- generated timestamp
- published flag
- coach flag
- bracket flag
- label
- event/tournament linkage

Observed operational behaviors:

- result sets can be regenerated
- result sets can be published or unpublished
- result sets can be coach-only or public
- result sets can be deleted and recreated

Implication:

- result sets are managed operational artifacts with lifecycle, not immutable history snapshots

## Important Distinctions To Preserve Later

The source corpus suggests these distinctions must remain explicit:

- round result vs overall standings
- standings vs speaker awards
- ordinary standings vs bracket results
- public result vs coach-only result
- generated artifact vs export representation

## Likely Release 1 Critical Result Artifacts

Based on current evidence, the most critical result artifacts appear to be:

- round results
- entry standings
- break/bracket outputs
- speaker awards if required by target market
- basic CSV/export equivalents for standings

## Likely Later Or Conditional Result Artifacts

Likely conditional or later-scope artifacts:

- long-tail specialized governing-body result packets
- full sweeps variants
- niche statistics and historical ranking outputs

## Suggested Next Deeper Pass

If more source analysis is done here, the next version should enumerate for each artifact:

- label used in UI
- generation trigger
- persisted or ephemeral
- publication control points
- common audience
- Release 1 criticality
