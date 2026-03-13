# Capability Domains: Batch E

## Domain 1: Results, Breaks, Tiebreakers, and Publication

### Results, Breaks, Tiebreakers, and Publication

**Description:** Computes ranked results from raw ballot scores using configurable tiebreaker chains, manages advancement (breaks) from prelim rounds into elimination rounds, generates publishable result sets (final places, prelim seeds, speaker awards, brackets), and controls multi-tier publication of schematics, results, and feedback to coaches, competitors, and the public.

**Primary users:** Tournament directors, tab staff, coaches (view-only for published results), public (view-only for published results)

**Main entry points:**
- `/tabbing/results/index.mhtml` -- Per-event results viewer with round-by-round tiebreaker display
- `/tabbing/break/index.mhtml` -- Break/advancement interface showing ordered entries with tiebreaker columns
- `/tabbing/publish/index.mhtml` -- Final results publication manager (generate result sets, publish/unpublish, upload files)
- `/tabbing/publish/display.mhtml` -- View a generated result set (or redirect to bracket view)
- `/tabbing/publish/bracket.mhtml` -- Bracket visualization for elim result sets
- `/panel/publish/index.mhtml` -- Round-by-round schematic and score publication (primary/secondary/feedback tiers)
- `/index/results/index.mhtml` -- Public-facing tournament results search and display

**Major sub-capabilities:**

