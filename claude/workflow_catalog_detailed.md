# Detailed Workflow Catalog — Release 1 Critical Workflows

## Overview

This document provides implementation-ready workflow specifications for the 10 most critical workflows in Tabroom. These are the workflows required for every tournament, and are prioritized for Release 1 of the rebuild.

Each workflow is documented with preconditions, triggers, detailed steps with system behavior, decision points, business rules, settings that alter behavior, format variations, error handling, outputs, and source file references.

**Key:** Throughout this document, "EAV setting" refers to the entity-attribute-value settings stored in `*_setting` tables (e.g., `event_setting`, `tourn_setting`, `category_setting`).

---

## 1. Pair a Debate Round

### Workflow Name
Pair a Debate Round (Powermatched / Preset / Bracket)

### Primary Role(s)
Tabber, Tournament Director

### Preconditions
- Event exists with type = "debate" (or "wsdc", "mock_trial")
- Round exists with a timeslot assigned
- Active entries exist in the event (entry.active = 1)
- For powermatched rounds: at least one prior round has completed results
- Protocol (tiebreaker set) assigned to the round
- Schedule is set with round numbers

### Trigger
Tabber navigates to Paneling > Sections, selects round, and clicks "Section" (or the round is included in a bulk "Section all" operation). The action dispatches to `panel_master.mhtml`.

### Steps

1. **Validate permissions and inputs** (`panel_master.mhtml`)
   - System checks the user has tabber/owner permission for the event
   - System determines pairing type from `round.type`:
     - "prelim" or "preset" -> `pair_preset.mas`
     - "highlow" -> `pair_debate.mas` (powermatched, top vs bottom within bracket)
     - "highhigh" -> `pair_debate.mas` (powermatched, top vs top within bracket)
   - If bulk operation, iterate over multiple events/rounds

2. **Dump existing pairing** (`/funclib/round_dump.mas`)
   - System deletes all existing panels, ballots, and scores for this round
   - Logged with person who initiated the action

3. **Gather entry data**
   - Load all active entries for the event
   - Load school and region information for each entry
   - Load hybrid strike/conflict data

4. **Determine pairing algorithm based on round type:**

   **4a. Preset Pairing** (`pair_preset.mas`)
   - Load opponent history for all entries across all prior prelim rounds
   - Determine side constraints: odd-numbered prior rounds = side-constrained (entries must switch sides)
   - If `sidelock_against` is set to a specific round, lock sides relative to that round
   - If round robin (`round_robin` setting), generate full round-robin schedule
   - For non-round-robin:
     - Score each potential matchup with penalties:
       - Same school: 1,000,000,000,000,000,000 penalty (effectively forbidden unless `school_debates_self`)
       - Hybrid conflict: same penalty as school
       - Hit before: 1,000,000,000,000 * number of times met
       - Wrong side (both need same side): 100,000,000,000
     - Build brackets based on win records from prior rounds
     - Sort entries by seed position within brackets
     - Greedily assign opponents from lowest-penalty candidates
     - Run iterative swap improvement (up to 100 passes) to find global minimum
   - Assign sides: snake by seed position if not side-constrained; honor side dues if constrained
   - Create Panel records, create Ballot records with side assignments

   **4b. Powermatched Pairing** (`pair_debate.mas`)
   - Compute results through `order_entries.mas` to determine current standings
   - Compute win-loss records via `entry_wins.mas`
   - Compute bye history via `entry_byes.mas`
   - Build brackets by win record (e.g., all 3-0s, all 2-1s, etc.)
   - Determine pullup candidates when brackets have odd numbers:
     - Score pullup desirability based on method:
       - "sop" (default): seed + opposition seeds — lower SOP gets pulled up
       - "oppwin": opponent wins — lower opp-wins gets pulled up
       - "lowseed": lowest seed gets pulled up (APDA style)
       - "middle": same as oppwin
     - Penalty for being pulled up twice unless `pullup_repeat` enabled
   - Score each potential matchup with weighted penalties:
     - Same school: 1e18 (unless `school_debates_self`)
     - Met before: 1e12 per meeting
     - Wrong side: 1e11 (or 1e8 if `pullup_minimize`)
     - Pullup across brackets: 1e4 (or 1e14 if `pullup_minimize`)
     - Position difference: scaled by rank^3
   - Greedy initial assignment, then iterative swap improvement (100 passes)
   - Side assignment: honor side dues from prior rounds; snake remaining
   - Handle BYE: if odd number of entries, insert synthetic BYE entry at worst bracket position
     - `autobye_nojudge`: automatically bye entries when insufficient judges, preferring entries with fewer prior byes
   - Create Panel and Ballot records

5. **Set round as paired**
   - `round.paired_at` = current timestamp
   - `round.update()`

6. **Redirect to schematic view** (`/panel/schemat/show.mhtml`)
   - Tabber reviews the generated pairing

7. **Manual adjustments** (optional)
   - Tabber can swap entries between panels (`/panel/schemat/move_debate.mhtml`)
   - Tabber can swap sides (`/panel/schemat/debate_sides_swap.mhtml`)

8. **Assign judges** (separate workflow, see Workflow 3)

9. **Assign rooms** (separate workflow)

10. **Publish** (see publish step in workflow notes)
    - Set `round.published = 1` (full) or `2` (without judges)
    - System calls `docshare_rooms.mas`, `online_usage.mas`, `publish_flips.mas`
    - Logged as tabbing action

### Decision Points
| Decision | Options | Determined By |
|----------|---------|---------------|
| Pairing algorithm | Preset / Powermatched (HL) / Powermatched (HH) | `round.type` |
| Side constraints | Constrained / Unconstrained | Odd/even prior round count + `no_side_constraints` setting |
| Pullup method | SOP / OppWin / LowSeed / Middle | `event_setting.pullup_method` |
| School self-debate | Allowed / Forbidden | `event_setting.school_debates_self` |
| Bye handling | Manual / Auto (no-judge) | `event_setting.autobye_nojudge` |
| Region constraints | None / Avoid / Constrain | `event_setting.region_constrain`, `event_setting.region_avoid`, `tourn_setting.ncfl` |
| Bracket basis | Win-loss / Ballots | `event_setting.bracket_by_ballots` |
| Round robin mode | Yes / No | `event_setting.round_robin` |

### Key Business Rules Invoked
- BR-PAIR-001: Entries from the same school must not debate each other (unless `school_debates_self`)
- BR-PAIR-002: Entries should not meet the same opponent twice (1e12 penalty per repeat)
- BR-PAIR-003: Side balance must be maintained in side-constrained rounds
- BR-PAIR-004: Pullups go to the weakest entry in the lower bracket (by pullup method)
- BR-PAIR-005: An entry should not be pulled up more than once (unless `pullup_repeat`)
- BR-PAIR-006: BYE goes to the lowest-seeded entry
- BR-PAIR-007: Hybrid entries inherit school conflicts from both schools

### Settings That Affect Behavior
| Setting | Table | Effect |
|---------|-------|--------|
| `pullup_method` | event_setting | SOP/oppwin/lowseed/middle |
| `pullup_repeat` | event_setting | Allow same entry pulled up multiple times |
| `pullup_minimize` | event_setting | Dramatically increase pullup penalty |
| `powermatch` | event_setting | SOP vs seed ordering within brackets |
| `no_side_constraints` | event_setting | Disable automatic side balancing |
| `school_debates_self` | event_setting | Allow same-school matchups |
| `bracket_by_ballots` | event_setting | Use ballot count instead of win count for brackets |
| `region_constrain` | event_setting | Treat region like school for constraints |
| `region_avoid` | event_setting | Penalize same-region matchups |
| `round_robin` | event_setting | Generate full round-robin schedule |
| `panel_labels` | event_setting | "letters" uses A,B,C; default uses 1,2,3 |
| `autobye_nojudge` | event_setting | Auto-assign byes when judges insufficient |
| `hybrids_can_hit` | event_setting | Allow hybrid entries to hit their other school |
| `prevent_hitting_pullup_twice` | event_setting | Extra penalty for hitting a pullup opponent again |
| `aff_label` / `neg_label` | event_setting | Custom side labels |
| `nsda_district` | tourn_setting | District tournament special rules |
| `ncfl` | tourn_setting | NCFL region-as-school behavior |

### Format Variations
- **Debate (standard):** Full workflow as described
- **WUDC:** Uses `pair_wudc.mas` — 4-team bracket pairing, completely different algorithm
- **WSDC:** Treated as debate with additional team-point scoring
- **Mock Trial:** Treated as debate type
- **Congress/Speech:** Use completely different workflows (Workflows 2)

### Error/Exception Handling
- No entries in event: system redirects to schedule setup page
- Round 1 set to powermatched: error message "impossible to powermatch round 1"
- No timeslot on round: error message, abort
- Odd entries without available BYE: system auto-creates BYE entry
- District round > 2 set to preset: system auto-changes to powermatched with warning

### Outputs/State Changes
- **Created:** Panel records (one per debate), Ballot records (one per entry per judge per panel)
- **Modified:** `round.paired_at` timestamp set
- **Ballot fields set:** `entry`, `panel`, `side`, `bye`, `audit` (bye panels auto-audited)
- **Panel fields set:** `round`, `letter`, `flight`, `bracket`, `bye`

