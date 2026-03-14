# Settings Classification Register

## Purpose

This document is the first-pass attempt to classify the current settings/tag surface into more meaningful categories than “just settings.”

Its purpose is to help later design work avoid treating every legacy tag as the same kind of thing.

This is not yet a full tag-by-tag exhaustive register. It is a classification framework with representative examples.

## Source Basis

Primary inputs:

- `FIRST_PASS_CONFIGURATION_SETTINGS_SPEC.md`
- `FIRST_PASS_SETTINGS_TAG_INVENTORY.md`
- `FIRST_PASS_DATA_STATE_MODEL.md`
- `SOURCE_CONFLICT_AND_AMBIGUITY_REGISTER.md`

## Classification Categories

For later design purposes, the source corpus suggests five main categories:

1. durable configuration
2. workflow state / status
3. audit / attribution / history markers
4. publication / visibility controls
5. specialized or parameterized variants

## 1. Durable Configuration

Definition:

- values intended to define tournament behavior, defaults, formatting, or business rules over a meaningful period of time

Representative examples:

- `judge_per`
- `rounds_per`
- `num_judges`
- `powermatch`
- `prefs`
- `speaker_protocol`
- `point_increments`
- `min_points`
- `max_points`
- `online_mode`
- `online_hybrid`
- `judge_publish_results`
- `schem_designation`
- `school_codes`
- `logo`

Likely future treatment:

- strong typing
- grouped configuration objects
- validation rules

## 2. Workflow State / Status

Definition:

- values that represent the current operational state of an entity or a one-off per-tournament override state

Representative examples:

- `open_prefs`
- `off_waitlist`
- `dq`
- `no_elims`
- `exclude_from_sweeps`
- `use_for_breakout`
- `ignore_results`
- `session_lock`
- `public_signup_pending`
- `results_published`

Why this matters:

- these should not be modeled as timeless configuration if they actually change during workflows

## 3. Audit / Attribution / History Markers

Definition:

- values whose primary purpose is to say who did something, when it happened, or whether it has been reviewed

Representative examples:

- `dropped_at`
- `dropped_by`
- `rejected_by`
- `public_signup_at`
- `public_signup_by`
- `notes_processed`
- `comments_reviewed`
- `disaster_checked`

Why this matters:

- these likely belong in activity/history models or explicit event logs rather than generic configuration stores

## 4. Publication / Visibility Controls

Definition:

- values controlling who can see what and when

Representative examples:

- `anonymous_public`
- `live_updates`
- `judge_publish_results`
- `motion_publish`
- `flip_published`
- `strikes_published`
- `publish_paradigms`
- `include_room_notes`

Why this matters:

- the current platform clearly treats publication as a first-class domain, not just a UI toggle

## 5. Specialized / Parameterized Variants

Definition:

- values that represent a family of related settings or organization-specific branches rather than one standalone stable field

Representative examples:

- `breakout_*`
- `round_robin_*`
- `quiz_ignore_*`
- `vaccine_<tourn_id>`
- `exempt_<tourn_id>`
- `no_<district_id>`
- `nsda_district_*`

Why this matters:

- these may need structured submodels or typed collections instead of a flat settings table

## Scope-Based Classification Notes

## Tournament Scope

Most likely durable configuration:

- deadlines
- caps
- branding
- broad mode flags

Most likely mixed config/state:

- published reporting switches
- current deadlines already passed or re-opened via overrides

## Event Scope

Most likely durable configuration:

- scoring model
- ballot model
- pairing strategy
- labels and visibility defaults

Most likely mixed config/state:

- event-specific re-openings or temporary operational overrides

## Round Scope

Most likely workflow state / publication:

- `motion`
- `ignore_results`
- `disaster_checked`
- `flip_published`
- per-round publication detail controls

Implication:

- round settings are disproportionately likely to be stateful rather than timeless

## Entry / Judge / Person Scope

Most likely mixed:

- some values are profile data or durable preference data
- others are tournament-local operational markers

Representative examples:

- durable-ish:
  - `paradigm`
  - `neutral`
  - `diverse`
  - `tab_rating`
- stateful:
  - `public_signup_pending`
  - `rejected_by`
  - `dq`
  - `open_prefs`

## High-Priority Tags For Future Explicit Modeling

These look like strong candidates to become first-class modeled concepts later rather than remain generic tags:

- `prefs`
- `judge_per`
- `rounds_per`
- `num_judges`
- `powermatch`
- `online_mode`
- `online_hybrid`
- `speaker_protocol`
- `no_elims`
- `dq`
- `ignore_results`
- `anonymous_public`
- `judge_publish_results`
- `breakout_*`

## Tags That Look Especially Dangerous To Leave Generic

These appear risky because they encode product-critical state or complex behavior:

- `ignore_results`
- `dq`
- `no_elims`
- `open_prefs`
- `off_waitlist`
- `disaster_checked`
- `flip_published`
- `strikes_published`
- `results_published`
- `public_signup_*`

## Suggested Next Deeper Pass

The next more granular version of this register should likely classify tags by:

- scope
- category from this register
- suspected lifecycle
- candidate future model owner
- Release 1 relevance

That deeper pass is still source-analysis compatible and can happen here later if useful.
