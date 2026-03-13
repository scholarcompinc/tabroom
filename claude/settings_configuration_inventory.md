# Settings and Configuration Inventory

## Overview

Tabroom's configuration system uses **19 EAV (Entity-Attribute-Value) `*_setting` tables** that store flexible key-value pairs against nearly every entity type. This is the single largest source of product behavior variation and one of the most complex areas to catalog.

**Scale:** Approximately **570+ unique setting tags** identified across the three most-used scopes alone:
- `event_setting`: **231 unique tags**
- `tourn_setting`: **193 unique tags**
- `category_setting`: **145 unique tags**

Additional settings exist on `round`, `panel`, `person`, `chapter`, `circuit`, `region`, `school`, `entry`, `student`, `judge`, `jpool`, `rpool`, `protocol`, and `tabroom` scopes.

**Source:** Extracted via grep of `*_settings->{"..."}` patterns across the entire `web/` directory.

---

## Setting Scopes

### 1. Tournament-Level Settings (`tourn_setting`)
**193 unique tags identified**

**Registration & Deadlines:**
- `closed_entry` — Close registration
- `drop_deadline` — Deadline for drops
- `freeze_deadline` — Freeze registration changes
- `judge_deadline` — Judge registration deadline
- `fine_deadline` — Fine assessment deadline
- `fifty_percent_deadline` — 50% fee deadline
- `hundred_percent_deadline` — 100% fee deadline
- `release_deadline` — Release deadline
- `script_deadline` — Script submission deadline
- `supp_deadline` — Supplemental deadline
- `bill_deadline` — Billing deadline
- `hide_deadlines` — Hide deadlines from public
- `overall_cap` — Tournament-wide entry cap
- `school_overall_cap` — Per-school entry cap

**Financial:**
- `enable_credit` — Enable credit card payment
- `authorizenet_enable` — AuthorizeNet payment gateway
- `authorizenet_api_login` — AuthorizeNet API login
- `authorizenet_transaction_key` — AuthorizeNet transaction key
- `authorizenet_client_key` — AuthorizeNet client key
- `authorizenet_cc_fee` — Credit card processing fee
- `authorizenet_ach_enable` — ACH payment enable
- `authorizenet_ach_fee` — ACH processing fee
- `paypal_enable` — PayPal payment
- `tmoney_enable` — TMoney payment system (NSDA)
- `tmoney_staging` — TMoney staging environment
- `nsda_billing` — NSDA billing integration
- `nsda_billing_bonds` — NSDA billing for judge bonds
- `nsda_billing_entries` — NSDA billing for entries
- `nsda_billing_fines` — NSDA billing for fines
- `nsda_billing_judges` — NSDA billing for judges
- `no_registration_fees` — Disable fees
- `drop_fine` — Fine for late drops
- `add_fine` — Fine for late additions
- `forfeit_judge_fine` — Fine for judge forfeit
- `forfeit_judge_fine_elim` — Fine for judge forfeit in elims
- `first_forfeit_multiplier` — Multiplier for first forfeit
- `per_person_fee` — Per-person fee
- `per_student_fee` — Per-student fee
- `invoice_address` — Invoice address
- `invoice_message` — Invoice message
- `invoice_waitlist` — Include waitlist in invoices
- `concession_name` — Concession store name
- `concession_invoice` — Include concessions in invoice
- `currency` — Currency code
- `refund_information` — Refund policy text

**Display & Communication:**
- `hide_codes` — Hide entry codes
- `fontsize` — Print font size
- `col_space` — Column spacing
- `logo` — Tournament logo
- `disclaimer` — Tournament disclaimer
- `bias_statement` — Bias/equity statement
- `competitor_form_message` — Competitor form message
- `judgebond_message` — Judge bond message
- `hotel_message` — Hotel information
- `onsite_notes` — Onsite notes
- `instructions_url` — Instructions URL
- `suppconn_message` — Supplemental connection message
- `limit_info` — Limit info display
- `publish_schools` — Publish school list
- `show_book` — Show rulebook

