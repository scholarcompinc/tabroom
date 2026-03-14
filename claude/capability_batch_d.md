# Capability Domains: Batch D

## 1. Pairing, Paneling, and Schematics

### Pairing, Paneling, and Schematics

**Description:** This domain encompasses the algorithmic core of tournament management: generating matchups (debate) or section assignments (speech/congress), assigning judges to panels, assigning rooms, managing side constraints, handling byes and forfeits, and displaying/publishing the resulting schematics. It contains the most computationally complex code in Tabroom, with multiple distinct pairing algorithms for different competitive formats (debate powermatching, speech snaking, congress chamber assignment, WUDC British Parliamentary, preset/round-robin) and extensive constraint-satisfaction logic.

**Primary users:**
- Tournament directors and tab staff (primary operators who trigger pairing, assign judges/rooms, manipulate results)
- Coaches and competitors (consumers who view published schematics and receive blast notifications)
- Judges (receive assignments via blast/email)

**Main entry points:**
- `/panel/schemat/show.mhtml` -- Main schematic view page; dispatches to format-specific display components (`show_debate.mas`, `show_speech.mas`, `show_congress.mas`, `show_wudc.mas`)
- `/panel/round/index.mhtml` -- Round management landing page
- `/panel/round/event.mhtml` -- Event-level round overview
- `/panel/judge/index.mhtml` -- Judge pool management
- `/panel/room/index.mhtml` -- Room pool management
- `/panel/manipulate/` -- All manual adjustment endpoints
- `/panel/publish/` -- Publishing controls
- `/panel/report/` -- Printable reports (postings, ballots, labels)

**Major sub-capabilities:**

*Pairing Algorithms:*
- **Debate powermatching** (`web/panel/round/pair_debate.mas`, ~900 lines) -- The primary debate pairing engine. Implements bracket-by-bracket powermatching with configurable methods (SOP, high-high, high-low). Handles side constraints, pullup logic, bye assignment, school/conflict preclusion, and bracket collapse when pairing is impossible. Contains subfunctions: `setbracket`, `pairbracket`, `justmakeitwork`, `pullup`, `make_pullup_array`, `count_possible_opponents`, `unpaired`, `write_round`, `erase_current_round`.
- **Alternative powermatch** (`web/panel/round/pair_powered.mas`) -- A penalty-based optimization approach to debate pairing. Assigns numerical penalties for school conflicts (10^18), repeat matchups (10^12), wrong side (10^11), pullups (10^4). Used as fallback ("disaster mode") when the primary algorithm fails.
- **Speech section snaking** (`web/panel/round/pair_speech.mas`, `web/panel/round/snake_speech.mas`) -- Assigns competitors to sections using weighted penalty scoring. Penalties vary by context: NSDA Districts (school=10^9, title=10^4, repeat=100), NSDA Nationals (school/district/region=10^5, repeat=10^5, order=100), regular tournaments (school=10^6, repeat=10^3). Tracks speaker order history and avoids same-school, same-region pairings.
- **Congress chamber pairing** (`web/panel/round/pair_congress.mas`) -- Assigns competitors to legislative chambers. Supports PO (Presiding Officer) contest cloning, and uses penalties for school (10^12), state (10^8), bill_topic (10^6), author (10^4). Tracks recency via `congress_recency.mhtml`.
- **WUDC pairing** (`web/panel/round/pair_wudc.mas`) -- British Parliamentary 4-team-per-room format. Assigns positions (OG/OO/CG/CO) based on accumulated ranks. Tracks position history and avoids same-school conflicts.
- **Preset/round-robin pairing** (`web/panel/round/pair_preset.mas`) -- Generates preset round pairings respecting region, school, and area constraints. Handles side constraints and sidelock configurations.
- **Elimination bracket pairing** (`web/panel/round/pair_bracket.mhtml`) -- Pairs specific win brackets in elim rounds.

*Side Assignment:*
- **Serpentine (snake) sides** (`web/panel/manipulate/snake_sides.mhtml`) -- Assigns sides in a serpentine pattern by seed, with special handling for large schools (>30% of draw placed on same side).
- **Random sides** (`web/panel/manipulate/random_sides.mhtml`)
- **Manual side swap** (`web/panel/schemat/debate_sides_swap.mhtml`, `debate_side_save.mhtml`)
- **Sidelock against specific round** (via round setting `sidelock_against`)
- **WUDC side assignment** (`web/panel/schemat/wudc_sides.mhtml`, `wudc_side_save.mhtml`)

*Judge Assignment:*
- **Automated judge assignment** (`web/panel/round/judges.mhtml`) -- Clears existing assignments and reassigns judges to panels respecting judge pools (jpools), flight structure, preference ratings, conflicts, and rounds-per limits. Handles congress parliamentarian assignment across tied sessions.
- **Manual judge operations** (`web/panel/round/manual_judges.mhtml`, `manual_judge_save.mhtml`)
- **Judge pool management** (`web/panel/judge/jpool.mhtml`, `jpool_create.mhtml`, `edit_jpools.mhtml`, `jpool_autopopulate.mhtml`) -- Create/edit judge pools, assign judges to pools, link pools to rounds.
- **Standby judges** (`web/panel/judge/standbys.mhtml`, `standby_create.mhtml`)
- **Judge availability** (`web/panel/judge/availability.mhtml`, `event_availability.mhtml`)
- **NSDA Nationals pool operations** (`web/panel/judge/nats_pool_*.mhtml`)
- **Schemat-level judge add/remove/swap** (`web/panel/schemat/judge_add.mhtml`, `judge_rm.mhtml`, `judge_remove.mhtml`, `judge_push.mhtml`, `flight_judge_swap.mhtml`, `flight_judge_save.mhtml`, `chair_switch.mhtml`, `chair_confirm.mhtml`)

