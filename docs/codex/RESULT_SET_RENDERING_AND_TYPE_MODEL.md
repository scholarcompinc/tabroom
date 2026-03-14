# Result-Set Rendering and Type Model

## Purpose

This document captures the first-pass model of how the current platform renders result sets generically across different subject types.

Its purpose is to make explicit that one `result_set` mechanism is used for multiple kinds of ranked objects, not just event entries.

This is a descriptive source-analysis artifact, not a future data-model recommendation.

## Source Basis

Primary evidence used:

- `web/funclib/results_table.mas`
- `web/index/tourn/results/event_results.mhtml`
- `web/index/tourn/results/prelims_table.mhtml`
- `web/tabbing/publish/generate_results.mhtml`
- `web/tabbing/publish/generate_sweeps.mhtml`
- prior codex result-set docs

## First-Pass Conclusions

High-confidence observations:

- `result_set` is a generic ranked artifact container
- result rows can represent entries, students, schools, and other scoped entities
- the generic results renderer infers subject type from populated fields
- result sets may also carry metadata about protocols, sweep sets, sections, circuits, and ballot payloads

## Subject Types Visible In Source

The generic renderer appears to support at least these result subject types:

## 1. Entries

Observed in:

- prelim seeds
- final places
- brackets
- many ordinary event result sets

Typical metadata:

- entry code
- entry name
- school
- section/chamber in some contexts

## 2. Students

Observed in:

- speaker awards
- student-scoped sweeps outputs

Typical metadata:

- student first/last name
- linked entry and school
- award-specific ranking/tiebreak values

## 3. Schools

Observed in:

- school-scoped sweeps outputs
- possibly tournament-wide aggregate award outputs

Typical metadata:

- school code/name
- state/region
- aggregate point values and counts

## Subject-Type Inference

Observed inference logic in `results_table.mas`:

- if school and entry exist, treat as entries
- else if school exists, treat as schools
- else if student exists, treat as students
- else if entry exists, treat as entries

Implication:

- subject type is not always explicitly encoded as a dedicated result-set kind
- it can be inferred from the shape of the stored result rows

## Metadata Dimensions Carried By Result Sets

Observed metadata:

- label
- generated timestamp
- tournament
- event
- circuit
- protocol reference
- sweep-set reference
- section/chamber mapping
- code style

Implication:

- result sets are not just collections of ranks; they are typed presentation objects with context

## Detail Payloads Carried By Result Values

Observed detail payload examples:

- tiebreak values
- ballot strings
- JSON ballot payloads for section/chamber style displays
- point totals
- counted-entry totals
- placement descriptors

Implication:

- a result set can function as both a leaderboard and a rich report container

## Renderer Behaviors Visible In Source

Observed behaviors:

- generic table rendering for published result sets
- special-case handling for `Ballots` payloads
- sweep-set metadata shown differently from protocol-backed event standings
- section/chamber grouping support
- school/state/region contextual display

Implication:

- generic rendering exists, but it is still sensitive to result-set subtype and payload conventions

## Result-Set Families That Use The Generic Renderer

High-confidence examples:

- ordinary standings-like result sets
- speaker awards
- sweeps outputs
- some tournament-wide aggregate outputs

Less direct or separate render paths:

- prelims table
- bracket views
- round-result views

## Important Distinctions To Preserve Later

- generic result-set rendering is not the same thing as round-result rendering
- one result-set mechanism supports multiple ranked subject types
- sweep-set-backed outputs differ semantically from protocol-backed standings even if rendered similarly
- some result artifacts are document-like payloads rather than simple rank/value tables

## Open Questions For Later Validation

- whether any current result-set families rely on undocumented renderer conventions not obvious in source
- whether schools/students/entries should be treated as explicit result subject types in later requirements normalization
