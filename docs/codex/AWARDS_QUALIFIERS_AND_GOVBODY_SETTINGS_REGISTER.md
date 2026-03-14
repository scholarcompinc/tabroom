# Awards, Qualifiers, and Governing-Body Settings Register

## Purpose

This document captures the first-pass register of settings and configurable rule inputs that materially shape awards, qualification, and governing-body result-posting behavior in the current Tabroom platform.

Its purpose is to make explicit that this area depends heavily on configuration, not just algorithm code.

This is a descriptive source-analysis artifact, not a future settings design.

## Source Basis

Primary evidence used:

- `web/tabbing/publish/generate_results.mhtml`
- `web/tabbing/publish/generate_sweeps.mhtml`
- `web/tabbing/results/top_novice.mas`
- `web/tabbing/results/sweep_tourn.mas`
- `web/funclib/nsda/qualifier_count.mas`
- `web/funclib/nsda/post_points.mas`
- `web/tabbing/publish/index.mhtml`
- `web/tabbing/report/results_table_print.mas`

## First-Pass Conclusions

High-confidence observations:

- awards/qualification behavior is highly settings-driven
- the relevant settings live at multiple scopes: tournament, event, entry, round, and sweep-rule level
- some settings are pure enablement flags, while others materially alter policy and outputs

## Tournament-Level Settings

Observed first-pass examples:

- `nsda_district`
- `nsda_nats`
- `nsda_ms_nats`
- `nsda_online_nats`
- `nsda_district_questions`
- `nsda_district_level_force`

Observed role:

- determine whether district/nationals-specific qualification and reporting logic applies
- carry district questionnaire metadata used by later workflows
- alter event-entry and output behavior for NSDA-affiliated modes

## Event-Level Settings

Observed first-pass examples:

- `speaker_protocol`
- `top_novice`
- `honorable_mentions`
- `speaker_min_speeches`
- `bid_round`
- `bid_limit`
- `weekend`
- `supp`
- `nsda_qual_force`
- `nsda_qual_override`
- `nsda_qual_percent`
- `nsda_qual_penalty`
- `nsda_qual_max`
- `nsda_qual_nohousepilot`
- `nsda_points_posted`
- `judge_publish_results`
- `breakouts`
- `breakout_<n>_label`
- `breakout_<n>_students`

Observed role:

- enable or shape speaker awards
- determine qualifying-bid behavior
- govern district qualifier counts and overrides
- partition result outputs by breakout or weekend
- mark external posting state

## Entry-Level Settings

Observed first-pass examples:

- `exclude_from_sweeps`
- `sweeps`
- `nsda_vacate`
- `coach_points`
- breakout entry flags such as `breakout_<n>`

Observed role:

- exclude an entry from specific award calculations
- inject manual/external sweep points
- mark vacancy/qualification complications
- provide coach-point linkage and other external-reporting metadata

## Round-Level Settings

Observed first-pass examples:

- `ignore_results`
- `use_for_breakout`
- `coach_points`

Observed role:

- exclude rounds from standings/posting
- bind rounds to breakout-specific advancement/result universes
- attach certain posting/reporting semantics at round level

## Sweep-Rule and Sweep-Set Configuration

Observed first-pass examples:

- `novice_only`
- `multiply_entrysize`
- `multiplier`
- `skip_rounds`
- `ignore_round`
- `exclude_breakouts`
- `coachover_advance`
- event-type and event-level sweep filters

Observed role:

- define the scoring policy and scope of sweep awards
- determine which rounds and participants count
- alter how special cases such as coachovers affect points

## Settings Grouped By Function

## 1. Enablement Flags

Examples:

- `speaker_protocol`
- `bid_round`
- `nsda_district`
- `nsda_nats`

Meaning:

- without these, the related output family may not exist or may be inapplicable

## 2. Policy/Computation Settings

Examples:

- `nsda_qual_force`
- `nsda_qual_override`
- `nsda_qual_percent`
- `nsda_qual_penalty`
- `speaker_min_speeches`
- sweep-rule tags

Meaning:

- these directly change who wins, qualifies, or is counted

## 3. Partitioning / Filtering Settings

Examples:

- `breakouts`
- `breakout_<n>_label`
- `breakout_<n>_students`
- `exclude_from_sweeps`
- `ignore_results`
- `weekend`
- `supp`

Meaning:

- these create alternate sub-populations or remove data from a given output

## 4. Posting / State Flags

Examples:

- `nsda_points_posted`
- publish flags on result sets

Meaning:

- these affect whether an external or public output is considered already posted or ready

## Settings That Look Especially Load-Bearing

Based on current evidence, these settings look especially important:

- `speaker_protocol`
- `top_novice`
- `bid_round`
- `nsda_district`
- `nsda_nats`
- `nsda_qual_*`
- `ignore_results`
- `breakouts` / breakout labels and filters
- `exclude_from_sweeps`
- sweep-rule tags such as `exclude_breakouts` and `coachover_advance`

## Important Distinctions To Preserve Later

- display option vs policy input
- event-scope rule vs tournament-scope affiliation mode
- per-entry exclusion vs event-wide qualification rule
- generated state flag vs standing configuration

## Open Questions For Later Validation

- which of these settings are still commonly used versus legacy compatibility residue
- whether some `nsda_qual_*` branches can be normalized into clearer policy objects later
- which award/qualifier settings are mandatory for the target market slice
