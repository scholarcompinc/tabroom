# First-Pass Settings Tag Inventory

## Purpose

This document captures a first-pass inventory of concrete setting/tag names visible in the current Tabroom codebase.

Its purpose is to:

- make the configuration surface more concrete,
- identify high-frequency and high-signal settings,
- show where configuration overlaps with workflow state,
- guide later detailed settings extraction.

This is not an exhaustive or canonical tag dictionary yet.

## Source Basis

Primary evidence used:

- `rg`/`perl` extraction of `setting("...")` usage across:
  - `web/setup`
  - `web/register`
  - `web/panel`
  - `web/tabbing`
  - `web/user`
  - `web/funclib`
  - `web/index`
- code patterns using `tag => "..."`, `setting_name = "..."`, and related setting writes
- `doc/sql/current-schema.sql`

Important limitation:

- this is a code-usage inventory, not a full database inventory
- dynamic tags, dead tags, or data-only tags may be underrepresented or overrepresented
- counts reflect code references, not runtime usage frequency

## High-Frequency Tags From `setting("...")` Usage

Top observed examples from code references:

| Approx. Count | Tag |
|---|---|
| 46 | `ncfl` |
| 33 | `judge_per` |
| 32 | `breakout_` |
| 28 | `pairing_seed` |
| 27 | `notes` |
| 26 | `qualifiers` |
| 25 | `coaches` |
| 24 | `nsda_district` |
| 24 | `weekend` |
| 23 | `no_codes` |
| 23 | `phone` |
| 22 | `aff_label` |
| 22 | `neg_label` |
| 21 | `forfeits_rank_last` |
| 21 | `prefs` |
| 20 | `rounds_per` |
| 19 | `po_contest` |
| 19 | `prelim_jpool` |
| 18 | `equal_elims` |
| 17 | `email` |
| 17 | `nsda_district_questions` |
| 17 | `num_judges` |

Interpretation:

- judging burden, pairing, district specialization, and registration/contact details are highly represented
- configuration is not confined to tournament setup; it reaches deeply into live tabbing and public views

## High-Frequency Tags From General `tag => ...` Style Usage

Top observed examples from broader tag references:

| Approx. Count | Tag |
|---|---|
| 103 | `winloss` |
| 73 | `rank` |
| 69 | `online_hybrid` |
| 66 | `point` |
| 65 | `supp` |
| 47 | `chapter` |
| 41 | `online_mode` |
| 37 | `paradigm` |
| 30 | `diverse` |
| 30 | `registrant` |
| 30 | `tab_rating` |
| 30 | `weekend` |
| 25 | `po` |
| 24 | `rejected_by` |
| 23 | `nsda_district` |
| 22 | `ignore_results` |

Interpretation:

- some of the most frequently referenced tags are not just config tags; they are score/result tags and operational state markers
- later extraction must keep “settings tags” and “result/score tags” distinct even though both use `tag`

## Thematic Tag Groups

## 1. Tournament Identity, Display, and Publication

Observed examples:

- `logo`
- `schemat_display`
- `school_codes`
- `no_codes`
- `hide_codes`
- `schem_designation`
- `schem_orientation`
- `results_published`
- `judge_publish_results`
- `anonymous_public`
- `live_updates`
- `include_room_notes`
- `motion_publish`

What these appear to control:

- public-facing pairings/results visibility
- labeling and posting styles
- what information is hidden or exposed

## 2. Registration and Entry Management

Observed examples:

- `max_entry`
- `min_entry`
- `cap`
- `school_cap`
- `overall_cap`
- `school_overall_cap`
- `waitlist_rank`
- `off_waitlist`
- `drop_deadline`
- `fine_deadline`
- `drop_fine`
- `open_prefs`
- `dq`
- `no_elims`
- `exclude_from_sweeps`
- `preferred_flight`
- `ballot_notes`
- `qualifiers`
- `atlarge`
- `lastchance`
- `source_entry`

What these appear to control:

- whether an entry may register, compete, break, or affect awards
- tournament-specific overrides and late-stage adjustments

Important note:

- several of these function as lifecycle or eligibility state, not just static settings

## 3. Judge Supply, Burden, and Assignment

Observed examples:

- `judge_per`
- `rounds_per`
- `prelim_jpool`
- `prelim_jpool_name`
- `pref_jpool`
- `hired_jpool`
- `hired_rounds`
- `num_judges`
- `judge_surcharge`
- `hired_fee`
- `round_hire_fee`
- `uncovered_entry_fee`
- `judge_deadline`
- `no_judge_warnings`
- `judge_publish_results`
- `judge_sheet_notice`
- `standby_timeslot`
- `tab_rating`
- `neutral`
- `diverse`

What these appear to control:

- obligation math
- available judge pools
- how many judges are needed per round type
- assignment quality or special judge traits

## 4. Preferences, Strikes, and Conflicts

Observed examples:

- `prefs`
- `prefs_only`
- `self_prefs`
- `school_strikes`
- `free_strike`
- `strike_start`
- `strike_end`
- `conflicts`
- `both_sides_same`
- `no_prefs`
- `noprefs`
- `side_based_prefs`