*Room Assignment:*
- **Automated room assignment** (`web/panel/room/assign.mhtml`) -- Assigns rooms to panels by timeslot, round type, site, or event.
- **Room pool management** (`web/panel/room/rpool.mhtml`, `rpool_create.mhtml`, `rpool_edit.mhtml`, `rpool_clone.mhtml`) -- Create/manage room pools (rpools) linked to rounds.
- **Manual room operations** (`web/panel/round/manual_rooms.mhtml`, `manual_room_save.mhtml`)
- **Room import** (`web/panel/room/import_csv.mhtml`)
- **Room reserve** (`web/panel/room/reserve.mhtml`, `reserve_save.mhtml`)
- **Schemat-level room save** (`web/panel/schemat/room_save.mhtml`, `panel_room_save.mhtml`)

*Schematic Display and Manipulation:*
- **Format-specific views** (`web/panel/schemat/show_debate.mas`, `show_speech.mas`, `show_congress.mas`, `show_wudc.mas`) -- Each displays appropriate columns (sides, judges, rooms, speaker order, results) for its format.
- **Panel view/edit** (`web/panel/schemat/panel_view.mhtml`) -- Detailed view of a single panel.
- **Entry swap/move** (`web/panel/manipulate/debate_swap.mhtml`, `entry_move.mhtml`, `move_speech.mhtml`, `entry_edit.mhtml`, `entry_rm.mhtml`)
- **Round edit views** (`web/panel/manipulate/debate_round_edit.mhtml`, `speech_round_edit.mhtml`, `congress_round_edit.mhtml`, `wudc_round_edit.mhtml`)
- **Bracket manipulation** (`web/panel/manipulate/bracket_edit.mhtml`, `bracket_save.mhtml`, `collapse_bracket.mhtml`, `reset_bracket.mhtml`, `dump_bracket.mhtml`)
- **Panel creation/deletion** (`web/panel/manipulate/create_panels.mhtml`, `panel_rm.mhtml`, `empty_rm.mhtml`, `dump_panel.mhtml`)
- **Manual pairing** (`web/panel/round/manual_debate.mhtml`, `manual_speech.mhtml`, `web/panel/manipulate/manual_debate.mhtml`, `manual_speech.mhtml`, `manual_wudc.mhtml`, `manual_powermatch.mhtml`, `manual_pullup.mhtml`, `manual_rebalance.mhtml`)
- **Manual section trimming** (`web/panel/manipulate/trim_sections.mhtml`)
- **Free judges** (`web/panel/manipulate/free_judges.mhtml`, `deactivate_judges.mhtml`)
- **Replace judge** (`web/panel/manipulate/replace_judge.mhtml`)

*Speaker Order:*
- **Speaker order management** (`web/panel/schemat/speaker_order.mhtml`, `speaker_order_save.mhtml`)
- **Speaker order improvement** (`web/panel/round/speaker_order.mhtml`, `speaker_order_improve.mhtml`)
- **Correct orders utility** (`web/funclib/correct_orders.mas`) -- Fixes duplicate or missing speaker order values.

*Blast/Publishing:*
- **Blast pairing** (`web/panel/schemat/blast_pairing.mas`, `blast.mhtml`, `blast_message.mas`, `blast_schedule.mhtml`, `blast_delete.mhtml`) -- Sends pairing notifications via email to competitors, judges, and followers. Uses external `indexcards` API.
- **Section blast** (`web/panel/schemat/section_blast.mhtml`)
- **Publish controls** (`web/panel/publish/publish_switch.mhtml`, `publish_everything.mhtml`) -- Toggle round publication status.
- **Online ballots toggle** (`web/panel/schemat/online_ballots.mhtml`)

*Reports:*
- **Postings** (`web/panel/report/postings.mhtml`, `list_postings.mhtml`, `half_postings.mhtml`, `giant_postings.mhtml`, `bigass_posting.mhtml`)
- **Schematic print** (`web/panel/report/schematic.mhtml`)
- **Ballot printing** (`web/panel/report/print_ballots.mhtml`, `ballot_table.mhtml`, `ballot_labels.mhtml`)
- **Judge reports** (`web/panel/report/judge_chart.mhtml`, `judge_labels.mhtml`, `judge_points.mhtml`)
- **Chamber reports** (`web/panel/report/chamber_roster.mhtml`, `chamber_report.mhtml`)
- **Congress scoresheets** (`web/panel/report/congress_scoresheet.mhtml`)
- **Seating charts** (`web/panel/report/seating.mhtml`)
- **Strike cards** (`web/panel/report/strike_cards.mhtml`)
- **Room master** (`web/panel/report/rooms_master.mhtml`)
- **Disaster report** (`web/panel/report/disasters.mhtml`)
- **Double entry check** (`web/panel/report/double_entry.mhtml`)
- **Preference/experience reports** (`web/panel/report/pref_experience.mhtml`)
- **Slideshow** (`web/panel/report/slideshow.mhtml`)
- **Placards** (`web/panel/report/placards.mhtml`)

*Data Preparation:*
- **Pairing hash construction** (`web/funclib/make_pairing_hash.mas`) -- Builds the core data structure for debate pairing: loads all active entries with school/region/pairing_school info, computes win records, seeds (SOP/opponent seeds), side due, and preclusion matrix (school, hit-before, region conflicts).
- **Entry wins** (`web/funclib/entry_wins.mas`) -- Computes win/loss records per round.
- **Bracket display** (`web/funclib/bracket_show.mas`) -- Renders elimination brackets for published results.
- **Disaster check** (`web/panel/schemat/disaster_check.mhtml`) -- Pre-publication validation checking for missing judges, rooms, entries; school conflicts; side issues.