**Registration Features:**
- `ask_regions` — Ask for region during registration
- `ask_observers` — Allow observer registration
- `category_adult_contact` — Require adult contact per category
- `require_adult_contact` — Require adult contact
- `allow_dashboard_drops` — Allow drops from coach dashboard
- `no_school_judges` — No school-provided judges
- `no_waitlist_double_entry` — No double-entry for waitlisted
- `mock_trial_registration` — Mock trial registration mode
- `mailing_address` — Require mailing address
- `account_contacts` — Account contact requirements
- `entry_upload` — Allow entry upload
- `entry_release` — Entry release management
- `registration_packet` — Registration packet upload
- `require_hotel_confirmation` — Require hotel confirmation
- `student_form_confirm` — Student form confirmation
- `student_form_label` — Student form label

**District/NSDA:**
- `district` — District tournament flag
- `district_eligible` — District eligibility
- `district_required` — District required
- `district_regions` — District regions
- `nsda_district` — NSDA district
- `nsda_district_questions` — District questions
- `nsda_nats` — NSDA nationals
- `nsda_ms_nats` — NSDA middle school nationals
- `nsda_online_nats` — NSDA online nationals
- `nsda_members_only` — NSDA members only
- `nsda_nonquals` — NSDA non-qualifiers
- `nsda_speech_method` — NSDA speech pairing method
- `nsda_extemp_topics` — Extemp topics
- `ceda_nationals` — CEDA nationals flag

**Misc:**
- `backtab` — Back-tabulation mode
- `avoid_others_rooms` — Avoid using other events' rooms
- `double_entry` — Double-entry allowed
- `double_max` — Max double entries
- `first_school_code` — Starting school code
- `ncfl` — NCFL tournament flag
- `ncfl_codes` — NCFL code format
- `legion` — Legion tournament flag
- `has_debate` — Tournament includes debate
- `has_speech` — Tournament includes speech
- `backup_followers` — Backup follower notification
- `forfeit_notify_coaches` — Notify coaches of forfeits
- `async_events` — Async/asynchronous events
- `chatwoot_enable` — Chatwoot integration
- `campus_zone` — Campus/online zone
- `store_carts` — Concession store carts
- `shipping_address` — Shipping for concessions

### 2. Category-Level Settings (`category_setting`)
**145 unique tags identified**

**Judge Management:**
- `deadline` — Prefs deadline
- `details_deadline` — Judge details deadline
- `default_mjp` — Default mutual judge preference scheme
- `coach_ratings` — Enable coach ratings
- `cumulate_prefs` — Cumulate preferences
- `conflicts` — Conflict tracking mode
- `conflict_message` — Conflict notification message
- `auto_conflict_hires` — Auto-conflict hired judges
- `cannot_cancel_hires` — Prevent hire cancellation
- `close_switcheroo` — Close judge switches
- `diversity_notice` — Diversity notice
- `diverse_judge_weight` — Diversity weight in assignment

**Ballot Display:**
- `ballot_entry_first_names` — Show first names on ballots
- `ballot_entry_names` — Show entry names
- `ballot_entry_school_codes` — Show school codes
- `ballot_entry_titles` — Show titles
- `ballot_judge_phones` — Show judge phones
- `ballot_region_codes` — Show region codes
- `ballot_school_codes` — Show school codes
- `ballot_school_names` — Show school names
- `ballot_signature` — Require signature
- `ballot_speakerorders` — Show speaker orders
- `ballot_times` — Show times

**Audit & Entry:**
- `audit_method` — Audit method (single/double)
- `auto_bond` — Auto judge bonds