What these appear to control:

- whether and how preference systems are used
- who may interact with them
- timing windows and special constraint behavior

## 5. Pairing, Seeding, and Break Logic

Observed examples:

- `pairing_seed`
- `powermatch`
- `seed_round`
- `seed_presets`
- `round_robin`
- `round_robin_`
- `equal_elims`
- `double_elimination`
- `breakouts`
- `breakout_`
- `use_for_breakout`
- `registered_seed`
- `allow_rank_ties`
- `forfeits_never_break`
- `forfeits_rank_last`
- `sidelock_against`
- `sidelock_elims`
- `no_side_constraints`
- `truncate_fill`
- `speaker_protocol`

What these appear to control:

- how entries are seeded and paired
- who advances
- how forfeits/byes affect standings
- how round types interact with ranking logic

## 6. Ballots, Scoring, and Results

Observed examples:

- `point_increments`
- `min_points`
- `max_points`
- `coach_points`
- `po_contest`
- `student_ballot`
- `speakers_rubric`
- `ballot_rubric`
- `ballot_rules`
- `ballot_rules_chair`
- `combined_ballots`
- `speaker_max_scores`
- `speaker_min_speeches`
- `break_point`
- `ignore_results`

What these appear to control:

- ballot payload expectations
- score scales
- rubric-driven scoring
- whether a round should count in standings

## 7. Online, Hybrid, and Remote Competition

Observed examples:

- `online_mode`
- `online_hybrid`
- `online_ballots`
- `use_normal_rooms`
- `campus_observers`
- `campus_test_private`
- `video_link`

What these appear to control:

- whether pairings/ballots assume physical rooms, virtual rooms, or hybrid behavior
- whether remote observation or special routing is needed

## 8. Financial and Commerce Behavior

Observed examples:

- `per_student_fee`
- `per_person_fee`
- `hotel`
- `hotel_message`
- `currency`
- `purchase_order`
- `purchase_order_at`
- `purchase_order_by`
- `refund_address`
- `refund_payable`
- `refund_method`

What these appear to control:

- registration pricing
- invoicing/payment workflow
- hotel/housing communication

## 9. District, Nationals, and Circuit Specialization

Observed examples:

- `nsda_district`
- `nsda_district_questions`
- `nsda_district_email`
- `nsda_district_sw_email`
- `nsda_district_ballot_header`
- `nsda_district_open`
- `nsda_district_deadline`
- `nsda_nats`
- `nats_category`
- `nsda_house_bloc`
- `nsda_strikes`
- `nsda_ratings`
- `nsda_paid`
- `nsda_membership`
- `ncfl`
- `naudl`
- `usa_wsdc`

What these appear to control:

- special governing-body rules
- district qualification administration
- specialized documents and exports
- alternate pairing/break/reporting behavior

## 10. Identity, Contact, and Self-Service

Observed examples:

- `contact`
- `contact_name`
- `contact_email`
- `contact_number`
- `email`
- `phone`
- `coaches`
- `registrant`
- `paradigm`
- `paradigm_quiz`
- `reg_answers`
- `public_signup`
- `public_signup_at`
- `public_signup_by`

What these appear to control:

- who the tournament or school contacts are
- self-service judge signup and paradigm flows
- questionnaire/registration-answer capture

## Tags That Behave Like State

These are especially important because they look like settings but behave like workflow state or audit fields:

- `dropped_at`
- `dropped_by`
- `rejected_by`
- `public_signup_pending`
- `public_signup_at`
- `notes_processed`
- `off_waitlist`
- `results_published`
- `ignore_results`
- `session_lock`
- `comments_reviewed`

Implication:

- later modeling work must separate durable configuration from operational state and audit markers

## Dynamic or Patterned Tags

Observed patterns:

- `breakout_*`
- `quiz_ignore_*`
- `vaccine_<tourn_id>`
- `exempt_<tourn_id>`
- `no_<district_id>`
- `round_robin_*`

Implication:

- not all tags are a finite static dictionary
- some settings are parameterized families and may require explicit target-state modeling instead of one generic settings table

## First-Pass Priority Tags For Deeper Extraction

The highest-value tags to extract in detail next are:

- `prefs`
- `judge_per`
- `rounds_per`
- `num_judges`
- `powermatch`
- `pairing_seed`
- `online_mode`
- `online_hybrid`
- `speaker_protocol`
- `forfeits_never_break`
- `forfeits_rank_last`
- `ignore_results`
- `no_elims`
- `dq`
- `exclude_from_sweeps`
- `breakout_*`
- `nsda_district*`
- `ncfl`

## Open Questions For Later Passes

- which tags are still live in production versus legacy residue?
- which tags belong to a normalized first-class model instead of a generic settings store?
- which tags are user-configurable versus system-managed?
- which tags need strong validation and typed contracts in a future platform?
- which tags are mutually dependent and should be modeled as one configuration object?