*Other:*
- **Flip management** (`web/panel/schemat/flips.mhtml`, `panel_flip_save.mhtml`)
- **Debate bye management** (`web/panel/manipulate/debate_bye.mhtml`)
- **Round log** (`web/panel/schemat/round_log.mhtml`)
- **Round settings edit** (`web/panel/schemat/settings_edit.mas`, `settings_save.mhtml`, `setting_switch.mhtml`)
- **Round CSV exports** (`web/panel/schemat/round_csv.mhtml`, `event_csv.mhtml`, `list_csv.mhtml`, `roster_csv.mhtml`, `timeslot_csv.mhtml`, `cloud_csv.mhtml`)
- **Round import/upload** (`web/panel/schemat/import_round.mhtml`, `upload_debate.mhtml`, `upload_backup.mhtml`)
- **Round dump** (`web/panel/schemat/round_dump.mhtml`, `round_dump_entries.mhtml`, `round_dump_judges.mhtml`, `round_dump_rooms.mhtml`)
- **Prefs report** (`web/panel/schemat/prefs_report.mhtml`)
- **Preset editing** (`web/panel/schemat/preset_edit.mas`, `create_presets.mhtml`)
- **Congress snake display** (`web/panel/schemat/snake_congress.mas`, `snake_congress_cards.mhtml`, `show_snake.mhtml`)
- **Seating assignment** (`web/panel/schemat/seating_assign.mhtml`, `seating_view.mhtml`, `seating_move.mhtml`, `seating_rooms_save.mhtml`, `seating_chart.mhtml`, `seating_print.mhtml`)
- **Rubrics** (`web/panel/schemat/rubrics.mhtml`)
- **Mass round creation** (`web/panel/round/mass_create.mhtml`, `mass_judges.mhtml`)
- **Motions** (`web/panel/round/motions.mhtml`, `motions_save.mhtml`) -- WUDC/BP debate motion management.
- **Runoff** (`web/panel/round/runoff.mhtml`, `runoff_schedule.mhtml`)
- **Timeslot merge** (`web/panel/round/timeslot_merge.mhtml`)
- **Congress recency tracking** (`web/panel/round/congress_recency.mhtml`, `congress_recency_report.mhtml`)
- **Category check** (`web/panel/round/category_check.mhtml`)

**Key entities:**
- `Tab::Panel` (`web/lib/Tab/Panel.pm`) -- Core panel/section entity. Columns: id, letter, round, room, flight, bye, bracket, publish, started, timestamp. Has many: ballots, student_votes, logs (ChangeLog). TEMP columns for display: opp, pos, side, entryid, judge, audit, timeslotid, roomname, eventname, judgenum, panelsize, ada, speakerorder.
- `Tab::PanelSetting` (`web/lib/Tab/PanelSetting.pm`) -- Key-value settings on panels. Columns: id, panel, tag, value, value_date, value_text, setting, timestamp.
- `Tab::Round` (`web/lib/Tab/Round.pm`) -- Round entity. Columns: id, type, name, label, flighted, published, post_primary, post_secondary, post_feedback, paired_at, start_time, site, event, runoff, protocol, timeslot, timestamp. Types include: prelim, preset, snaked_prelim, highhigh, highlow, elim, final, runoff. Has many: jpools (via JPoolRound), rpools (via RPoolRound), autoqueues, settings (RoundSetting), panels, results.
- `Tab::RoundSetting` (`web/lib/Tab/RoundSetting.pm`) -- Key-value settings on rounds.
- `Tab::Ballot` (`web/lib/Tab/Ballot.pm`) -- Links judge/entry/panel. Columns: id, judge, panel, entry, speakerorder, side, audit, bye, forfeit, tv, approved, chair, seat, entered_by, audited_by, judge_started, started_by, timestamp. Has many: scores.
- `Tab::ChangeLog` (`web/lib/Tab/ChangeLog.pm`) -- Audit trail. Columns: id, tag, tourn, school, person, count, description, judge, entry, event, category, chapter, circuit, round, panel, new_panel, old_panel, fine, deleted, created_at, timestamp.
- `Tab::StudentVote` (`web/lib/Tab/StudentVote.pm`) -- Congress peer voting. Columns: id, tag, value, panel, entry, voter, entered_by, entered_at, timestamp.

**Key settings/configuration:**
- *Event settings (via EventSetting):*
  - `powermatch` -- Powermatching method: "sop" (seed + opp seeds), "highhigh"
  - `pullup_method` -- How to select pullup teams: "sop" default
  - `pullup_repeat` -- Allow repeat pullups
  - `pullup_minimize` -- APDA-style pullup minimization
  - `no_side_constraints` -- Disable side locking
  - `bracket_by_ballots` -- Use ballot count rather than wins for bracketing
  - `school_debates_self` -- Allow same-school matchups
  - `region_constrain` / `region_avoid` -- Region-level constraints
  - `aff_label` / `neg_label` -- Side labels
  - `autobye_nojudge` -- Auto-bye when insufficient judges
  - `online_mode` -- Online competition mode (sync, nsda_campus, etc.)
  - `snake_sides_huge_schools` -- Override large-school side snaking
  - `round_robin` -- Round robin format
  - `combined_ballots` -- Combined ballot entry mode
  - `speaker_priority_first` -- Speech speaker order priority
  - `ask_for_titles` -- Track piece titles for speech
  - `point_increments` -- whole/half/tenths/fourths
  - `flight_rooms_only` -- Only flight rooms, not judges
  - `blind_mode` -- Hide codes until publish
  - `prevent_hitting_pullup_twice` -- Prevent repeat pullup matches

- *Round settings (via RoundSetting):*
  - `sidelock_against` -- Lock sides against specific round, or "NONE"/"RANDOM"
  - `num_judges` -- Number of judges per panel
  - `flight_a_round` / `flight_b_round` -- Linked flight rounds
  - `disaster_checked` -- JSON log of disaster checks
  - `use_normal_rooms` -- Override online mode for this round
  - `use_for_breakout` -- Breakout round indicator

