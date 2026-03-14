# Results Generation, Regeneration, and Mutability Model

## Purpose

This document captures the first-pass model of how result artifacts are generated, replaced, and maintained in the current Tabroom platform.

Its purpose is to make explicit that many important result artifacts are generated operational outputs with labels, lifecycle, and replacement behavior.

This is a descriptive inventory of current behavior, not a future implementation recommendation.

## Source Basis

Primary evidence used:

- `web/tabbing/publish/generate_results.mhtml`
- `web/tabbing/publish/generate_sweeps.mhtml`
- `web/tabbing/publish/index.mhtml`
- `web/tabbing/publish/result_set_switch.mhtml`
- `web/tabbing/publish/rset_switch.mhtml`
- `web/tabbing/results/order_entries.mas`
- `web/tabbing/results/order_speakers.mas`
- prior codex results/settings/publication docs

## First-Pass Conclusions

High-confidence observations:

- important result artifacts are generated rather than hand-authored
- generation commonly replaces prior artifacts with the same event/label identity
- generation and publication are independent choices
- generated artifacts often persist supporting metadata such as keys, ballot strings, cached JSON, and percentile

## Generation Families Visible In Source

Observed first-pass result-generation families:

- prelim seeds
- final places
- speaker awards
- prelims table
- brackets
- chamber results
- NDCA outputs
- TOC bids
- sweeps outputs

Implication:

- “publish results” is really a family of generation workflows, not one monolithic job

## Label-Centered Artifact Identity

Observed behavior:

- generation code derives human-readable labels such as `Prelim Seeds`, `Final Places`, `Speaker Awards`, `Prelims Table`, `Bracket`, and chamber-result labels
- before regeneration, prior `result_value`, `result`, and `result_set` rows for the same event/label are deleted

Implication:

- at least for many result families, artifact identity is effectively `(event, label)` rather than immutable version lineage

## Regeneration Behavior

Observed sequence in source:

1. select target event(s) and optional subtype filters
2. resolve breakout / label context where relevant
3. delete prior result-value rows for the event/label
4. delete prior result rows for the event/label
5. delete prior result-set row for the event/label
6. create a fresh result set
7. create result keys and result values
8. optionally mark the new set published

Implication:

- regeneration is replacement, not incremental patching

## Data Added During Generation

Observed generated structures:

- `result_set` row with label, generated timestamp, event/tournament linkage, publish state
- `result` rows with entry or student, rank, place, and sometimes round/panel linkage
- `result_key` rows defining sortable/result-display columns
- `result_value` rows carrying tiebreak values, round labels, ballots, and protocol-linked values
- cached compressed JSON for the prelims-table variant
- percentile updates written after ranking

Implication:

- generated result artifacts are richer than a simple standings snapshot

## Mutability Dimensions

The source suggests at least four separate mutability dimensions:

## 1. Regeneration

- same logical artifact can be recalculated and replaced

## 2. Publication State

- published/unpublished can be toggled after generation

## 3. Audience State

- coach-only visibility can differ from public visibility

## 4. Scope Filters

- breakout filters
- event-family filters
- limits on how many results to generate
- specialized qualifiers / governing-body variants

## Generation Inputs That Shape Artifact Contents

Observed first-pass inputs:

- last prelim or final round selection
- event format
- protocol and tiebreak configuration
- breakout filters and breakout labels
- event settings such as speaker protocol
- publish flag at generation time
- optional limit parameters
- specialized event settings such as NDCA/TOC-related options

Implication:

- result generation depends heavily on tournament configuration and round-state context, not just raw scores

## Examples Of Generated Payload Shape

### Prelim seed / final places style sets

Observed contents:

- ranked entries
- tiebreak values
- ballot strings
- percentile
- round references in some final/elim cases

### Speaker awards

Observed contents:

- ranked students
- student-to-entry linkage
- speaker-protocol tiebreak values
- ballot-like display strings

### Prelims table

Observed contents:

- compressed cached JSON
- round metadata
- panel and judge structure
- entry metadata
- per-round results data
- tiebreak key descriptions

### Chamber results

Observed contents:

- round/chamber grouping
- section-rank-style results
- per-entry result rows tied to chamber context

## Operational Meaning

The source corpus suggests the operator model is:

- generate a results artifact when a tournament stage is ready
- inspect it
- decide whether to publish it publicly or coach-only
- regenerate it if inputs or corrections change

Implication:

- generated result artifacts are operational products used during live administration, not just static end-of-event archives

## Important Distinctions To Preserve Later

- generated result set vs raw standings computation
- replaceable labeled artifact vs immutable audit history
- generation-time publish choice vs later publish toggle
- entry-based result artifact vs student-based result artifact
- cached document-style artifact vs relationally rendered artifact

## Open Questions For Later Validation

- which generated result families are commonly regenerated during live tournaments versus only once near completion
- whether operators treat regeneration as routine or exceptional for specific result families
- whether any result families need explicit version-history requirements in a future implementation