- *Tiebreaker computation engine* (`order_entries.mas`, `order_speakers.mas`): Multi-tier tiebreaker calculation supporting 30+ tiebreaker types. Entries are ranked by successive tiebreaker tiers; each tier can aggregate multiple tiebreaker values. Supports composite tiebreakers (one protocol referencing another via `child` field), high/low dropping, truncation, multipliers, chair-only vs. scorer-only filtering, and violation penalties.
- *Tiebreaker types* (from `tiebreak_name.mas`, `tiebreaks_explain.mhtml`, `tiebreak_types.mas`): winloss, ballots, losses, ranks, reciprocals, points, opp_wins, opp_points, opp_ranks, opp_seed, headtohead, judgevar, judgevar2, preponderance, downs, judgepref, chair_ranks, non_chair_ranks, seed, three_way_point, three_way_recip, 3way_pts_worst, 3way_rcp_worst, entry_vote_one, entry_vote_all, student_rank, best_po, po_points, NSDA Points, Rounds.
- *Tiebreaker configuration* (`/setup/rules/tiebreaks.mhtml`, `tiebreak_save.mhtml`, `tiebreak_edit.mas`): UI for creating protocol (tiebreaker set) objects, adding tiebreakers with priority, count scope (all/prelim/elim/specific round), high/low dropping, truncation, multiplier, composite child protocol, chair filtering, and violation penalty.
- *Protocol settings* (`tiebreak_types.mas`): Determines which score tags (winloss, rank, point, refute, entry_winloss, best_po) are needed for a given protocol by walking the tiebreaker chain including composites.
- *Result set generation* (`generate_results.mhtml`): Creates result sets of type: Final Places, Prelim Seeds, Speaker Awards, Bracket, Chamber Results, Prelims Table, NDCA Points, TOC Bids. Deletes previous result set with same label before regenerating. Each result consists of a `Result` row (rank, place, entry/student link) with associated `ResultValue` rows for each tiebreaker column plus raw ballot strings.
- *Prelim seed ordering*: Ranks entries by tiebreaker computation from last prelim round. Stores rank and all tiebreaker values as `ResultValue` rows.
- *Final places*: Iterates through final, elim rounds, then prelim in reverse order. Elim-round losers get the round label (e.g., "Quarters"). Final round entries get ordinate place (1st, 2nd, Co-Champion). Entries not in elims get "Prelim" designation.
- *Speaker awards*: Orders individual students using a separate `speaker_protocol` event setting. Stores per-student results with ballot strings.
- *Chamber results (Congress)*: Uses `congress_ties.mas` to group rounds by session lock, then ranks entries within each chamber/section.
- *Prelims table*: Generates a full JSON data blob (compressed into `result_set.cache`) containing per-round, per-panel, per-judge score detail for a full prelim results table display.
- *Bracket generation* (`generate_bracket.mas`): Creates a bracket `ResultSet` (with `bracket=1` flag) for visualization of elimination rounds.
- *Percentile calculation*: After result set generation, updates `result.percentile` as `((total - rank + 1) / total) * 100`.
- *Breakout support*: Results can be scoped to breakout subgroups (e.g., novice division within an event) using `breakout_N_label` and `breakout_N_students` event settings.
- *Break/advancement - Debate* (`break_debate.mhtml`): Single- and double-elimination bracket advancement. Determines winners/losers by win/loss tiebreaker from last elim round. Bracket seeds are carried forward; the `panel.bracket` field tracks bracket position. Handles power-of-2 bracket padding with byes. For double elimination: tracks losses count, splits winners (0 losses) and losers (1 loss) brackets, checks for repeat matchups and flips bracket if needed. Special NSDA district 4-entry/3-qualifier logic is hardcoded.
- *Break/advancement - Speech* (`break_speech.mhtml`): Advances top N entries from prelims into elim sections. Uses `equal_elims` protocol setting to draw equal numbers from each section. Supports region avoidance (NCFL snake). Creates new round with specified number of panels and distributes entries across sections.
- *Break/advancement - Congress* (`break_congress.mhtml`): Similar to speech break but specific to Congress format. Clears existing round and re-distributes.
- *Break/advancement - WUDC* (`break_wudc.mhtml`): Break for Worlds-style (British Parliamentary) debate.
- *Break menu* (`break/menu.mas`): Calculates default break sizes based on event type and remaining rounds (powers of 2 for debate, sections of 6-7 for speech, sections of 12 for congress). Special NSDA Nationals break size logic. Provides form for specifying start/end seed, round type, timeslot, protocol, and site for the new round.
- *Publication tiers*: Three independent publication levels for round results: `post_primary` (W/L or rank), `post_secondary` (points, debate rank), `post_feedback` (RFD, comments). Each has four visibility levels: 0=None, 1=Coaches, 2=Competitors+Coaches, 3=Public. Schematic publication (`round.published`) has separate levels: 0=Not Public, 1=Public, 2=Public without judges, 3=Entry List only, 5=Show entries their chambers (Congress).
- *Result set publication* (`result_set_switch.mhtml`): Toggle `published` and `coach` flags on result sets. `published` makes results publicly visible; `coach` makes them visible only to coaches (when not fully published).
- *Publish all rounds* (`publish_all_rounds.mhtml`): Bulk publish all round results and feedback.
- *Mass publish* (`panel/publish/publish_everything.mhtml`): Bulk set publication levels for schematics, primary, secondary, and feedback across events/timeslots/categories.
- *Blast results* (`blast_results.mas`): Emails debate round results to entry followers and school followers when `judge_publish_results` event setting is enabled.
- *Result file uploads* (`upload_results.mhtml`): Upload arbitrary result files (PDFs, etc.) to S3, tagged to event or tournament-wide, with publish/coach visibility toggles.
- *CSV export* (`csv.mhtml`, `results_csv.mas`, `speakers_csv.mhtml`): Export results and speaker awards to CSV format.
- *NSDA qualifiers* (`nsda_qualifiers.mhtml`): Generate district qualifier result sheets, post qualifier registrations to nationals.
- *NSDA Nationals ranks* (`nationals_ranks.mhtml`): Special ranking for NSDA Nationals events.
- *NDCA points* (delegates to `/tabbing/report/ndca/points.mhtml`): Baker, Dukes & Bailey, NDCA PF point calculations.
- *TOC bids* (delegates to `/tabbing/report/toc/post_bids.mhtml`): Generate and optionally email TOC bid reports.
- *Public results display*: `web/index/results/` provides public-facing results including circuit stats, speaker rankings, debate stats, TOC bids, NDCA standings, NDT/CEDA points, qualifiers, team lifetime records, RPI detail.