### Post-conditions
- Every active entry appears in exactly one panel (or has a BYE)
- No same-school matchups exist (unless allowed)
- Side constraints are satisfied where possible
- `round.paired_at` is set to current time

### Source Files
- `/web/panel/round/panel_master.mhtml` — orchestrator/dispatcher
- `/web/panel/round/pair_debate.mas` — powermatched pairing (highlow/highhigh)
- `/web/panel/round/pair_preset.mas` — preset pairing
- `/web/panel/round/pair_bracket.mhtml` — bracket/elim pairing
- `/web/panel/round/pair_wudc.mas` — WUDC 4-team pairing
- `/web/panel/schemat/show.mhtml` — schematic display
- `/web/panel/publish/publish_switch.mhtml` — publish toggle
- `/web/funclib/round_dump.mas` — clear existing pairing
- `/web/funclib/entry_wins.mas` — compute win records
- `/web/funclib/entry_byes.mas` — compute bye history
- `/web/tabbing/results/order_entries.mas` — compute standings/seeds

---

## 2. Panel a Speech Round

### Workflow Name
Panel a Speech Round (Snake Algorithm)

### Primary Role(s)
Tabber, Tournament Director

### Preconditions
- Event exists with type = "speech"
- Round exists with timeslot assigned
- Active entries exist in the event
- Number of sections (panels) determined (based on min/max/default panel size settings)
- For snaked rounds (round > 1): prior round results exist

### Trigger
Tabber navigates to Paneling > Sections for the event, sets number of sections, and clicks "Section." Dispatched through `panel_master.mhtml`.

### Steps

1. **Determine number of sections**
   - System calculates based on: `min_panel_size` (default 5), `max_panel_size` (default 8), `default_panel_size` (default 6)
   - `num_panels = ceil(num_entries / default_panel_size)`
   - Tabber can override the calculated number
   - `max_size = ceil(entries / panels)`, `min_size = floor(entries / panels)`

2. **Dispatch to algorithm** (`panel_master.mhtml`)
   - For round 1 (prelim): `pair_speech.mas` (cold snake, no prior data)
   - For subsequent prelims or snake-seeded rounds: `snake_speech.mas` (seeded snake)

3. **Cold Snake — Round 1** (`pair_speech.mas`)
   - Load all active entries with school and region data
   - Build hit matrix from any prior rounds in this event
   - Define penalty weights:
     - Same school in section: 1,000,000 (district: 1,000,000,000)
     - NSDA Nats: school = 100,000; district = 100,000; region = 100,000; repeat = 100,000; speaker order = 100
     - Same entry repeated in section: 1,000 (district: 100)
     - Same school repeated: 10
     - Region constrain: 1,000,000; region avoid: 100,000
   - Iterate through entries, greedily assign each to the section with lowest conflict score
   - Run 7 iterations of pairwise swap improvement:
     - For each section with score > 0, try swapping each entry with entries from other sections
     - Accept swap if combined score decreases
   - **Speaker order assignment:**
     - Track each entry's prior speaking positions
     - Sort entries to minimize repeat positions
     - Handle double-entry scheduling conflicts (entries competing in two events simultaneously)
     - `speaker_priority_first`: if set, double-entered competitors speak first

4. **Seeded Snake — Subsequent Rounds** (`snake_speech.mas`)
   - Compute standings from prior round via `order_entries.mas`
   - Assign seeds based on standings (place = seed)
   - Build hit matrix: who has been in sections with whom in prior rounds
   - Track "last hit" (entries in same section in immediately prior round)
   - Penalty weights for snaked rounds:
     - Same school: 1,000,000
     - Same region: 100,000 (if tournament has regions) or 1,000
     - Seed average deviation: 1,000
     - Last-round hit: 100
     - Any prior hit: 10
   - **Initial snake:** Sort entries by seed, snake across sections (1→N, N→1, 1→N...)
   - **School conflict resolution:** If a school conflict exists in a section, find the closest-seed entry to swap with from another section (shift chain swap to minimize disruption)
   - **Iterative improvement (4 passes):** For each section with score > 0, try swapping entries with entries from other sections within 2 seed positions

5. **Save panels to database**
   - Create Panel records (or reuse existing)
   - Create Ballot records for each entry in each section, with `speakerorder` set
   - Assign existing judges to ballots if judges were pre-assigned

6. **Set round paired**
   - `round.paired_at` = now

7. **Review and adjust**
   - Tabber reviews sections on schematic view
   - Manual adjustments via `move_speech.mhtml` if needed

### Decision Points
| Decision | Options | Determined By |
|----------|---------|---------------|
| Number of sections | Calculated / Manual override | Panel size settings + tabber input |
| Algorithm | Cold snake / Seeded snake | Round number (1 vs >1) and `round.type` |
| School conflict handling | Penalty-based / Forbidden | Tournament type (district vs regular) |
| Region handling | None / Avoid / Constrain | `region_constrain`, `region_avoid`, `ncfl`, `nsda_nats` |
| Seed basis | Prior round results / Custom protocol / Pairing seed | `seed_basis` parameter, `seed_presets`, `seed_round` |
| Entry limit (cuts) | All entries / Top N | `limit_to` parameter |
| Speaker order priority | First / Late | `speaker_priority_first` |
| Section labels | Numbers / Letters | `panel_labels` setting |

### Key Business Rules Invoked
- BR-SPEECH-001: Entries from the same school must not appear in the same section (1M penalty)
- BR-SPEECH-002: Entries should not repeat opponents across rounds (penalty-weighted)
- BR-SPEECH-003: Speaker order should vary across rounds — no entry should speak in the same position twice
- BR-SPEECH-004: Section strength should be balanced (seed average penalty)
- BR-SPEECH-005: Double-entered students get accommodated speaker order positions
- BR-SPEECH-006: School percentage limit (`school_percent_limit`) can cap how many sections a large school appears in

### Settings That Affect Behavior
| Setting | Table | Effect |
|---------|-------|--------|
| `min_panel_size` | event_setting | Minimum entries per section (default 5) |
| `max_panel_size` | event_setting | Maximum entries per section (default 8) |
| `default_panel_size` | event_setting | Target section size (default 6) |
| `panel_labels` | event_setting | "letters" for A/B/C, numbers otherwise |
| `region_constrain` | event_setting | Treat region as hard constraint |
| `region_avoid` | event_setting | Penalize same-region in section |
| `speaker_priority_first` | event_setting | Double-entered students speak first |
| `school_percent_limit` | event_setting | Max % of sections a school can appear in |
| `seed_presets` | event_setting | Use prior round results as seeds for preset rounds |
| `seed_round` | round_setting | Specific round to use for seeding |
| `ask_for_titles` | event_setting | Track piece titles (NSDA Nats uses for title-clash avoidance) |
| `nsda_district` / `nsda_nats` / `ncfl` | tourn_setting | Special tournament penalty weights |

### Format Variations
- **Speech (standard):** Full workflow as described
- **Congress:** Uses `pair_congress.mas` — chamber assignment with PO tracking, legislation assignment, recency tracking
- **Debate:** Uses completely different workflow (Workflow 1)

### Error/Exception Handling
- Zero sections requested: error message "cannot create zero sections"
- No entries: event page shows "no entries" message
- No prior round for seeded snake: returns without action
- Flight B round tracking: second-flighted sections assigned to alternate round record

### Outputs/State Changes
- **Created:** Panel records, Ballot records with `speakerorder`
- **Modified:** `round.paired_at` timestamp
- **Ballot fields:** `entry`, `panel`, `speakerorder`, optionally `judge`

### Post-conditions
- Every active entry appears in exactly one section
- No same-school entries share a section (if achievable)
- Speaker order varies from prior rounds
- Section sizes are balanced (differ by at most 1)

### Source Files
- `/web/panel/round/panel_master.mhtml` — orchestrator
- `/web/panel/round/pair_speech.mas` — cold snake (round 1)
- `/web/panel/round/snake_speech.mas` — seeded snake (round 2+)
- `/web/panel/round/event.mhtml` — section count configuration UI
- `/web/panel/round/speaker_order.mhtml` — speaker order management
- `/web/panel/round/speaker_order_improve.mhtml` — speaker order optimization

---

## 3. Assign Judges to a Round

### Workflow Name
Assign Judges to a Debate Round

### Primary Role(s)
Tabber, Tournament Director

### Preconditions
- Round is paired (panels and ballots exist)
- Judge pools (JPools) assigned to the round
- Judges registered and assigned to pools
- Category has pref/rating configuration set
- Timeslot assigned to round

### Trigger
Tabber clicks "Assign Judges" on the schematic view page. Dispatches to `debate_judge_assign.mhtml`.

### Steps

1. **Clear existing judge assignments** (`/funclib/round_clear_judges.mas`)
   - Remove all current judge assignments from ballots for this round
   - Preserve bye panels (do not assign judges to byes)
   - Optionally limit to a bracket range (`min_bracket`, `max_bracket`)

2. **Load entry data for all panels**
   - Query all ballots across the entire category (not just this event) to build complete judge history
   - Track for each entry: school, region, area (diocese region), event, side, seed
   - Track which panels are online-hybrid
   - Build panel-to-entries mapping

