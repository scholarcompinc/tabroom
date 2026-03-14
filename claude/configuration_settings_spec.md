# Configuration and Settings Spec

## Purpose

This document converts the Phase 1 settings inventory (570+ EAV tags) into a rationalized specification suitable for strongly-typed schema design. It groups settings by functional area, identifies Release 1 critical settings, and proposes simplification candidates.

---

## Design Principles for Rebuild

1. **No EAV tables.** Every setting becomes a typed column, a JSON column with schema validation, or a related entity.
2. **Settings belong to the entity they configure.** Tournament settings become columns on the tournament table (or a typed tournament_config table).
3. **Validation at the schema level.** Enums, constraints, and defaults replace the current "any string" approach.
4. **Explicit defaults.** Every setting has a documented default value.
5. **Settings interactions are documented.** Where settings at different scopes interact, the precedence is explicit.

---

## Settings Groups by Functional Area

### Group 1: Registration Configuration

**Scope:** Tournament + Event

| Setting | Current Tag(s) | Type | Default | R1? | Notes |
|---------|---------------|------|---------|-----|-------|
| Registration open | `closed_entry` (inverted) | boolean | true (open) | Yes | Inverted logic in legacy |
| Overall entry cap | `overall_cap` | integer | null (no cap) | Yes | |
| Per-school entry cap | `school_overall_cap` | integer | null | Yes | |
| Per-event entry cap | `cap` (event) | integer | null | Yes | |
| Per-school-per-event cap | `school_cap` (event) | integer | null | Yes | |
| School percentage limit | `school_percent_limit` (event) | decimal | null | No | Rare |
| Registration deadline | multiple deadline tags | datetime | null | Yes | |
| Freeze deadline | `freeze_deadline` | datetime | null | Yes | |
| Drop deadline | `drop_deadline` | datetime | null | Yes | |
| Supplemental deadline | `supp_deadline` | datetime | null | No | |
| Waitlist enabled | `no_waitlist` (event, inverted) | boolean | true | Yes | Inverted logic |
| Waitlist all entries | `waitlist_all` (event) | boolean | false | No | |
| Waitlist ranking | `waitlist_rank` (event) | string | null | No | |
| Allow hybrid entries | `hybrids` (event) | boolean | false | No | R2 |
| Allow mavericks | `mavericks` (event) | boolean | false | Yes | |
| Allow TBA entries | `always_tba` (event) | boolean | false | No | |
| Double entry allowed | `double_entry` (tourn) | boolean | false | Yes | |
| Double entry max | `double_max` (tourn) | integer | null | Yes | |
| Require adult contact | `require_adult_contact` (tourn) | boolean | false | Yes | |
| Entry upload allowed | `entry_upload` (tourn) | boolean | false | No | |
| Registration notice text | `registration_notice` (event) | text | null | No | |

**Simplification candidates:**
- `closed_entry` → rename to `registration_open` with non-inverted boolean
- `no_waitlist` → rename to `waitlist_enabled` with non-inverted boolean
- Consolidate multiple deadline tags into a `registration_deadlines` JSON object or related table

### Group 2: Scoring and Ballot Configuration

**Scope:** Event (via Protocol)

| Setting | Current Tag(s) | Type | Default | R1? | Notes |
|---------|---------------|------|---------|-----|-------|
| Point scale | `point_scale` (event) | enum | varies | Yes | e.g., "30", "6-1" |
| Point increments | `point_increments` (event) | enum | "1" | Yes | whole/half/tenth |
| Max points | `max_points` (event) | decimal | null | Yes | |
| Min points | `min_points` (event) | decimal | null | Yes | |
| Allow point ties | `point_ties` (event) | boolean | false | Yes | |
| Allow low-point wins | `allow_lowpoints` (event) | boolean | false | Yes | |
| Allow rank ties | `allow_rank_ties` (event) | boolean | false | Yes | |
| Number of speeches | `number_of_speeches` (event) | integer | 1 | Yes | Congress |
| Chair ballot only | `chair_ballot_only` (event) | boolean | false | Yes | |
| Chair determines W/L | `chair_winloss` (event) | boolean | false | Yes | |
| Chair scores separately | `chair_scores` (event) | boolean | false | No | |
| Combined ballots | `combined_ballots` (event) | boolean | false | No | |
| Online ballots enabled | `online_ballots` (event) | boolean | false | Yes | |
| Rubric ballot | `ballot_rubric` (event) | boolean | false | No | |
| Request RFDs | `rfd_plz` (event) | boolean | false | Yes | |
| Request comments | `comments_plz` (event) | boolean | false | No | |
| Scorer max | `scorer_max` (event) | integer | null | No | |
| Audit method | `audit_method` (category) | enum | "single" | Yes | single/double |