**Key entities:**
- `Tab::Result` (`web/lib/Tab/Result.pm`): Core result row. Columns: id, rank, place, result_set, entry, student, school, round, panel, timestamp, percentile. Links to entry, round, student, result_set. Has many result_values.
- `Tab::ResultSet` (`web/lib/Tab/ResultSet.pm`): Container for a set of results. Columns: id, tag, label, tourn, event, circuit, qualifier, sweep_set, sweep_award, bracket, published, coach, code, generated, timestamp, cache. Links to tourn, event, circuit, sweep_set, sweep_award. Has many results and result_keys.
- `Tab::ResultKey` (`web/lib/Tab/ResultKey.pm`): Column definition for a result set. Columns: id, result_set, tag, description, no_sort, sort_desc, timestamp. Defines a tiebreaker column (e.g., "Wins", "Points", "Ballots").
- `Tab::ResultValue` (`web/lib/Tab/ResultValue.pm`): Individual value in a result row. Columns: id, priority, value, result, result_key, protocol, timestamp. Links result to result_key; priority determines column ordering.
- `Tab::Tiebreak` (`web/lib/Tab/Tiebreak.pm`): Tiebreaker definition within a protocol. Columns: id, name, count, count_round, truncate, truncate_smallest, multiplier, violation, result, priority, highlow, highlow_count, highlow_threshold, highlow_target, child, protocol, chair, timestamp. `child` references another Protocol for composite tiebreakers.
- `Tab::Protocol` (not directly read, but central): A tiebreaker set assigned to rounds. Contains multiple Tiebreak rows. Has protocol_setting for options like `equal_elims`, `forfeits_rank_last`.

**Key settings/configuration:**
- *Protocol settings*: `equal_elims` (advance equal from each section), `forfeits_rank_last`
- *Event settings*: `speaker_protocol` (protocol for speaker awards), `bid_round` (TOC bid round number), `baker`/`dukesandbailey`/`ndca_public_forum` (NDCA point systems), `judge_publish_results` (email results to followers), `results_published`, `breakouts` (number of breakout groups), `breakout_N_label`/`breakout_N_students`/`breakout_N_delete`, `top_novice`, `honorable_mentions`, `elim_method`, `double_elimination`, `school_debates_self`, `round_robin`, `aff_label`/`neg_label`, `panel_labels`, `show_totals`, `point_increments`, `qualifier_*` (qualifier method settings), `region_avoid`, `online_mode`, `exclude_from_sweeps` (entry setting)
- *Round settings*: `session_lock` (Congress chamber grouping), `ignore_results`, `use_for_breakout`, `seed_round`, `show_chair`, `motion`, `public_feedback`
- *Entry settings*: `no_elims` (ineligible to clear), `exclude_from_sweeps`, `sweeps` (manual sweep points), `dq`
- *Tourn settings*: `nsda_district`, `nsda_nats`, `ncfl`, `advance_target` (district break percentage), `regions`, `require_disaster_check`, `backup_followers`
- *Tiebreak fields*: `name` (tiebreaker type), `count` (all/prelim/elim/specific/previous), `count_round` (for specific), `highlow` (0=none, 1=both, 3=best, 4=worst, 5=only best N), `highlow_count`, `highlow_threshold`, `highlow_target`, `truncate`, `truncate_smallest`, `multiplier`, `violation`, `result` (win/loss/split filter), `priority` (tier), `child` (composite protocol), `chair` (chair/nonchair/all)

**Key reports/exports:**
- Final Places result set
- Prelim Seeds result set
- Speaker Awards result set
- Bracket visualization
- Chamber Results (Congress)
- Prelims Results Table (JSON-backed)
- NDCA Points report
- TOC Bid report (with optional email)
- District Qualifiers sheet
- CSV exports of results and speakers
- Uploaded result files (PDF etc.)
- Public circuit stats, speaker rankings, debate stats