3. **Load judge data**
   - Query all judges in assigned JPools for this round
   - For each judge, determine:
     - School affiliation and conflicts
     - Region and area (diocese)
     - Prior round assignments (how many rounds judged, which entries seen)
     - Availability for this timeslot (check against shifts, other commitments)
     - Tab rating and coach preference ratings/strikes
     - Online/hybrid capability
   - Calculate `rounds_owed` = obligation minus rounds already judged

4. **Build conflict matrix**
   - Hard conflicts (judge cannot see entry):
     - Same school as entry (unless event allows)
     - Explicit strike/conflict in database
     - Hybrid school conflict
     - Region conflict (if `ncfl` or `region_constrain`)
   - Soft preferences (weighted penalties):
     - Tab rating: higher-rated judges assigned to higher-bracket debates
     - Coach prefs: ordinal/tiered/percentage ratings translated to numerical penalty
     - Prior judging: penalty for judging same entry again
     - Burden balance: penalty for over/under-judging relative to obligation

5. **Score each judge-panel combination**
   - For each panel, for each eligible judge:
     - Sum conflict penalties (school, region, entry-seen-before)
     - Add pref penalty (based on coach ratings for entries in panel)
     - Add burden penalty (judges who owe more rounds get preference)
     - For multi-judge panels: chair assignment based on highest tab rating
   - Account for `best_judges_highest_seed`: if set, highest-seeded entries get best-rated judges

6. **Greedy assignment with optimization**
   - Sort panels by bracket (highest bracket = most wins = highest priority)
   - For each panel, assign the judge(s) with lowest total penalty
   - Handle flighted rounds: judge can cover both flights unless timing conflict
   - Handle `num_judges` > 1: assign multiple judges, designate chair
   - Run swap optimization to improve overall assignment quality

7. **Save assignments**
   - Update ballot records: set `ballot.judge` for each entry-judge combination
   - Set `ballot.chair` for chair judges in multi-judge panels

8. **Log the action**
   - Record: "Re-assigned the judges for round X of Event"
   - Include bracket range if specified

### Decision Points
| Decision | Options | Determined By |
|----------|---------|---------------|
| Number of judges per panel | 1 / 3 / custom | `round_setting.num_judges` |
| Pref scheme | None / Ordinal / Tiered / Percentile | `category_setting.prefs` |
| Tab ratings used | Yes / No | `category_setting.tab_ratings` |
| Coach ratings used | Yes / No | `category_setting.coach_ratings` |
| Best judges to top seeds | Yes / No | `event_setting.best_judges_highest_seed` |
| Invert ratings | Yes / No | `event_setting.invert_ratings` |
| Flighting | 1 / 2+ flights | `round.flighted` |
| Judge can see same school | No / Yes | `event_setting.school_debates_self` |
| Region constraints on judges | None / Avoid / Hard | `ncfl`, `region_constrain` |

### Key Business Rules Invoked
- BR-JUDGE-001: Judge must not be from same school as any entry in panel
- BR-JUDGE-002: Judge must not have a recorded strike/conflict against any entry
- BR-JUDGE-003: Judges should be balanced in burden (rounds judged vs obligation)
- BR-JUDGE-004: Coach preferences should be respected (weighted by pref system)
- BR-JUDGE-005: Chair judge should be the highest-rated judge on multi-judge panels
- BR-JUDGE-006: Judge should not judge the same entry more than once if avoidable
- BR-JUDGE-007: Bye panels get no judges

### Settings That Affect Behavior
| Setting | Table | Effect |
|---------|-------|--------|
| `num_judges` | round_setting | Judges per panel (1 or 3 typical) |
| `prefs` | category_setting | Pref scheme type |
| `tab_ratings` | category_setting | Enable tab quality ratings |
| `coach_ratings` | category_setting | Enable coach ratings |
| `no_prefs` | event_setting | Disable prefs for this event |
| `best_judges_highest_seed` | event_setting | Assign best judges to top seeds |
| `invert_ratings` | event_setting | Invert rating scale |
| `online_hybrid` | event_setting | Track online/in-person compatibility |
| `flight_rooms_only` | event_setting | Flights share rooms but not judges |
| `rounds_per` | category_setting | Judge obligation in rounds |

### Format Variations
- **Debate:** Full workflow as described; multi-judge panels common in elims
- **Speech:** Judge assignment integrated into `pair_speech.mas`; judges assigned directly to sections with speaker-order ballots
- **Congress:** Judges are panelists/scorers assigned to chambers; PO is a separate role

### Error/Exception Handling
- No timeslot: error "assigning judging is impossible without a timeslot"
- Insufficient judges: system assigns available judges, leaves panels unjudged
- All judges conflicted for a panel: panel left without judge (flagged in disaster check)

### Outputs/State Changes
- **Modified:** Ballot records updated with `judge` and `chair` fields
- **Logged:** Tabbing log entry for judge assignment

### Post-conditions
- Every non-bye panel has the required number of judges assigned
- No hard conflicts exist between assigned judges and entries
- Judge burden is approximately balanced

### Source Files
- `/web/panel/round/debate_judge_assign.mhtml` — main judge assignment algorithm
- `/web/panel/round/judges.mhtml` — judge overview/status
- `/web/panel/round/manual_judges.mhtml` — manual judge adjustments
- `/web/panel/round/manual_judge_save.mhtml` — save manual changes
- `/web/panel/round/mass_judges.mhtml` — bulk judge operations
- `/web/funclib/round_clear_judges.mas` — clear existing assignments
- `/web/funclib/round_available_judges.mas` — compute available judge count

---

## 4. Enter a Ballot Online (Judge)

### Workflow Name
Online Ballot Entry by Judge

### Primary Role(s)
Judge

### Preconditions
- Judge has a Tabroom account linked to their judge record (`judge.person` is set)
- Judge is assigned to a panel (ballot records exist with this judge and panel)
- Round is published (`round.published > 0`)
- Ballots are not yet confirmed (`ballot.audit = 0`)

### Trigger
Judge logs into Tabroom, navigates to their judging assignments (home page shows current assignments), clicks on the round/panel to enter ballot. System loads `ballot_confirm.mhtml` which routes to `ballot.mhtml`.

### Steps

1. **Authentication and authorization** (`ballot_confirm.mhtml`)
   - Verify `judge.person.id == person.id` (or person is site_admin)
   - Find ballots for this judge + panel combination
   - Verify ballots exist and are unconfirmed (`audit = 0`)
   - If already confirmed: redirect with "ballots were already confirmed" message

2. **Load ballot form** (`ballot.mhtml`)
   - De-duplicate panel ballots (`panel_dedupe.mas`)
   - Load event settings, round settings, category settings
   - Determine ballot type from `event.type`:
     - Debate: win/loss decision + speaker points (per student)
     - Speech: ranks (+ optional points)
     - Congress: ranks/scores + PO scoring + legislation votes
     - WSDC: redirect to `wsdc_ballot.mhtml` for sub-scored ballots
     - Legion format: redirect to `legion_ballot.mhtml`
   - Determine which score types are needed via `tiebreak_types.mas`:
     - `point`: speaker points needed
     - `rank`: ranks needed
     - `ballot`: win/loss decision needed
     - `tv`: time violation tracking (unless `no_judge_violations`)
   - Load point range constraints:
     - `min_points` / `max_points` from event settings
     - `point_increments`: whole/half/tenths/fourths
     - Sub-scores for WSDC: Content, Style, Strategy, POI
   - Display bias statement (if tournament started after Aug 2021 or `bias_statement` set)
   - Show ballot header if NSDA tournament
   - Display entry names based on category settings:
     - `ballot_entry_names`: show entry names on ballot
     - `ballot_entry_first_names`: show first names
     - `ballot_school_names` / `ballot_school_codes`: show school info

3. **Judge enters scores**
   - **Debate ballot:**
     - Select winner (side 1 / side 2)
     - Enter speaker points for each student on each team
     - Validate: no low-point wins unless `no_lpw` is false or `allow_lowpoints` is set
     - Enter RFD (reason for decision) — text field
     - Enter comments per entry — text field
     - Optional: time violations, rubric scores
   - **Speech ballot:**
     - Enter rank for each entry (1 = best, N = worst)
     - Validate: no duplicate ranks
     - Enter optional points
     - Enter comments per entry
   - **Congress ballot:**
     - Enter ranks and/or scores per entry
     - Enter PO scores
     - Enter student vote results (if applicable)

4. **Submit ballot** (`ballot_save.mhtml`)
   - Validate all required fields are present
   - Authorization check: `judge.person.id == person.id` or site_admin
   - Find unconfirmed ballots (`ballot.audit = 0`)
   - Process time entries (if `ballot_times` category setting)
   - Save RFD as Score record (`tag = 'rfd'`)
   - Save per-entry comments as Score records (`tag = 'comments'`)
   - **For debate:**
     - Save win/loss as Score record (`tag = 'ballot'`, `value = 1` for winner)
     - Save speaker points per student as Score records (`tag = 'point'`)
     - Validate point ranges against min/max settings
     - Check for low-point wins
   - **For speech:**
     - Save ranks as Score records (`tag = 'rank'`)
     - Validate rank uniqueness
   - Save time violation scores if applicable
   - Handle dropped entries: auto-audit those ballots