- *Tournament settings:*
  - `nsda_district` / `nsda_nats` / `ncfl` -- Organization-specific pairing rules
  - `regions` -- Enable region tracking

**Key reports/exports:**
- Postings (various sizes/formats)
- Schematic prints
- Ballot PDFs
- Judge assignment charts
- Room master reports
- CSV exports (round, event, roster, timeslot, cloud)
- Strike cards
- Congress scoresheets
- Seating charts
- Preset draw sheets
- NSDA elim bios
- NCFL reports
- Slideshow mode

**Integrations/dependencies:**
- `indexcards` API (`$Tab::indexcards_url`) -- External service for blast notifications
- Email/SMS notification system (blast sends to person.email/phone)
- Judge preference system (ratings from Tab::Rating via `panel_ratings.mas`, `judge_use.mas`)
- Results/tiebreak system (pairing reads previous round results via `order_entries.mas`, `entry_wins.mas`)
- Entry/school system (reads school, region, chapter for conflict detection)
- Follower system (blast to followers of entries)

**Documentation sources:**
- Inline code comments in `pair_debate.mas` (extensive algorithm documentation)
- Error log output built into pairing code for debugging
- `disaster_check.mhtml` contains implicit documentation of all validation rules

**Code sources:**
- `web/panel/` -- Primary directory (~120 files across subdirectories)
  - `web/panel/round/` -- Pairing algorithms and round management (38 files)
  - `web/panel/schemat/` -- Schematic display, manipulation, blast (92 files)
  - `web/panel/judge/` -- Judge pool management (31 files)
  - `web/panel/room/` -- Room pool management (27 files)
  - `web/panel/manipulate/` -- Manual adjustments (35 files)
  - `web/panel/publish/` -- Publication controls (3 files)
  - `web/panel/report/` -- Printable reports (33 files)
- `web/funclib/make_pairing_hash.mas` -- Core pairing data structure builder
- `web/funclib/entry_wins.mas` -- Win/loss computation
- `web/funclib/correct_orders.mas` -- Speaker order correction
- `web/funclib/bracket_show.mas` -- Bracket rendering
- `web/funclib/round_blast.mas` -- Blast email/notification assembly
- `web/funclib/blast_results.mas`, `blast_flips.mas`, `blast_tabbers.mas`
- `web/funclib/panel_*.mas` -- Panel utility functions (entries, judges, scores, ratings, etc.)
- `web/funclib/congress_ties.mas`, `congress_names.mas`, `results_congress.mas`
- `web/lib/Tab/Panel.pm`, `PanelSetting.pm`, `Round.pm`, `RoundSetting.pm`, `Ballot.pm`

**Complexity level:** High

This is the most algorithmically complex area of Tabroom. The debate pairing engine alone (`pair_debate.mas`) is ~900 lines of nested constraint-satisfaction logic with multiple fallback strategies (clean pairing -> justmakeitwork -> alternative pullups -> bracket collapse -> disaster mode). The speech pairing uses weighted penalty optimization across multiple constraint dimensions. Five distinct pairing algorithms serve different competitive formats, each with format-specific constraints.

**Feature frequency / criticality:** Every tournament

Pairing is the core operational function of every tournament. Every round of every event at every tournament requires pairing, judge assignment, and room assignment. Failures in pairing block the entire tournament.

**Notes and open questions:**
- The `pair_debate.mas` code contains a commented-out reference to `pair_powered.mas` as an alternative engine (`pullup_minimize eq "nobody_today_still_does_not_work"`), suggesting the penalty-based approach was attempted but had issues.
- The `pair_powered.mas` file uses a completely different approach (numerical penalty optimization) compared to `pair_debate.mas` (bracket-based greedy matching). The relationship between these two and when each is used needs clarification.
- Debug mode (`$debugme`) is hardcoded with `undef $debugme` -- toggle requires code change.
- There is extensive special-casing for NSDA Districts, NSDA Nationals, and NCFL throughout the pairing code.
- The `make_pairing_hash.mas` file builds an in-memory data structure for all entries; scalability for very large tournaments may be a concern.
- Congress chamber pairing supports "bloc_school" penalties for NSDA districts when house_chambers matches num_panels.
- The `justmakeitwork` subroutine in `pair_debate.mas` is a fallback that relaxes constraints; documenting exactly which constraints it relaxes would be valuable.
- WUDC pairing assigns 4 teams per panel with position tracking -- a distinct data model from 2-team debate.
- The pairing code uses Class::DBI ORM throughout, which may have performance implications for large tournaments.

---

## 2. Ballots, Scoring, and Audits

### Ballots, Scoring, and Audits

**Description:** This domain covers the complete lifecycle of ballot data: entering scores from paper ballots, receiving online ballots from judges, validating and auditing entered data, computing results using configurable tiebreaker protocols, generating standings, managing breaks to elimination rounds, and producing result reports. It is the second most algorithmically complex area, particularly in the tiebreak computation engine (`order_entries.mas`).

**Primary users:**
- Tab room staff (enter paper ballots, audit entries, manage results)
- Judges (enter online ballots via `/user/judge/ballot.mhtml`)
- Tournament directors (review results, manage breaks, publish standings)
- Coaches and competitors (view published results)

