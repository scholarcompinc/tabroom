# Algorithmically Complex Areas -- Existing Platform Catalog

This document catalogs the highest-risk algorithmic areas in the Tabroom
codebase for purposes of a platform rebuild. Each section describes the
complexity, source files, documentation state, and behavioral notes
discovered through direct code analysis.

---

## 1. Debate Powermatching

**Area name:** Debate Powermatching (High-Low / High-High pairing)

**Why it is complex:**
Powermatching is a constrained optimization problem. Entries must be paired
within win-brackets while simultaneously satisfying multiple soft
constraints: no same-school matchups, no repeat opponents, side-balance
(alternating aff/neg), pullup fairness, and positional ordering (SOP vs
seed-based). The algorithm builds an N-by-N opponent-score matrix using
weighted penalty values spanning 18 orders of magnitude (from 0.01 position
penalties up to 10^18 same-school penalties), performs a greedy initial
assignment, then runs up to 100 iterative swap-improvement passes attempting
pairwise swaps that reduce total penalty. This is essentially a minimum-cost
matching problem solved via local search heuristic rather than a globally
optimal algorithm.

Key sub-problems:
- Pullup selection (who gets pulled from a lower bracket) with multiple
  methods: SOP (seed + opponent seed), opponent wins, low seed (APDA), with
  repeat-pullup prevention
- Side constraint enforcement on even-numbered rounds
- Bye assignment for odd-count fields
- High-high vs high-low bracket ordering
- Bracket-by-ballots variant

There is also a second, independent powermatching implementation in
`pair_debate.mas` (~1,878 lines) which is the primary code path. It
reimplements the same logic with a different structure and includes extensive
error logging. The older `pair_powered.mas` (~847 lines) is referenced but
currently bypassed via commented-out code.

**Primary source files:**
- `/home/jeloni/tabroom/web/panel/round/pair_debate.mas` (1,878 lines -- primary path)
- `/home/jeloni/tabroom/web/panel/round/pair_powered.mas` (847 lines -- older implementation, partially active)
- `/home/jeloni/tabroom/web/panel/round/pair_bracket.mhtml` (205 lines -- manual bracket pairing UI)
- `/home/jeloni/tabroom/web/funclib/make_pairing_hash.mas` (builds entry data for bracket pairing)
- `/home/jeloni/tabroom/web/funclib/entry_wins.mas` (win-loss record calculation)
- `/home/jeloni/tabroom/web/funclib/entry_byes.mas` (bye tracking)
- `/home/jeloni/tabroom/web/tabbing/results/order_entries.mas` (seeding/ordering for powermatching basis)