5. **Confirm ballot** (`ballot_confirm.mhtml`)
   - Set `ballot.audit = 1` for all ballots for this judge + panel
   - Set `ballot.entered_by = person.id`
   - Record timestamp
   - Check if all judges in panel have now submitted: if so, panel is complete
   - Display confirmation page with submitted scores
   - Show link to RFD if entered

6. **Notification**
   - Tab room status dashboard updates to show ballot received
   - If all ballots for round are in, round status shows complete

### Decision Points
| Decision | Options | Determined By |
|----------|---------|---------------|
| Ballot format | Debate/Speech/Congress/WSDC/Legion | `event.type` + tournament settings |
| Score types needed | Points/Ranks/Wins/TV | Protocol tiebreaker configuration |
| Low-point win allowed | Yes / No | `event_setting.no_lpw`, `event_setting.allow_lowpoints` |
| Point increment | Whole/Half/Tenths/Fourths | `event_setting.point_increments` |
| RFD required | Yes / No | Category/event settings |
| Names visible | Yes / No | `category_setting.ballot_entry_names` |
| Sub-scores | None / WSDC categories | `event.type == wsdc` |

### Key Business Rules Invoked
- BR-BALLOT-001: Only the assigned judge (or site admin) can enter/confirm the ballot
- BR-BALLOT-002: Speaker points must fall within configured min/max range
- BR-BALLOT-003: Low-point wins are flagged/prevented unless explicitly allowed
- BR-BALLOT-004: Ranks must be unique per judge (no ties in speech)
- BR-BALLOT-005: Once confirmed (audit=1), ballot cannot be re-entered by judge (must contact tab)
- BR-BALLOT-006: Dropped/inactive entries are auto-audited
- BR-BALLOT-007: Bias statement must be shown (post-Aug 2021 tournaments)

### Settings That Affect Behavior
| Setting | Table | Effect |
|---------|-------|--------|
| `no_lpw` | event_setting | Prevent low-point wins |
| `allow_lowpoints` | event_setting | Override LPW prevention |
| `point_increments` | event_setting | Point granularity |
| `min_points` / `max_points` | event_setting | Point range |
| `ballot_entry_names` | category_setting | Show names on ballot |
| `ballot_school_names` | category_setting | Show schools on ballot |
| `ballot_times` | category_setting | Track speech times |
| `bias_statement` | tourn_setting | Custom bias statement text |
| `no_judge_violations` | event_setting | Hide TV entry |
| `team_points` | event_setting | Points at team not student level |
| `ballot_rubric` | event_setting | Use rubric-based scoring |

### Format Variations
- **Debate:** Win/loss + student-level points; RFD field
- **Speech:** Ranks (1-N), optional points; comments per entry
- **Congress:** Ranks/scores + PO scoring + student vote tracking
- **WSDC:** Sub-scored ballots (Content, Style, Strategy, POI) with separate min/max per category

### Error/Exception Handling
- Judge not assigned to panel: redirect with error
- Ballots already confirmed: redirect with "already confirmed" message
- Invalid point range: validation error on submission
- Missing required fields: form validation prevents submission
- Duplicate ranks in speech: validation error

### Outputs/State Changes
- **Created:** Score records (tag: ballot, point, rank, rfd, comments, tv, time, etc.)
- **Modified:** Ballot records: `audit` set to 1, `entered_by` set to person ID
- **State:** Panel completion status updates (visible on status dashboard)

### Post-conditions
- All score records exist for this judge's ballot
- Ballot.audit = 1 (confirmed)
- Ballot.entered_by = judge's person ID
- Status dashboard reflects updated completion

### Source Files
- `/web/user/judge/ballot.mhtml` — ballot entry form (main display)
- `/web/user/judge/ballot_save.mhtml` — process and save ballot data
- `/web/user/judge/ballot_confirm.mhtml` — confirm/finalize ballot
- `/web/user/judge/ballot_rubric.mas` — rubric-based ballot display
- `/web/user/judge/ballot_recency.mas` — congress recency tracking
- `/web/funclib/tiebreak_types.mas` — determine which score types needed
- `/web/funclib/panel_dedupe.mas` — clean up duplicate ballot records

---

## 5. Enter a Ballot (Tab Room)

### Workflow Name
Tab Room Ballot Entry (with Double-Entry and Audit)

### Primary Role(s)
Tabber, Tab Room Staff

### Preconditions
- Round is paired with panels and ballots existing
- User has tabber permission (not checker-only)
- Protocol (tiebreaker set) assigned to the round

### Trigger
Tab room staff navigates to Tabbing > Ballot Entry, selects a timeslot or event, then clicks on a specific panel to enter scores. Can also navigate from schematic view via panel link.

### Steps

1. **Navigate to ballot entry dashboard** (`/tabbing/status/status.mhtml`)
   - System displays all rounds for the selected timeslot/event
   - For each panel, shows status:
     - Ballot submitted (audit = 1) vs pending
     - Judge started (judge_started flag)
     - Bye/forfeit status
   - Color-coded: green = complete, yellow = partial, red = missing

2. **Select panel for entry** (`/tabbing/entry/panel.mhtml`)
   - Load panel data: entries, judges, existing scores
   - Verify tabber permission (not checker-only)
   - Load event settings for point ranges, score types
   - Determine ballot structure from tiebreak types
   - Display ballot entry form with:
     - Entry codes and names
     - Student names for student-level scoring
     - Score input fields based on protocol
     - Bye/forfeit toggle buttons
     - Side labels (aff/neg for debate)

3. **Enter scores** (staff fills in paper ballot data)
   - **Debate:**
     - Select winner via win/loss radio buttons or dropdown
     - Enter speaker points per student
     - Enter optional RFD and comments
     - Mark bye or forfeit if applicable
   - **Speech:**
     - Enter rank for each entry
     - Enter optional points
   - **Congress:**
     - Enter ranks and scores
     - Enter PO scores
     - Student vote entry

4. **Save ballot** (`/tabbing/entry/panel_save.mhtml`)
   - Process each ballot for entries in the panel:
     - Clean up duplicate ballots (delete extras)
     - Process scores by tag type:
       - `rank`: save/update rank score; validate no duplicates (in student-level scoring)
       - `point`: save/update speaker points per student
       - `ballot`: save/update win/loss decision
       - `rubric`: save rubric category scores
       - `po`: save PO scores (Congress)
     - Handle bye: if `bye_[ballot_id]` set, mark ballot as bye
     - Handle forfeit: if `forfeit_[ballot_id]` set, mark ballot as forfeit
   - For WSDC: process sub-scores (Content, Style, Strategy, POI)
   - Track changes for audit trail
   - Set `ballot.audit = 1` and `ballot.entered_by = person.id`
   - Log the change

5. **Double-entry verification** (optional)
   - If tournament uses double-entry:
     - First entry saves scores with `ballot.audit = 0` (entered but not verified)
     - Second staff member enters same ballot independently
     - System compares the two entries
     - If they match: `ballot.audit = 1` (verified)
     - If discrepancy: flag for resolution, show differences
   - Screen audit mode (`/tabbing/entry/screen_audit.mhtml`): review and approve entered ballots on screen

6. **Audit** (`/tabbing/entry/audit.mhtml`)
   - Tabber reviews entered ballots
   - Can mark ballots as audited (`ballot.audit = 1`)
   - Records `ballot.audited_by` with person ID
   - Audit flag tracks verification level

7. **Closeout** (`/tabbing/entry/closeout.mhtml`)
   - Batch operation to close out remaining ballots
   - Handle blanks correction (`blanks_correct.mhtml`)
   - Combined entry mode (`combined.mhtml`) for rapid multi-ballot entry

### Decision Points
| Decision | Options | Determined By |
|----------|---------|---------------|
| Score entry level | Team / Student | `event_setting.team_points`, event type |
| Double-entry required | Yes / No | Tournament practice (not a system setting, procedural) |
| Rapid entry mode | Normal / Rapid | Tabber choice (`rapid.mhtml`) |
| Audit mode | Screen / Paper | Tabber choice |
| Combined entry | Normal / Combined | Tabber choice (for multi-judge panels) |
| Point granularity | Whole/Half/Tenths/Fourths | `event_setting.point_increments` |

### Key Business Rules Invoked
- BR-TAB-001: Only tabbers (not checkers) can enter ballots from admin side
- BR-TAB-002: Double-entry requires matching scores to confirm
- BR-TAB-003: Audit trail must record who entered and who audited
- BR-TAB-004: Bye ballots auto-set audit = 1
- BR-TAB-005: Duplicate ballot records are automatically cleaned up
- BR-TAB-006: Dropped entry ballots auto-audited

### Settings That Affect Behavior
| Setting | Table | Effect |
|---------|-------|--------|
| `point_increments` | event_setting | Point entry granularity |
| `min_points` / `max_points` | event_setting | Enforced point range |
| `team_points` | event_setting | Points at team level |
| `ballot_rubric` | event_setting | Rubric-based scoring |
| `ballot_entry_titles` | category_setting | Show piece titles |
| `ballot_times` | category_setting | Track speech times |
| `wsdc_subtotal_ballot` | event_setting | WSDC sub-score entry |
| `status_by_entry` | event_setting | Track status per entry vs per ballot |
| `dont_poke_entries` | event_setting | Disable entry status checking |
| `backtab` | tourn_setting | High-precision decimal points (.001) |