**Main entry points:**
- `/tabbing/entry/index.mhtml` -- Ballot entry landing page (by timeslot, event, judge)
- `/tabbing/entry/panel.mhtml` -- Per-panel ballot entry form
- `/tabbing/entry/audit.mhtml` -- Audit interface for reviewing entered ballots
- `/tabbing/entry/screen_audit.mhtml` -- Screen-based audit view
- `/tabbing/entry/combined.mhtml` -- Combined ballot view (all judges for a round)
- `/tabbing/entry/card.mhtml` -- Entry card (lookup by code or name)
- `/tabbing/entry/rapid.mhtml` -- Rapid data entry mode
- `/tabbing/results/index.mhtml` -- Results/standings view
- `/tabbing/break/index.mhtml` -- Break to elimination rounds
- `/tabbing/status/dashboard.mhtml` -- Tournament status dashboard
- `/tabbing/publish/index.mhtml` -- Results publishing
- `/tabbing/report/index.mhtml` -- Tabbing reports
- `/user/judge/ballot.mhtml` -- Online ballot entry (judge-facing)
- `/user/judge/ballot_save.mhtml` -- Online ballot save/validation
- `/user/judge/ballot_confirm.mhtml` -- Online ballot confirmation

**Major sub-capabilities:**

*Ballot Entry (Tab Room):*
- **Panel-based entry** (`web/tabbing/entry/panel.mhtml`, `panel_save.mhtml`) -- Primary ballot entry form. Loads tiebreak types to determine which score fields to show (winloss, rank, points, refutation, best_po, TV). Handles per-student scoring for debate, per-entry scoring for speech. Supports WSDC sub-scores (Content, Style, Strategy, POI). Validates point increments (whole, half, tenths, fourths). Tracks `entered_by` and `audited_by` persons.
- **Combined ballot view** (`web/tabbing/entry/combined.mhtml`, `combined_audit.mhtml`, `combined_winner.mhtml`) -- View/enter all ballots across judges for a round.
- **Rapid entry mode** (`web/tabbing/entry/rapid.mhtml`, `rapid_switch.mhtml`) -- Streamlined data entry interface.
- **Entry card view** (`web/tabbing/entry/card.mhtml`, `card_save.mhtml`) -- Look up entry by code or last name, view all ballots.
- **Panel extras** (`web/tabbing/entry/panel_extras_save.mhtml`) -- Additional per-panel data entry.
- **Side switching** (`web/tabbing/entry/switch_sides.mhtml`) -- Swap sides on entered ballots.
- **Bye management** (`web/tabbing/entry/bye_switch.mhtml`) -- Toggle bye status.
- **Forfeit handling** -- Controlled via ballot.forfeit column.
- **Closeout** (`web/tabbing/entry/closeout.mhtml`, `closeout_save.mhtml`) -- Close out rounds, finalize scores.
- **Blank correction** (`web/tabbing/entry/blanks_correct.mhtml`) -- Fix missing/blank ballot data.
- **Limit management** (`web/tabbing/entry/limit.mhtml`) -- Control which events/rounds are available for entry.

*Online Ballot Entry (Judge-Facing):*
- **Standard ballot** (`web/user/judge/ballot.mhtml`) -- Full online ballot form. Supports debate (win/loss, ranks, speaker points), speech (ranks, points), congress (ranks, best PO). Validates LPW (low-point wins) unless `no_lpw` or `allow_lowpoints` settings. Supports WSDC sub-point categories, TV (time violation) marking.
- **WSDC ballot** (`web/user/judge/wsdc_ballot.mhtml`) -- Specialized for WSDC with sub-point categories.
- **Legion ballot** (`web/user/judge/legion_ballot.mhtml`) -- Specialized ballot format.
- **Ballot save/validation** (`web/user/judge/ballot_save.mhtml`) -- Validates submitted ballots: checks authorization (judge.person matches person), verifies ballots are unfinished (audit=0), validates entry activity, checks for LPW violations, processes score tags (winloss, rank, point, refute, best_po, subpoints, rubric, rfd, comments). Sets `ballot.audit` to mark completion.
- **Ballot confirmation** (`web/user/judge/ballot_confirm.mhtml`) -- Confirmation step after submission.
- **Ballot rubric** (`web/user/judge/ballot_rubric.mas`, `ballot_rubric_single.mas`) -- Rubric-based scoring interface.
- **Congress recency** (`web/user/judge/ballot_recency.mas`) -- Recency tracking for congress scoring.

*Score Data Model:*
- **Score tags** -- The `Score.tag` field determines score type:
  - `winloss` -- Binary win (1) or loss (0)
  - `rank` -- Numerical rank (1st, 2nd, etc.)
  - `point` -- Speaker/performance points
  - `refute` -- WSDC refutation points
  - `best_po` -- Congress best presiding officer
  - `po` -- Congress PO scoring
  - `subpoints` -- JSON blob of WSDC sub-categories (Content, Style, Strategy, POI)
  - `rubric` -- JSON rubric scoring
  - `rfd` -- Reason for decision (text, stored compressed)
  - `comments` -- Judge comments (text, stored compressed)
  - `tv` -- Time violation
- **Per-student vs per-entry scoring** -- In debate, scores are per-student (each debater gets individual speaker points). In speech, scores are typically per-entry (ballot-level rank/points). The `studpoints` flag controls this.
- **Team points** -- Override mode where points are per-team rather than per-student (`event_settings.team_points` or `ballot_rubric`).
- **Score positions** -- `Score.position` tracks speaker position (1st speaker, 2nd speaker, etc.).
- **Score speech** -- `Score.speech` column.
- **Score content** -- `Score.content` stores compressed text (RFDs, comments) via `Tab::Utils::compress/uncompress`.

*Audit System:*
- **Double-entry audit** (`web/panel/report/double_entry.mhtml`) -- Compares independently entered ballots.
- **Screen audit** (`web/tabbing/entry/screen_audit.mhtml`, `screen_audit_save.mhtml`) -- On-screen audit interface showing all ballots for a timeslot, with color-coding for audit status.
- **Audit tracking** -- `ballot.audit` field: 0=not entered, 1+=entered/confirmed. `ballot.entered_by` and `ballot.audited_by` track who entered and audited.
- **Score delta tracking** -- `panel_save.mhtml` tracks changes (`$score_delta`, `$change` strings) for changelog.
- **ChangeLog** (`web/lib/Tab/ChangeLog.pm`, `web/funclib/log.mas`) -- All score changes logged with person, timestamp, description, old_panel/new_panel. Tags include 'tabbing' for score-related changes.
- **Section audit report** (`web/tabbing/report/section_audit.mhtml`)
- **Print audit** (`web/tabbing/report/print_audit.mhtml`, `audit_print.mas`, `audit_csv.mas`, `audit_table.mas`)