**Documentation quality:** Code-only. There are inline comments explaining
penalty weights and the rationale for some decisions (e.g., "APDA and
therefore insane"), but no external documentation of the algorithm. The code
contains hardcoded debug entry IDs (592621, 594635, 595467) and extensive
debug logging blocks that are unreachable.

**Likely need for expert review:** Yes. Two parallel implementations exist
with subtly different behaviors. The penalty weight system spans 18 orders of
magnitude and the interaction between constraints is non-obvious. The
iterative swap improvement is a local search that may not find global optima.
The APDA, NCFL, and NSDA variants add special-case behavior. Domain experts
must confirm the correctness of pullup, side-constraint, and bracket logic.

**Key behavioral notes:**
- Penalty hierarchy: same-school (10^18) > hit-before (10^12) > wrong-side
  (10^11) > pullup (10^4, or 10^14 if pullup_minimize enabled)
- Pullup score doubles for entries already pulled up (unless pullup_repeat
  is set)
- `pair_debate.mas` starts with `our $debugme = 1; undef $debugme;`
  suggesting active development/debugging
- Region constraints can replace school constraints via `region_constrain`
  setting
- Hybrid school conflicts are tracked via Strike table with type='hybrid'
- Byes are placed at position 1 in the worst bracket with an extremely high
  pullup score (num_entries * 10^18) to prevent pullup
- Side assignment uses a snaking pattern for non-side-locked rounds

---

## 2. Speech Paneling

**Area name:** Speech Section/Panel Assignment

**Why it is complex:**
Speech paneling must distribute entries across sections while minimizing
same-school hits, repeat-opponent hits, same-region hits, and optionally
balancing seeds and speaker-order positions. The algorithm uses a two-phase
approach: (1) greedy initial assignment where each entry is placed in the
section that produces the lowest penalty score, and (2) iterative swap
improvement (up to 7 passes) that tries all pairwise entry swaps between
sections to reduce total penalty. This is an NxM assignment problem with
O(n^2) section-scoring and O(n^3) swap evaluation, making it computationally
expensive for large fields.

Speaker order assignment is embedded: after sectioning, entries are ordered
within each section by historical speaker-order totals, then iteratively
shuffled (up to 15 passes) to avoid repeat positions and consecutive-round
same-positions. Double-entry (same student in multiple events in the same
timeslot) handling adds further complexity.

The system has three separate scoring weight configurations: standard
tournaments, NSDA Districts, and NSDA Nationals, each with different penalty
hierarchies.

**Primary source files:**
- `/home/jeloni/tabroom/web/panel/round/pair_speech.mas` (1,012 lines)
- `/home/jeloni/tabroom/web/panel/round/snake_speech.mas` (seed-based snaking variant)
- `/home/jeloni/tabroom/web/panel/round/nsda/snake_speech.mas` (NSDA-specific snaking)
- `/home/jeloni/tabroom/web/panel/round/nsda/speaker_order.mas` (NSDA speaker order assignment, 305 lines)
- `/home/jeloni/tabroom/web/funclib/event_speaker_order.mas`

**Documentation quality:** Code-only. Comments exist for some weight choices
(e.g., "These four are absolute and given the size should always be
achievable") but the algorithmic strategy is undocumented.

**Likely need for expert review:** Yes. The scoring function (`score_section`)
is called O(n^3) times during swap improvement. The speaker-order randomization
uses `int(rand($size_of_section))` which can produce position 0, and the
`splice` operations on the entries array during iteration may cause
unpredictable reordering. The `school_percent_limit` feature adds forbidden
section constraints that interact with the greedy placement.

**Key behavioral notes:**
- Standard weights: school=10^6, repeat=10^3, repeat_school=10
- NSDA Nationals weights: school=10^5, district=10^5, region=10^5,
  repeat=10^5, order=100
- Districts weights: school=10^9, title=10^4, repeat=100, repeat_school=10
- Section scoring uses a `stringify` cache keyed by sorted entry-ID strings
- `school_percent_limit` prevents any school from having more than X% of
  sections, implemented via `forbidden_sections` hash
- Seed-based paneling (`seed_basis` parameter) tries to equalize average
  seed across sections after constraint satisfaction
- Flighted rounds split sections across two round objects
- Commented-out code suggests title-collision and repeat-region avoidance
  were attempted but disabled

---

## 3. Congress Grouping/Chambering

**Area name:** Congress Chamber Assignment

**Why it is complex:**
Congress chambering is a variant of speech paneling with additional
constraints unique to Congressional Debate: bill/legislation authorship
separation, same-state avoidance, same-district avoidance (at NSDA
Nationals), bloc-school handling (for districts with house/senate
chambering), and PO (Presiding Officer) contest cross-linking. Sessions are
"tied" together -- multiple scoring rounds share the same chamber
assignments, so changes propagate across all linked sessions.

The system supports three modes: "wipe" (create new chambers from scratch),
"realign" (copy first session's chambers to later sessions), and "single"
(copy to just one session). The scoring function (`score_chamber`) evaluates
all pairwise entry combinations for constraint violations with a cached
scoring system. Iterative swap improvement runs up to 5 passes.

Seed distribution is an additional optimization pass that runs after
constraint satisfaction, attempting entry swaps that maintain the same
constraint score while equalizing total seed values across chambers.

**Primary source files:**
- `/home/jeloni/tabroom/web/panel/round/pair_congress.mas` (913 lines)
- `/home/jeloni/tabroom/web/panel/round/congress_chambers.mhtml` (UI for chamber setup)
- `/home/jeloni/tabroom/web/panel/round/congress_recency.mhtml` (recency-based speaker order)
- `/home/jeloni/tabroom/web/funclib/congress_ties.mas` (linked session resolution)
- `/home/jeloni/tabroom/web/funclib/congress_names.mas`

**Documentation quality:** Weak/code-only. No external documentation of the
chambering algorithm. Some inline comments explain weight rationale.

**Likely need for expert review:** Yes. The interaction between NSDA Nationals
constraints (state, region, district, authorship, autoqual) creates a
complex constraint space. The PO contest linking (copying chambers from a
parent event) has separate logic that bypasses the main chambering algorithm.
The bloc_school feature for NSDA Districts uses negative penalty values
(subtracting from school penalty), which is unusual.

**Key behavioral notes:**
- NSDA Nationals penalties: state=10^8, region=10^6, district=100033 (not
  a round number -- intentional to break ties), author=10^4, autoqual=100,
  name=1
- Standard penalties: school=10^12, state=10^8, bill_topic=10^6,
  author=10^4, name=1
- NCFL uses region penalty equal to school penalty (diocese-as-school)
- `bloc_school` penalty is subtracted (negative) when two entries share a
  bloc, effectively allowing same-school within-bloc
- Pairwise scores are cached in `${$entries}{"cache"}{$entry}{$other}` to
  avoid recomputation
- Default chamber size: min=15, max=30, target=20
- Seed equalization pass runs 5 iterations, swapping entries that maintain
  identical constraint scores while moving seed totals closer together
- Typo in code: `"panel_lables"` instead of `"panel_labels"` on line 789

---

## 4. Judge Assignment Optimization

**Area name:** Debate Judge Assignment

**Why it is complex:**
This is the most complex single file in the codebase (2,120 lines). It solves
a constrained assignment problem: assign judges to panels while respecting
hard constraints (school conflicts, entry strikes, time strikes,
repeat-judging, event strikes) and optimizing soft constraints (MJP/MPJ
preference ratings, mutuality, judge diversity, obligation-based workload
balancing, bracket priority, seed priority, tab ratings, regional avoidance).

The algorithm:
1. Builds a complete judge-panel scoring matrix factoring in ~15 different
   scoring dimensions
2. Runs 30 iterations of: random initial assignment, then iterative swap
   improvement (trying all judge-pair swaps to reduce total score)
3. Selects the best iteration

Additional complexity from:
- Multi-judge panels (e.g., 3-judge elim panels)
- Multi-flight rounds (linking panels across flights for same-judge)
- NCFL diocese-region constraints
- "Caring quota" that deprioritizes MJP quality for brackets that cannot
  clear (mathematically eliminated entries)
- Online/hybrid tournament room-type matching
- Obligation tracking (rounds_per system for judge workload distribution)
- Tab ratings vs coach ratings vs ordinal prefs -- three different
  preference systems
- `best_judges_highest_seed` option for APDA-style tournaments

**Primary source files:**
- `/home/jeloni/tabroom/web/panel/round/debate_judge_assign.mhtml` (2,120 lines)
- `/home/jeloni/tabroom/web/panel/round/debate_judge_panel.mhtml` (speech judge assignment variant)
- `/home/jeloni/tabroom/web/panel/round/mass_judges.mhtml` (batch judge assignment)
- `/home/jeloni/tabroom/web/funclib/judges_by_pref.mas` (preference aggregation)
- `/home/jeloni/tabroom/web/funclib/judge_use.mas` (obligation/usage tracking)
- `/home/jeloni/tabroom/web/funclib/round_pref_data.mas` (preference data for display)
- `/home/jeloni/tabroom/web/funclib/panel_judgeadd.mas` (single judge addition)
- `/home/jeloni/tabroom/web/funclib/strike_judges.mas` (strike application)
- `/home/jeloni/tabroom/web/funclib/free_strikes.mas`
- `/home/jeloni/tabroom/web/funclib/category_strikes.mas`

**Documentation quality:** Weak. Comments appear sporadically (e.g., "This
takes care of the diocese region thing that baffles me and is best not
confronted directly"), but the overall algorithm structure is undocumented.
Weight values are stored in category_settings but their interaction is not
documented anywhere.

**Likely need for expert review:** Yes. This is the highest-risk area for
rebuild. The scoring formula combines 15+ dimensions with configurable
weights, and correctness depends on the relative magnitude of these weights.
The 30-iteration random-start approach suggests the problem space is too
large for deterministic solutions. NCFL, NSDA Nationals, APDA, and WSDC each
have special-case behavior woven throughout.

**Key behavioral notes:**
- Conflict penalties: entry_strike=10^8, event_strike=10^8, region_constrain=5*10^6, NCFL_dioregion=5*10^6
- Configurable weights with defaults: mutuality=40, preference=15, default_mjp=2, diversity=1, suckage=3, round_burn_avoid=3, prefer_hireds=10, meatspace=2
- "Caring quota" reduces preference weight to 1% for brackets below `break_point`
- `use_priority` score balances obligation fulfillment -- judges further from their obligation target get priority
- Person-level cross-category conflict detection: if judge John Smith is judging LD at 3pm, his Policy alter ego is marked as busy
- Bracket score amplifies judge-panel scores by bracket level (higher brackets get exponentially better judges)
- Judge rating tiers can be marked as "strike" tiers, automatically creating conflicts
- `allow_repeat_prelim_side` allows repeat judging if entry is on opposite side

---

## 5. Speaker Order Assignment

**Area name:** Speech Speaker Order / Speaking Position Assignment

**Why it is complex:**
Speaker order must ensure each entry speaks in different positions across
rounds, avoiding repeat positions and ensuring fair distribution across
early/mid/late slots. The algorithm is a randomized iterative improvement:
entries are sorted by cumulative past speaking order (highest total first),
then iteratively shuffled up to 15 times, moving entries that have a repeated
position to a random new position via `splice`. Double-entered students
(competing in two events in the same timeslot) must be placed early or late
in their sections to allow travel time.

The improvement algorithm (`speaker_order_improve.mhtml`) works post-hoc,
identifying entries with missing early/mid/late coverage and swapping them
with entries that have excess coverage in the needed category.

**Primary source files:**
- `/home/jeloni/tabroom/web/panel/round/pair_speech.mas` (lines 712-828, embedded in paneling)
- `/home/jeloni/tabroom/web/panel/round/nsda/speaker_order.mas` (305 lines, NSDA variant)
- `/home/jeloni/tabroom/web/panel/round/speaker_order.mhtml` (display/audit view)
- `/home/jeloni/tabroom/web/panel/round/speaker_order_improve.mhtml` (166 lines, post-hoc improvement)
- `/home/jeloni/tabroom/web/funclib/ncfl_speakers.mas` (early/mid/late status check)
- `/home/jeloni/tabroom/web/funclib/swap_orders.mas`
- `/home/jeloni/tabroom/web/funclib/correct_orders.mas`

**Documentation quality:** Code-only. The early/mid/late thresholds are
hardcoded (early < 3, mid 3-5, late > 5) without documentation of why these
boundaries were chosen.

**Likely need for expert review:** Yes. The randomized shuffle approach using
`int(rand(N))` and `splice` during iteration has subtle bugs: position 0 can
be generated, and the splice-during-iteration pattern can duplicate entries
(handled by a post-loop `grep` deduplication). The NSDA California-method
elims skip speaker order entirely.

**Key behavioral notes:**
- Early positions: 1-2; Mid positions: 3-5; Late positions: 6+
- Double-entry students placed at position 1 (if `speaker_priority_first`)
  or position 7 (if late), or inverse of their other-event position
- Post-hoc improvement (`find_other`) only swaps if the donor has 2+ rounds
  in the needed category
- Max 15 shuffle iterations per section; typically converges much earlier
- NSDA California methods 2 and 3 skip speaker order for elims entirely

---

## 6. Room Quality / ADA Matching

**Area name:** Room Assignment with ADA and Quality Constraints

**Why it is complex:**
Room assignment must satisfy ADA accessibility requirements, minimize entry
room-moves between rounds, handle room reservations for specific judges,
manage room pools (rpools) shared across events, handle flighted rounds
(multiple debates in same room), and optionally assign rooms by bracket
quality (best rooms to highest brackets). Congress rooms must propagate
across tied sessions.

The algorithm processes panels in priority order: ADA panels first, then
reserved-judge panels, then bracket-priority, then remaining. For each panel,
it checks preferred rooms (based on previous-round room assignments, with
aff-side getting priority for room retention), reserved rooms, and falls back
to general pool.

**Primary source files:**
- `/home/jeloni/tabroom/web/panel/round/rooms.mhtml` (866 lines)
- `/home/jeloni/tabroom/web/panel/room/assign.mhtml` (batch room assignment)
- `/home/jeloni/tabroom/web/funclib/round_panels_ada.mas` (ADA-flagged panel retrieval)
- `/home/jeloni/tabroom/web/funclib/room_strikes.mas` (room-entry conflict data)
- `/home/jeloni/tabroom/web/funclib/clean_rooms.mas` (available room filtering)
- `/home/jeloni/tabroom/web/funclib/site_room_strikes.mas`
- `/home/jeloni/tabroom/web/funclib/hybrid_panels.mas` (online/hybrid room URL assignment)

**Documentation quality:** Code-only. No documentation of the room assignment
priority algorithm or ADA matching logic.

**Likely need for expert review:** No, with caveats. The algorithm is
relatively straightforward (priority-ordered greedy assignment), but the
interaction between room reservations, room strikes, flighting, ADA, and
room pools creates edge cases. Mock trial has special headcount-based room
capacity matching.

**Key behavioral notes:**
- ADA panels are always processed first (sorted to front)
- Neg-side (side 2) entry's previous room is preferred; aff-side room is
  secondary preference
- `bracket_rooms` setting sorts panels by bracket and rooms by quality
- Mock trial rooms sorted by capacity (head count = students + observers)
- Room pools shared across events: rooms in fewer pools are preferred
  (via `rpool_count` sorting)
- `roomlock_against` setting allows locking judge rooms to a reference round
- `same_room_timeslot` allows judge room persistence within a timeslot
- Online/hybrid events skip room assignment entirely unless `use_normal_rooms` is set
- `repeat_rooms` setting limits assignment to rooms used in previous round
- Fallback pool (`@reserve`) used when primary rooms exhausted
- Congress tied sessions inherit room assignments from first session by letter

---

## 7. Burden Calculation (Judge Workload)

**Area name:** Judge Obligation/Burden Calculation

**Why it is complex:**
Judge burden tracks how many entries a school has competing in events served
by a judge pool, compared to how many judges that school provides. This
ratio determines whether a school is meeting its judging obligation. The
calculation must span across events within a category, account for
waitlisted/dropped entries, and handle judge pool boundaries.

The obligation system in judge assignment (`debate_judge_assign.mhtml`) goes
further: it computes a "use priority" score for each judge based on their
obligation (hired + regular), rounds already judged, and remaining rounds.
Judges with more remaining obligation get higher priority. This interacts
with the `rounds_per` category setting that caps judge usage.

**Primary source files:**
- `/home/jeloni/tabroom/web/funclib/jpool_burden.mas` (67 lines -- per-school burden calculation)
- `/home/jeloni/tabroom/web/funclib/judge_use.mas` (round-by-round usage tracking)
- `/home/jeloni/tabroom/web/panel/round/debate_judge_assign.mhtml` (lines 900-975, obligation scoring)
- `/home/jeloni/tabroom/web/panel/judge/nats_pool_check.mhtml`
- `/home/jeloni/tabroom/web/panel/judge/nats_pool_totals.mhtml`

**Documentation quality:** Code-only. The burden formula is embedded in
SQL queries and inline Perl without documentation.

**Likely need for expert review:** No. The core calculation is a ratio
(entries / judges per school per pool). The complexity is primarily in the
integration with judge assignment scoring, which is documented above.

**Key behavioral notes:**
- Burden = entries in pool / judges in pool per school
- Waitlisted entries excluded by default; configurable via `judges_waitlist`
- Dropped entries excluded by default; configurable via `drops_no_burden`
- NSDA Nationals multiplies usage priority by 10,000 for primary-event
  rounds and 100 for secondary-event rounds
- `rounds_per` system: judges marked "out" when rounds_judged >= obligation
- Remaining-round percentage drives exponential priority scoring:
  `round_diff ^ round_burn_avoid` (default exponent = 3)

---

## 8. Conflicts/Strikes/Prefs Application

**Area name:** Conflict, Strike, and Preference System

**Why it is complex:**
The conflict system has multiple overlapping mechanisms: school-based
conflicts (auto-generated), entry-level strikes (registrant or tab-created),
regional conflicts, diocese-region conflicts (NCFL), event strikes, elim
strikes, time/departure strikes, hybrid school conflicts, and rating-tier
strikes. These are resolved differently depending on context:
- Pairing: school conflicts prevent matchups
- Judge assignment: all conflict types prevent judge-entry assignments
- MJP/MPJ preferences layer on top as soft constraints

The strike table stores heterogeneous data (entry, school, region,
dioregion, time range, event), and the application logic must correctly
interpret each type. Rating tiers can be marked as "strike" or "conflict"
tiers, converting preference ratings into hard constraints.

**Primary source files:**
- `/home/jeloni/tabroom/web/panel/round/debate_judge_assign.mhtml` (lines 514-593, strike processing)
- `/home/jeloni/tabroom/web/funclib/entry_conflicts.mas` (entry-level conflict retrieval)
- `/home/jeloni/tabroom/web/funclib/chapter_conflicts.mas` (chapter/school conflicts)
- `/home/jeloni/tabroom/web/funclib/school_conflicts.mas`
- `/home/jeloni/tabroom/web/funclib/person_conflict.mas`
- `/home/jeloni/tabroom/web/funclib/category_strikes.mas`
- `/home/jeloni/tabroom/web/funclib/free_strikes.mas`
- `/home/jeloni/tabroom/web/funclib/event_strike_judges.mas`
- `/home/jeloni/tabroom/web/funclib/event_selfstrike.mas`
- `/home/jeloni/tabroom/web/funclib/event_judgeprefs.mas`
- `/home/jeloni/tabroom/web/funclib/trpc_category_strikes.mas`
- `/home/jeloni/tabroom/web/funclib/strike_judges.mas`
- `/home/jeloni/tabroom/web/funclib/strike_name.mas`

**Documentation quality:** Code-only. No documentation of conflict type
hierarchy, resolution order, or interaction between conflict types.

**Likely need for expert review:** Yes. The strike/conflict system is the
foundation of fairness in all pairing and judge assignment. Incorrect
conflict resolution can produce rule violations. The interaction between
school conflicts, regional conflicts, hybrid conflicts, and MJP-tier strikes
is complex and undocumented. Time strikes use timezone-aware datetime
comparisons.

**Key behavioral notes:**
- Strike types in database: conflict, entry, elim, event, hybrid, region,
  dioregion, school, time, departure
- School conflicts are auto-applied unless `allow_judge_own` setting is on
- Region conflicts enforced at two levels: `region_avoid` (soft, +100 score)
  and `region_constrain` (hard, +5,000,000 score)
- Hybrid strikes: if school A has a hybrid conflict with entry X, ALL judges
  from school A are conflicted with entry X
- Time strikes compare round timeslot start/end against strike start/end
  with timezone conversion
- Rating tier strikes: if a tier is marked `strike` or `conflict`, any
  rating in that tier becomes a hard conflict
- `allow_repeat_judging` bypasses repeat-entry conflict generation entirely
- `allow_repeat_prelim_side` allows repeat judging if entry on opposite side
- `allow_repeat_elims` / `disallow_repeat_drop` control elim repeat rules

---

## 9. Tiebreak Logic

**Area name:** Tiebreak/Results Ordering System

**Why it is complex:**
This is the single largest file in the codebase (4,069 lines). It implements
a fully configurable, multi-tiered tiebreak system that supports 30+
tiebreak metrics for debate, speech, and congress. Each tournament defines a
"protocol" (tiebreak set) with prioritized tiebreak rules. The system
calculates all metrics, then sorts entries by the priority chain until ties
are broken.

Supported tiebreak types include: winloss, ballots, ranks, reciprocals,
points, opp_wins, opp_points, opp_ballots, opp_seed, headtohead, downs,
preponderance, judgepref, judgevar, three_way_point, best_po, student_rank,
entry_vote, and more.

Each tiebreak can be configured with:
- Count scope: all rounds, prelim only, elim only, specific round
- High-low dropping (drop N highest/lowest scores)
- Truncation (cap max rank)
- Multiplier
- Chair-only filtering
- Composite references (tiebreak based on another protocol's results)
- Violation flags (for code-of-conduct violations)

The system handles forfeits, byes, DQs, drops, and team-based aggregation
differently per event type (debate, speech, congress, WSDC, mock trial,
WUDC).

**Primary source files:**
- `/home/jeloni/tabroom/web/tabbing/results/order_entries.mas` (4,069 lines -- the core engine)
- `/home/jeloni/tabroom/web/tabbing/results/order_speakers.mas` (speaker award ordering)
- `/home/jeloni/tabroom/web/tabbing/results/results_table.mas` (results display)
- `/home/jeloni/tabroom/web/tabbing/results/results_csv.mas` (CSV export)
- `/home/jeloni/tabroom/web/funclib/tiebreak_types.mas` (199 lines -- determines which score types are needed)
- `/home/jeloni/tabroom/web/funclib/tiebreak_name.mas` (display name mapping)
- `/home/jeloni/tabroom/web/funclib/district_tiebreakers.mas` (NSDA Districts preset tiebreak creation)
- `/home/jeloni/tabroom/web/funclib/congress_ties.mas` (congress session linking for tiebreaks)

**Documentation quality:** Weak. The file has some comments (e.g., "i do so
hate you Perl" and "Shut up") but no documentation of the tiebreak
evaluation algorithm. The 30+ metric implementations are undocumented.
District tiebreakers are created programmatically in `district_tiebreakers.mas`
which serves as implicit documentation of NSDA Districts rules.

**Likely need for expert review:** Yes, absolutely. This is the most critical
file for competitive integrity. The tiebreak system must perfectly match
community-understood rules (NSDA, NCFL, WUDC, APDA, etc.). High-low
dropping, truncation, composite tiebreaks, and forfeit handling all have
edge cases. The file explicitly guards against composite-of-composite
tiebreaks (which would cause infinite loops) and opponent-seed composites
(circular dependency). The `$seed = $tourn->start->epoch` line uses
tournament start date as random seed for deterministic tiebreaking, which is
undocumented behavior.

**Key behavioral notes:**
- Composite tiebreaks reference another protocol's full results as a single
  tiebreak value (e.g., "IE Prelim Composite" at NSDA Nationals)
- Composite-of-composite is explicitly blocked with error message
- opp_seed in composites is blocked (circular dependency)
- Forfeits can be configured to rank last via `forfeits_rank_last`
- `forfeits_never_break` prevents forfeiting entries from advancing
- High-low dropping supports threshold (minimum rounds before dropping)
  and target (which score to drop -- high, low, or both)
- `truncate_smallest` normalizes scores across unequal section sizes
- Score types that are skipped: rfd, comments, time, title, categories,
  rubric, po, strike, no_strike
- Round types normalized: highlow, snaked_prelim, highhigh all treated as
  "prelim" for scoring purposes
- WSDC adds "refute" score type when ranks are in the tiebreak set

---

## 10. Break/Advancement Logic

**Area name:** Break to Elimination Rounds / Advancement

**Why it is complex:**
Break logic differs fundamentally between debate and speech:

**Debate breaks** (`break_debate.mhtml`, 711 lines):
- Entries are seeded from results ordering, then placed into a power-of-2
  bracket (e.g., 16 entries -> 4-seed bracket with 1v16, 2v15, etc.)
- Non-power-of-2 fields generate byes for top seeds
- Double elimination tracks winners bracket and losers bracket separately,
  with bracket-factor calculations that alternate between 2x and 1.5x
  multipliers per elimination round
- Same-school matchups in elims auto-resolve: lower seed advances
  automatically (scores created as 1-0)
- Side assignment uses `round_elim_dueaff.mas` to determine who should be aff
- NSDA Districts 4-to-3 qualifier scenario has hardcoded special-case logic

**Speech breaks** (`break_speech.mhtml`):
- Entries ranked by results ordering, top N advance to next round
- Multiple sections created with snaking assignment
- NCFL uses 5-judge finals panels
- Region avoidance applied during speech elim paneling

**Primary source files:**
- `/home/jeloni/tabroom/web/tabbing/break/break_debate.mhtml` (711 lines)
- `/home/jeloni/tabroom/web/tabbing/break/break_speech.mhtml`
- `/home/jeloni/tabroom/web/tabbing/break/break_congress.mhtml`
- `/home/jeloni/tabroom/web/tabbing/break/break_wudc.mhtml`
- `/home/jeloni/tabroom/web/tabbing/break/ncfl_snake.mhtml`
- `/home/jeloni/tabroom/web/tabbing/break/ready_status.mas`
- `/home/jeloni/tabroom/web/funclib/event_breakable.mas` (167 lines -- determines if events are ready to advance)

**Documentation quality:** Code-only. The double-elimination bracket logic
has no external documentation. The NSDA Districts 4-to-3 special case is
commented as "HARD CODE THIS NONSENSE UNAPOLOGETICALLY!"

**Likely need for expert review:** Yes. Double elimination bracket
calculation uses a complex multiplier system (alternating 2x and 1.5x per
round depth), losers-bracket seeding inverts seeds, and repeat-debate
detection triggers bracket reversal. The power-of-2 bracket function and
same-school auto-advancement are correctness-critical.

**Key behavioral notes:**
- `bracket()` function: finds next power of 2 >= input
- Master seed = bracket_target + 1; opponent = master_seed - my_seed
- Same-school elim matchups: lower seed auto-wins, panel marked as bye,
  ballots marked as audited
- Double elimination losers bracket inversion: if seed <= num_winners,
  new_seed = (2 * num_winners + 1) - seed
- Repeat-debate detection in losers bracket: if any pairing repeats a
  previous elim matchup, bracket is "flipped" (offset by +/-1)
- NSDA Districts: when 4 entries remain and 3 qualify, winners get byes and
  losers debate for 3rd qualifier spot
- `no_elims` entry setting prevents specific entries from advancing
- Elim-to-elim advancement uses bracket position from previous round
  (winner inherits panel bracket number), not seed

---

## 11. Sweepstakes Calculation

**Area name:** Sweepstakes / Team Awards Calculation

**Why it is complex:**
Sweepstakes aggregates individual entry results into school-level scores
with configurable rules. The system supports nested sweep sets (parent sets
can contain child sets), per-event limits, per-school limits, wildcard
entries, and multiple counting strategies (by entry, by person, by student
with max_per_person caps).

The recursive `sweep_set()` function processes child sets first, then
applies the parent set's rules to aggregate results. This tree structure
allows complex award schemes (e.g., "Best 2 debate + best 2 speech entries"
as children under a "Overall" parent).

Point assignment comes from `sweep_tourn.mas` which maps tournament results
to point values based on configurable rules, event types, round counts, and
multipliers.

**Primary source files:**
- `/home/jeloni/tabroom/web/tabbing/results/sweep_schools.mas` (304 lines -- school sweepstakes)
- `/home/jeloni/tabroom/web/tabbing/results/sweep_students.mas` (student sweepstakes)
- `/home/jeloni/tabroom/web/tabbing/results/sweep_tourn.mas` (point calculation engine)
- `/home/jeloni/tabroom/web/tabbing/results/nsda_sweepstakes.mhtml` (NSDA-specific sweepstakes)
- `/home/jeloni/tabroom/web/tabbing/results/nsda_points.mhtml`

**Documentation quality:** Code-only. The recursive sweep_set structure and
rule application order are undocumented.

**Likely need for expert review:** No, with caveats. The algorithm is
straightforward (sort entries by points, apply counting limits, sum per
school), but the recursive child-set processing and the interaction between
`entries`, `events`, `event_entries`, and `wildcards` limits creates
edge cases that need careful testing.

**Key behavioral notes:**
- Hybrid entries split points 50/50 between both schools
- Rules: `events` (max events counted), `entries` (max entries counted),
  `wildcards` (extra entries beyond limits), `event_entries` (max per event)
- `one_per_person`: only one entry per student counts
- `by_person`: counts by person rather than entry
- `max_entry_persons`: caps points from any single entry
- `set_limit` and `set_event_limit`: per-child-set caps
- Child sets can have their own limits independent of parent
- Results sorted by points descending; limits applied in order so highest-scoring entries are counted first

---

## 12. Ballot Validation and Audit Logic

**Area name:** Ballot Validation, Audit, and Result Integrity

**Why it is complex:**
The audit system determines whether ballots are "complete" (audited) and
ready for results calculation. It operates at three levels:
1. **Score-based audit**: ballots with non-null scores are marked audited
2. **Bye/forfeit audit**: bye and forfeit ballots are auto-audited
3. **Zero-round cleanup**: ballots without scores and without bye/forfeit
   flags are marked un-audited

Bracket recalculation is performed for powered rounds: the panel bracket
value is set to the highest win count among entries in that panel.

The `panel_winner.mas` function determines debate winners using a vote-count
approach (count of winloss=1 scores > count of winloss=0 scores), supporting
both single-judge and multi-judge panels. It returns the winner, their side,
and the vote record (e.g., "2-1").

The `ballot_panel_fix.mas` handles data repair scenarios. The
`event_breakable.mas` determines tournament-wide advancement readiness by
checking for unaudited ballots across all rounds.

**Primary source files:**
- `/home/jeloni/tabroom/web/funclib/round_audit.mas` (85 lines -- round-level audit)
- `/home/jeloni/tabroom/web/funclib/panel_winner.mas` (113 lines -- winner determination)
- `/home/jeloni/tabroom/web/funclib/ballot_panel_fix.mas` (data repair)
- `/home/jeloni/tabroom/web/funclib/event_breakable.mas` (167 lines -- advancement readiness)
- `/home/jeloni/tabroom/web/funclib/event_ballots.mas`
- `/home/jeloni/tabroom/web/funclib/round_ballots.mas`
- `/home/jeloni/tabroom/web/funclib/round_ballot_json.mas`
- `/home/jeloni/tabroom/web/funclib/round_ballot_strings.mas`
- `/home/jeloni/tabroom/web/funclib/timeslot_ballots.mas`
- `/home/jeloni/tabroom/web/funclib/other_ballots.mas`

**Documentation quality:** Code-only. The three audit SQL statements in
`round_audit.mas` are clear but undocumented regarding edge cases (e.g., what
happens when a ballot has some null and some non-null scores).

**Likely need for expert review:** No, but thorough testing is needed. The
audit logic is straightforward SQL but the winner-determination query in
`panel_winner.mas` uses a correlated subquery comparison (wins > losses)
that must handle edge cases like split decisions, no-shows, and partially
entered results. The `event_breakable.mas` readiness check must correctly
identify when all ballots in all rounds are complete, accounting for drops,
byes, and forfeits.

**Key behavioral notes:**
- Audit is a boolean flag on each ballot; batch-set via three SQL UPDATE
  statements per round
- Score-based audit: `ballot.audit = 1 WHERE score.value IS NOT NULL`
- Bye/forfeit audit: `ballot.audit = 1 WHERE panel.bye=1 OR ballot.bye=1 OR ballot.forfeit=1`
- Zero cleanup: `ballot.audit = 0 WHERE bye!=0 AND forfeit!=0 AND no scores exist`
  (Note: this condition appears inverted -- `bye != 0` should likely be `bye = 0`)
- Panel bracket recalculation only runs for powered rounds (not prelim, not elim)
- Winner determination: majority of winloss=1 scores across all judges
- `event_breakable.mas` checks three conditions: had-final (event complete),
  had-single (single elim panel done), advance-me (all ballots audited)
- The system distinguishes between "in progress", "ready to advance", and
  "done" states

---

## Summary Risk Matrix

| Area | Complexity | Lines of Code | Documentation | Expert Review | Risk Level |
|------|-----------|--------------|---------------|--------------|------------|
| Tiebreak Logic | Very High | 4,069 | Weak | Required | CRITICAL |
| Judge Assignment | Very High | 2,120 | Weak | Required | CRITICAL |
| Debate Powermatching | Very High | 2,725 (combined) | Code-only | Required | CRITICAL |
| Speech Paneling | High | 1,012 | Code-only | Required | HIGH |
| Congress Chambering | High | 913 | Code-only | Required | HIGH |
| Break/Advancement | High | 711+ | Code-only | Required | HIGH |
| Conflicts/Strikes | High | Distributed | Code-only | Required | HIGH |
| Speaker Order | Medium | ~500 (distributed) | Code-only | Recommended | MEDIUM |
| Room Assignment | Medium | 866 | Code-only | Not required | MEDIUM |
| Sweepstakes | Medium | 304+ | Code-only | Not required | MEDIUM |
| Burden Calculation | Low | 67 | Code-only | Not required | LOW |
| Ballot Audit | Low | 85 | Code-only | Not required | LOW |

**Overall assessment:** The codebase has zero external algorithm
documentation. All business logic is embedded in Mason/Perl templates
without separation between algorithm and presentation. The three most
critical areas (tiebreaks, judge assignment, powermatching) total over 8,900
lines of algorithmic code with extensive special-casing for NSDA, NCFL, APDA,
WSDC, and WUDC rule sets. A rebuild must either preserve exact behavioral
parity (requiring exhaustive regression testing against production data) or
explicitly document and gain community approval for any behavioral changes.