**Registration:**
- `alt_max` — Max alternates
- `ask_alts` — Ask for alternates
- `ask_paradigm` — Require paradigm
- `ask_parli` — Ask parliamentary details
- `code_start` — Starting code number
- `custom_rounds_per` — Custom rounds per judge
- `dio_min` — DIO minimum
- `departure_notice` — Departure notice
- `departure_times` — Departure time tracking

**Judge Assignment:**
- `commitment_bump_after` — Bump commitment after N rounds
- `commitment_bump_unit` — Commitment bump unit
- `count_elims` — Count elims in burden
- `allow_region_panels` — Allow same-region panels
- `allow_school_panels` — Allow same-school panels

### 3. Event-Level Settings (`event_setting`)
**231 unique tags identified — the largest setting scope**

**Pairing & Paneling:**
- `powermatch` — Powermatching mode
- `pullup_method` — Pullup handling method
- `pullup_minimize` — Minimize pullups
- `pullup_repeat` — Allow repeat pullups
- `prevent_hitting_pullup_twice` — Prevent double pullups
- `avoid_school_hits` — Avoid same-school matchups
- `school_debates_self` — Allow same-school debates
- `region_avoid` — Avoid same-region matchups
- `region_constrain` — Region constraints
- `region_judge_forbid` — Forbid same-region judges
- `state_constraint_threshold` — State constraint threshold
- `round_robin` — Round robin format
- `seed_presets` — Seed preset rounds
- `snake_sides_huge_schools` — Snake sides for large schools
- `no_side_constraints` — Remove side constraints
- `no_elim_sidelocks` — No sidelocks in elims
- `allow_repeat_prelim_side` — Allow repeat prelim sides
- `allow_repeat_elims` — Allow repeat elim matchups
- `allow_repeat_judging` — Allow repeat judges
- `default_panel_size` — Default section size
- `min_panel_size` — Minimum section size
- `max_panel_size` — Maximum section size
- `section_sort` — Section sorting method
- `sort_precedence` — Sort precedence

**Scoring & Ballots:**
- `point_scale` — Point scale
- `point_increments` — Point increments (whole/half/tenth)
- `point_ties` — Allow point ties
- `max_points` — Maximum speaker points
- `min_points` — Minimum speaker points
- `allow_lowpoints` — Allow low-point wins
- `allow_rank_ties` — Allow rank ties
- `scorer_max` — Maximum scorers
- `number_of_speeches` — Speeches per entry
- `chair_ballot_only` — Chair ballot only
- `chair_scores` — Chair scores separately
- `chair_winloss` — Chair determines win/loss
- `chair_only_outstanding` — Chair only for outstanding speaker
- `combined_ballots` — Combined ballots
- `parli_ballot` — Parliamentary ballot format
- `parli_noautofill` — No autofill for parli
- `online_ballots` — Online ballot entry
- `student_online_ballots` — Student online ballots
- `ballot_rubric` — Rubric ballot
- `ballot_rubric_single` — Single rubric ballot
- `roles_rubric` — Roles rubric
- `speakers_rubric` — Speakers rubric
- `rfd_plz` — Request RFDs
- `comments_plz` — Request comments

**Results & Breaks:**
- `break_point` — Break point (how many advance)
- `breakouts` — Breakout rounds
- `clearing_threshold` — Clearing threshold
- `elim_method` — Elimination method
- `double_elimination` — Double elimination
- `bracket_by_ballots` — Bracket by ballots
- `bracket_rooms` — Bracket room assignment
- `advance_overall` — Advance from overall standings
- `show_averages` — Show averages
- `show_totals` — Show totals
- `show_panel_averages` — Show panel averages
- `show_panel_letters` — Show panel letters
- `hide_final_decision` — Hide final decision
- `hide_panel_decision` — Hide panel decision
- `autopublish_results` — Auto-publish results
- `anonymous_public` — Anonymous public results
- `no_opponent_results` — Hide opponent results