**Proposed structure:** Create a `scoring_protocol` entity with typed columns rather than scattered event settings.

### Group 3: Pairing Configuration

**Scope:** Event

| Setting | Current Tag(s) | Type | Default | R1? | Notes |
|---------|---------------|------|---------|-----|-------|
| Powermatching enabled | `powermatch` (event) | boolean | true | Yes | Debate |
| Pullup method | `pullup_method` (event) | enum | null | Yes | |
| Minimize pullups | `pullup_minimize` (event) | boolean | false | Yes | |
| Prevent hitting pullup twice | `prevent_hitting_pullup_twice` | boolean | false | Yes | |
| Avoid school hits | `avoid_school_hits` (event) | boolean | true | Yes | |
| School debates self | `school_debates_self` (event) | boolean | false | No | |
| Region avoidance | `region_avoid` (event) | boolean | false | No | |
| Region constraint | `region_constrain` (event) | enum | null | No | |
| State constraint threshold | `state_constraint_threshold` | integer | null | No | |
| No side constraints | `no_side_constraints` (event) | boolean | false | Yes | |
| Allow repeat prelim side | `allow_repeat_prelim_side` | boolean | false | Yes | |
| Allow repeat elim matchups | `allow_repeat_elims` (event) | boolean | false | Yes | |
| Allow repeat judging | `allow_repeat_judging` (event) | boolean | false | Yes | |
| Round robin format | `round_robin` (event) | boolean | false | Yes | |
| Seed presets | `seed_presets` (event) | boolean | false | No | |
| Default panel/section size | `default_panel_size` (event) | integer | varies | Yes | Speech |
| Min panel size | `min_panel_size` (event) | integer | null | Yes | |
| Max panel size | `max_panel_size` (event) | integer | null | Yes | |
| Section sort method | `section_sort` (event) | enum | null | No | |

**Proposed structure:** Create a `pairing_config` entity per event with typed columns.

### Group 4: Judge Configuration

**Scope:** Category + Event

| Setting | Current Tag(s) | Type | Default | R1? | Notes |
|---------|---------------|------|---------|-----|-------|
| Prefs mode | `default_mjp` (category) | enum | "none" | Yes | none/tiered/ordinal/caps/percentage |
| Prefs deadline | `deadline` (category) | datetime | null | Yes | |
| Coach ratings enabled | `coach_ratings` (category) | boolean | false | Yes | |
| Cumulate prefs | `cumulate_prefs` (category) | boolean | false | No | |
| Conflicts tracking mode | `conflicts` (category) | enum | null | Yes | |
| Auto-conflict hires | `auto_conflict_hires` (category) | boolean | false | No | |
| Max pref | `max_pref` (event) | integer | null | Yes | |
| Max no-break pref | `max_nobreak_pref` (event) | integer | null | No | |
| No prefs | `no_prefs` (event) | boolean | false | Yes | |
| Self-strike allowed | `self_strike` (event) | boolean | false | No | |
| No judge burden | `no_judge_burden` (event) | boolean | false | No | |
| No judge violations | `no_judge_violations` (event) | boolean | false | No | |
| Best judges for highest seed | `best_judges_highest_seed` | boolean | false | No | |
| Auto chairs | `auto_chairs` (event) | boolean | false | Yes | |
| Diverse judge weight | `diverse_judge_weight` (cat) | decimal | null | No | |
| Commitment bump after | `commitment_bump_after` (cat) | integer | null | No | |
| Rounds per judge | `custom_rounds_per` (category) | integer | null | No | |

**Proposed structure:** `judge_pool_config` entity with typed columns for judge management settings.

### Group 5: Results and Break Configuration

**Scope:** Event + Protocol