**Integrations/dependencies:**
- S3 storage for uploaded result files (`$Tab::s3_url`)
- Indexcards service (`$Tab::indexcards_url`) for qualifier posting
- Email notification system (`send_notify.mas`) for blast_results and TOC bids
- NSDA membership system for district qualifiers and nationals registration
- Lingua::EN::Numbers::Ordinate for ordinal formatting (1st, 2nd, etc.)
- Math::Round for rounding break targets
- JSON for ballot string encoding and prelims table cache
- Tab::Utils::compress for cache compression

**Documentation sources:**
- `web/setup/rules/tiebreaks_explain.mhtml` -- Comprehensive guide to all tiebreaker types with explanations
- `web/funclib/tiebreak_name.mas` -- Human-readable tiebreaker name generation
- Code comments in `break_debate.mhtml` (bracket seeding logic, double-elimination)
- tournament-manual.pdf (referenced but not directly read here)

**Code sources:**
- `web/tabbing/results/` -- Results computation: `index.mhtml`, `order_entries.mas`, `order_speakers.mas`, `csv.mhtml`, `results_csv.mas`, `speakers.mhtml`, `speakers_csv.mhtml`, `results_table.mas`, `nsda_qualifiers.mhtml`, `nsda_sweepstakes.mhtml`, `nsda_points.mhtml`, `nats_order.mas`, `top_novice.mas`, `roles.mhtml`, `see_order.mhtml`
- `web/tabbing/break/` -- Break/advancement: `index.mhtml`, `menu.mas`, `break_debate.mhtml`, `break_speech.mhtml`, `break_congress.mhtml`, `break_wudc.mhtml`, `ncfl_snake.mhtml`, `ready_status.mas`
- `web/tabbing/publish/` -- Publication management: `index.mhtml`, `generate_results.mhtml`, `generate_bracket.mas`, `display.mhtml`, `bracket.mhtml`, `bracket_wudc.mhtml`, `publish_all.mhtml`, `publish_all_rounds.mhtml`, `result_set_switch.mhtml`, `result_set_adjust.mhtml`, `rset_switch.mhtml`, `set_delete.mhtml`, `delete_result.mhtml`, `upload_results.mhtml`, `file_delete.mhtml`, `file_switch.mhtml`, `nationals_ranks.mhtml`, `register_nationals.mhtml`, `result_set_add.mhtml`
- `web/panel/publish/` -- Round publication: `index.mhtml`, `publish_everything.mhtml`, `publish_switch.mhtml`
- `web/funclib/` -- Supporting libraries: `blast_results.mas`, `congress_ties.mas`, `tiebreak_types.mas`, `tiebreak_name.mas`, `district_tiebreakers.mas`, `results_debate.mas`, `results_speech.mas`, `results_congress.mas`, `results_wudc.mas`, `results_table.mas`, `round_results.mas`, `round_blast.mas`, `entry_results.mas`, `region_results.mas`, `region_result_values.mas`, `tourn_result_sets.mas`, `tourn_round_results.mas`, `tourn_results_events.mas`
- `web/index/results/` -- Public results display: `index.mhtml`, `circuit_stats.mhtml`, `speaker_rankings_by_circuit.mhtml`, `debate_stats2.mhtml`, `toc_bids.mhtml`, `ndca_standings.mhtml`, `ndt_ceda_points.mhtml`, `qualifiers.mhtml`, `team_results.mhtml`, `team_lifetime_record.mhtml`, `rpi_detail.mhtml`, `speaker_detail.mhtml`, and more
- `web/setup/rules/` -- Configuration: `tiebreaks.mhtml`, `tiebreaks_explain.mhtml`, `tiebreak_save.mhtml`, `tiebreak_rm.mhtml`, `tiebreak_edit.mas`
- `web/lib/Tab/` -- ORM models: `Result.pm`, `ResultSet.pm`, `ResultKey.pm`, `ResultValue.pm`, `Tiebreak.pm`

