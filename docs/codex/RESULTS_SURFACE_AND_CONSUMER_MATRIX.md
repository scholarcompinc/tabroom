# Results Surface and Consumer Matrix

## Purpose

This document captures the first-pass matrix of where result artifacts are surfaced in the current Tabroom platform and which audiences appear to consume them.

Its purpose is to distinguish:

- operator calculation surfaces,
- operator review/export surfaces,
- public tournament result views,
- downloadable/static result artifacts,
- specialized downstream consumers.

This is a descriptive inventory, not a future UI partition.

## Source Basis

Primary evidence used:

- `web/index/tourn/results/menu.mas`
- `web/index/tourn/results/index.mhtml`
- `web/index/tourn/results/event_results.mhtml`
- `web/index/tourn/results/round_results.mhtml`
- `web/index/tourn/results/prelims_table.mhtml`
- `web/index/tourn/results/bracket.mhtml`
- `web/index/tourn/results/debate_cumesheet.mhtml`
- `web/tabbing/results/index.mhtml`
- `web/tabbing/results/speakers.mhtml`
- `web/tabbing/results/see_order.mhtml`
- `web/tabbing/results/csv.mhtml`
- prior codex reporting/results/publication docs

## First-Pass Conclusions

High-confidence observations:

- there is not one results screen
- operator, coach, public, and downloadable consumers use different surfaces
- some result artifacts are rendered dynamically while others are exposed as published files
- the platform mixes live round transparency, standings, awards, and reference records in one broader results area

## Surface Families

## 1. Operator Calculation and Review Surfaces

Observed examples:

- `tabbing/results/index.mhtml`
- `tabbing/results/speakers.mhtml`
- `tabbing/results/see_order.mhtml`
- `tabbing/results/nsda_qualifiers.mhtml`

Observed user intent:

- inspect standings as of a selected round
- inspect speaker awards
- generate exports and print views
- review qualifier outputs and governing-body-specific ordering

Observed audience:

- tab room operators
- tournament directors
- specialized governing-body admins in some flows

## 2. Public Tournament Results Navigation

Observed examples:

- `index/tourn/results/menu.mas`
- `index/tourn/results/index.mhtml`
- `index/tourn/results/event_results.mhtml`
- `index/tourn/results/round_results.mhtml`
- `index/tourn/results/bracket.mhtml`
- `index/tourn/results/prelims_table.mhtml`

Observed user intent:

- browse published result sets by event
- browse published round results
- view brackets and prelim tables
- download published result files

Observed audience:

- public viewers
- coaches
- competitors
- judges checking public outcomes

## 3. Published File Surfaces

Observed examples:

- event-scoped `result` files in S3-backed links
- tournament-wide `result` files in the same public menu

Observed user intent:

- download final packets, PDFs, or prepared result artifacts

Observed audience:

- public viewers
- coaches
- tournament staff

## 4. Specialized Historical / Reference Result Views

Observed examples:

- `index/tourn/results/debate_cumesheet.mhtml`
- linked speaker-detail and team-record pages

Observed user intent:

- inspect cumulative debate records and speaker placement detail
- traverse into deeper statistical or historical reference views

Observed audience:

- coaches
- interested public users
- circuit/community users doing research

## Surface Matrix

| Surface | Primary Artifact Type | Audience | Notes |
|---|---|---|---|
| `tabbing/results/index.mhtml` | operator standings, prelim ordering, qualifier tools | Operator | Primary internal results workbench |
| `tabbing/results/speakers.mhtml` | speaker awards standings | Operator | Dedicated speaker-result lane |
| `tabbing/results/see_order.mhtml` | sortable order / NSDA-oriented ordering | Operator / specialized admin | Bridges standings and qualification reporting |
| `tabbing/results/csv.mhtml` | CSV export | Operator / integration | Structured machine-readable output |
| `index/tourn/results/round_results.mhtml` | one-round result display | Public / coach / competitor | Uses format-specific result renderer |
| `index/tourn/results/event_results.mhtml` | published result-set table | Public / coach | Generic published standings-like result-set view |
| `index/tourn/results/prelims_table.mhtml` | published prelims table | Coach-oriented result artifact, publicly gated by publication | Special document-style artifact |
| `index/tourn/results/bracket.mhtml` | published bracket set | Public / coach | Distinct rendering path keyed by bracket flag |
| `index/tourn/results/menu.mas` | event/tournament result navigation | Public / coach | Aggregates result sets, round results, and downloadable files |
| published `result` files | downloadable packets/reports | Public / coach / staff | Static artifact channel separate from rendered pages |
| `debate_cumesheet.mhtml` and linked records | cumulative/statistical views | Public / coach / researcher | More reference-oriented than live-ops critical |

## Consumer Types Visible In Source

## 1. Tournament Participants

Likely needs:

- round results
- published standings
- bracket visibility
- downloadable final postings

## 2. Coaches

Likely needs:

- everything in ordinary public result views
- prelims-table style detailed artifact
- coach-only visibility where enabled

## 3. Operators

Likely needs:

- calculate and inspect standings before publication
- compare by round
- export to CSV/PDF
- manage qualifiers and awards

## 4. External / Ecosystem Consumers

Likely needs:

- downloadable files
- CSVs
- specialized NSDA/TOC/qualification outputs

## Dynamic Rendered Versus Static Artifact Split

Observed split:

- some results are rendered from live result-set data
- some are rendered from round/score tables directly
- some are downloadable published files
- some specialized artifacts embed cached JSON or precomputed payloads

Implication:

- later requirements work should not assume one delivery mechanism for all result artifacts

## Navigation Behavior Worth Preserving As Fact

Observed behavior:

- public results navigation groups by event and tournament-wide artifacts
- bracket result sets route to a distinct bracket page
- if a recent published round exists, the menu may redirect users to round results by default
- published result files coexist with rendered result views in the same navigation area

Implication:

- the existing platform treats “results” as a mixed navigation domain, not just a standings page

## Likely Release-1-Critical Result Surfaces

Based on current evidence:

- operator standings/review surface
- public round results
- public published standings/result-set view
- bracket display
- basic downloadable final result artifacts

## Likely Later Or Conditional Result Surfaces

- deep cume-sheet / historical reference views
- niche governing-body result views
- long-tail statistical drilldowns

## Open Questions For Later Validation

- how heavily coaches actually use the prelims-table artifact relative to ordinary result sets
- which downloadable published files are considered non-negotiable at live tournaments
- whether some historical/reference views are more mission-critical than the source evidence suggests