*Student Voting (Congress):*
- **Student vote entry** (`web/tabbing/entry/student_vote.mhtml`, `student_vote_save.mhtml`, `student_vote_switch.mhtml`) -- Congress peer evaluation system.
- **PO vote** (`web/tabbing/entry/po_vote_save.mhtml`) -- Presiding Officer voting.
- **Kill switch** (`web/tabbing/entry/ks_switch.mhtml`) -- Emergency toggle.

*Results and Standings:*
- **Results ordering engine** (`web/tabbing/results/order_entries.mas`) -- The core tiebreak computation engine. Processes configurable tiebreak protocols with priorities. Handles debate-specific (winloss, ballots, opp_wins, headtohead, SOP), speech-specific (ranks, reciprocals, points), and congress-specific (ranks, best_po, student_rank) tiebreakers. Supports composite tiebreakers (child protocols), truncation (drop high/low), multipliers, chair-only tiebreakers, and round-specific counts (all, previous, specific, prelim, elim).
- **Speaker standings** (`web/tabbing/results/speakers.mhtml`, `order_speakers.mas`, `speakers_csv.mhtml`) -- Individual speaker awards.
- **Results display** (`web/tabbing/results/results_table.mas`, `see_order.mhtml`)
- **Results CSV export** (`web/tabbing/results/csv.mhtml`, `results_csv.mas`)
- **NSDA points** (`web/tabbing/results/nsda_points.mhtml`)
- **NSDA qualifiers** (`web/tabbing/results/nsda_qualifiers.mhtml`)
- **NSDA sweepstakes** (`web/tabbing/results/nsda_sweepstakes.mhtml`)
- **NSDA Nationals order** (`web/tabbing/results/nats_order.mas`)
- **Sweepstakes** (`web/tabbing/results/sweep_students.mas`, `sweep_schools.mas`, `sweep_tourn.mas`)
- **Top novice** (`web/tabbing/results/top_novice.mas`)
- **Roles** (`web/tabbing/results/roles.mhtml`)

*Break to Elims:*
- **Debate break** (`web/tabbing/break/break_debate.mhtml`) -- Seeds entries into elimination bracket based on prelim results. Creates elim round, assigns protocol/site/timeslot. Supports breakouts (sub-brackets by label).
- **Speech break** (`web/tabbing/break/break_speech.mhtml`)
- **Congress break** (`web/tabbing/break/break_congress.mhtml`)
- **WUDC break** (`web/tabbing/break/break_wudc.mhtml`)
- **NCFL snake** (`web/tabbing/break/ncfl_snake.mhtml`)
- **Ready status** (`web/tabbing/break/ready_status.mas`) -- Check if round is ready to break.

*Publishing:*
- **Generate results** (`web/tabbing/publish/generate_results.mhtml`) -- Materializes results into Result/ResultSet/ResultValue tables.
- **Generate bracket** (`web/tabbing/publish/generate_bracket.mas`) -- Creates bracket result sets.
- **Generate sweepstakes** (`web/tabbing/publish/generate_sweeps.mhtml`)
- **Publish all** (`web/tabbing/publish/publish_all.mhtml`, `publish_all_rounds.mhtml`)
- **Result set management** (`web/tabbing/publish/result_set_add.mhtml`, `result_set_switch.mhtml`, `result_set_adjust.mhtml`, `set_delete.mhtml`, `delete_result.mhtml`)
- **Upload results** (`web/tabbing/publish/upload_results.mhtml`)
- **Display** (`web/tabbing/publish/display.mhtml`)
- **Bracket views** (`web/tabbing/publish/bracket.mhtml`, `bracket_wudc.mhtml`)
- **File management** (`web/tabbing/publish/file_switch.mhtml`, `file_delete.mhtml`)
- **NSDA integration** (`web/tabbing/publish/register_nationals.mhtml`, `nationals_ranks.mhtml`)
- **SW District** (`web/tabbing/publish/swdistrict.mhtml`, `sw_nsda_students.mas`, `sw_nsda_save.mhtml`)

*Status Dashboard:*
- **Dashboard** (`web/tabbing/status/dashboard.mhtml`) -- Real-time overview of all events showing ballot entry completion, timing, and issues.
- **Panel confirm** (`web/tabbing/status/panel_confirm.mhtml`) -- Confirm panel completion.
- **Blast missing** (`web/tabbing/status/blast_missing.mhtml`) -- Notify about missing ballots.
- **Dashboard ignore** (`web/tabbing/status/dashboard_ignore.mhtml`) -- Mark items to ignore on dashboard.