**Complexity level:** High

The tiebreaker computation engine (`order_entries.mas`) is among the most complex components in the codebase. It handles 30+ tiebreaker types, composite protocols, high/low dropping, truncation, section-rank equalization, round-robin pod ranking, and format-specific logic for debate, speech, congress, and WUDC. The break logic adds significant complexity with single/double elimination bracket management, bracket seed tracking, repeat-matchup detection, and hardcoded NSDA district special cases. The publication system has multiple independent visibility tiers (schematic, primary, secondary, feedback) each with four levels, plus separate result-set-level publication.

**Feature frequency / criticality:** Every tournament

Results computation and breaks are used at every tournament. Publication is used at every tournament. Tiebreaker configuration happens during tournament setup. The specific result types (NDCA, TOC, district qualifiers) are used at subset of tournaments but are high-criticality for those events.

**Notes and open questions:**
- `order_entries.mas` is very large (likely 500+ lines) and deeply procedural; it is the core ranking engine and a likely candidate for the most complex single component to rewrite.
- The `Tiebreak.child` composite mechanism allows one protocol to reference another, but composites-of-composites are explicitly forbidden (with a user-facing error message in `order_entries.mas`).
- Congress session locking (`congress_ties.mas`) groups rounds for combined ranking; the logic is non-trivial with fallback behaviors for invalid locks.
- The `result_set.cache` field stores compressed JSON for prelims tables, suggesting a pattern where complex result data is pre-computed and cached.
- NSDA district break logic contains multiple hardcoded special cases (4-entry/3-qualifier scenario, specific debate final protocol name matching).
- `result_set.bracket` is a boolean flag that changes how the result set is displayed (bracket visualization vs. table).
- The `Result.percentile` field is computed post-generation but it is unclear where/how it is consumed.
- Break target calculation in `menu.mas` uses different formulas per event type (powers of 2 for debate, multiples of 6/7 for speech, multiples of 12 for congress).
- The WUDC break and bracket paths (`break_wudc.mhtml`, `bracket_wudc.mhtml`) are separate files, suggesting British Parliamentary format has substantially different break logic.

---

## Domain 2: Sweepstakes and Awards

### Sweepstakes and Awards

**Description:** Computes aggregate team/school/individual awards across multiple events at a tournament, using configurable point systems (sweep rules). Sweepstakes awards aggregate entry-level results (from sweepstakes rules applied to scores) into school, individual, or entry rankings. Circuit-level sweep awards aggregate results across tournaments within a circuit over a season.

**Primary users:** Tournament directors (configure and generate), coaches (view school standings), public (view published results)

**Main entry points:**
- `/setup/rules/sweeps.mhtml` -- Configure sweepstakes rule sets (setup UI)
- `/tabbing/publish/index.mhtml` -- Generate sweepstakes results (sidebar form)
- `/tabbing/publish/generate_sweeps.mhtml` -- Execute sweepstakes generation
- `/tabbing/results/nsda_sweepstakes.mhtml` -- NSDA-specific sweepstakes display

**Major sub-capabilities:**

