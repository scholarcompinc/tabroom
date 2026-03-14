# Awards Distribution and Pickup Operation Model

## Purpose

This document captures the first-pass model of how the current Tabroom platform supports awards distribution and pickup operations after results are determined.

Its purpose is to isolate the physical/operational awards desk workflow from pure result generation.

This is a descriptive source-analysis artifact, not a future PMC workflow design.

## Source Basis

Primary evidence used:

- `web/tabbing/report/awards_pickup.mhtml`
- `web/tabbing/report/pickup_switch.mhtml`
- `web/tabbing/report/pickup.mhtml`
- `web/tabbing/report/awards_school.mhtml`
- `web/tabbing/report/awards_ceremony.mhtml`
- `web/tabbing/report/index.mhtml`

## First-Pass Conclusions

High-confidence observations:

- the platform treats awards distribution as a live operational workflow
- pickup tracking is school-based, not just result-based
- awards distribution depends on both elimination participation and speaker-award results

## Workflow Shape

## 1. Identify Eligible Schools

Observed logic:

- schools with entries in elimination rounds are included
- schools with speaker-award-winning students are also included
- the two populations are merged into one pickup list

Implication:

- awards desk operations are based on recognition participation, not just a final places table

## 2. Present Awards Desk Queue

Observed UI characteristics:

- sortable table of schools
- state/country grouping cues
- count of relevant entries/students
- link into school-specific award details

Implication:

- this is an operational distribution list, not a generic standings page

## 3. Track Pickup State

Observed mechanism:

- school-level `picked_up` setting
- toggle switch per school
- explicit endpoints to mark picked up / not picked up
- refresh polling to keep the list current

Implication:

- pickup state is persistent operational state

## 4. Drill Into School Award Details

Observed behavior:

- school-specific awards page exists
- pickup flow redirects back to school awards context after state change

Implication:

- distribution likely involves a school-level packet or checklist, not just one Boolean toggle

## Inputs To Awards Distribution

Observed first-pass inputs:

- elimination-round participation
- `Speaker Awards` result set
- school/chapter geography
- school setting `picked_up`
- nationals mode affects some thresholds/counts

Implication:

- awards distribution depends on recognition outputs from multiple sources

## Operational Meaning

The source suggests a real-world workflow:

- determine which schools have awards to collect
- sort and manage pickup at an awards desk
- mark collected/uncollected state live
- support ceremony/distribution staff during or after awards

## Why This Matters

This is easy to miss if one only studies standings and result sets. But the current platform treats:

- awards generation,
- ceremony reporting,
- and awards distribution/pickup

as connected but distinct operational workflows.

## Important Distinctions To Preserve Later

- award winner determination vs award packet distribution
- school-level pickup state vs entry/student placement
- ceremony order vs distribution queue

## Open Questions For Later Validation

- how heavily this pickup workflow is used in ordinary tournaments versus nationals/district contexts
- whether operators expect audit/history of pickup toggles
- whether awards-school detail pages are considered mandatory or convenience tooling