### Format Variations
- **Debate:** Win/loss + student speaker points; side-specific display
- **Speech:** Ranks per entry, optional points; speaker order display
- **Congress:** Ranks + scores + PO + student votes; multiple score types
- **WSDC:** Sub-category points (Content, Style, Strategy, POI) per student

### Error/Exception Handling
- Panel not found: abort with "no such panel"
- No tiebreaker set: redirect with "you do not have tiebreakers set"
- Duplicate ballot records: auto-cleaned during save
- Invalid point values: saved but may cause warnings in results

### Outputs/State Changes
- **Created/Modified:** Score records for each score type
- **Modified:** Ballot records: `audit`, `entered_by`, `bye`, `forfeit`, `chair`, `tv`
- **Modified:** Panel records: `bye` (panel-level bye)
- **Logged:** Change descriptions for audit trail

### Post-conditions
- Scores exist for all required score types
- Ballot.audit = 1 for completed ballots
- Status dashboard reflects updated state

### Source Files
- `/web/tabbing/entry/panel.mhtml` — ballot entry form (tab room)
- `/web/tabbing/entry/panel_save.mhtml` — save ballot data
- `/web/tabbing/entry/audit.mhtml` — audit interface
- `/web/tabbing/entry/screen_audit.mhtml` — screen audit mode
- `/web/tabbing/entry/screen_audit_save.mhtml` — save screen audit
- `/web/tabbing/entry/combined.mhtml` — combined multi-judge entry
- `/web/tabbing/entry/combined_audit.mhtml` — combined audit
- `/web/tabbing/entry/rapid.mhtml` — rapid entry mode
- `/web/tabbing/entry/closeout.mhtml` — batch closeout
- `/web/tabbing/entry/blanks_correct.mhtml` — fix blank ballots
- `/web/tabbing/entry/index.mhtml` — ballot entry event selection
- `/web/tabbing/status/status.mhtml` — status dashboard

---

## 6. Compute Results for a Round

### Workflow Name
Compute and Display Round Results/Standings

### Primary Role(s)
Tabber (initiator), System (computation)

### Preconditions
- Round has been paired
- Ballots have been entered (at least partially)
- Protocol (tiebreaker set) assigned to the round via `round.protocol`
- Protocol has tiebreakers configured with priorities

### Trigger
Results are computed automatically when:
- Tabber views schematic page (results shown inline)
- Tabber navigates to Results pages
- Tabber initiates a break (calls `order_entries.mas` internally)
- Powermatching next round (calls `order_entries.mas` for seeding)

### Steps

1. **Determine protocol and tiebreakers** (`order_entries.mas`)
   - Load `round.protocol` — the tiebreaker set (Protocol record)
   - Load protocol settings
   - Load event and tournament settings
   - Verify protocol exists; if not, abort with error
   - If NSDA Nats speech: override to "IE Prelim Composite" protocol
   - Safety check: reject composite-of-composite tiebreakers (infinite loop prevention)
   - Safety check: reject "opp_seed" in composite sets (self-referential)

2. **Build tiebreaker configuration**
   - For each tiebreak in the protocol (ordered by priority):
     - Record: name, count scope (prelim/elim/previous/specific/all), multiplier
     - Record: high-low drop configuration (drop highest N, drop lowest N, threshold)
     - Record: chair-only flag, violation flag
     - Record: child protocol reference (for composite tiebreakers)
     - Record: truncation settings
   - Group tiebreakers by priority tier

3. **Gather raw scoring data**
   - Query all ballots, scores, and panels across all rounds of the event
   - Filter based on tiebreaker count scope:
     - "prelim": only prelim rounds up to current
     - "elim": only elim/final rounds
     - "previous": all rounds before current
     - "specific": only a specific round
     - "all": everything
   - Exclude rounds with `ignore_results` setting
   - Exclude non-score tags: rfd, comments, time, title, categories, rubric, po, strike
   - Track per ballot: entry, judge, scores, bye status, forfeit status, panel bye

4. **Calculate tiebreakers per entry**
   - For each entry, for each tiebreaker:
     - **Wins ("winloss"):** Count ballots where `tag='ballot' AND value=1`
     - **Ballots ("ballots"):** Raw ballot wins
     - **Points ("point"):** Sum speaker points across ballots
     - **Ranks ("rank"):** Sum/average ranks
     - **Reciprocals ("recip"):** Sum of 1/rank values
     - **Opponent wins ("opp_wins"):** Sum of opponents' win counts
     - **Opponent seed ("opp_seed"):** Average of opponents' seed positions
     - **Head-to-head:** Check direct matchup results
     - **Coin flip ("coinflip"):** Random tiebreaker (seeded by tournament start)
     - **Custom composites:** Recursive calculation using child protocol
   - Apply multipliers
   - Apply high-low drops (remove best N and/or worst N scores)
   - Apply truncation
   - Handle forfeits and byes:
     - Forfeits: entry gets loss, forfeit flag set
     - Byes: entry gets averaged points or win (configurable)
     - `forfeits_never_break`: forfeited entries excluded from breaks

5. **Sort entries by tiebreakers**
   - Sort by priority 1 tiebreaker first, then 2, then 3, etc.
   - Each tiebreaker has direction: "down" (higher is better, e.g., wins) or "up" (lower is better, e.g., ranks)
   - Assign place numbers (entries tied on all tiebreakers share same place)

6. **Return results data structure**
   - `entries_ref`: hash of place -> [entry_ids]
   - `tbs_ref`: hash of entry_id -> tiebreaker_id -> value
   - `desc_ref`: hash of tiebreaker_id -> description ("W", "Pts", etc.)
   - `tier_dir`: hash of tiebreaker_id -> direction ("up"/"down")
   - Additional: forfeit flags, panel ranks, panel letters, long descriptions

7. **Display results** (various display templates)
   - Schematic view shows inline results on `show.mhtml`
   - Debate results: `debate_results.mas`
   - Speech results: `speech_results.mas`
   - WUDC results: `wudc_results.mas`
   - Results page: `/tabbing/results/index.mhtml`

### Decision Points
| Decision | Options | Determined By |
|----------|---------|---------------|
| Tiebreaker set | Multiple configured protocols | `round.protocol` |
| Count scope | Prelim / Elim / All / Specific | `tiebreak.count` |
| High-low drops | None / Drop high / Drop low / Both | `tiebreak.highlow`, `tiebreak.highlow_count` |
| Bye treatment | Win / Average / Custom | Protocol settings |
| Forfeit treatment | Loss / Exclude from breaks | `forfeits_never_break` |
| Composite calculation | Direct / Child protocol | `tiebreak.child` |
| Chair-only scoring | All judges / Chair only | `tiebreak.chair` |

### Key Business Rules Invoked
- BR-RESULT-001: Tiebreakers are applied in priority order; equal entries share a place
- BR-RESULT-002: High-low drops remove extreme scores only when sufficient rounds exist (threshold check)
- BR-RESULT-003: Byes receive averaged opponent scores
- BR-RESULT-004: Forfeits count as losses; forfeiting entries can be excluded from breaks
- BR-RESULT-005: Composite tiebreakers cannot reference other composites (infinite loop prevention)
- BR-RESULT-006: Dropped/DQ entries are excluded from standings
- BR-RESULT-007: Rounds with `ignore_results` are excluded

### Settings That Affect Behavior
| Setting | Table | Effect |
|---------|-------|--------|
| `round_robin` | event_setting | Affects bracket calculation |
| `team_points` | event_setting | Points aggregated at team vs student level |
| `nsda_district` / `nsda_nats` | tourn_setting | Special protocols and composite handling |
| `forfeits_never_break` | protocol_setting | Exclude forfeit entries from breaks |
| `ignore_results` | round_setting | Exclude round from results |

### Format Variations
- **Debate:** Primary tiebreakers: wins, speaker points, opponent wins; ballots as secondary
- **Speech:** Primary: ranks, reciprocals, points; average-based aggregation
- **Congress:** Complex multi-dimensional scoring; PO scores, legislation, recency
- **WUDC:** Team points across 4-team panels; special scoring rules

### Error/Exception Handling
- No protocol: abort with "round does not have a tiebreaker set"
- No round number: abort with warning about missing timeslot
- Composite-of-composite: error displayed, computation halted
- opp_seed in composite: error displayed, computation halted

### Outputs/State Changes
- **No persistent state changes** — results are computed on-the-fly from Score records
- **Returns:** Sorted entry list, tiebreaker values, place numbers
- **Optional:** ResultSet/Result records created during break (see Workflow 7)

### Post-conditions
- Entries are ordered by tiebreaker priority
- Place numbers assigned (ties get same place)
- Results displayable on schematic, results pages

### Source Files
- `/web/tabbing/results/order_entries.mas` — main results computation engine
- `/web/tabbing/results/order_speakers.mas` — individual speaker results
- `/web/tabbing/results/index.mhtml` — results display page
- `/web/tabbing/results/results_table.mas` — results table display component
- `/web/panel/schemat/debate_results.mas` — inline debate results
- `/web/panel/schemat/speech_results.mas` — inline speech results
- `/web/panel/schemat/wudc_results.mas` — inline WUDC results
- `/web/funclib/entry_wins.mas` — win record calculation
- `/web/funclib/tiebreak_types.mas` — determine score types from protocol
- `/web/funclib/opponent_records.mas` — opponent win/loss records

