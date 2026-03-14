# Results Concept Normalization Register

## Purpose

This document captures a first-pass normalization of results-domain concepts visible in the current Tabroom platform.

Its purpose is to reduce drift later when these concepts are discussed or re-modeled elsewhere.

This is not a target data model. It is a concept dictionary for the results domain.

## Source Basis

Primary inputs:

- `FIRST_PASS_RESULTS_AND_ADVANCEMENT_DEEP_DIVE.md`
- `RESULT_ARTIFACT_AND_RESULT_SET_MATRIX.md`
- `PUBLICATION_AND_ARTIFACT_INVENTORY.md`
- result-related route and schema surfaces

## Core Results Concepts

## 1. Raw Score

Current meaning:

- one scored value attached to a ballot, sometimes also to a student/speech/position

Primary source structures:

- `score`

Examples:

- `winloss`
- `point`
- `rank`
- `speech`
- `refute`

Normalization note:

- a raw score is not the same thing as a ranking result

## 2. Ballot Outcome

Current meaning:

- the per-ballot interpreted competitive outcome for an entry in a panel

Primary source structures:

- `ballot`
- related `score` rows

Examples:

- win/loss
- bye
- forfeit
- chair/non-chair distinctions

Normalization note:

- ballot outcome is still below the standings layer

## 3. Round Result

Current meaning:

- what is shown for one round after publication thresholds are met

Primary sources:

- `results_debate.mas`
- `results_speech.mas`
- `results_congress.mas`

Normalization note:

- round result is a presentation artifact for one round, not the same as full event standings

## 4. Standings / Ordered Entries

Current meaning:

- ranked ordering of entries within an event at a given point, computed by protocol and tiebreak rules

Primary sources:

- `order_entries.mas`

Normalization note:

- standings are computed, not manually stored as a single primary table

## 5. Speaker Order / Speaker Awards Ranking

Current meaning:

- per-student ranking derived from a dedicated speaker protocol and underlying score data

Primary sources:

- `order_speakers.mas`

Normalization note:

- this is a sibling result system to entry standings, not a subset of the same artifact

## 6. Result Set

Current meaning:

- a persisted generated artifact representing some ranked output, bracket, qualifier list, or similar publishable result grouping

Primary source structures:

- `result_set`
- `result`
- `result_key`
- `result_value`

Normalization note:

- result set is a container artifact, not equivalent to one round or one standings table

## 7. Bracket

Current meaning:

- a result-set-backed advancement structure used for elimination progression

Primary evidence:

- `result_set.bracket`
- break generation flows

Normalization note:

- bracket is both a result artifact and an operational advancement artifact

## 8. Final Places / Qualifiers

Current meaning:

- a specialized ranked output for placement, advancement, or governing-body qualification

Primary evidence:

- `Final Places` labels
- qualifier flows and result generation

Normalization note:

- this is not necessarily identical to generic event standings

## 9. Publication State

Current meaning:

- the visibility state controlling when and to whom an artifact or detail level is visible

Primary evidence:

- `round.published`
- `post_primary`
- `post_secondary`
- `post_feedback`
- `result_set.published`
- `result_set.coach`

Normalization note:

- publication state belongs to artifacts and detail levels, not just to events as a whole

## 10. Break

Current meaning:

- the selection of advancing entries from a ranked field into a subsequent elimination or breakout structure

Primary evidence:

- `tabbing/break/*`

Normalization note:

- break is a process and an outcome, not just a label on results

## 11. Sweepstakes

Current meaning:

- award-oriented aggregation across events/tournaments based on configurable sweep rules

Primary evidence:

- `sweep_*`
- sweep result/report flows

Normalization note:

- sweepstakes is not ordinary standings; it is a separate award aggregation engine

## 12. Qualification Pipeline

Current meaning:

- the process and artifacts used to determine qualifiers, alternates, or governing-body advancement status

Primary evidence:

- `qualifier`
- district/national result flows

Normalization note:

- qualification is downstream of standings but not reducible to them

## Concept Distinctions That Matter

These distinctions appear especially important:

- raw score vs round result
- round result vs standings
- standings vs result set
- result set vs export
- standings vs speaker awards
- bracket vs final places
- publication state vs computed result content

## Terms That Should Be Used Carefully Later

These terms are easy to blur if not handled carefully:

- “results”
- “standings”
- “bracket”
- “published”
- “qualifier”
- “awards”
- “speaker”

Recommended discipline:

- use the more specific term whenever possible

## Suggested Next Use Of This Register

Use this register later to:

- keep implementation briefs precise,
- avoid mixing artifact layers in architecture discussions,
- align ScholarComp-side results language to the source behavior without copying legacy implementation structure.
