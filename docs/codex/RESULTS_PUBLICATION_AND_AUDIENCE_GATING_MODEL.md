# Results Publication and Audience Gating Model

## Purpose

This document captures the first-pass publication and audience-gating model for results-related artifacts in the current Tabroom platform.

Its purpose is to separate:

- artifact generation from artifact visibility,
- round-result posting from result-set publication,
- public visibility from coach-only visibility,
- per-format detail thresholds from general publication state.

This is a descriptive model of current behavior, not a target authorization design.

## Source Basis

Primary evidence used:

- `web/tabbing/publish/index.mhtml`
- `web/tabbing/publish/generate_results.mhtml`
- `web/tabbing/publish/result_set_switch.mhtml`
- `web/tabbing/publish/rset_switch.mhtml`
- `web/index/tourn/results/index.mhtml`
- `web/index/tourn/results/event_results.mhtml`
- `web/index/tourn/results/prelims_table.mhtml`
- `web/index/tourn/results/round_results.mhtml`
- `web/funclib/results_debate.mas`
- `web/funclib/results_speech.mas`
- `web/funclib/results_congress.mas`
- `web/funclib/region_results.mas`
- prior codex results/publication docs

## First-Pass Conclusions

High-confidence observations:

- results publication is not one switch
- round artifacts and result sets are governed separately
- visibility is audience-sensitive
- some detail layers are independently gated within one published artifact
- coach-only visibility is a first-class state, not just an operator convention

## Publication Layers

## 1. Round Publication

Observed controls:

- `round.published`
- `round.post_primary`
- `round.post_secondary`
- `round.post_feedback`
- `panel.publish`
- event setting `judge_publish_results`

Observed effect:

- determines whether one-round public result views are visible
- also determines how much detail is visible within the round result

Observed public entry points:

- `index/tourn/results/round_results.mhtml`
- `funclib/results_debate.mas`
- `funclib/results_speech.mas`
- `funclib/results_congress.mas`

## 2. Result-Set Publication

Observed controls:

- `result_set.published`
- `result_set.coach`
- `result_set.bracket`
- generated timestamp and label metadata

Observed effect:

- determines whether persisted standings-like artifacts are accessible from public/coach result pages

Observed public entry points:

- `index/tourn/results/index.mhtml`
- `index/tourn/results/event_results.mhtml`
- `index/tourn/results/prelims_table.mhtml`

## 3. Specialized Derived Publication

Observed examples:

- sweeps generation with publish flag
- qualification outputs
- region/coaching result views that accept either public or coach-visible result sets

Implication:

- some downstream consumers accept broader visibility than strict public pages do

## Audience Categories Visible In Source

## 1. Public

Observed characteristics:

- may access published round results
- may access published result sets
- is blocked when result sets are unpublished
- cannot use coach-only visibility in ordinary public result pages

## 2. Coach-Scoped Audience

Observed characteristics:

- certain result tables are tagged for coach use
- `result_set.coach` exists as a separate visibility state
- some funclib helpers explicitly accept `published = 1 or coach = 1`

Implication:

- the product distinguishes “not public yet, but visible to coaches” from both operator-only and public

## 3. Operator / Admin

Observed characteristics:

- can generate result sets
- can publish and unpublish
- can delete and regenerate labeled sets
- can manage public/coach visibility separately

## Gating By Artifact Family

| Artifact Family | Primary Visibility Controls | Notes |
|---|---|---|
| Round result pages | `round.published`, `post_primary`, `post_secondary`, `post_feedback`, `panel.publish`, `judge_publish_results` | Detail threshold varies by format and score type |
| Generic result sets | `result_set.published` | Public event result page requires published state |
| Coach-oriented prelims table | `result_set.published`, downstream `tag => 'coach'` rendering, `result_set.coach` in related helpers | Suggests a distinct coach-audience path even when based on same underlying set |
| Brackets | `result_set.published`, `result_set.bracket` | Public entry redirects to bracket view when bracket flag is set |
| Region / downstream result helpers | `result_set.published or result_set.coach` | Broader visibility predicate than public pages |
| Sweeps / specialized outputs | generation-time publish flag plus downstream routing | Publication handled as part of generation flow |

## Detail Threshold Model Inside Round Results

The source strongly suggests that one round can expose different detail levels.

### Debate-like events

Observed gating:

- primary posting unlocks win/loss outcome
- secondary posting unlocks points and additional score detail
- panel-level publication may unlock results when `judge_publish_results` is enabled

### Speech

Observed gating:

- primary posting unlocks rank visibility
- secondary posting unlocks points
- panel-level publication may unlock result visibility when enabled

### Congress

Observed gating:

- primary posting unlocks rank visibility
- secondary posting unlocks points and speech values
- feedback/post-feedback is also part of the visibility condition

Implication:

- publication state is not just “visible / not visible”
- it is a layered disclosure model

## Coach-Only Visibility Signals

Observed direct signals:

- publish UI exposes a distinct `Coach Only` column
- public checkbox and coach-only checkbox are mutually constrained in the UI
- public publication hides the coach-only toggle for the same set
- helper code in region result functions treats `published = 1 or coach = 1` as visible

Implication:

- coach-only result visibility should be treated as a real artifact state in later requirements work

## Mutability Signals

Observed operational behavior:

- publish all / unpublish all operations exist
- individual result-set publish state can be toggled
- existing labeled result sets are deleted and regenerated during new generation flows

Implication:

- publication state is mutable
- generated result artifacts are operationally replaceable, not append-only history

## Public-Facing Entry Rules Visible In Source

High-confidence examples:

- `index/tourn/results/index.mhtml` rejects unpublished result sets
- `index/tourn/results/event_results.mhtml` rejects unpublished result sets
- `index/tourn/results/prelims_table.mhtml` rejects unpublished result sets
- `index/tourn/results/round_results.mhtml` rejects rounds below posting threshold unless by-panel publication is allowed

## Important Distinctions To Preserve Later

- round posting state is not the same thing as result-set publication state
- public visibility is not the same thing as coach visibility
- one artifact can expose partial details before exposing full details
- bracket visibility rides on result-set publication but is presented differently
- generation and publication are related but separate actions

## Open Questions For Later Validation

- whether ordinary coach-only result sets have dedicated end-user navigation outside helper flows
- whether `result_set.coach` is consistently enforced across all downstream result consumers
- which specialized results should be treated as public, coach-only, or operator-only by default