- *Sweep set configuration* (`/setup/rules/sweeps.mhtml`, `sweep_set_save.mhtml`, `sweep_add.mhtml`, `sweep_rm.mhtml`, `sweep_types_add.mhtml`): Create named sweep sets per tournament. Each set specifies which events are included (by specific event, by event type, or by event level), and contains rules that define how points are calculated. UI has three tabs: "setup" (event selection), "rules" (current rules), "add_rules" (add new rules). Sets support parent/child composition via `SweepInclude` -- a sweep set can include results from child sets.
- *Sweep rule types* (from `sweep_tourn.mas` and `sweep_rule_save.mhtml`): Rules define how scores translate to points. Key rule tags include:
  - `points_per` -- Points awarded per round competed
  - `winloss` / `ballots` -- Points based on wins or ballot count
  - `ranks` / `points` / `reciprocals` -- Points based on judge scores
  - `place` -- Points based on final placement
  - `seed` -- Points based on prelim seed
  - `elim_wins` -- Points for winning elim rounds
  - `novice_only` -- Restrict to novice entries
  - `multiply_entrysize` -- Multiply points by number of students on entry
  - `multiplier` -- General point multiplier
  - `skip_rounds` / `ignore_round` -- Exclude specific rounds
  - `exclude_breakouts` -- Exclude breakout rounds
  - `entries` -- Max entries counted per school
  - `events` -- Max events counted per school
  - `event_entries` -- Max entries per event per school
  - `wildcards` -- Additional entries beyond the cap
  - `one_per_person` -- Count only one entry per student
  - `by_person` -- Count by person rather than entry
  - `max_entry_persons` -- Maximum persons counted per entry
  - `set_limit` / `set_event_limit` -- Limits on child set contributions per school
- *Sweep rule detail fields* (`SweepRule.pm`): Each rule has: `tag` (rule type), `value` (point value), `place` (placement threshold), `count` (round scope: all/prelim/elim/specific), `count_round` (specific round number), `rev_min` (reverse minimum), `truncate`, `protocol` (specific tiebreaker set to reference).
- *Sweep computation - by entry* (`sweep_tourn.mas`): Core computation engine. Iterates over all entries in sweep-set-specified events, pulls all scores from the database (filtered by skip_rounds, exclude_breakouts, exclude_from_sweeps entry setting), then applies each sweep rule to compute points per entry. Handles byes, forfeit detection, round-type filtering, and manual sweep point overrides (`entry_setting.sweeps`). Returns entries hash with points and rules hash with rule metadata.
- *Sweep computation - by school* (`sweep_schools.mas`): Aggregates entry-level points into school totals. Applies school-level limits: max entries counted (`entries` rule), max events (`events` rule), max entries per event (`event_entries` rule), wildcard entries beyond caps. Supports hybrid entries (split points between two schools). Recursively processes child sweep sets via `SweepInclude`. Tracks per-school counted entries, event counts, and wildcard counts. Returns schools hash with aggregated points, entry counts, and detail strings.
- *Sweep computation - by individual* (`sweep_students.mas`): Aggregates entry-level points into individual student totals. Similar limit logic as schools but applied per-student. Returns students hash with points, entry counts, and subtotals breakdown.
- *Sweep result set generation* (`generate_sweeps.mhtml`): Three scopes: "schools", "entries", "students". For each scope: deletes previous result set with same label, computes results, creates `ResultSet` with `sweep_set` link, creates `Result` rows with rank/place, creates `ResultKey` and `ResultValue` rows for points, counted entries, total entries, and student detail. Handles tie detection (appends "-T" to place). Supports `limit` to restrict to top N places. Links result set back to `sweep_set` for reference.
- *Circuit-level sweep awards* (`SweepAward.pm`, `SweepAwardEvent.pm`): Circuit-scoped awards that aggregate across tournaments. `SweepAward` has: name, description, circuit, target (school/student/entry), count, min_entries, min_schools, period. Links to multiple `SweepSet` instances and `ResultSet` instances. `SweepAwardEvent` defines event categories for circuit awards with name, abbr, level.
- *NSDA sweepstakes* (`nsda_sweepstakes.mhtml`): Special NSDA district/nationals sweepstakes display. Creates "Yearly Sweeps", "Speech", and "Debate" sweep sets. Computes combined and per-type school standings.
- *NSDA district sweepstakes setup* (`district_tiebreakers.mas`): Automatically creates Congress tiebreaker protocols with `equal_elims` and `forfeits_rank_last` settings during district tournament setup.
- *Cumulative awards* (referenced in `sweeps.mhtml`): For NSDA Nationals, support cumulative award rules that aggregate across rounds/days.
- *Sweep event filtering* (`SweepEvent.pm`): Links sweep sets to specific events or event types/levels. Supports filtering by `event_type` and `event_level` and `nsda_category`. When no specific events are linked, filtering falls back to type/level constraints via SQL.

