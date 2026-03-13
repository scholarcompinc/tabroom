# First-Pass Configuration and Settings Spec

## Purpose

This document is the first-pass configuration and settings specification for the current Tabroom platform.

Its purpose is to make the current configuration surface visible before:

- normalization,
- service mapping,
- API design,
- UI redesign.

This is not yet a target-state configuration design.

## Source Basis

Primary evidence used in this pass:

- `web/lib/Tab/*Setting.pm`
- setting access patterns across `web/setup`, `web/register`, `web/panel`, `web/tabbing`, `web/user`, and `web/funclib`
- tournament manual setup and rules sections
- public docs at `docs.tabroom.com`

Direct evidence of setting model scope:

- `CategorySetting`
- `ChapterSetting`
- `CircuitSetting`
- `EntrySetting`
- `EventSetting`
- `JPoolSetting`
- `JudgeSetting`
- `PanelSetting`
- `PersonSetting`
- `ProtocolSetting`
- `RPoolSetting`
- `RegionSetting`
- `RoundSetting`
- `SchoolSetting`
- `Setting`
- `StudentSetting`
- `TabroomSetting`
- `TournSetting`
- `WeekendSetting`

## First-Pass Conclusions

High-confidence observations:

- the product is heavily configuration-driven
- settings exist at many scopes, not just tournament scope
- a large part of product behavior is represented through EAV-style setting tables
- configuration likely controls:
  - registration behavior
  - judging behavior
  - pairing and assignment behavior
  - scoring and tiebreak behavior
  - publication behavior
  - financial/billing behavior
  - online/hybrid behavior
  - district/NSDA/nationals specialization

Implication for later work:

- configuration should be treated as a major product domain, not as support plumbing

## Setting Scopes

## 1. Global / Platform Scope

Observed forms:

- `TabroomSetting`
- `Setting`

Likely uses:

- platform defaults
- global labels/metadata
- site-wide or system-wide options

Confidence:

- medium

## 2. Person / Identity Scope

Observed forms:

- `PersonSetting`
- `StudentSetting`
- `JudgeSetting`
- `SchoolSetting`
- `ChapterSetting`

Examples seen in source:

- `force_password_change`
- `paradigm`
- `reg_answers`
- `first_year`
- `neutral`
- `original_school`
- `nsda_paid`
- `coaches`
- `self_prefs`
- `contact_name`
- `contact_email`
- `contact_number`

Likely uses:

- identity/profile state
- affiliation metadata
- self-service preferences
- judge-specific qualifications and answers
- school/chapter contact and billing metadata

## 3. Tournament Scope

Observed forms:

- `TournSetting`
- `WeekendSetting`

Examples seen in source:

- `drop_deadline`
- `judge_deadline`
- `fine_deadline`
- `freeze_deadline`
- `overall_cap`
- `school_overall_cap`
- `school_codes`
- `logo`
- `notes`
- `track_reg_changes`
- `nsda_nats`
- `nsda_district`
- `mock_trial_registration`
- `per_student_fee`
- `per_person_fee`
- `hotel_message`

Likely uses:

- registration windows and caps
- branding and public tournament configuration
- financial deadlines and fee setup
- affiliation/specialized tournament mode flags
- global operational rules for a tournament

## 4. Category / Division Scope

Observed forms:

- `CategorySetting`

Examples seen in source:

- `rounds_per`
- `judge_per`
- `hired_jpool`
- `hired_fee`
- `uncovered_entry_fee`
- `missing_judge_fee`
- `tab_room`
- `nats_category`
- `neutrals`
- `no_codes`
- `weekend`
- `judge_sheet_notice`

Likely uses:

- judge burden and hiring math
- category-specific judging behavior
- visibility and coding conventions
- category-level financial and scheduling behavior

## 5. Event Scope

Observed forms:

- `EventSetting`

Examples seen in source:

- `aff_label`
- `neg_label`
- `online_mode`
- `online_hybrid`
- `online_ballots`
- `combined_ballots`
- `min_points`
- `max_points`
- `team_points`
- `point_increments`
- `round_robin`
- `break_point`
- `judge_publish_results`
- `speaker_protocol`
- `ballot_rubric`
- `strike_card_message`
- `show_totals`
- `hide_codes`
- `no_codes`
- `waitlist_rank`
- `drop_fine`
- `fine_deadline`

Likely uses:

- event-format configuration
- ballot structure and display
- scoring model and increments
- publication rules
- online/hybrid behavior
- registration/waitlist behavior
- side/label and result visibility settings

## 6. Round Scope

Observed forms:

- `RoundSetting`

Examples seen in source:

- `motion`
- `include_room_notes`
- `show_chair`
- `exclude_scoresheets`
- `session_lock`
- `ignore_results`
- `disaster_checked`
- `strikes_due`
- `flip_published`

Likely uses:

- round-specific publication and document behavior
- congress/debate round metadata
- operational validation/recovery state
- strike-card/pairing timing behavior