---

## 7. Run a Break (Elimination Bracket)

### Workflow Name
Break to Elimination Rounds

### Primary Role(s)
Tabber, Tournament Director

### Preconditions
- Preliminary rounds are complete (all ballots entered and audited)
- Results computed via protocol/tiebreaker set
- Break round timeslot exists
- Protocol for the elimination round exists
- Site assigned (unless online event)

### Trigger
Tabber navigates to Tabbing > Breaks, selects the "from" round (last prelim or previous elim), configures break parameters, and clicks to execute the break.

### Steps

1. **Configure break parameters** (`/tabbing/break/index.mhtml`)
   - Select "from" round (the round whose results determine the break)
   - Select or create "into" round (the elimination round to break into)
   - Specify break range: start seed and end seed (e.g., seeds 1-16 for an octas break)
   - Select round type: "elim" / "final"
   - Select protocol for the elim round
   - Select timeslot and site
   - Select breakout (if multiple break tracks)
   - Optionally specify override for number of entries

2. **Validate inputs** (`break_debate.mhtml`)
   - Verify "from" round exists and user has permission for its event
   - Verify "into" round is in the same event (cannot break across events)
   - Verify round is not breaking into itself
   - Verify required fields: timeslot, protocol, start/end seeds, round type
   - For elim-to-elim breaks, start/end are not required (winners advance automatically)

3. **Create or prepare target round**
   - If "into" round does not exist: create Round record with timeslot, protocol, site, label, type
   - Set default `num_judges` = 3 for elim/final rounds
   - Renumber rounds for the event (`renumber_rounds.mas`)
   - If "into" round exists: clear it (`round_dump.mas`)

4. **Compute standings from source round**
   - Call `order_entries.mas` with the "from" round
   - Get ordered entry list by place/seed

5. **Determine advancing entries**

   **5a. Prelim-to-elim break:**
   - Iterate through entries by seed
   - Include entries from `start` seed to `end` seed
   - Exclude entries with `no_elims` entry setting (ineligible for elims)
   - Create Result records in the bracket ResultSet: entry, seed, place

   **5b. Elim-to-elim advancement:**
   - Load bracket positions from the previous elim round's panels
   - Winners advance: entries with `tbs[entry][1] == 1` (won the ballot)
   - Losers drop out (standard single elimination)
   - Bracket seed is inherited from panel bracket position
   - **Double elimination** (`double_elimination` event setting):
     - Winners bracket: entries with 0 losses advance to winners bracket
     - Losers bracket: entries with 1 loss advance to losers bracket
     - Losers' bracket seed is adjusted relative to winners' bracket size
     - Check for repeat matchups: if losers' bracket would create a repeat, reverse bracket seeding
     - When only 2 entries remain: create finals

6. **Build elimination bracket**
   - Target bracket must be a power of 2 (round up: 12 entries -> 16-team bracket)
   - Compute `master_seed = target_bracket + 1`
   - Create matchups: seed N vs seed (master_seed - N)
     - E.g., in 16-team bracket: 1v16, 2v15, 3v14, etc.
   - Entries without opponents get byes (partial bracket)
   - Handle walk-over rounds (same-school matchups in elims)

7. **Create panels and ballots for elim round**
   - For each matchup: create Panel record with `bracket` = lower seed
   - Create Ballot records with side assignments:
     - Side 1 (aff) = higher-seeded entry (if side-constrained)
     - Side 2 (neg) = lower-seeded entry
     - If no side constraints: snake sides
   - Bye panels: create panel with `bye = 1`

8. **Handle losers bracket** (double elimination only)
   - Create separate panels for losers' bracket matchups
   - Bracket target calculation based on initial break size and current round number
   - Exponent-based formula: `bracket_target = (first_elim_target / 2^exponent) * multiplier + 1`

9. **Set round as paired**
   - `round.paired_at = now`

10. **Redirect to schematic for review**

### Decision Points
| Decision | Options | Determined By |
|----------|---------|---------------|
| Break type | Prelim-to-elim / Elim-to-elim | Source round type |
| Bracket size | Power of 2 >= entries | Computed from break range |
| Elimination format | Single / Double | `event_setting.double_elimination` |
| Side assignment in elims | Constrained / Snake | Event settings |
| Judges for elim | 1 / 3 / custom | `round_setting.num_judges` (default 3 for elims) |
| School walk-over | Bye / Debate | Same-school detection |
| Breakout track | None / Named breakout | `breakout` parameter |

### Key Business Rules Invoked
- BR-BREAK-001: Break bracket must be a power of 2
- BR-BREAK-002: Top seed plays bottom seed in each half-bracket (1v16, 2v15, etc.)
- BR-BREAK-003: Entries marked `no_elims` are excluded from the break
- BR-BREAK-004: Same-school matchups in elims may be awarded as byes/walk-overs
- BR-BREAK-005: Winners of each elim round advance to the corresponding bracket position
- BR-BREAK-006: Double-elimination: 0-loss entries go to winners bracket, 1-loss to losers bracket
- BR-BREAK-007: Cannot break an event into a different event

### Settings That Affect Behavior
| Setting | Table | Effect |
|---------|-------|--------|
| `double_elimination` | event_setting | Enable double-elimination bracket |
| `school_debates_self` | event_setting | Allow same-school elim matchups |
| `no_elims` | entry_setting | Exclude entry from break |
| `use_for_breakout` | round_setting | Mark round as breakout track |
| `nsda_district` | tourn_setting | District qualifier count calculation |

### Format Variations
- **Debate:** Full bracket elimination as described
- **Speech:** `break_speech.mhtml` — entries advance to final rounds with re-snaking
- **Congress:** `break_congress.mhtml` — chamber advancement with session tracking
- **WUDC:** `break_wudc.mhtml` — 4-team bracket advancement

### Error/Exception Handling
- Missing timeslot/protocol/seeds: redirect with error messages
- Breaking into self: error "You cannot advance a round into itself"
- No bracket numbers on panels: error "Your elim panels are horked"
- Missing round type: error "Missing round type"

### Outputs/State Changes
- **Created:** Round record (if new), Panel records, Ballot records, Result records, ResultSet
- **Modified:** `round.paired_at` set
- **Result records:** Track bracket seeding: entry, seed, place in ResultSet with `bracket = 1`

### Post-conditions
- Elimination round exists with correct bracket structure
- All advancing entries have Panel and Ballot records
- Bracket positions correctly assigned (power of 2)
- Byes assigned where bracket is partial

### Source Files
- `/web/tabbing/break/index.mhtml` — break configuration UI
- `/web/tabbing/break/break_debate.mhtml` — debate break execution
- `/web/tabbing/break/break_speech.mhtml` — speech break execution
- `/web/tabbing/break/break_congress.mhtml` — congress break execution
- `/web/tabbing/break/break_wudc.mhtml` — WUDC break execution
- `/web/tabbing/break/ready_status.mas` — break readiness check
- `/web/tabbing/results/order_entries.mas` — standings for break
- `/web/funclib/renumber_rounds.mas` — renumber rounds after creating new round

---

## 8. Register a School at a Tournament

### Workflow Name
Register a School at a Tournament

### Primary Role(s)
Coach, Chapter Admin

### Preconditions
- Coach has a Tabroom account with confirmed email
- Coach has a chapter (school) record they administer
- Tournament exists with registration open (`tourn.reg_end` > now)
- Coach has appropriate chapter permissions

### Trigger
Coach navigates to their chapter's tournaments page, finds the tournament, and clicks "Register." This dispatches to `create.mhtml`.

### Steps

1. **Validate eligibility** (`create.mhtml`)
   - Check email is confirmed (not `email_unconfirmed`)
   - Check registration deadline not passed (`tourn.reg_end`)
   - Verify coach has chapter authorization (`/user/chapter/auth.mas`)
   - Check for existing registration: if school already registered, redirect to entry page

2. **Tournament-specific eligibility checks**
   - **NSDA Nationals:**
     - Chapter must be marked as `highschool`
     - Sync NSDA chapter data
     - Track nationals appearances
   - **NSDA MS Nationals:**
     - Chapter must NOT be `highschool`
   - **NSDA Members Only:**
     - Chapter must have NSDA membership (`chapter.nsda`)
   - **Districts Required:**
     - Chapter must have NSDA membership
     - Chapter must have attended district tournament this season
   - **School Districts Required:**
     - Verify NSDA membership

3. **Create school record** (`/funclib/school_create.mas`)
   - Create School record: `tourn`, `chapter`, registering person info
   - School inherits: name, state, region from chapter
   - Returns school ID or error

4. **Post-creation setup**
   - For NSDA Nationals: sync official name from NSDA, update appearances
   - If tournament has a disclaimer: redirect to `disclaimer.mhtml` for acceptance
   - Otherwise: redirect to entry page `entry.mhtml`

5. **Accept disclaimer** (if required)
   - Coach reads and accepts tournament disclaimer
   - System records: `school_setting.disclaimed = person_id`, `school_setting.disclaimed_at = now`