**Key entities:**
- `Tab::SweepSet` (`web/lib/Tab/SweepSet.pm`): Tournament-level sweep configuration. Columns: id, tourn, name, sweep_award, timestamp. Has many rules (`SweepRule`), sweep_events (`SweepEvent`), children/parents (`SweepInclude`). Custom `rule()` method provides get/set interface for sweep rules by tag. Custom `scopes()` method for JSON-encoded scope data.
- `Tab::SweepRule` (`web/lib/Tab/SweepRule.pm`): Individual rule within a sweep set. Columns: id, sweep_set, tag, value, place, count, count_round, rev_min, truncate, protocol, timestamp.
- `Tab::SweepEvent` (`web/lib/Tab/SweepEvent.pm`): Links sweep set to events. Columns: id, sweep_set, event, event_type, event_level, nsda_category, sweep_award_event, timestamp.
- `Tab::SweepInclude` (`web/lib/Tab/SweepInclude.pm`): Parent-child relationship between sweep sets. Columns: id, parent, child, timestamp. Enables compositional sweep sets.
- `Tab::SweepAward` (`web/lib/Tab/SweepAward.pm`): Circuit-level award definition. Columns: id, name, description, circuit, target, count, min_entries, min_schools, period, timestamp. Has many sweep_sets and result_sets.
- `Tab::SweepAwardEvent` (`web/lib/Tab/SweepAwardEvent.pm`): Event category for circuit awards. Columns: id, name, abbr, level, sweep_set, timestamp.
- `Tab::ResultSet` -- Used to store generated sweep results (linked via `sweep_set` and `sweep_award` columns).
- `Tab::Result` -- Individual ranked rows in sweep results (linked to school or student rather than entry for school/individual scopes).

**Key settings/configuration:**
- *Sweep set rules* (stored in `sweep_rule` table via `SweepSet->rule()` method):
  - `entries` -- Max entries counted per school
  - `events` -- Max distinct events counted per school
  - `event_entries` -- Max entries per event per school
  - `wildcards` -- Extra entries beyond caps
  - `one_per_person` -- One entry per student
  - `by_person` -- Count per person
  - `max_entry_persons` -- Max persons counted per entry
  - `novice_only` -- Restrict to novice entries
  - `multiply_entrysize` -- Multiply by team size
  - `multiplier` -- General multiplier
  - `skip_rounds` / `ignore_round` -- Round exclusions
  - `exclude_breakouts` -- Exclude breakout rounds
  - `set_limit` -- Per-school limit from child sets
  - `set_event_limit` -- Per-school per-event limit from child sets
  - `cumulative` -- Cumulative award rule (NSDA Nationals)
- *Entry settings*: `exclude_from_sweeps` (exclude entry from sweep calculations), `sweeps` (manual sweep point override)
- *Event settings*: Various breakout settings affect which entries are included

**Key reports/exports:**
- School sweepstakes result set (with points, counted entries, total entries columns)
- Individual sweepstakes result set (with points, counted entries, total entries, subtotals by entry)
- Entry sweepstakes result set (with event, per-rule-breakdown, total points, student detail)
- NSDA district sweepstakes (combined, speech, debate breakdowns)
- NSDA Nationals cumulative awards
- All sweep results publishable with same publish/coach toggle as regular result sets

**Integrations/dependencies:**
- Depends on the results/tiebreaker system for entry scoring data
- References `entry_setting.exclude_from_sweeps` and `entry_setting.sweeps` for per-entry overrides
- Hybrid entry support requires cross-referencing `student.chapter` with `school.chapter` to detect students from different schools on the same entry
- NSDA membership integration for district sweepstakes
- JSON encoding for student subtotals in individual scope results

