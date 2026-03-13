# Tabroom Business Rules Catalog

**Phase 2 Requirements Artifact -- Tabroom Rebuild Project**
**Generated:** 2026-03-13
**Status:** Draft -- Requires Expert Review

---

## Table of Contents

1. [Pairing Logic](#1-pairing-logic)
2. [Judge Assignment Logic](#2-judge-assignment-logic)
3. [Tiebreak Logic](#3-tiebreak-logic)
4. [Break/Advancement Logic](#4-breakadvancement-logic)
5. [Ballot Validation Logic](#5-ballot-validation-logic)
6. [Sweepstakes Logic](#6-sweepstakes-logic)
7. [Judge Obligation / Burden Logic](#7-judge-obligation--burden-logic)
8. [Conflict/Strike Application Logic](#8-conflictstrike-application-logic)

---

## 1. Pairing Logic

### 1.1 Debate Powermatching -- Penalty-Based Optimization

- **Rule name:** Debate Powermatch Opponent Scoring
- **Description:** Assigns numeric penalty scores to every possible opponent pairing in a bracket, then uses greedy matching with iterative swap optimization to minimize total penalty cost.
- **Trigger:** When a powermatched (highlow or highhigh) debate round is paired.
- **Inputs:**
  - Entry win/loss records from previous rounds
  - Seed position of each entry (computed from `order_entries.mas` tiebreakers)
  - Opponent history (who has debated whom)
  - Side history (which side each entry was on last)
  - School/region affiliations
  - Hybrid school strike records
  - Pullup history (how many times each entry has been pulled up)
  - Event settings: `pullup_method`, `pullup_repeat`, `powermatch`, `school_debates_self`, `no_side_constraints`, `bracket_by_ballots`, `pullup_minimize`, `hybrids_can_hit`, `region_constrain`, `prevent_hitting_pullup_twice`
- **Logic:**

  **Penalty weights (from `pair_powered.mas`):**
  | Constraint | Standard Penalty | With `pullup_minimize` |
  |---|---|---|
  | Same school | 1,000,000,000,000,000,000 (1e18) | Same |
  | Hit before | 1,000,000,000,000 (1e12) per occurrence | Same |
  | Wrong side | 100,000,000,000 (1e11) | 100,000,000 (1e8) |
  | Pullup | 10,000 | 100,000,000,000,000 (1e14) |
  | Hit pullup again | 1,000,000,000 (1e9) | Same |

  **Bracket formation:** Entries grouped by win record (`entry_record`). Within each bracket, entries sorted by SOP (seed + average opponent seed) or by seed alone depending on `powermatch` setting.

  **Pullup selection methods** (controlled by `pullup_method` setting):
  - `sop` (default): Higher pullup score = harder to pull up. Score = `(num_entries * 2) - opp_seed`. Entries already pulled up get doubled score (unless `pullup_repeat` enabled).
  - `lowseed`: Score = `(num_entries * 2) - entry_seed`. APDA-specific method.
  - `oppwin` / `middle`: Score based on opponent win totals.

  **Pullup penalty formula:**
  ```
  pullup_penalty += (abs(opponent_record - entry_tier) ^ 8) * pullup_penalty_base
  pullup_penalty += pullup_penalty_base * pullup_score_of_entry
  ```
  The exponent of 8 makes multi-bracket pullups extremely expensive.

  **Position penalty (within bracket):**
  ```
  position_penalty = (worst_position - opponent_position) * ((num_entries - rank)^3) * 0.01
  ```

  **Matching algorithm:**
  1. Greedy: iterate entries by record (desc) then position. For each unmatched entry, pick the opponent with lowest bidirectional penalty score.
  2. Swap optimization: up to 100 passes. For each entry, test swapping opponents with every other entry. Accept the swap if `old_score - new_score > 0`. Stop when no swaps improve the total.

  **Highhigh mode:** When `round->type eq "highhigh"`, within-bracket ordering is reversed so highest-seeded entries face each other.

- **Outputs/effects:** Creates `Panel` and `Ballot` records with side assignments and bracket values.
- **Format applicability:** Debate
- **Source files:** `/home/jeloni/tabroom/web/panel/round/pair_powered.mas`
- **Confidence:** High
- **Expert review needed:** Yes -- the exponent values (8, 3) and penalty magnitudes are critical tuning parameters.

---

### 1.2 Debate Preset Pairing

- **Rule name:** Preset Round Pairing with Side Constraints
- **Description:** Pairs prelim/preset rounds using a constraint-based approach with sidelocking.
- **Trigger:** When a prelim or preset debate round is paired.
- **Inputs:**
  - Entry records, ballot history, school/region affiliations
  - Event settings: `sidelock_against`, `no_side_constraints`, `nsda_district`, `round_robin`, `ncfl`
  - Prior round side assignments
- **Logic:**

  **Side constraint determination (from `pair_debate.mas`):**
  - Side constraints activate on odd-numbered previous rounds (`round_count % 2`)
  - Disabled by `no_side_constraints` setting or `sidelock_against` = "NONE" or "RANDOM"
  - When constrained: if last round entry was side 1 (aff), they are due side 2 (neg), and vice versa

  **Sidelocking against specific rounds:** The `sidelock_against` setting allows constraining sides relative to a specific other round rather than the immediately previous one.

  **NSDA District override:** If tournament is NSDA district, not round robin, and round > 2, the round type is forced to "highlow" (powermatched) and the user is redirected.

  **NCFL override:** If tournament setting is `ncfl`, both `region_constrain` and `region_avoid` are enabled automatically.

- **Outputs/effects:** Calls into `pair_powered.mas` or generates pairings directly.
- **Format applicability:** Debate
- **Source files:** `/home/jeloni/tabroom/web/panel/round/pair_debate.mas`, `/home/jeloni/tabroom/web/panel/round/pair_preset.mas`
- **Confidence:** High
- **Expert review needed:** Yes -- the NSDA district round-2 cutoff and NCFL auto-region behavior.

---

### 1.3 Side Assignment in Powermatched Rounds

- **Rule name:** Side Assignment via Snake or Constraint
- **Description:** Determines which entry gets which side in each debate.
- **Trigger:** After opponents are matched in a powermatched round.
- **Inputs:** Side due history, side constraints flag, seed order.
- **Logic:**
  1. If side constraints are active and an entry has a `side_due`, assign that side.
  2. If one entry has `side_due` and the other does not, the other gets the opposite.
  3. If neither has `side_due`, use a snaking counter (`sides++ % 2 + 1`) based on seed order to distribute sides evenly.
  4. If both entries are on the same side, force the opponent to the other side.
  5. BYE goes to the "short" side (whichever has fewer entries due) in side-constrained rounds.
- **Outputs/effects:** `Ballot.side` values (1 = aff, 2 = neg).
- **Format applicability:** Debate
- **Source files:** `/home/jeloni/tabroom/web/panel/round/pair_powered.mas` (lines 763-842)
- **Confidence:** High
- **Expert review needed:** No

---

### 1.4 Bye Assignment

- **Rule name:** Bye Assignment for Odd-Numbered Fields
- **Description:** Handles odd numbers of entries by inserting a BYE as a pseudo-entry.
- **Trigger:** When `num_entries % 2` is odd.
- **Inputs:** Entry bye history, entry records, side due counts.
- **Logic:**
  - BYE is placed in the worst bracket (record = 0).
  - BYE position is set to 1 (top of bracket, lowest seed).
  - BYE pullup score is set to `num_entries * 1e18` (effectively never pulled up).
  - BYE side is set to the "short" side if there's an imbalance.
  - Entries who have had more byes previously are sorted to avoid receiving another.
  - `met_before{entry}{"BYE"}` is set from `entry_byes` to penalize double-byes.
- **Outputs/effects:** One panel marked `bye => 1`.
- **Format applicability:** Debate
- **Source files:** `/home/jeloni/tabroom/web/panel/round/pair_powered.mas` (lines 369-406)
- **Confidence:** High
- **Expert review needed:** No

---

### 1.5 Auto-Bye for Insufficient Judges

- **Rule name:** Autobye When Judge Count Insufficient
- **Description:** Automatically assigns byes to entries when there are not enough judges to cover all debates.
- **Trigger:** When `autobye_nojudge` event setting is enabled and `available_judges * num_flights < num_debates`.
- **Inputs:** Available judge count, number of flights, number of entries, prior bye counts.
- **Logic:**
  - Calculate `num_debates = floor(num_entries / (2 * num_judges))`
  - If `num_debates > available_judges * num_flights`, remove entries with fewest prior byes from the pairing pool (they get byes)
  - Entries are shuffled first, then sorted by bye count (ascending), so those with most byes are most likely to debate
- **Outputs/effects:** Excluded entries receive bye panels.
- **Format applicability:** Debate
- **Source files:** `/home/jeloni/tabroom/web/panel/round/pair_powered.mas` (lines 132-179)
- **Confidence:** High
- **Expert review needed:** No

---

### 1.6 Speech Section Placement

- **Rule name:** Speech Section Placement with Penalty Scoring
- **Description:** Assigns speech entries to sections (panels) to minimize same-school, same-region, and repeat-opponent conflicts.
- **Trigger:** When a speech round is paired.
- **Inputs:**
  - Entry school, region, state affiliations
  - Prior section assignments (who has been in which section)
  - Speaker order history
  - Tournament type flags: `nsda_district`, `nsda_nats`, `ncfl`, `region_constrain`, `region_avoid`
  - Seed basis (if seeded pairing)
  - `school_percent_limit` setting
- **Logic:**

  **Penalty weights (from `pair_speech.mas`, vary by tournament type):**

  | Context | School | Region | District | Repeat Entry | Repeat School | Speaker Order | Title |
  |---|---|---|---|---|---|---|---|
  | Standard | 1,000,000 | (if enabled) 1,000,000 or 100,000 | -- | 1,000 | 10 | -- | -- |
  | NSDA District | 1,000,000,000 | -- | -- | 100 | 10 | -- | 10,000 |
  | NSDA Nationals | 100,000 | 100,000 | 100,000 | 100,000 | -- | 100 | -- |

  **Section scoring function `score_section()`:**
  - For each pair of entries in a section, accumulates penalties for: same school (doubled if already hit own school), same region (doubled similarly), same state, repeat opponent, same speaker order positions
  - If `seed_basis` is set, penalizes sections whose average seed deviates from the global seed average

  **Placement algorithm:**
  1. Initial placement: iterate entries (sorted by school size descending for large schools). For each entry, test all sections; pick the one with lowest incremental score. Use round-robin key rotation.
  2. `school_percent_limit`: If set, forbids schools with more entries than `(num_sections * limit%)` from being placed in randomly selected forbidden sections.
  3. Swap optimization: 7 passes. For each section with score > 0, for each entry, try swapping with every entry in every other section. Accept if total score decreases.

- **Outputs/effects:** Creates Panel and Ballot records with `speakerorder` assignments.
- **Format applicability:** Speech
- **Source files:** `/home/jeloni/tabroom/web/panel/round/pair_speech.mas`
- **Confidence:** High
- **Expert review needed:** Yes -- the NSDA Nationals penalty weights and district-specific title handling.

---

### 1.7 Speech Speaker Order Assignment

- **Rule name:** Speaker Order Optimization
- **Description:** Assigns speaker order within sections to avoid repeated positions.
- **Trigger:** After section placement is complete.
- **Inputs:** Prior speaker orders for each entry, double-entry constraints.
- **Logic:**
  1. Sort entries by total prior speaker order sum (descending -- those who have spoken late get placed early).
  2. For up to 15 iterations: if an entry has already spoken in their assigned position in a prior round, randomly relocate them.
  3. If an entry spoke in the same position in the immediately previous round, definitely move them.
  4. Double-entry handling: if a student is in another event at the same timeslot, position them early (position 1) if `speaker_priority_first` is set, or late (position 7) otherwise, based on the other event's speaker order.
- **Outputs/effects:** `Ballot.speakerorder` values.
- **Format applicability:** Speech
- **Source files:** `/home/jeloni/tabroom/web/panel/round/pair_speech.mas` (lines 712-828)
- **Confidence:** High
- **Expert review needed:** No

---

### 1.8 Speech Snake Pairing (Elims/Seeded)

- **Rule name:** Snake Algorithm for Speech Elims
- **Description:** Seeds entries into sections using a snake (fold) pattern, then optimizes for school conflicts.
- **Trigger:** When a speech elim/snaked round is created via `snake_speech.mas`.
- **Inputs:** Entry seeds (from results or pairing_seed setting), school/region affiliations, prior opponent history.
- **Logic:**

  **Snake pattern:**
  1. Sort entries by seed.
  2. Alternating direction: first row goes panel 1,2,3,...N; second row goes N,...,3,2,1; and so on (standard fold/snake).
  3. For prelim rounds: resolve school conflicts by finding the nearest-seed entry in a different panel with no school conflict, then performing a "swap train" -- shifting all entries between the conflict and the swap target by one position.

  **Scoring weights (from `snake_speech.mas`):**
  | Constraint | Penalty |
  |---|---|
  | Same school | 1,000,000 |
  | Same region | 100,000 (if regions enabled) or 1,000 |
  | Average seed deviation | 1,000 |
  | Last-round hit | 100 |
  | Prior hit | 10 |

  **Post-snake optimization:** 4 passes of pairwise swap testing. Only swaps entries within 2 seed positions of each other.

- **Outputs/effects:** Panel and Ballot records with speaker order.
- **Format applicability:** Speech
- **Source files:** `/home/jeloni/tabroom/web/panel/round/snake_speech.mas`
- **Confidence:** High
- **Expert review needed:** No

---

### 1.9 Congress Chamber Assignment

- **Rule name:** Congress Chamber Placement
- **Description:** Assigns congress entries to chambers (panels) minimizing school, state, region, and legislation authorship conflicts.
- **Trigger:** When congress round is paired with `congress_method` = "wipe" (full reassignment).
- **Inputs:**
  - Entry school, state, region, district affiliations
  - Legislation authorship
  - Bill topics
  - Seed values, autoqual status
  - Student last names (for same-name avoidance)
  - NSDA house bloc assignments
  - `school_percent_limit` setting
- **Logic:**

  **Penalty weights (from `pair_congress.mas`):**

  | Context | School | State | Region | District | Bill Topic | Author | Autoqual | Name |
  |---|---|---|---|---|---|---|---|---|
  | Standard | 1e12 | 1e8 | (if regions) 1e9 | -- | 1e6 | 10,000 | -- | 1 |
  | NSDA Nationals | -- | 1e8 | 1e6 | 100,033 | -- | 10,000 | 100 | 1 |
  | NSDA District (house) | 1e12 | 1e8 | -- | -- | -- | -- | -- | 1 |

  **NCFL override:** Region penalty set equal to school penalty.

  **NSDA District house chambers:** When `nsda_district` and `house_chambers == num_panels`, a `bloc_school` penalty of 1e6 is applied. This *reduces* the penalty for same-school entries that share a bloc assignment (they are deliberately placed together).

  **Chamber scoring function `score_chamber()`:**
  - Pairwise comparisons with penalty caching
  - Same school check (with bloc exception for districts)
  - State, region, district constraint checks
  - Same last name check
  - Both authors check
  - Both autoqual check
  - Same bill topic check

  **Optimization:** 5 passes of pairwise swap optimization across chambers.

  **Seed balancing (if `seed_presets` enabled):** After constraint optimization, 5 additional passes attempt to equalize total seed sums across chambers by swapping entries that don't worsen constraint scores.

  **Chained rounds (realign/single):** Congress sessions can be "chained" -- multiple rounds share the same chamber assignments. The `realign` method copies chambers from the first round in the chain to all tied rounds.

- **Outputs/effects:** Panel records with chamber assignments, ballots with speaker order, and `congress_recency` updates.
- **Format applicability:** Congress
- **Source files:** `/home/jeloni/tabroom/web/panel/round/pair_congress.mas`
- **Confidence:** High
- **Expert review needed:** Yes -- the NSDA district bloc_school negative penalty and NCFL region override.

---

### 1.10 Repeat Matchup Avoidance

- **Rule name:** Repeat Matchup Prevention
- **Description:** Penalizes pairing entries who have already debated each other.
- **Trigger:** During powermatch opponent scoring.
- **Inputs:** `met_before` hash built from all prior rounds' ballot records.
- **Logic:**
  - `opponent_score += hit_before_penalty (1e12) * met_before_count`
  - Multiple meetings are multiplicatively penalized
  - In speech: `hits{scores}{repeat}` penalty (values vary: 1000 standard, 100000 at NSDA Nationals, 100 at NSDA Districts)
- **Outputs/effects:** Higher penalty scores for repeat matchups make them less likely to be selected.
- **Format applicability:** All
- **Source files:** `/home/jeloni/tabroom/web/panel/round/pair_powered.mas` (lines 434-440), `pair_speech.mas`
- **Confidence:** High
- **Expert review needed:** No

---

## 2. Judge Assignment Logic

### 2.1 Debate Judge Assignment -- Multi-Dimensional Scoring

- **Rule name:** Judge Panel Score Computation
- **Description:** Scores every judge-panel combination on multiple dimensions, then assigns judges greedily by panel priority.
- **Trigger:** When automated judge assignment is run for a debate round.
- **Inputs:**
  - Judge pool (from category or jpool), with school/region/area affiliations
  - Entry school/region/area affiliations per panel
  - Strike records (conflict, entry, event, time, hybrid, region, school, dioregion, elim types)
  - MJP ratings (tier-based or ordinal/percentile)
  - Tab ratings, coach ratings
  - Judge use statistics (rounds judged, obligation, percentage)
  - Category settings: `mutuality`, `preference`, `default_mjp`, `diverse_judge_weight`, `sucktastic_judge_weight`, `round_burn_avoid`, `mjp_prefer_hireds`, `mjp_meatspace`
  - Event settings: `region_avoid`, `region_constrain`, `allow_judge_own`, `allow_repeat_judging`, `allow_repeat_elims`, `online_hybrid`, `best_judges_highest_seed`, `invert_ratings`, `no_prefs`, `nobreak_noprefs`, `max_pref`, `max_nobreak_pref`
- **Logic:**

  **Default weight values (from `debate_judge_assign.mhtml`):**
  | Parameter | Default Value |
  |---|---|
  | `mutuality` | 40 |
  | `preference` | 15 (100 if `invert_ratings`) |
  | `default_mjp` | 2 |
  | `diversity` | 1 |
  | `suckage` (bad-judge weight) | 3 |
  | `round_burn_avoid` | 3 |
  | `prefer_hireds` | 10 |
  | `meatspace` (online/hybrid mismatch) | 2 |

  **Per-judge-per-panel score computation:**
  ```
  score = 0
  For each entry in panel:
    score += 50000 if judge has "avoid" (prior judging)
    score += 100000000 if judge has hard strike against entry
    score += 100000000 if judge struck from entry's event
    score += 100/10 if region_avoid and same region/area
    score += 5000000 if region_constrain and same region
    score += 5000000 if NCFL and same dioregion
    score += mjp_rating * preference (per entry)
    score += default_mjp * preference if no rating
    score += 500000 if ordinal prefs and judge has "avoid"
    score += 500000 if mjp_rating > max_pref or max_nobreak_pref

  score += abs(mjp_diff_between_entries) * mutuality * caring_quota
  score += roundcount * 0.1
  score += use_priority (obligation-based)
  score *= bracket_score (higher brackets get better judges)
  score *= seed_score (in prelims, higher seeds get better judges)
  ```

  **Caring quota:** For brackets where entries cannot clear (based on `break_point` and round number), preference weight is reduced to 1% (`caring_quota = 0.01`).

  **Use priority (obligation balancing):**
  - `round_diff = remaining_prelims * judge_use_percentage / 100`
  - If positive: `use_priority -= round_diff ^ round_burn_avoid`
  - Judges who have judged more get higher use_priority (less likely to be assigned)
  - At NSDA Nationals: enormous multipliers (10000x, 100x) on use counts

  **Judge selection:** After scoring, judges are assigned greedily -- panels are processed in bracket order (highest first), and for each panel the lowest-scoring (best) available judge is assigned.

- **Outputs/effects:** `Ballot.judge` assignments.
- **Format applicability:** Debate (speech uses a different simpler assignment in `pair_speech.mas`)
- **Source files:** `/home/jeloni/tabroom/web/panel/round/debate_judge_assign.mhtml`
- **Confidence:** High
- **Expert review needed:** Yes -- the interaction of bracket_score multiplication with other factors, and the NCFL elim rating bonus (A=50000, B=2500).

---

### 2.2 Strike/Conflict Enforcement in Judge Assignment

- **Rule name:** Hard and Soft Strikes During Judge Assignment
- **Description:** Applies different types of strikes as either hard conflicts or soft penalties.
- **Trigger:** During judge assignment scoring.
- **Inputs:** Strike records from the `strike` table.
- **Logic:**

  **Hard conflicts (score += 100,000,000):**
  - `conflict` or `entry` type strikes: judge cannot see entry (or any entry from strike's school)
  - MJP tier marked as `strike` or `conflict`: treated as hard strike
  - Same school as entry (unless `allow_judge_own`)
  - Same region as entry (if `region_constrain` or NCFL)
  - Prior judging of same entry (unless `allow_repeat_judging`)
  - Event-level strikes

  **Soft penalties:**
  - Prior judging: score += 50,000 ("avoid")
  - Region avoid: score += 100 (region match), score += 10 (area match)
  - Prior same-side judging allowed if `allow_repeat_prelim_side` and different side

  **Time/departure strikes:** Checked against round timeslot. If strike period overlaps round, judge is marked "out" (unavailable).

  **Hybrid strikes:** When `type eq "hybrid"`, all judges from the struck school get conflict entries for the hybrid entry.

- **Outputs/effects:** Judges marked as conflicted or penalized, affecting assignment scoring.
- **Format applicability:** All
- **Source files:** `/home/jeloni/tabroom/web/panel/round/debate_judge_assign.mhtml` (lines 514-593)
- **Confidence:** High
- **Expert review needed:** No

---

## 3. Tiebreak Logic

### 3.1 Entry Tiebreak System (Debate/General)

- **Rule name:** Configurable Multi-Tier Tiebreak Resolution
- **Description:** Computes composite tiebreaker scores for entries based on a configurable protocol with prioritized tiebreaker tiers.
- **Trigger:** When results are computed for any round (via `order_entries.mas`).
- **Inputs:**
  - Protocol (tiebreaker set) with ordered tiebreakers, each having: name, priority, count (round scope), multiplier, highlow, highlow_count, highlow_threshold, chair flag, violation penalty, truncate, truncate_smallest, child (for composites)
  - Score records: winloss, point, rank, refute (WSDC reply), ballot, po
  - Ballot records: forfeit, bye, TV flags
  - Event settings: `team_points`, `round_robin`, `show_averages`, `online_mode`
- **Logic:**

  **Supported tiebreaker types (from `order_entries.mas`):**

  | Name | Direction | Description |
  |---|---|---|
  | `winloss` | down (higher better) | Win/loss record |
  | `ballots` | down | Individual ballot wins |
  | `points` | down | Speaker points (or team points) |
  | `ranks` | up (lower better) | Judge ranks |
  | `reciprocals` | down | 1/rank values summed |
  | `opp_wins` | down | Average opponent win record |
  | `opp_ballots` | down | Average opponent ballot count |
  | `opp_seed` | down | Average opponent seed |
  | `opp_points` | down | Average opponent speaker points |
  | `opp_ranks` | up | Average opponent ranks |
  | `judgevar` | down | Z-score adjusted points (standardized by each judge's mean/stddev, then rescaled to tournament mean/stddev) |
  | `judgevar2` | down | Z-score adjusted points using per-debater sample stats |
  | `head2head` | -- | Head-to-head record between tied entries |
  | `coinflip` | up | Deterministic pseudo-random: `(entry_id * seed_epoch) mod 10^5` |
  | `child` (composite) | varies | References another Protocol for sub-computation |
  | `result` | varies | Uses opponent results from a separate protocol |

  **Round scoping (`count` field):**
  - `all`: all rounds
  - `prelim`: prelim rounds only (includes highlow, highhigh types)
  - `elim`: elim rounds only
  - `previous` / `last_elim`: only the most recent elim
  - `specific`: a named round number
  - `before_end`: N rounds before the last round

  **High/low score dropping:**
  - `highlow = 1`: drop both best and worst (count = highlow_count each)
  - `highlow = 3`: drop best only
  - `highlow = 4`: drop worst only
  - For ranks: "best" = lowest rank (shifted out), "worst" = highest rank (popped off)
  - For points: "best" = highest (popped off), "worst" = lowest (shifted out)
  - `highlow_threshold`: minimum number of ballots required before dropping applies

  **Truncation:**
  - Hard cap: `truncate` -- ranks above this value are capped to it
  - Floating cap: `truncate_smallest` -- ranks capped to the smallest section size for that round
  - Rank cannot exceed section size in speech/congress

  **Composite tiebreakers:** A tiebreaker can reference a `child` Protocol. The system recursively calls `order_entries.mas` with the child protocol to compute sub-rankings. Composites of composites are explicitly blocked to prevent infinite loops.

  **Multiplier:** Each tiebreaker tier's total is multiplied by `tb.multiplier`.

  **Final sorting:** Tiers are sorted in reverse priority order. Within each tier, entries are sorted by their computed total (ascending for "up" direction, descending for "down").

  **Tie detection:** Entries with identical tier strings across all tiebreakers receive the same place number.

  **Format-specific behaviors:**
  - `wsdc` / `mock_trial` / `wudc`: treated as "debate" type
  - `speech` / `congress`: ranks cannot exceed panel entry count
  - DQ handling: in speech/congress, DQ'd entries' rank thresholds adjust other entries' ranks downward

  **Forfeit handling:** If `forfeits_never_break` protocol setting is enabled, forfeited entries are sorted to the bottom.

- **Outputs/effects:** Returns ordered entry arrays by place, tiebreaker values per entry per tier, tier descriptions, forfeit flags, and tier directions.
- **Format applicability:** All
- **Source files:** `/home/jeloni/tabroom/web/tabbing/results/order_entries.mas`
- **Confidence:** High
- **Expert review needed:** Yes -- the composite tiebreaker recursion, judgevar formula, and multiplier interactions.

---

### 3.2 Speaker Tiebreak System

- **Rule name:** Individual Speaker Awards Tiebreaking
- **Description:** Computes speaker-level (rather than entry-level) tiebreakers for individual speaker awards.
- **Trigger:** When speaker awards are computed via `order_speakers.mas`.
- **Inputs:** Same as entry tiebreakers but computed per student, not per entry. Additional settings: `speaker_max_scores`, `speaker_min_speeches`, `speakers_rubric`, `include_wsdc_reply`.
- **Logic:**

  **Key differences from entry tiebreakers:**
  - Operates on `student` entities rather than `entry` entities
  - `speaker_max_scores`: caps the number of scores counted (drops excess, keeping best)
  - `speaker_min_speeches`: entries with too many missed speeches are demoted
  - `mavericks` setting: if "nothing", unspeaking competitors get 0 points instead of being excluded
  - `judgepref` tiebreaker: when exactly 2 entries are tied on all other tiebreakers, breaks the tie by counting how many judges ranked one above the other
  - `rankinround`: ranks each entry within their section based on overall results, then sums those section-ranks
  - `judgevar`: `((score - judge_avg) / judge_stddev) * total_stddev + total_avg`
  - `judgevar2`: `((score - judge_avg) / judge_stddev) * judge_sample_stddev + judge_sample_avg`

  **Elim filtering:** In elim/final rounds, only students who actually have ballots in that round are included.

  **Bye handling in round robin:** `bye_min` tracks the minimum byes any student has; used for normalization.

- **Outputs/effects:** Ordered student arrays by place, with tiebreaker values.
- **Format applicability:** All (primarily speech and congress for individual awards)
- **Source files:** `/home/jeloni/tabroom/web/tabbing/results/order_speakers.mas`
- **Confidence:** High
- **Expert review needed:** Yes -- the `judgepref` mutual preference tiebreaker and `judgevar2` formula.

---

## 4. Break/Advancement Logic

### 4.1 Debate Break and Bracket Construction

- **Rule name:** Debate Elim Break
- **Description:** Advances entries from prelims (or prior elims) into elimination rounds, constructing proper brackets.
- **Trigger:** When an admin creates a new elim round from a source round.
- **Inputs:**
  - Source round results (from `order_entries.mas`)
  - Start/end seed range
  - `no_elims` entry setting (ineligible entries)
  - `school_debates_self` event setting
  - `double_elimination` event setting
- **Logic:**

  **Prelim-to-elim break:**
  1. Compute seedings using `order_entries.mas` on the source round.
  2. Select entries from `start` seed through `end` seed, skipping entries with `no_elims` setting.
  3. Compute `target_bracket` = next power of 2 >= number of breaking entries.
  4. Pair using standard bracket: seed 1 vs seed N, seed 2 vs seed N-1, etc. where N = `target_bracket + 1 - seed`.
  5. Entries without an opponent in the bracket receive a bye.

  **Elim-to-elim break:**
  1. Winners advance: entries with `winloss = 1` in the previous elim.
  2. Bracket position inheritance: winners keep their panel's bracket number.
  3. The new bracket target is calculated from the number of winners.

  **Double elimination:**
  1. Entries with 0 losses advance to winners bracket (bracket from panel).
  2. Entries with 1 loss advance to losers bracket.
  3. Losers' bracket seed is inverted: `(num_winners * 2 + 1) - seed` for top-half seeds.
  4. Losers' bracket target: `(first_elim_target / 2^exponent) * multiplier + 1` where multiplier alternates between 2 and 1.5 for successive rounds.
  5. Repeat detection: if a losers' bracket pairing would repeat a prior elim matchup, brackets are flipped by offsetting seeds by +/-1.
  6. When only 2 entries remain, it becomes a final.

  **Same-school bypass in elims:** If two entries from the same school are paired, the higher seed automatically wins (bye assigned with winloss scores).

  **Side assignment in elims:** Uses `/funclib/round_elim_dueaff.mas` to determine which entry is due aff, and swaps sides if needed.

  **NSDA District special case:** When exactly 4 entries remain and 3 qualify, winners get byes and losers debate each other (bracket seed 3).

- **Outputs/effects:** New Round, Panel, Ballot, and ResultSet records. Result entries with seed/place values in the Bracket result set.
- **Format applicability:** Debate
- **Source files:** `/home/jeloni/tabroom/web/tabbing/break/break_debate.mhtml`
- **Confidence:** High
- **Expert review needed:** Yes -- the double elimination bracket math and NSDA district 4-to-3 logic.

---

### 4.2 Speech Break and Elim Sectioning

- **Rule name:** Speech Elim Break
- **Description:** Advances top-seeded speech entries into elim or final rounds with snake sectioning.
- **Trigger:** When an admin creates a speech elim round.
- **Inputs:** Source round results, start/end range, number of panels, `elim_method` event setting, school/region/district affiliations, prior opponent history.
- **Logic:**

  **Snake methods (from `break_speech.mhtml`):**
  - Standard snake: entries sorted by seed, alternately filling panels forward then backward.
  - `snake_school`: after snake, swap entries (within 2 seed positions) to resolve school conflicts.
  - `snake_school_rank` / `snake_school_prelim_cume` / `snake_school_overall_cume`: swap only within same tiebreaker value.
  - `snake_school_force`: more aggressive swapping.
  - `ky_semis_snake`: reverses every 3rd row instead of every 2nd.
  - `nsda_snake`: full scoring with school (1e7), region (1e6 in early rounds, 0 in round 9+), top_seed (1e5), average (1e3), last_hit (1e3), hit (10).

  **Post-snake optimization:** 2 passes of pairwise swap testing using the same `score_panel()` function as `snake_speech.mas`.

  **Speaker order in elims:**
  - Sort by cumulative prior speaker order sum (descending).
  - For up to 10 iterations, relocate entries who have spoken in their assigned position.
  - NSDA Nationals finals: fully random order.
  - NCFL: no speaker order optimization.

  **NCFL elim handling:** Delegates to `ncfl_snake.mhtml` for a separate algorithm.

  **Judge defaults:** Speech elims default to 3 judges; NCFL finals default to 5.

- **Outputs/effects:** Panel and Ballot records with speaker order.
- **Format applicability:** Speech
- **Source files:** `/home/jeloni/tabroom/web/tabbing/break/break_speech.mhtml`
- **Confidence:** High
- **Expert review needed:** Yes -- the NSDA snake scoring weights and the ky_semis_snake variant.

---

## 5. Ballot Validation Logic

### 5.1 Score Range Validation

- **Rule name:** Speaker Point Range Enforcement
- **Description:** Validates that entered speaker points fall within the configured range and increment.
- **Trigger:** When a ballot is saved (both judge online entry and tab entry).
- **Inputs:**
  - `min_points` event setting (default: 0)
  - `max_points` event setting (default: 30)
  - `point_increments` event setting: "whole", "tenths", "half"
  - `point_ties` event setting
- **Logic:**
  - Points must be numeric
  - Points must fall within `[min_points, max_points]`
  - Rounding applied based on increment type:
    - `whole`: `int(points + 0.5)`
    - `tenths`: `round(points * 10) / 10`
    - `half`: `round(points * 2) / 2`
- **Outputs/effects:** Error message returned if validation fails; ballot not saved.
- **Format applicability:** All
- **Source files:** `/home/jeloni/tabroom/web/tabbing/entry/ballots/debate_save.mhtml` (lines 80-85, 155-185), `/home/jeloni/tabroom/web/user/judge/ballot_save.mhtml`
- **Confidence:** High
- **Expert review needed:** No

---

### 5.2 Low-Point Win Prevention

- **Rule name:** Low-Point Win (LPW) Detection
- **Description:** Prevents or warns when a winning entry has lower speaker points than the losing entry.
- **Trigger:** During ballot save.
- **Inputs:**
  - `no_lpw` event setting
  - `allow_lowpoints` event setting (overrides `no_lpw`)
  - Event type (speech/congress always have `no_lpw`)
- **Logic:**
  - `no_lpw` is set to true by default for speech and congress events
  - `allow_lowpoints` explicitly overrides and permits LPW
  - For debate: if `no_lpw` is set and the winner's points < loser's points, an error is raised
  - In tab entry (`debate_save.mhtml`): validates that higher-ranked entries have higher or equal points
    ```
    if rank_A < rank_B, then points_A >= points_B (or error)
    ```
- **Outputs/effects:** Error message; ballot is not saved if LPW detected and disallowed.
- **Format applicability:** All (always on for speech/congress, configurable for debate)
- **Source files:** `/home/jeloni/tabroom/web/user/judge/ballot_save.mhtml` (lines 97-99), `/home/jeloni/tabroom/web/tabbing/entry/ballots/debate_save.mhtml` (lines 130-136)
- **Confidence:** High
- **Expert review needed:** No

---

### 5.3 Rank Validation

- **Rule name:** Rank Uniqueness and Range
- **Description:** Validates that ranks are unique, numeric, and within range.
- **Trigger:** During ballot save for speech/congress events with rank tiebreakers.
- **Inputs:** Number of entries in the section, submitted ranks.
- **Logic:**
  - Ranks must be numeric
  - Ranks must be unique (no ties in ranks)
  - Ranks must be in range `[1, num_entries_in_section]`
  - Rank order must be consistent with points (higher-ranked entries must have >= points)
- **Outputs/effects:** Error messages; ballot not saved if validation fails.
- **Format applicability:** Speech, Congress
- **Source files:** `/home/jeloni/tabroom/web/tabbing/entry/ballots/debate_save.mhtml` (lines 101-137)
- **Confidence:** High
- **Expert review needed:** No

---

### 5.4 Double-Entry Ballot Verification (Audit System)

- **Rule name:** Ballot Audit Flag
- **Description:** Tracks whether a ballot has been confirmed/audited.
- **Trigger:** On ballot creation and during scoring.
- **Inputs:** `ballot.audit` field.
- **Logic:**
  - New ballots created with `audit = 0` (unconfirmed)
  - Bye ballots created with `audit = 1` (auto-confirmed)
  - Judge online entry: ballots remain `audit = 0` until tab confirms
  - Tab entry sets `audit` upon save
  - Dropped entries' ballots are auto-set to `audit = 1`
  - The audit system does not implement traditional "double entry" (two independent entries compared); rather it flags ballots as confirmed by tab staff
- **Outputs/effects:** `ballot.audit` field controls whether results are considered final.
- **Format applicability:** All
- **Source files:** `/home/jeloni/tabroom/web/user/judge/ballot_save.mhtml` (lines 34, 73-79), `/home/jeloni/tabroom/web/tabbing/entry/ballots/debate_save.mhtml`
- **Confidence:** Medium -- the actual double-entry verification mechanism may be more distributed across the codebase.
- **Expert review needed:** Yes

---

## 6. Sweepstakes Logic

### 6.1 Sweepstakes Point Calculation

- **Rule name:** Sweepstakes Points Engine
- **Description:** Computes sweepstakes points per entry based on a configurable rule set, then aggregates to schools.
- **Trigger:** When sweepstakes results are computed via `sweep_tourn.mas`.
- **Inputs:**
  - SweepSet with associated SweepRules
  - Entry scores (winloss, rank, point) across rounds
  - Entry placement/seeding from result sets
  - Event type, field size
  - Entry student count
  - Rules: `novice_only`, `multiply_entrysize`, `multiplier`, `skip_rounds`, `exclude_breakouts`
- **Logic:**

  **Supported sweepstakes rule types (from `sweep_tourn.mas`):**

  | Rule Tag | Description |
  |---|---|
  | `points_per` | Fixed points per round participated |
  | `points_per_rank` | Fixed points for achieving exact rank N |
  | `points_per_rank_above` | Fixed points for achieving rank <= N |
  | `rev_per_rank` | Reverse scoring: `place - rank` per ballot (minimum = `rev_min`) |
  | `cume` | Points if cumulative rank total equals exactly N |
  | `cume_above` | Points if cumulative rank total <= N |
  | `points_per_comp_rank` | Points based on composite (section) rank |
  | `rev_per_comp_rank` | Reverse points based on composite section rank |
  | `seed` | Points for exact seed N in final placement |
  | `seed_above` | Points for seed <= N |
  | `seed_above_percent` | Points for top N% of field |
  | `rev_seed` | Reverse seed scoring: `field_size - seed` |
  | `nsda_place` | NSDA-specific: baseline (10 for debate, 9 for speech, 9 for congress) minus place, adjusted for field size |
  | `ballot_win` / `ballot_loss` | Points per individual ballot win/loss |
  | `round_win` / `round_loss` | Points per round win/loss (majority ballots) |
  | `prorated_ballots` | 3x for sweep, 2x for split win, 1x for split loss |
  | `round_bye` | Points for bye rounds |
  | `points_per_po_round` | Points per round elected as Presiding Officer |
  | `coachover_advance` | Points for advancing from a coach-over round |
  | `manual` | Manually entered sweepstakes points per entry |
  | `minimum` | Minimum points floor for any scored entry |

  **Round scoping:** Each rule has a `count` (all, prelim, elim, specific, last_prelims, before_end) and optional `count_round` for specifics.

  **NSDA place scoring formula:**
  ```
  baseline = 10
  baseline-- if speech or congress
  unless congress:
    if max_entry > 1: baseline-- if field < 50, baseline-- if field < 30
    if max_entry == 1: baseline-- if field < 58, baseline-- if field < 38
  points = baseline - place (for places 1-6 only)
  ```

  **Post-computation adjustments:**
  - `multiply_entrysize`: multiply points by number of students on entry
  - `multiplier`: multiply all points by a fixed factor
  - `minimum`: if total < minimum, set to minimum

- **Outputs/effects:** Per-entry points hash returned to `sweep_schools.mas`.
- **Format applicability:** All
- **Source files:** `/home/jeloni/tabroom/web/tabbing/results/sweep_tourn.mas`
- **Confidence:** High
- **Expert review needed:** Yes -- the NSDA place formula and prorated_ballots logic.

---

### 6.2 School Sweepstakes Aggregation

- **Rule name:** School Sweepstakes Aggregation with Caps
- **Description:** Aggregates entry-level sweepstakes points to schools with configurable limits.
- **Trigger:** When school sweepstakes are computed via `sweep_schools.mas`.
- **Inputs:** Per-entry points from `sweep_tourn.mas`, SweepSet rules.
- **Logic:**

  **Cap/limit rules (from SweepSet):**
  | Rule | Effect |
  |---|---|
  | `entries` | Max number of entries per school that count |
  | `events` | Max number of distinct events per school that count |
  | `event_entries` | Max entries per event per school |
  | `wildcards` | Number of "extra" entries allowed beyond the entries/events cap |
  | `one_per_person` | Each student can only contribute once |
  | `by_person` | Points counted per person rather than per entry |
  | `max_entry_persons` | Max number of students per entry that contribute |
  | `set_limit` | Per-child-set limit on entries per school |
  | `set_event_limit` | Per-child-set per-event limit |

  **Hybrid entries:** Points are split 50/50 between the entry's school and the hybrid school.

  **Hierarchical sweep sets:** SweepSets can have children. Child sets are computed recursively, with their own cap rules applied before rolling up.

  **Processing order:** Entries are processed in descending point order, so highest-scoring entries are counted before caps kick in.

- **Outputs/effects:** School-level point totals with breakdowns.
- **Format applicability:** All
- **Source files:** `/home/jeloni/tabroom/web/tabbing/results/sweep_schools.mas`
- **Confidence:** High
- **Expert review needed:** No

---

## 7. Judge Obligation / Burden Logic

### 7.1 NSDA Nationals Judge Burden Calculation

- **Rule name:** NSDA Nationals Per-School Judge Burden
- **Description:** Calculates how many judge rounds a school owes based on their entries.
- **Trigger:** When checking school judge obligations.
- **Inputs:**
  - School entries per event
  - `nats_judge_burden` event setting (burden per entry)
  - `nats_jpool` event setting (which judge pool)
  - `nats_screwy_burden` event setting
  - `no_judge_burden` event setting
  - `rejected_by` entry setting (rejected entries don't count)
  - Category settings: `min_burden`, `max_burden`, `minimum_supplied`
- **Logic:**

  **Standard burden:**
  ```
  burden = entry_count * burden_per_entry
  ```

  **"Screwy burden" (NSDA special):**
  ```
  For each entry 1..N:
    if entry_number is odd: burden += full_burden
    if entry_number is even: burden += ceil(full_burden / 2)
  ```
  (Effectively: first entry pays full, second pays half, third pays full, etc.)

  **Aggregation:**
  - Burden is computed per event, then summed per jpool
  - Total burden across all jpools is ceiling-rounded
  - `min_burden`: if total > 0 and total < min, set to min
  - `max_burden`: if total > max, set to max
  - `minimum_supplied`: if school has entries but no judges in other categories, force minimum

  **Exclusions:**
  - Entries with `rejected_by` setting are excluded
  - Events with `no_judge_burden` are excluded
  - Unconfirmed entries are excluded

- **Outputs/effects:** Burden totals per jpool and overall, plus hire eligibility per jpool.
- **Format applicability:** All (NSDA Nationals specific)
- **Source files:** `/home/jeloni/tabroom/web/funclib/judgemath/nats_burden.mas`
- **Confidence:** High
- **Expert review needed:** Yes -- the "screwy burden" alternating pattern.

---

### 7.2 Judge Use Tracking / Round Burn

- **Rule name:** Judge Round Burn Avoidance
- **Description:** Tracks how many rounds each judge has been used and factors this into assignment priority.
- **Trigger:** During judge assignment.
- **Inputs:** Judge rounds judged, obligation, `rounds_per` category setting, remaining prelim count.
- **Logic:**
  - If `rounds_per` is set: judges who have judged >= their obligation are marked "out"
  - `round_diff = remaining_prelims * use_percentage / 100`
  - Priority penalty: `-(round_diff ^ round_burn_avoid)` (default round_burn_avoid = 3)
  - Newbie bonus: judges who have judged less get lower use_priority (more likely to be assigned)
  - Without `rounds_per`: simple round count * round_burn_avoid penalty
  - In elims: round_burn_avoid is reduced by 3 when min_bracket is set
- **Outputs/effects:** `use_priority` values that influence judge-panel scoring.
- **Format applicability:** All
- **Source files:** `/home/jeloni/tabroom/web/panel/round/debate_judge_assign.mhtml` (lines 900-975)
- **Confidence:** High
- **Expert review needed:** No

---

## 8. Conflict/Strike Application Logic

### 8.1 Person-Level Conflict Propagation

- **Rule name:** Chapter Conflict to Tournament Strike Propagation
- **Description:** Converts person-level conflicts (stored in the `conflict` table) into tournament-level strikes when a school registers.
- **Trigger:** When `chapter_conflicts.mas` is called (typically during school creation/registration).
- **Inputs:** School's chapter ID, tournament judges linked to persons with conflicts against that chapter.
- **Logic:**
  ```sql
  SELECT judge.* FROM judge, category, conflict
  WHERE category.tourn = [tournament]
    AND judge.category = category.id
    AND judge.person != 0
    AND judge.person = conflict.person
    AND conflict.chapter = [school's chapter]
  ```
  For each matched judge, create a Strike record:
  ```perl
  Tab::Strike->create({
    tourn      => tournament_id,
    judge      => judge_id,
    type       => "school",
    school     => school_id,
    registrant => 1
  })
  ```
- **Outputs/effects:** Strike records of type "school" linking the judge to the school.
- **Format applicability:** All
- **Source files:** `/home/jeloni/tabroom/web/funclib/chapter_conflicts.mas`
- **Confidence:** High
- **Expert review needed:** No

---

### 8.2 Strike Type Taxonomy

- **Rule name:** Strike Types and Their Application
- **Description:** Defines how different strike types are evaluated during judge assignment.
- **Trigger:** During judge assignment when strikes are loaded.
- **Inputs:** Strike records with type, judge, entry, school, region, dioregion, event, start/end times.
- **Logic:**

  **Strike types and their effects (from `debate_judge_assign.mhtml` lines 529-593):**

  | Strike Type | Scope | Effect |
  |---|---|---|
  | `conflict` | Judge vs entry (or school) | Hard conflict: score += 1e8. If school-based, applies to all entries from that school. |
  | `entry` | Judge vs specific entry | Hard conflict: score += 1e8. |
  | `school` | Judge vs school | Hard conflict against all entries from that school. |
  | `region` | Judge vs region | Hard conflict against all entries from that region. |
  | `dioregion` | Judge vs diocese region (NCFL) | Hard conflict against all entries from that diocese region. |
  | `hybrid` | School vs entry | All judges from the struck school get conflicts with the hybrid entry. |
  | `event` | Judge vs event | Judge excluded from all panels in that event. |
  | `elim` | Judge limited to prelims | Judge excluded from elim/final/runoff rounds of the struck event. |
  | `time` | Judge vs time range | If strike period overlaps the round's timeslot, judge is marked unavailable. |
  | `departure` | Judge leaving at time | Same as `time` -- if departure window overlaps round, judge unavailable. |

  **MJP tier strikes:** Rating tiers can be marked with `strike` or `conflict` flags. If a judge's MJP tier for an entry is marked as such, it becomes a hard conflict.

  **Implicit school conflicts:** Unless `allow_judge_own` is set, every judge automatically has a conflict with all entries from their own school. If `region_constrain` or `region_judge_forbid` or NCFL, judges also conflict with all entries from their region.

- **Outputs/effects:** Populated `judges{"conflicts"}` and `judges{"out"}` hashes used in scoring.
- **Format applicability:** All
- **Source files:** `/home/jeloni/tabroom/web/panel/round/debate_judge_assign.mhtml` (lines 479-593)
- **Confidence:** High
- **Expert review needed:** No

---

### 8.3 Busy Judge Detection (Cross-Event Conflicts)

- **Rule name:** Judge Availability Across Events
- **Description:** Detects when a judge is already assigned to another panel in an overlapping timeslot.
- **Trigger:** During judge assignment.
- **Inputs:** Round timeslots, existing ballot assignments, judge pools.
- **Logic:**
  - Query finds judges who have ballot assignments in any round whose timeslot overlaps the current round's timeslot.
  - Also checks across person links: if `judge.person = other_judge.person` and the other judge is busy, the current judge is marked busy.
  - Exception: async online events are excluded from overlap checks.
  - Judges in "standby" jpools for the current timeslot are excluded from the available pool.
- **Outputs/effects:** `judges{"out"}{$judge}` incremented, preventing assignment.
- **Format applicability:** All
- **Source files:** `/home/jeloni/tabroom/web/panel/round/debate_judge_assign.mhtml` (lines 618-765)
- **Confidence:** High
- **Expert review needed:** No

---

## Appendix: Key Event Settings Reference

These settings appear throughout the business rules and control major behavioral branches:

| Setting | Type | Effect |
|---|---|---|
| `powermatch` | Event | Powermatching method: "sop" (default), "seed", "highhigh" |
| `pullup_method` | Event | Pullup selection: "sop" (default), "oppwin", "middle", "lowseed" |
| `pullup_minimize` | Event | Massively increases pullup penalty, reduces wrong-side penalty |
| `pullup_repeat` | Event | Allows entries to be pulled up multiple times |
| `no_side_constraints` | Event | Disables automatic side alternation |
| `school_debates_self` | Event | Allows same-school matchups |
| `bracket_by_ballots` | Event | Forms brackets by ballot count rather than round wins |
| `region_constrain` | Event | Hard constraint: same region treated like same school |
| `region_avoid` | Event | Soft constraint: penalizes same region |
| `allow_judge_own` | Event | Allows judges to judge entries from their own school |
| `allow_repeat_judging` | Event | Allows judges to judge entries they've seen before |
| `allow_repeat_elims` | Event | Allows repeat judging in elims specifically |
| `no_lpw` | Event | Prevents low-point wins |
| `double_elimination` | Event | Enables double-elimination bracket format |
| `online_hybrid` | Event | Enables online/in-person hybrid mode |
| `best_judges_highest_seed` | Event | Prioritizes best judges for highest-seeded panels |
| `school_percent_limit` | Event | Limits school representation per section to N% |
| `seed_presets` | Event | Uses seeded prelim pairing |
| `nsda_district` | Tourn | Enables NSDA district-specific rules |
| `nsda_nats` | Tourn | Enables NSDA Nationals-specific rules |
| `ncfl` | Tourn | Enables NCFL (Catholic league) rules |

---

## Appendix: Penalty Magnitude Summary

All pairing/assignment systems use penalty-based scoring where violations of different severity have different magnitudes. The penalty hierarchy (from most to least severe) is:

**Debate pairing (pair_powered.mas):**
```
Same school:        1e18
Hit before:         1e12 (per occurrence)
Wrong side:         1e11 (or 1e8 with pullup_minimize)
Hit pullup again:   1e9
Pullup penalty:     1e4 (or 1e14 with pullup_minimize)
Position penalty:   varies (fractional)
```

**Speech pairing (pair_speech.mas):**
```
School:   1e6 (standard) / 1e5 (NSDA nats) / 1e9 (districts)
Region:   1e6 (constrain) / 1e5 (avoid/nats)
District: 1e5 (NSDA nats)
Repeat:   1e3 (standard) / 1e5 (NSDA nats) / 1e2 (districts)
Order:    1e2 (NSDA nats)
Title:    1e4 (districts)
```

**Congress pairing (pair_congress.mas):**
```
School:      1e12
Region:      1e9
State:       1e8
Bill topic:  1e6
Author:      1e4
Name:        1
```

**Judge assignment (debate_judge_assign.mhtml):**
```
Hard strike:        1e8
Event exclusion:    1e8
Region constrain:   5e6
NCFL dioregion:     5e6
Ordinal avoid:      5e5
MJP cap exceeded:   5e5
Region avoid:       100/10
Online mismatch:    2 (meatspace weight)
```