**Registration:**
- `cap` — Entry cap
- `school_cap` — Per-school cap
- `school_percent_limit` — School percentage limit
- `max_entry` — Max entry size
- `min_entry` — Min entry size
- `no_waitlist` — Disable waitlist
- `waitlist_all` — Waitlist all entries
- `waitlist_rank` — Waitlist ranking
- `field_waitlist` — Field size waitlist
- `field_report` — Field report
- `deadline` — Event-specific deadline
- `freeze_deadline` — Event-specific freeze
- `fine_deadline` — Event-specific fine deadline
- `drop_fine` — Event drop fine
- `registration_notice` — Registration notice text
- `hybrids` — Allow hybrid entries
- `hybrids_can_hit` — Hybrids can hit own school
- `mavericks` — Allow mavericks (incomplete teams)
- `enter_me_twice` — Allow double entry
- `always_tba` — Always show TBA

**Display & Labels:**
- `aff_label` — Affirmative label
- `neg_label` — Negative label
- `aff_string` — Affirmative string
- `neg_string` — Negative string
- `chair_label` — Chair judge label
- `panel_labels` — Panel labels
- `schem_designation` — Schematic designation
- `schem_orientation` — Schematic orientation
- `blind_mode` — Hide judge names from public
- `code_hide` — Hide entry codes
- `code_start` — Starting entry code
- `separate_codes` — Separate code ranges
- `judge_codes_only` — Show judge codes only
- `description` — Event description
- `result_description` — Results description
- `resolution` — Debate resolution
- `topic` — Topic

**Congress-Specific:**
- `congress_entry_cards` — Congress entry cards
- `congress_placard_designator` — Placard designator
- `congress_placard_nologo` — No logo on placard
- `congress_placard_noschools` — No schools on placard
- `congress_placard_title` — Placard title
- `congress_seating_entrycodes` — Seating entry codes
- `congress_seating_entrynames` — Seating entry names
- `congress_seating_schoolcodes` — Seating school codes
- `congress_seating_schoolnames` — Seating school names
- `ask_for_po` — Ask for presiding officer
- `po_contest` — PO contest
- `po_points_required` — PO points required
- `po_protocol` — PO protocol
- `leadership_protocol` — Leadership protocol
- `student_vote` — Student voting
- `student_vote_message` — Student vote message
- `enforce_equal_speeches` — Enforce equal speeches

**WUDC-Specific:**
- `include_wsdc_reply` — Include WSDC reply speeches
- `usa_wsdc` — USA WSDC format
- `wsdc_by_win_average` — WSDC by win average
- `wsdc_cap_repel` — WSDC cap repel
- `wsdc_no_rfd` — WSDC no RFD
- `wsdc_subtotal_ballot` — WSDC subtotal ballot

**Online/Hybrid:**
- `online_mode` — Online tournament mode
- `online_hybrid` — Hybrid mode
- `online_ballots` — Online ballot entry
- `online_entry_display` — Online entry display
- `online_judge_display` — Online judge display
- `online_instructions` — Online instructions
- `online_prepend_role` — Prepend role in online
- `online_public` — Public online viewing
- `online_support` — Online support link
- `online_event_match` — Online event matching
- `campus_room_limit` — Campus room limit
- `disable_video_link` — Disable video link
- `show_async_links` — Show async links
- `show_async_to_entries` — Show async to entries
- `auto_docshare` — Auto document sharing
- `live_updates` — Live score updates
- `start_button` — Start button
- `start_button_text` — Start button text

**Misc:**
- `bid_round` — TOC bid round
- `top_novice` — Top novice award
- `honorable_mentions` — Honorable mentions
- `baseline` — Baseline scoring
- `baker` — Baker format
- `big_questions` — Big Questions format
- `dukesandbailey` — Dukes and Bailey format
- `mock_trial_feedback` — Mock trial feedback
- `ndca_public_forum` — NDCA Public Forum
- `speaker_protocol` — Speaker protocol
- `speaker_max_scores` — Speaker max scores
- `speaker_min_speeches` — Speaker min speeches
- `speaker_priority_first` — Speaker priority first
- `team_points` — Team points
- `team_total_line` — Team total line