6. **Entry dashboard** (`entry.mhtml`)
   - Display school's tournament registration status
   - Show fees summary, empty entry warnings
   - Verify adult contact info if required (`require_adult_contact` setting)
   - Show category warnings for missing entries
   - Entry/judge/fee tabs available for further action

### Decision Points
| Decision | Options | Determined By |
|----------|---------|---------------|
| Registration allowed | Yes / No (deadline passed) | `tourn.reg_end` vs current time |
| NSDA eligibility | Various checks | `nsda_district`, `nsda_nats`, `nsda_members_only` settings |
| Disclaimer required | Yes / No | `tourn_setting.disclaimer` |
| Adult contact required | Yes / No | `tourn_setting.require_adult_contact` |
| Contact format | Simple / Account-based | `tourn_setting.account_contacts` |

### Key Business Rules Invoked
- BR-REG-001: Registration is closed after `tourn.reg_end` deadline
- BR-REG-002: Email must be confirmed before registration
- BR-REG-003: Each chapter can only register once per tournament (redirect to existing if duplicate)
- BR-REG-004: NSDA tournament types have membership requirements
- BR-REG-005: District tournaments require prior district attendance (for LCQ)
- BR-REG-006: Disclaimer must be accepted before proceeding (if configured)

### Settings That Affect Behavior
| Setting | Table | Effect |
|---------|-------|--------|
| `disclaimer` | tourn_setting | Require disclaimer acceptance |
| `require_adult_contact` | tourn_setting | Require contact info before entries |
| `account_contacts` | tourn_setting | Use account-based contact tracking |
| `nsda_district` | tourn_setting | NSDA district tournament rules |
| `nsda_nats` | tourn_setting | NSDA Nationals rules |
| `nsda_ms_nats` | tourn_setting | NSDA MS Nationals rules |
| `nsda_members_only` | tourn_setting | Require NSDA membership |
| `school_districts_required` | tourn_setting | Require district attendance |
| `hide_codes` | tourn_setting | Hide entry codes during registration |
| `supp_team_show_coaches` | tourn_setting | Supplemental team assignment |
| `track_reg_changes` | tourn_setting | Log all registration changes |

### Format Variations
- No format-specific variations — this workflow is the same for all event types.

### Error/Exception Handling
- Email unconfirmed: redirect to confirmation page
- Deadline passed: redirect with "registration deadline was [date]" message
- Already registered: redirect to existing school entry page
- Not high school for Nationals: redirect with error
- No NSDA membership: redirect with error and contact info
- School creation fails: redirect to home with error

### Outputs/State Changes
- **Created:** School record linking chapter to tournament
- **Created:** school_settings for disclaimer acceptance (if applicable)
- **Modified:** NSDA sync data (if NSDA tournament)

### Post-conditions
- School record exists for this chapter + tournament combination
- Coach can now add entries and judges
- Disclaimer accepted (if required)

### Source Files
- `/web/user/enter/create.mhtml` — registration handler
- `/web/user/enter/entry.mhtml` — entry dashboard (post-registration)
- `/web/user/enter/disclaimer.mhtml` — disclaimer display/acceptance
- `/web/user/enter/details.mhtml` — school registration details
- `/web/user/chapter/tourn_register.mhtml` — alternate registration path
- `/web/funclib/school_create.mas` — school record creation

---

## 9. Add Entries to an Event

### Workflow Name
Add Entries to an Event

### Primary Role(s)
Coach, Chapter Admin

### Preconditions
- School is registered at the tournament (School record exists)
- Event exists and is open for registration
- Coach has chapter authorization
- Registration deadline not passed (or within drop deadline)
- Adult contact info provided (if required by tournament)

### Trigger
Coach navigates to their school's entry page, selects an event, and adds students. The primary paths are:
- `by_person.mhtml` — add by student from roster
- `by_person_add.mhtml` — create entry for selected student
- Or direct entry via event-specific interfaces

### Steps

1. **Select event** (`entry.mhtml` dashboard)
   - Display available events grouped by category
   - Show current entry count vs cap for each event
   - Check if event has waitlist
   - Coach clicks event to manage entries

2. **View available students** (`by_person.mhtml`)
   - Load chapter student roster
   - Show which students are already entered in events at this tournament
   - Show eligibility status per event:
     - Double-entry restrictions
     - NSDA eligibility (if applicable)
     - Grade level restrictions

3. **Select students and create entry** (`by_person_add.mhtml`, `by_person_save.mhtml`)
   - For individual events: select 1 student
   - For team events: select `min_entry` to `max_entry` students (e.g., 2 for LD/Policy teams)
   - System validates:
     - Student not already entered in this event
     - Double-entry rules satisfied (check against other events in same timeslots)
     - Event not at capacity (or entry goes to waitlist)
   - Create Entry record:
     - `event`, `school`, `active = 1`
     - Assign code based on `event.code_style` (auto-generated or coach-specified)
     - Set `name` based on student name(s)
   - Create EntryStudent records linking entry to student(s)
   - If event at capacity and waitlist enabled: `entry.waitlist = 1`

4. **Configure entry details** (`details_save.mhtml`, `entry_edit.mas`)
   - Set entry name and code (if registration-style codes)
   - Set ADA accessibility requirement
   - Set piece title and author (if speech event with `ask_for_titles`)
   - Set seed designation (if APDA: full/half/free)
   - Set video link (if online event)
   - Set breakout eligibility (if self-registration for breakouts)
   - Handle hybrid entries (students from multiple schools)

5. **Waitlist processing** (if applicable)
   - Entry added to waitlist (`entry.waitlist = 1`)
   - Coach can set waitlist rank order (`waitlist_rank` setting)
   - Tournament director can admit from waitlist later

6. **Fee calculation**
   - System recalculates school fees (`/funclib/school_fees.mas`)
   - Entry fee added to school invoice
   - Display updated fee total

7. **Confirmation**
   - Entry appears in school's entry list
   - If `track_reg_changes`: log the change with description
   - Redirect back to entry management page

### Decision Points
| Decision | Options | Determined By |
|----------|---------|---------------|
| Entry type | Individual / Team | `event_setting.max_entry`, `event_setting.min_entry` |
| Code assignment | Auto / Coach-entered | `event.code_style` (register/initials/auto) |
| Capacity handling | Accept / Waitlist / Reject | Event cap settings, waitlist settings |
| Double-entry allowed | Yes / No / Restricted | Double-entry rules, timeslot conflicts |
| Title required | Yes / No | `event_setting.ask_for_titles` |
| Hybrid entry | Yes / No | `event_setting.online_hybrid` or cross-school entry |
| Seed type | None / Full / Half / Free | `event_setting.apda` |

### Key Business Rules Invoked
- BR-ENTRY-001: Each student can appear in at most one entry per event
- BR-ENTRY-002: Double-entry across events must respect scheduling conflicts
- BR-ENTRY-003: Entry cap: if event is full, new entries go to waitlist
- BR-ENTRY-004: Entry codes must be unique within the tournament
- BR-ENTRY-005: Minimum and maximum team size enforced
- BR-ENTRY-006: Registration deadline enforced (entries blocked after deadline)
- BR-ENTRY-007: Dropped entries assessed drop fee (if configured)

### Settings That Affect Behavior
| Setting | Table | Effect |
|---------|-------|--------|
| `max_entry` / `min_entry` | event_setting | Team size limits |
| `ask_for_titles` | event_setting | Require piece title/author |
| `apda` | event_setting | APDA seed designations |
| `online_hybrid` | event_setting | Enable hybrid entry mode |
| `code_style` | event field | Code generation method |
| `waitlist_rank` | event_setting | Coach-ranked waitlist |
| `hide_codes` | tourn_setting | Hide codes during registration |
| `script_deadline` | tourn_setting | Deadline for piece title changes |
| `drop_deadline` | tourn_setting | Deadline for dropping entries |
| `track_reg_changes` | tourn_setting | Log all entry changes |

### Format Variations
- **Debate:** 2-person teams typical; side-specific roles
- **Speech:** Individual entries; piece title/author required for many events
- **Congress:** Individual entries; legislation preferences
- **WSDC:** Large team entries (5+ members)

### Error/Exception Handling
- Student not on roster: must be added to chapter roster first
- Duplicate entry: error "student already entered in this event"
- Code conflict: error "code already taken, please select another"
- Deadline passed: redirect with deadline message
- No adult contact: blocked with "contact information incomplete" message

### Outputs/State Changes
- **Created:** Entry record, EntryStudent record(s)
- **Created:** entry_settings for title, author, seed, etc.
- **Modified:** School fees recalculated
- **Logged:** Registration change (if tracking enabled)

### Post-conditions
- Entry exists in the event with active status
- Student(s) linked to entry
- Code assigned
- Fees updated

### Source Files
- `/web/user/enter/entry.mhtml` — entry dashboard
- `/web/user/enter/entry_edit.mas` — entry edit form component
- `/web/user/enter/details_save.mhtml` — save entry details
- `/web/user/enter/by_person.mhtml` — student-based entry view
- `/web/user/enter/by_person_add.mhtml` — add entry for student
- `/web/user/enter/by_person_save.mhtml` — save student-based entry
- `/web/user/enter/entry_drop.mhtml` — drop an entry
- `/web/user/enter/entry_csv.mhtml` — CSV import of entries
- `/web/user/enter/students.mhtml` — student roster management
- `/web/user/enter/student_save.mhtml` — save student changes
- `/web/funclib/school_fees.mas` — fee calculation