**Documentation sources:**
- `web/setup/rules/sweeps.mhtml` -- UI labels explain rule purposes
- `web/funclib/sweeps/show_rules.mas`, `setup_set.mas`, `add_rules.mas` -- Rule display and configuration components (referenced but in funclib/sweeps/)
- Code comments in `sweep_schools.mas` and `sweep_tourn.mas`

**Code sources:**
- `web/setup/rules/sweeps.mhtml` -- Sweep set configuration UI
- `web/setup/rules/sweep_set_save.mhtml` -- Create/update sweep set
- `web/setup/rules/sweep_set_delete.mhtml` -- Delete sweep set
- `web/setup/rules/sweep_add.mhtml` -- Add events to sweep set
- `web/setup/rules/sweep_rm.mhtml` -- Remove events from sweep set
- `web/setup/rules/sweep_types_add.mhtml` -- Add event types to sweep set
- `web/setup/rules/sweep_rule_save.mhtml` -- Save sweep rule
- `web/setup/rules/sweep_rule_rm.mhtml` -- Delete sweep rule
- `web/setup/rules/sweep_rule_cumulative.mhtml` -- Cumulative award rule (NSDA)
- `web/tabbing/publish/generate_sweeps.mhtml` -- Generate sweep results
- `web/tabbing/publish/sw_nsda_save.mhtml` -- NSDA sweep save
- `web/tabbing/publish/sw_nsda_students.mas` -- NSDA student sweeps
- `web/tabbing/publish/swdistrict.mhtml` -- District sweep display
- `web/tabbing/results/sweep_schools.mas` -- School-level sweep computation
- `web/tabbing/results/sweep_students.mas` -- Individual-level sweep computation
- `web/tabbing/results/sweep_tourn.mas` -- Entry-level sweep computation (core engine)
- `web/tabbing/results/nsda_sweepstakes.mhtml` -- NSDA sweepstakes display
- `web/lib/Tab/SweepSet.pm`, `SweepRule.pm`, `SweepEvent.pm`, `SweepInclude.pm`, `SweepAward.pm`, `SweepAwardEvent.pm` -- ORM models

**Complexity level:** High

The sweep computation involves recursive child-set processing, multiple scoping rules (entries, events, wildcards, per-person limits), hybrid entry point splitting, manual override handling, and complex score aggregation across round types. The rule system is highly configurable with many interacting parameters.

**Feature frequency / criticality:** Most tournaments

Most tournaments of significant size generate school sweepstakes. Individual sweepstakes are used at some tournaments. NSDA district and nationals sweepstakes are critical for those specific tournament types. Circuit-level awards are used by circuits tracking season-long standings.

**Notes and open questions:**
- The `sweep_schools.mas` file contains a recursive `sweep_set()` subroutine that processes child sweep sets. This recursion could theoretically be deep if sweep sets are heavily nested, though in practice nesting is likely shallow.
- The `SweepSet->rule()` method in the ORM provides a get/set interface that searches, creates, updates, or deletes `SweepRule` rows -- it is a custom accessor pattern rather than standard ORM access.
- The `SweepSet->scopes()` method references a `scope` column that is not listed in the `columns(All)` declaration, suggesting it may be in the database but not exposed in the ORM definition, or it may be vestigial.
- Hybrid entry handling (splitting points between two schools) is based on detecting `student.chapter != school.chapter`, which could have edge cases with multi-school teams.
- The `exclude_from_sweeps` entry setting and `sweeps` manual override provide escape hatches for edge cases but add complexity to the computation path.
- `SweepAward` (circuit-level) and `SweepAwardEvent` appear to be used for circuit season standings but the full circuit sweep computation flow was not fully traced in this analysis.
- The `target` field on `SweepAward` (school/student/entry) suggests circuit awards can target different aggregation levels, but the generation mechanism was not fully explored.
- It is unclear whether circuit-level sweep computation happens within Tabroom or is handled by a separate service.