### 4. Round-Level Settings (`round_setting`)
**12 unique tags identified**

- `flip_blasted` — Flip blast sent
- `flip_published` — Flip results published
- `flip_round_deadline` — Flip deadline
- `nsda_confirmed` — NSDA confirmed
- `num_judges` — Number of judges for this round
- `showrooms_from` — Show rooms from (linked round)
- `sides_not_set` — Sides not yet assigned
- `strikes_auto` — Auto-generated strikes
- `strikes_blasted` — Strikes blast sent
- `strikes_instapublish` — Instant-publish strikes
- `strikes_published` — Strikes published
- `timeslot_merge` — Merge timeslots

### 5. Other Setting Scopes

**Panel settings (`panel_setting`):** Per-section overrides (room quality, special notes)

**Person settings (`person_setting`):** User preferences, notification settings, paradigm data, `force_password_change`, `email_unconfirmed`, `system_administrator`

**Chapter settings (`chapter_setting`):** Chapter-level preferences

**Circuit settings (`circuit_setting`):** Circuit-level configuration

**School settings (`school_setting`):** Tournament-specific school configuration

**Judge settings (`judge_setting`):** Per-judge tournament settings

**Entry settings (`entry_setting`):** Per-entry tournament settings

**Student settings (`student_setting`):** Per-student settings

**JPool settings (`jpool_setting`):** Judge pool configuration

**RPool settings (`rpool_setting`):** Room pool configuration

**Protocol settings (`protocol_setting`):** Scoring protocol configuration

**Region settings (`region_setting`):** Region configuration

**Tabroom settings (`tabroom_setting`):** Global site-wide settings

---

## Settings Architecture Notes

1. **No enumeration:** There is no master list of valid setting tags. Tags are created ad-hoc in code. Any string can be used as a tag.
2. **No validation:** The EAV pattern means there's no schema-level validation of setting values.
3. **No documentation:** Most settings are undocumented except by their usage in code.
4. **Interaction complexity:** Settings at different scopes can interact (tournament settings vs event settings vs category settings), and the precedence rules are implicit in code.
5. **Value type dispatch:** Settings use a type indicator in the `value` column ("text", "date", "json") with actual values stored in `value_text` or `value_date` columns. Simple values use the `value` column directly.
6. **Compressed JSON:** Some settings store compressed JSON in `value_text`, making them opaque to direct database queries.

## Release 1 Critical Settings (Preliminary Assessment)

The following setting families appear critical for any functional tournament:

| Family | Scope | Count (est.) | Criticality |
|--------|-------|-------------|-------------|
| Pairing/side assignment | Event | ~25 | Critical |
| Scoring/ballot format | Event | ~30 | Critical |
| Registration/caps | Event + Tourn | ~20 | Critical |
| Results/tiebreakers | Event + Protocol | ~15 | Critical |
| Judge prefs/conflicts | Category | ~20 | Critical |
| Financial/payment | Tourn | ~25 | High |
| Display/labels | Event | ~20 | High |
| Online/hybrid | Event + Tourn | ~15 | Medium |
| Congress-specific | Event | ~15 | Medium |
| WUDC-specific | Event | ~10 | Low |
| District/NSDA | Tourn + Event | ~20 | Low (unless NSDA) |

## Open Questions

- Which settings are actively used vs legacy/deprecated?
- What are the valid values for each setting tag?
- What are the interaction rules between settings at different scopes?
- Which settings would benefit from being promoted to first-class schema columns?
- Are there settings that are functionally required but have no default, creating silent failures?
- How many setting tags have typos or inconsistent naming (`instruction_url` vs `instructions_url`, `invoice_wailtist` vs `invoice_waitlist`)?