*Reports:*
- **Score report** (`web/tabbing/report/score_report.mhtml`)
- **Raw ballots** (`web/tabbing/report/raw_ballots.mhtml`) -- Full raw ballot data dump.
- **Forfeits** (`web/tabbing/report/forfeits.mhtml`)
- **Violations** (`web/tabbing/report/violations.mhtml`)
- **Stats** (`web/tabbing/report/stats.mhtml`)
- **Reading/readingjudges** (`web/tabbing/report/reading.mhtml`, `readingjudges.mhtml`)
- **Judge work** (`web/tabbing/report/judge_work.mhtml`)
- **Congress scores** (`web/tabbing/report/congress_scores.mhtml`)
- **PO report** (`web/tabbing/report/po_report.mhtml`)
- **Roles** (`web/tabbing/report/roles.mhtml`)
- **Code lists** (`web/tabbing/report/code_list.mhtml`, `code_print.mhtml`, `codebreaker.mhtml`)
- **Awards** (`web/tabbing/report/awards_ceremony.mhtml`, `awards_school.mhtml`, `awards_script.mhtml`, `awards_refresh.mhtml`, `awards_pickup.mhtml`)
- **School results** (`web/tabbing/report/school.mhtml`, `school_results_print.mhtml`)
- **Sweepstakes** (`web/tabbing/report/sweep_schools.mhtml`, `sweep_students.mhtml`, `sweep_entries.mhtml`, `sweep_post.mhtml`, `sweep_schools_print.mhtml`, `sweep_students_print.mhtml`)
- **Pickup** (`web/tabbing/report/pickup.mhtml`, `pickup_switch.mhtml`)
- **Packet** (`web/tabbing/report/packet.mhtml`)
- **Pending ballots** (`web/tabbing/report/print_pending.mhtml`)
- **Round robin script** (`web/tabbing/report/round_robin_script.mhtml`)
- **Actual schedule** (`web/tabbing/report/actual_schedule.mhtml`)
- **Prelim order** (`web/tabbing/report/prelims_order.mhtml`)
- **Room cleaning** (`web/tabbing/report/room_cleaning.mhtml`)
- **NAUDL exports** (`web/tabbing/report/naudl_student_export.mhtml`, `naudl_tourn_export.mhtml`)
- **Legion** (`web/tabbing/report/legion_report.mhtml`, `send_legion.mhtml`)
- **CSV exports** (`web/tabbing/report/audit_csv.mas`, `last_round_csv.mhtml`, `event_speakers_csv.mhtml`)

*Sweepstakes:*
- **Sweepstakes entry/upload** (`web/tabbing/entry/sweeps.mhtml`, `sweeps_save.mhtml`, `upload_sweeps.mhtml`, `upload_sweeps_template.mhtml`)

**Key entities:**
- `Tab::Ballot` (`web/lib/Tab/Ballot.pm`) -- Central scoring entity. Links judge, panel, entry. Columns: id, judge, panel, entry, speakerorder, side, audit (0=unentered, 1+=entered), bye, forfeit, tv, approved, chair, seat, entered_by (Person), audited_by (Person), judge_started, started_by, timestamp. Has many: scores (Score).
- `Tab::Score` (`web/lib/Tab/Score.pm`) -- Individual score records. Columns: id, ballot, tag (winloss/rank/point/refute/best_po/po/subpoints/rubric/rfd/comments/tv), student, value, content (compressed text), speech, position, topic, timestamp. TEMP columns: panelid, entryid, roundtype, roundid, studentid, judgeid, bye, ballotid, chair, pullup.
- `Tab::Protocol` (`web/lib/Tab/Protocol.pm`) -- Tiebreak protocol definition. Columns: id, name, tourn, timestamp. Has many: tiebreaks, rounds, settings.
- `Tab::Tiebreak` (`web/lib/Tab/Tiebreak.pm`) -- Individual tiebreak rule. Columns: id, name, count (all/previous/specific/prelim/elim), count_round, truncate, truncate_smallest, multiplier, violation, result, priority, highlow, highlow_count, highlow_threshold, highlow_target, child (Protocol for composites), protocol, chair, timestamp.
- `Tab::Result` (`web/lib/Tab/Result.pm`) -- Materialized result record. Columns: id, rank, place, result_set, entry, student, school, round, panel, timestamp, percentile.
- `Tab::ResultSet` (`web/lib/Tab/ResultSet.pm`) -- Result set container. Columns: id, tag, label, tourn, event, circuit, qualifier, sweep_set, sweep_award, bracket, published, coach, code, generated, timestamp, cache.
- `Tab::ResultKey` (`web/lib/Tab/ResultKey.pm`) -- Metadata for result columns.
- `Tab::ResultValue` (`web/lib/Tab/ResultValue.pm`) -- Individual result values.
- `Tab::StudentVote` (`web/lib/Tab/StudentVote.pm`) -- Congress peer voting. Columns: id, tag, value, panel, entry, voter, entered_by, entered_at, timestamp.
- `Tab::ChangeLog` (`web/lib/Tab/ChangeLog.pm`) -- Audit trail for all changes.

**Key settings/configuration:**
- *Event settings:*
  - `point_increments` -- whole/half/tenths/fourths (step size for speaker points)
  - `team_points` -- Score at team level rather than student level
  - `ballot_rubric` -- Use rubric-based scoring
  - `no_lpw` -- Disallow low-point wins
  - `allow_lowpoints` -- Override LPW restriction
  - `combined_ballots` -- Combined ballot entry mode
  - `no_judge_violations` -- Disable time violation tracking
  - `breakouts` -- Number of breakout categories
  - `breakout_N_label` -- Breakout category labels
  - `min_content_points` / `max_content_points` -- WSDC sub-score ranges
  - `min_style_points` / `max_style_points`
  - `min_strategy_points` / `max_strategy_points`
  - `min_poi_points` / `max_poi_points`
  - `wsdc_subtotal_ballot` -- Enable WSDC sub-totaling
  - `online_mode` -- Online ballot submission mode
  - `not_nats` -- Exclude from NSDA Nationals special processing

- *Category settings:*
  - `ballot_entry_titles` -- Show titles on ballot entry
  - `ballot_times` -- Show times on ballot entry
  - `ballot_entry_names` / `ballot_entry_first_names` -- Show names on online ballots
  - `ballot_school_codes` / `ballot_school_names` -- Show school info on online ballots
  - `hide_codes` -- Hide entry codes
  - `rounds_per` -- Rounds per judge obligation