## 7. Panel / Pool Scope

Observed forms:

- `PanelSetting`
- `JPoolSetting`
- `RPoolSetting`

Examples seen in source:

- `registrant`
- `event_based`
- `standby_timeslot`
- `message`
- `show_judges`
- `prelim_jpool_name`
- `pref_jpool`

Likely uses:

- pool construction rules
- pool display/publication behavior
- operational messaging and assignment behavior
- standby and event-based pool semantics

## 8. Protocol / Scoring Scope

Observed forms:

- `ProtocolSetting`

Examples seen in source:

- `forfeits_never_break`
- score tag behavior such as `winloss`, `point`, `rank`

Likely uses:

- ballot/scoring semantics
- scoring tag behavior
- break/tiebreak-related scoring consequences

## 9. Entry Scope

Observed forms:

- `EntrySetting`

Examples seen in source:

- `rejected_by`
- `dropped_at`
- `dropped_by`
- `waitlist_rank`

Likely uses:

- entry status
- change tracking
- rejection/waitlist metadata
- operational audit trail

## Functional Setting Families

These families cut across the scope types above.

## Registration and Eligibility

Observed examples:

- `overall_cap`
- `school_overall_cap`
- `waitlist_rank`
- `max_entry`
- `min_entry`
- `double_entry`
- `double_max`
- `signup_deadline`
- `judges_waitlist`
- `release_forms`
- `eligibility_forms`

## Judging, Preferences, and Burden

Observed examples:

- `judge_per`
- `rounds_per`
- `hired_jpool`
- `hired_fee`
- `round_hire_fee`
- `missing_judge_fee`
- `neutral`
- `tab_rating`
- `parli`
- `standby`
- `standby_timeslot`
- `pref_jpool`
- `show_judges`

## Pairing and Assignment

Observed examples:

- `pairing_seed`
- `seed_round`
- `registered_seed`
- `sidelock_against`
- `no_side_constraints`
- `timeslot_merge`
- `seed_presets`
- `schem_designation`
- `tab_room`

## Ballots, Scoring, and Results

Observed examples:

- `combined_ballots`
- `speaker_protocol`
- `ballot_rubric`
- `min_points`
- `max_points`
- `team_points`
- `point_increments`
- `show_totals`
- `judge_publish_results`
- `forfeits_never_break`
- `forfeits_rank_last`
- `allow_rank_ties`
- `equal_elims`

## Publication and Visibility

Observed examples:

- `logo`
- `notes`
- `results_published`
- `flip_published`
- `show_chair`
- `no_opponent_results`
- `judges_ballots_visible`
- `hide_codes`
- `no_codes`

## Financials and Billing

Observed examples:

- `per_student_fee`
- `per_person_fee`
- `drop_fine`
- `fine_deadline`
- `judge_surcharge`
- `purchase_order`
- `purchase_order_by`
- `purchase_order_at`
- `currency`
- `hotel`
- `hotel_message`

## Online / Hybrid / Campus

Observed examples:

- `online_mode`
- `online_hybrid`
- `online_ballots`
- `use_normal_rooms`
- `video_link`
- `campus_observers`
- `campus_room_limit`
- `campus_test_private`

## District / NSDA / Specialized Behavior

Observed examples:

- `nsda_nats`
- `nsda_district`
- `nsda_district_questions`
- `nsda_notes`
- `nsda_paid`
- `nsda_points`
- `nsda_points_posted`
- `nats_category`
- `qualifiers`
- `bid_round`

## Output / Print / Document Behavior

Observed examples:

- `aff_label`
- `neg_label`
- `col_space`
- `row_space`
- `top_margin`
- `left_margin`
- `panel_labels`
- `student_vote_message`
- `judge_sheet_notice`

## First-Pass Release 1 Setting Priorities

Most likely Release 1 critical setting families:

- registration windows/caps
- event/category structure
- judge burden and preference settings
- pairing and scoring settings
- ballot/result visibility settings
- room/accessibility related settings
- core financial settings

Likely later or tranche-dependent:

- specialized district/nationals settings
- some print/layout tuning settings
- concessions and niche billing settings
- some online/hybrid/campus specialization

## Risks and Open Questions

- The settings surface is broader than entity counts alone suggest.
- Some settings likely behave as feature flags for specialized tournament modes.
- Multiple scopes may override or interact in ways that are not yet obvious.
- The public docs likely explain some settings, but not all interactions or edge cases.
- Later requirements work should separate:
  - must-preserve behavioral settings,
  - simplification candidates,
  - settings that are mostly legacy residue.

## Recommended Follow-On Work

1. Extract settings by scope from `docs.tabroom.com`
2. Build a more structured setting inventory with:
   - scope
   - type
   - default behavior
   - affected workflows
3. Identify Release 1 critical settings explicitly
4. Document setting interaction risks for pairing, judging, and results
