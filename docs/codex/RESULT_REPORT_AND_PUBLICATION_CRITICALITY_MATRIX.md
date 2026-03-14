# Result Report and Publication Criticality Matrix

## Purpose

This document captures the first-pass criticality ranking of result-related reports, views, and published artifacts visible in the current Tabroom platform.

Its purpose is to distinguish:

- live-operationally critical outputs,
- tournament-closing critical outputs,
- specialized or ecosystem outputs,
- lower-frequency reference/reporting outputs.

This is a descriptive triage artifact, not a final release decision.

## Source Basis

Primary evidence used:

- `web/index/tourn/results/menu.mas`
- `web/index/tourn/results/*`
- `web/tabbing/results/*`
- `web/tabbing/publish/*`
- `web/tabbing/report/*`
- prior codex results/report/publication docs

## Criticality Scale

For this document:

- `Tier A`: operationally critical in ordinary tournament execution
- `Tier B`: high-value, common, but not always blocking live operations
- `Tier C`: specialized, context-dependent, or less frequently used
- `Tier D`: niche, historical, or long-tail support output

## Matrix

| Artifact / Surface | Criticality | Why |
|---|---|---|
| Published round results | Tier A | Core transparency output during live rounds |
| Published standings / generic result sets | Tier A | Core post-round and post-event visibility path |
| Final Places | Tier A | Official placement artifact and upstream dependency for later flows |
| Bracket display / bracket result sets | Tier A | Required once elimination rounds exist |
| Operator standings review (`tabbing/results/index`) | Tier A | Core internal operating surface |
| Ballot/round-result publication controls | Tier A | Directly controls live result visibility |
| Speaker awards | Tier B | Common awards artifact, especially in speech/debate contexts |
| Prelims Table | Tier B | Detailed coach/operator artifact; not universal but clearly important |
| CSV result exports | Tier B | Common export/integration need |
| Printable standings / result printouts | Tier B | Often needed for ceremony/posting workflows |
| Sweepstakes outputs | Tier B/C | Important for many tournaments, but not universal at same criticality as final places |
| District qualifiers | Tier B/C | Mission-critical for district workflows, irrelevant for many ordinary tournaments |
| TOC bid outputs | Tier C | Important in specific circuits/events only |
| NSDA points posting | Tier C | External compliance/reporting workflow, not every tournament |
| Debate cume sheet | Tier C | Valuable reference output, but narrower than core standings/finals |
| Tournament-wide result files | Tier C | Useful distribution mechanism, but artifact-specific |
| Deep historical/statistical drilldowns | Tier D | Valuable reference layer, not core live operation |

## Tier A Notes

Tier A outputs appear load-bearing because they directly support one or more of:

- live participant visibility
- operator decision-making
- official completion of an event
- progression into elimination rounds or awards

From source evidence, the clearest Tier A items are:

- round results
- standings/result sets
- final places
- brackets
- operator review/generation controls

## Tier B Notes

Tier B outputs appear common and important, but more context-sensitive:

- speaker awards
- prelims table
- result exports
- printed result artifacts

These are strong candidates for broad support, but not every tournament depends on all of them equally.

## Tier C Notes

Tier C outputs tend to be:

- ecosystem-specific
- organization-specific
- format- or market-segment-specific

Examples:

- district qualifiers
- TOC bids
- NSDA posting flows
- cume-sheet style reference views

## How To Read This Matrix

This is not saying Tier C or D artifacts are unimportant. It is saying:

- they are less universal across the whole source corpus
- they often depend on specialized workflows or affiliations
- they are more likely to be tranche or target-market decisions

## Source Signals Used For Criticality

Signals considered:

- whether the artifact has dedicated public and operator entry points
- whether other workflows depend on it
- whether it appears in publication control surfaces
- whether it is format- or affiliation-specific
- whether it is used mid-tournament versus only at the end

## Important Cautions

- a Tier C artifact can be Tier A within a specific market slice such as NSDA districts
- criticality here is cross-platform/source-wide, not customer-segment specific
- later ScholarComp-side planning should re-rank these by target market

## Open Questions For Later Validation

- whether prelims table use is widespread enough to elevate it from Tier B to Tier A
- whether sweeps should be treated as Tier A for specific tournament segments
- which printed artifacts remain operationally mandatory despite digital publication