- *Protocol settings (via ProtocolSetting):*
  - `equal_elims` -- Equal elims format (section-based ranking)

- *Tiebreak configuration (via Tiebreak entity):*
  - Supported tiebreak names: `winloss`, `ballots`, `losses`, `ranks`, `reciprocals`, `points`, `opp_wins`, `opp_points`, `opp_ranks`, `opp_seed`, `headtohead`, `judgepref`, `judgevar`, `judgevar2`, `po_points`, `three_way_point`, `downs`, `preponderance`, `chair_ranks`, `non_chair_ranks`, `entry_vote_one`, `entry_vote_all`, `student_rank`, `best_po`
  - Count modes: `all`, `previous`, `specific` (round number), `prelim`, `elim`
  - Truncation: drop high/low N scores
  - Multiplier: weight factor
  - Chair-only: restrict to chair ballot
  - Child: composite tiebreaker referencing another Protocol
  - Violation: flag as violation rather than tiebreaker

**Key reports/exports:**
- Audit sheets (screen and print)
- Audit CSV
- Raw ballot dumps
- Score reports
- Awards scripts/ceremony reports
- School results
- Sweepstakes (students and schools)
- Code lists and codebreaker
- Speaker standings
- Results CSV
- Results table prints
- NSDA qualification reports
- NAUDL exports
- Legion reports
- Packet reports
- Forfeit reports
- Violation reports
- Judge work reports
- Congress score reports
- PO reports
- Bracket visualizations

**Integrations/dependencies:**
- **Pairing system** -- Ballot data is created when panels are paired (Ballot records created by pairing code). Results feed back into next-round pairing via `entry_wins.mas` and `make_pairing_hash.mas`.
- **Judge system** -- Ballots link to judges; online ballot entry authenticates via judge.person.
- **NSDA API** -- Results publishing can register nationals qualifiers, upload NSDA points.
- **NAUDL export** -- Student and tournament data export for NAUDL reporting.
- **Legion system** -- Alternate scoring/reporting system integration.
- **Tiebreak protocol system** -- Drives which score fields are shown on ballots and how results are computed.
- **ChangeLog system** -- All score modifications create audit trail entries.
- **indexcards API** -- Blast notifications for results.
- **ResultSet/Result materialization** -- Generated results stored for public display and circuit points.

**Documentation sources:**
- `order_entries.mas` contains extensive inline comments about tiebreak computation
- `tiebreak_types.mas` documents the mapping between tiebreak names and score tags
- `panel_save.mhtml` documents score processing logic inline
- `ballot_save.mhtml` documents validation rules

**Code sources:**
- `web/tabbing/` -- Primary directory (~100 files across subdirectories)
  - `web/tabbing/entry/` -- Ballot entry (30 files)
  - `web/tabbing/results/` -- Results computation and display (18 files)
  - `web/tabbing/break/` -- Break to elims (8 files)
  - `web/tabbing/publish/` -- Results publishing (23 files)
  - `web/tabbing/report/` -- Reports (50+ files)
  - `web/tabbing/status/` -- Status dashboard (6 files)
- `web/user/judge/ballot*.mhtml` -- Online ballot entry (8 files)
- `web/funclib/tiebreak_types.mas` -- Tiebreak type mapping
- `web/funclib/tiebreak_name.mas` -- Tiebreak display names
- `web/funclib/entry_wins.mas` -- Win computation
- `web/funclib/panel_scores.mas` -- Panel score retrieval
- `web/funclib/panel_winner.mas` -- Winner determination
- `web/funclib/results_congress.mas` -- Congress-specific results
- `web/funclib/blast_results.mas` -- Results blast notifications
- `web/lib/Tab/Ballot.pm`, `Score.pm`, `Protocol.pm`, `Tiebreak.pm`, `Result.pm`, `ResultSet.pm`, `ResultKey.pm`, `ResultValue.pm`, `StudentVote.pm`, `ChangeLog.pm`

**Complexity level:** High

The tiebreak computation engine (`order_entries.mas`) is one of the most complex pieces of code, supporting 25+ tiebreak types with composite tiebreakers, truncation, multipliers, round-specific counts, and format-specific logic (debate vs speech vs congress vs WUDC). The ballot save path must handle multiple score types simultaneously with validation. The break-to-elims logic must correctly seed entries across multiple breakout categories.

**Feature frequency / criticality:** Every tournament

Ballot entry and result computation happen at every tournament for every round. The audit system is critical for tournament integrity. Online ballots are used at most modern tournaments. Results publishing is the primary output of every tournament.

**Notes and open questions:**
- The `ballot.audit` field serves double duty: 0 means "not yet entered" and any positive value means "entered/confirmed." The exact semantics of values > 1 are unclear (possibly representing different audit states or double-entry confirmation).
- The `Score.content` field uses `Tab::Utils::compress/uncompress` for storing text data (RFDs, comments). The compression format should be documented.
- The relationship between `Protocol` and `Tiebreak` entities is many-to-many via the `child` field on Tiebreak -- a Tiebreak can reference another Protocol as a composite. This creates potential for circular references, which is explicitly checked in `order_entries.mas`.
- `order_entries.mas` has a hardcoded guard against composites-of-composites to prevent infinite loops and server overload.
- The `panel_save.mhtml` file (~350 lines) processes 10+ score types in a single function, making it a maintenance challenge.
- LPW (low-point win) validation is enforced at the online ballot save level but can be overridden by tab staff.
- The `ballot.tv` (time violation) and `ballot.approved` fields exist but their complete workflow is not fully documented.
- Congress scoring involves a separate `StudentVote` entity for peer evaluation, which is independent from the judge-based `Score` system.
- The `ResultSet.cache` field suggests result caching, but the cache invalidation strategy is unclear.
- NSDA-specific logic (districts, nationals, middle school nationals, billing) is deeply embedded throughout rather than abstracted.
