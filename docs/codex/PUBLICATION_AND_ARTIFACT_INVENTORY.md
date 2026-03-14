# Publication and Artifact Inventory

## Purpose

This document captures the first-pass inventory of publication-controlled artifacts and audience-visible outputs in the current Tabroom platform.

Its purpose is to answer:

- what gets published,
- at what granularity publication appears to occur,
- which artifacts are visible to which audiences,
- why publication appears to be a real product domain rather than a simple flag.

This is a descriptive source-analysis artifact, not a future UI or delivery design.

## Source Basis

Primary evidence used:

- publication-related scans across `web/index`, `web/tabbing`, `web/panel`, and `web/funclib`
- `funclib/results_debate.mas`
- `funclib/results_speech.mas`
- `funclib/results_congress.mas`
- `index/tourn/postings/*`
- `index/tourn/results/*`
- `tabbing/publish/*`
- `panel/schemat/show.mhtml`
- `panel/schemat/schemat_switch.mhtml`
- `FIRST_PASS_REPORTING_PRINT_EXPORT_DEEP_DIVE.md`

## First-Pass Conclusions

High-confidence observations:

- publication is multi-layered
- different artifacts publish at different times and detail levels
- some publication state is round-level, some panel-level, some result-set-level, and some setting-driven
- audience visibility is partially controlled by explicit publication state and partially by specialized visibility settings

## Publication-Controlled Artifact Types

The current source corpus shows at least these artifact families:

## 1. Pairings / Schematics

Observed evidence:

- `round.published`
- public postings under `index/tourn/postings/*`
- operator controls under `panel/schemat/*`

Observed notes:

- round publication appears to gate pairing visibility
- rounds can exist operationally before being visible publicly

## 2. Round Results

Observed evidence:

- `round.post_primary`
- `round.post_secondary`
- `round.post_feedback`
- `event_setting.judge_publish_results`
- round-result rendering helpers

Observed notes:

- result detail can be staged
- primary results, secondary results, and feedback are separate concerns

## 3. Result Sets / Standings Artifacts

Observed evidence:

- `result_set.published`
- `result_set.coach`
- public result pages under `index/tourn/results/*`
- publish surfaces under `tabbing/publish/*`

Observed notes:

- generated result sets are independently publishable
- publication is not limited to round-level results

## 4. Paradigms / Judge Pools

Observed evidence:

- published paradigm checks
- jpool publication checks

Observed notes:

- not all auxiliary information is public by default

## 5. Motions, Flips, and Round-Specific Metadata

Observed evidence:

- `motion_publish`
- `flip_published`
- related round settings and publish flows

Observed notes:

- publication covers not just pairings/results but also attached round metadata

## 6. Strike Cards / Clearing Lists / Special Lists

Observed evidence:

- `strikes_published`
- clearing-related publication messaging

Observed notes:

- specialty operational artifacts also have explicit visibility control

## Main Publication Control Points

## Round-Level Controls

Observed controls:

- `round.published`
- `round.post_primary`
- `round.post_secondary`
- `round.post_feedback`

Interpretation:

- these are the core publication axes for live round artifacts

## Panel-Level Controls

Observed controls:

- `panel.publish`

Interpretation:

- panel-level publication seems to support judge-specific or partial visibility cases

## Result-Set-Level Controls

Observed controls:

- `result_set.published`
- `result_set.coach`

Interpretation:

- generated standings/brackets can have their own publication lifecycle

## Setting-Based Visibility Controls

Observed examples:

- `anonymous_public`
- `live_updates`
- `judge_publish_results`
- `motion_publish`
- `include_room_notes`
- `publish_paradigms`

Interpretation:

- publication is partly entity-state and partly policy/configuration

## Audience Categories Visible In Source

First-pass audience groups implied by code:

- public viewers
- coaches / school-affiliated users
- judges
- tab room operators
- specialized admin/staff roles

Observed implication:

- different artifacts are intentionally exposed differently to different audiences

## Artifact Inventory By Audience

## Public-Facing Artifacts

Observed likely public artifacts:

- published pairings/postings
- public result sets
- public round results
- public paradigms where enabled
- public tournament pages and schedule data

## Coach / Team-Facing Artifacts

Observed likely coach-facing artifacts:

- standings and result sets marked as coach-visible
- updates / follow pages
- entry records and school-focused result views

## Judge-Facing Artifacts

Observed likely judge-facing artifacts:

- judge posting views
- ballots
- round-specific result detail when allowed

## Operator-Facing Artifacts

Observed operator-only or operator-primary artifacts:

- unpublished schematics
- disaster views
- print/report menus
- result-set generation and switching surfaces
- publication controls themselves

## Publication Granularity Observed

The current product appears to support publication at these granularities:

- tournament-level auxiliary pages/files
- event-level policies
- round-level pairing visibility
- round-level results detail
- panel-level visibility nuance
- result-set-level publication
- special artifact publication like paradigms, flips, strikes

## Important Publication Behaviors To Preserve In Substance

The source corpus strongly suggests these behaviors are important:

- pairings can be ready before being visible
- results can publish in layers
- detailed score visibility can lag behind basic outcome visibility
- some metadata like motions and flips have independent publication control
- public anonymity and live-updates policies alter visible detail

## Artifact Families Worth Treating Separately Later

These artifact families appear distinct enough that later planning should not merge them blindly:

- pairings/postings
- round results
- standings/result sets
- judge paradigms / judge pools
- print artifacts
- audit/disaster artifacts
- specialized qualification/award outputs

## Suggested Next Deeper Pass

If more source analysis is done here, the next version of this inventory should enumerate:

- artifact name
- source file(s)
- audience
- publication control points
- Release 1 necessity

That would still be a descriptive input and useful later in the ScholarComp repo.