---

## 10. Register Judges

### Workflow Name
Register Judges for a Tournament

### Primary Role(s)
Coach, Chapter Admin

### Preconditions
- School is registered at the tournament
- Judge category exists in the tournament
- Chapter has judges on its roster (ChapterJudge records)
- Judge registration deadline not passed
- Coach has chapter authorization

### Trigger
Coach navigates to their school's judge management page, selects a category, and adds judges from their chapter roster. This uses `judges.mhtml` (view) and `judge_save.mhtml` (create).

### Steps

1. **Select judge category** (`judges.mhtml`)
   - Display all categories with judge pools
   - Show current judge count vs obligation for each category
   - Show registration deadline (from `judge_deadline` or category-specific deadline)
   - For district tournaments: show weekend-specific deadlines
   - If only one category: auto-select it

2. **View available judges from chapter roster**
   - Load chapter judges not yet registered in this category
   - Display eligibility status:
     - Already registered elsewhere in this tournament
     - Linked to Tabroom account (if `linked_only` required)
     - Phone number present (if `link_phone_required`)
     - Campus test completed (if `link_campus_required`)
   - Show free judges (not assigned to any category yet)

3. **Select and register a judge** (`judge_save.mhtml`)
   - Validate chapter judge exists
   - **Account linking checks** (if `linked_only` setting):
     - Judge must have a linked Tabroom person account
     - If not linked: redirect to `judge_person_link.mhtml`
     - If `link_phone_required`: verify phone number > 2,000,000,000
     - If `link_campus_required`: verify campus test completion
   - Create Judge record:
     - `school`: school at this tournament
     - `first`, `last`: from ChapterJudge
     - `code`: auto-generated via `category_code.mas` (unless `no_codes`)
     - `person`: linked person account ID (0 if not linked)
     - `obligation`: from category settings (`max_rounds` or `rounds_per`)
     - `hired = 0`, `active = 1`
     - `category`: the judge category
     - `chapter_judge`: link to chapter roster
     - `registered_by`: person who registered the judge
   - Auto-create conflicts: `person_conflict.mas` generates school-based conflicts
   - Save special notes, tab rating (if provided)
   - If judge has a person account: create onsite contact record

4. **Set judge availability/shifts**
   - If category has shifts: assign judge to available shifts
   - `judge_shift.mhtml` / `judge_shift_flip.mhtml` for toggling shift availability
   - Calculate obligation based on shifts

5. **Manage judge details** (`judge_details.mhtml`, `judge_details_save.mhtml`)
   - Edit judge preferences, notes, special requirements
   - Set tab rating (admin)
   - Manage conflicts and strikes

6. **Judge obligation tracking**
   - System calculates obligation based on:
     - Number of entries the school has (`judge_per` entries per required judge)
     - Category rounds (`rounds_per`)
     - Existing judge rounds already committed
   - If school is under obligation: display warning
   - If over obligation: judge may be available for hire

7. **Log the registration** (if `track_reg_changes`)
   - Record: "Person X entered Category judge Code (First Last)"

### Decision Points
| Decision | Options | Determined By |
|----------|---------|---------------|
| Account linking required | Yes / No | `category_setting.linked_only` |
| Phone required | Yes / No | `category_setting.link_phone_required` |
| Campus test required | Yes / No | `category_setting.link_campus_required` |
| Code assignment | Auto / None | `category_setting.no_codes` |
| Obligation calculation | By entries / By rounds | `category_setting.judge_per`, `category_setting.rounds_per` |
| Shift-based scheduling | Yes / No | Category shift configuration |
| Hired judge marketplace | Enabled / Disabled | Judge hiring settings |

### Key Business Rules Invoked
- BR-JUDGE-REG-001: Judge deadline is enforced; registration blocked after deadline
- BR-JUDGE-REG-002: Linked accounts required for online tournaments
- BR-JUDGE-REG-003: School-based conflicts auto-generated on registration
- BR-JUDGE-REG-004: Judge obligation calculated from entry count and category rules
- BR-JUDGE-REG-005: Judge code must be unique within category
- BR-JUDGE-REG-006: FYO (First Year Out) judges tracked with custom label
- BR-JUDGE-REG-007: Hired judge deadline separate from regular deadline

### Settings That Affect Behavior
| Setting | Table | Effect |
|---------|-------|--------|
| `linked_only` | category_setting | Require Tabroom account link |
| `link_phone_required` | category_setting | Require phone number |
| `link_campus_required` | category_setting | Require campus room test |
| `no_codes` | category_setting | Skip code assignment |
| `judge_per` | category_setting | Entries per required judge |
| `rounds_per` | category_setting | Rounds per judge obligation |
| `max_rounds` | category_setting | Maximum rounds for obligation |
| `free_strikes_dont_count` | category_setting | Free strikes excluded from obligation |
| `fyo_label` | category_setting | Label for first-year judges |
| `deadline` | category_setting | Category-specific judge deadline |
| `open_switcheroo` | category_setting | Allow judge swaps after deadline |
| `hired_deadline` | category_setting | Deadline for hired judges |
| `judge_deadline` | tourn_setting | Tournament-wide judge deadline |
| `track_reg_changes` | tourn_setting | Log registration activity |

### Format Variations
- No significant format variations — judge registration is category-based, not event-type-based.
- Congress has PO/parliamentarian roles handled separately.

### Error/Exception Handling
- No chapter judge selected: redirect with "did not select a judge"
- Invalid chapter judge ID: abort with "No judge found on your roster"
- Not linked (when required): redirect to linking page
- No phone (when required): redirect with "does not have a valid phone number"
- Campus test not completed: redirect with detailed instructions
- Deadline passed: registration blocked
- No school record: redirect with "no active school entry" message

### Outputs/State Changes
- **Created:** Judge record in category
- **Created:** judge_settings for notes, tab_rating, special
- **Created:** Conflict/strike records (via `person_conflict.mas`)
- **Created:** Contact onsite record (if linked person)
- **Logged:** Registration change (if tracking enabled)

### Post-conditions
- Judge exists in category with active status
- Judge linked to chapter judge record and (optionally) person account
- Auto-conflicts generated for the judge's school
- Obligation tracking updated for the school

### Source Files
- `/web/user/enter/judges.mhtml` — judge management view
- `/web/user/enter/judge_save.mhtml` — create judge record
- `/web/user/enter/judge_details.mhtml` — edit judge details
- `/web/user/enter/judge_details_save.mhtml` — save judge details
- `/web/user/enter/judge_drop.mhtml` — drop a judge
- `/web/user/enter/judge_shift.mhtml` — manage judge shifts
- `/web/user/enter/judge_shift_flip.mhtml` — toggle shift availability
- `/web/user/enter/judge_person_link.mhtml` — link judge to person account
- `/web/user/enter/judges_eligible.mhtml` — check judge eligibility
- `/web/funclib/category_code.mas` — generate judge code
- `/web/funclib/person_conflict.mas` — generate auto-conflicts
- `/web/funclib/contact_onsite.mas` — create onsite contact record

---

## Cross-Cutting Concerns

### Authentication & Authorization
All workflows verify:
- User has valid session (`autohandler` chain)
- User has appropriate tournament permissions (`perms` hash checked at each entry point)
- Permission levels: site_admin > owner > tabber > checker > limited > coach
- Event-scoped permissions: some tabbers restricted to specific events/categories

### Logging
- Tabbing operations logged via `/funclib/log.mas` with type, event, round, person, description
- Registration changes logged when `track_reg_changes` is enabled
- All log entries include timestamp and person ID

### EAV Settings Pattern
Settings cascade through multiple levels:
1. `tourn_setting` — tournament-wide defaults
2. `category_setting` — category (debate/speech/congress) overrides
3. `event_setting` — event-specific overrides
4. `round_setting` — round-specific overrides
5. `entry_setting` — entry-specific flags

Many workflows load `all_settings()` at multiple levels and check in order.

### NSDA Integration
Multiple workflows have special paths for NSDA tournaments:
- District tournaments (`nsda_district`): special pairing rules, qualifier tracking, weekend-based deadlines
- National tournament (`nsda_nats`): strict eligibility, region/district constraints, composite results
- MS Nationals (`nsda_ms_nats`): middle school level restrictions
- NCFL (`ncfl`): diocese/region-based constraints

### Data Model Key Entities
| Entity | Role |
|--------|------|
| Tourn | Tournament record |
| Event | Competition division within a tournament |
| Category | Groups events that share judges (Debate/Speech/Congress) |
| Round | A specific round of an event |
| Panel | A single debate/section within a round |
| Ballot | Links entry + judge + panel; holds audit status |
| Score | Individual score data point (tag + value per ballot) |
| Entry | A competing unit (individual or team) |
| Judge | A judge registered in a category |
| School | A school's registration at a tournament |
| Chapter | Persistent school/program record |
| Protocol | Tiebreaker configuration set |
| Tiebreak | Individual tiebreaker within a protocol |
| JPool | Judge pool for round assignment |
| ResultSet | Container for break/bracket results |
| Result | Individual entry's bracket position |