| Setting | Current Tag(s) | Type | Default | R1? | Notes |
|---------|---------------|------|---------|-----|-------|
| Break point | `break_point` (event) | integer | null | Yes | |
| Elimination method | `elim_method` (event) | enum | null | Yes | |
| Double elimination | `double_elimination` (event) | boolean | false | No | |
| Bracket by ballots | `bracket_by_ballots` (event) | boolean | false | No | |
| Clearing threshold | `clearing_threshold` (event) | decimal | null | No | |
| Advance from overall | `advance_overall` (event) | boolean | false | No | |
| Show averages | `show_averages` (event) | boolean | false | Yes | |
| Show totals | `show_totals` (event) | boolean | false | Yes | |
| Auto-publish results | `autopublish_results` (event) | boolean | false | No | |
| Anonymous public results | `anonymous_public` (event) | boolean | false | No | |
| Hide final decision | `hide_final_decision` (event) | boolean | false | No | |
| Hide panel decision | `hide_panel_decision` (event) | boolean | false | No | |
| No opponent results | `no_opponent_results` (event) | boolean | false | No | |

### Group 6: Display and Labeling

**Scope:** Event + Tournament

| Setting | Current Tag(s) | Type | Default | R1? | Notes |
|---------|---------------|------|---------|-----|-------|
| Affirmative label | `aff_label` (event) | string | "Aff" | Yes | Debate |
| Negative label | `neg_label` (event) | string | "Neg" | Yes | Debate |
| Chair label | `chair_label` (event) | string | "Chair" | No | |
| Entry code style | code_start + format flags | enum | varies | Yes | 15+ styles |
| Hide codes | `hide_codes` (tourn) | boolean | false | No | |
| Blind mode | `blind_mode` (event) | boolean | false | Yes | |
| Schematic designation | `schem_designation` (event) | string | null | No | |
| Font size (print) | `fontsize` (tourn) | integer | null | No | |
| Resolution/topic | `resolution` (event) | text | null | Yes | Debate |

### Group 7: Ballot Display (Category-Level)

**Scope:** Category

| Setting | Current Tag(s) | Type | Default | R1? | Notes |
|---------|---------------|------|---------|-----|-------|
| Show entry names on ballot | `ballot_entry_names` | boolean | true | Yes | |
| Show first names | `ballot_entry_first_names` | boolean | false | No | |
| Show school codes | `ballot_school_codes` | boolean | false | Yes | |
| Show school names | `ballot_school_names` | boolean | false | No | |
| Show region codes | `ballot_region_codes` | boolean | false | No | |
| Show speaker orders | `ballot_speakerorders` | boolean | false | Yes | Speech |
| Show titles | `ballot_entry_titles` | boolean | false | No | |
| Show times | `ballot_times` | boolean | false | No | |
| Require signature | `ballot_signature` | boolean | false | No | |
| Show judge phones | `ballot_judge_phones` | boolean | false | No | |

**Proposed structure:** `ballot_display_config` object with typed fields, stored as a JSON column or related entity.

### Group 8: Financial Configuration

**Scope:** Tournament

| Setting | Current Tag(s) | Type | Default | R1? | Notes |
|---------|---------------|------|---------|-----|-------|
| Entry fees | via `tourn_fee` entity | decimal | 0 | Yes | Already an entity, not EAV |
| No registration fees | `no_registration_fees` | boolean | false | Yes | |
| Drop fine amount | `drop_fine` | decimal | null | Yes | |
| Add fine amount | `add_fine` | decimal | null | No | |
| Judge forfeit fine | `forfeit_judge_fine` | decimal | null | Yes | |
| Elim forfeit fine | `forfeit_judge_fine_elim` | decimal | null | No | |
| Forfeit multiplier | `first_forfeit_multiplier` | decimal | null | No | |
| Per-person fee | `per_person_fee` | decimal | null | No | |
| Per-student fee | `per_student_fee` | decimal | null | No | |
| Bill deadline | `bill_deadline` | datetime | null | Yes | |
| Currency | `currency` | string | "USD" | Yes | |
| Invoice message | `invoice_message` | text | null | No | |
| Invoice address | `invoice_address` | text | null | No | |
| Payment: Stripe enabled | (new — replaces multiple gateway settings) | boolean | false | Yes | |
| Payment: Stripe keys | (new) | encrypted string | null | Yes | |

**Simplification:** Replace 3 payment gateway configs (AuthorizeNet, PayPal, TMoney) with a single Stripe integration in Release 1.

### Group 9: Congress-Specific Settings

**Scope:** Event

| Setting | Current Tag(s) | Type | Default | R1? | Notes |
|---------|---------------|------|---------|-----|-------|
| Ask for PO | `ask_for_po` | boolean | false | R1? | |
| PO contest | `po_contest` | boolean | false | R1? | |
| PO points required | `po_points_required` | integer | null | R1? | |
| PO protocol | `po_protocol` | reference | null | R1? | |
| Leadership protocol | `leadership_protocol` | reference | null | R1? | |
| Student voting | `student_vote` | boolean | false | R1? | |
| Enforce equal speeches | `enforce_equal_speeches` | boolean | false | R1? | |
| Seating chart display options | `congress_seating_*` (4 tags) | boolean | false | R1? | |
| Placard options | `congress_placard_*` (4 tags) | string/boolean | varies | R1? | |

**Decision needed:** Is Congress format included in Release 1?

### Group 10: Online/Hybrid Settings

**Scope:** Event + Tournament

| Setting | Current Tag(s) | Type | Default | R1? | Notes |
|---------|---------------|------|---------|-----|-------|
| Online mode | `online_mode` (event) | boolean | false | R2 | |
| Hybrid mode | `online_hybrid` (event) | boolean | false | R2 | |
| Online ballots | `online_ballots` (event) | boolean | false | R2 | |
| Campus room limit | `campus_room_limit` (event) | integer | null | R2 | |
| Online entry display | `online_entry_display` (event) | enum | null | R2 | |
| Online judge display | `online_judge_display` (event) | enum | null | R2 | |
| Online instructions | `online_instructions` (event) | text | null | R2 | |
| Campus zone | `campus_zone` (tourn) | string | null | R2 | |
| Async events | `async_events` (tourn) | boolean | false | R2 | |

**All deferred to Release 2.**

---

## Settings Migration Strategy

### Phase 1: Identify and Classify
- [x] Extract all setting tags from code (done — 570+ tags)
- [ ] Classify each tag: active/deprecated/duplicate
- [ ] Document valid values for each tag
- [ ] Document defaults for each tag
- [ ] Document interaction rules between scopes

### Phase 2: Design Strongly-Typed Schema
- Group related settings into configuration entities
- Define typed columns with constraints
- Create enum types for multi-value settings
- Add JSON columns with JSON Schema for complex structured settings
- Define scope precedence rules explicitly

### Phase 3: Implement Migration
- Build setting extraction from legacy EAV tables
- Map legacy tags to new typed columns
- Handle typos and duplicates (`invoice_wailtist` → `invoice_waitlist`)
- Validate migrated data against new constraints

---

## Identified Typos and Duplicates

| Legacy Tag | Likely Correct | Issue |
|-----------|---------------|-------|
| `invoice_wailtist` | `invoice_waitlist` | Typo |
| `instruction_url` | `instructions_url` | Inconsistent |
| `hide_code` | `hide_codes` | Singular vs plural |
| `panel_lables` | `panel_labels` | Typo (found in code) |

---

## Release 1 Settings Count

| Group | R1 Settings | Total Settings | % Covered |
|-------|------------|---------------|-----------|
| Registration | ~15 | ~25 | 60% |
| Scoring/Ballot | ~15 | ~25 | 60% |
| Pairing | ~15 | ~20 | 75% |
| Judge | ~10 | ~20 | 50% |
| Results/Break | ~8 | ~15 | 53% |
| Display/Labels | ~6 | ~15 | 40% |
| Ballot Display | ~4 | ~10 | 40% |
| Financial | ~8 | ~20 | 40% |
| Congress | ~0-10 | ~15 | TBD |
| Online/Hybrid | 0 | ~15 | 0% |
| **Total** | **~80-90** | **~180** | **~50%** |

**~80-90 settings are Release 1 critical** out of the ~180 most-used settings (from the top 3 scopes). The remaining ~390 tags across all 19 scopes include many that are deprecated, niche, or NSDA-specific.
