# Data and State Model Pack

## Purpose

This document describes the major entity lifecycles, state transitions, and invariants that must be preserved in the rebuild. It builds on the Phase 1 data/entity inventory, adding lifecycle and state information extracted from code behavior.

---

## Entity Lifecycles

### Tournament Lifecycle

```
Requested → Created → Setup → Registration Open → Registration Frozen →
  Live (rounds running) → Completed → Archived
```

**States:**
- **Requested:** Tournament request submitted, awaiting approval
- **Created:** Basic record exists (name, dates, location)
- **Setup:** Events, categories, schedule, rules being configured
- **Registration Open:** Schools can register entries and judges
- **Registration Frozen:** No more changes to entries/judges (freeze_deadline passed)
- **Live:** Rounds are being paired, ballots entered, results computed
- **Completed:** All rounds finished, results published
- **Archived:** Historical record

**Key transitions:**
- Created → Setup: Tournament owner accesses `setup/tourn/main.mhtml`
- Setup → Registration Open: Implicit — tournament has tourn_settings populated
- Registration Open → Frozen: `freeze_deadline` passes
- Live → Completed: All events have final results published
- Note: There is no explicit state column. State is inferred from settings, dates, and round status.

**Invariant:** A tournament must have `tourn_settings` populated before any operations are allowed (enforced by autohandlers).

### Event Lifecycle

```
Created → Configured → Rounds Scheduled → Pairing Started →
  Prelims Running → Breaks Determined → Elims Running → Complete
```

**States are implicit** — determined by the state of the event's rounds:
- No rounds with panels → not yet paired
- All ballots audited in a round → round complete
- Break round created → breaks determined
- Final round complete → event complete

### Round Lifecycle

```
Created → Scheduled (in timeslot) → Paired (panels created) →
  Published (schematic visible) → Started (ballots in progress) →
  Complete (all ballots audited) → Results Computed
```

**Key state indicators:**
- `round` has panels → paired
- `round_setting.strikes_published` → strikes/side info published
- `round_setting.flip_published` → flip results published
- Panels have ballots with `audit = 1` → complete
- `result_set` exists for this round → results computed

### Entry Lifecycle

```
Registered → Active → [Waitlisted] → [Dropped/Withdrawn] →
  Competing → Eliminated/Completed
```

**Key columns:**
- `entry.active` — 1 = active, 0 = dropped/inactive
- Waitlist status inferred from settings and registration state
- Elimination status inferred from break/advancement records

### Ballot Lifecycle

```
Created (during pairing) → Started (judge begins) →
  Entered (scores submitted) → Audited (verified) → [Corrected]
```

**Key columns:**
- `ballot.judge_started` — datetime when judge began
- `ballot.started_by` — who started it
- `ballot.entered_by` — who entered scores
- `ballot.audited_by` — who verified
- `ballot.audit` — 0 = unaudited, 1 = audited
- `ballot.bye` — 1 = bye ballot
- `ballot.forfeit` — 1 = forfeit

### Judge Lifecycle (Tournament-Scoped)

```
Registered → Pool Assigned → [Shifts Set] → Available →
  Assigned (to panels) → Judging → Complete
```

**Key state indicators:**
- Judge exists → registered
- `jpool_judge` entries → pool assigned
- `judge_setting` for shifts → shifts configured
- Ballots exist for this judge → assigned to panels
- Ballots audited → complete

### School Lifecycle (Tournament-Scoped)

```
Registered → Entries Added → Judges Added → [Waitlisted Entries] →
  Invoice Generated → [Paid] → Active → Complete
```

**Key state indicators:**
- School exists → registered
- Entries exist → entries added
- Judges exist → judges added
- Invoice exists → invoiced
- Invoice.paid → paid

---

## Key State Transitions and Rules

### Pairing State Machine

```
                ┌─────────────────────────────────────┐
                │                                     │
Round Created ──┤──→ Panels Created ──→ Judges Assigned ──→ Rooms Assigned ──→ Published
                │         │                    │                │
                │         ▼                    ▼                ▼
                │    Manual Adjust       Manual Swap      Manual Change
                │         │                    │                │
                │         └────────────────────┴────────────────┘
                │                              │
                │                              ▼
                │                        Re-Published
                └─────────────────────────────────────┘
```

**Rules:**
1. Panels cannot be created if the round has no entries
2. Judges cannot be assigned until panels exist
3. Rooms cannot be assigned until panels exist
4. Publication can happen at any point after panels exist
5. Manual adjustments can happen after publication (re-publish required)
6. Creating new panels destroys existing panels and all associated ballots/scores

### Results State Machine

```
Ballots Entered → Ballots Audited → Results Computed → Results Published
                                           │
                                    ┌──────┴──────┐
                                    ▼              ▼
                              Preliminary     Speaker Awards
                               Seeds              │
                                    │              ▼
                                    ▼          Published
                              Break Round
                                Created
                                    │
                                    ▼
                              Elim Paired
                                    │
                                    ▼
                              (repeat ballot→results cycle)
```

**Rules:**
1. Results can only be computed when all ballots in a round are audited
2. Breaks require all preliminary rounds to have results
3. Elimination rounds follow the same ballot→audit→results cycle
4. Speaker awards are computed independently from team/entry results

---

## Key Invariants

### Data Integrity Invariants

1. **Ballot uniqueness:** `(entry, judge, panel)` must be unique (enforced by DB constraint)
2. **Ballot side-order uniqueness:** `(panel, judge, side, speakerorder)` must be unique
3. **One entry per student per event:** An entry_student cannot have the same student in multiple entries of the same event
4. **School belongs to tournament:** Every school has exactly one tourn and one chapter
5. **Entry belongs to event and school:** Every entry has exactly one event and one school
6. **Round belongs to event:** Every round has exactly one event
7. **Panel belongs to round:** Every panel has exactly one round
8. **Judge pool membership:** A judge can belong to multiple pools, but pools are per-category
9. **Permission scoping:** A permission row has exactly one scope (tourn, chapter, circuit, region, or district)

### Business Invariants

1. **No self-judging:** A judge cannot judge a panel containing their own school's entries (enforced by conflict/strike system)
2. **Side balance:** In debate, entries should have balanced side assignments across rounds (enforced by side assignment algorithm)
3. **Judge obligation:** Schools must provide judges proportional to their entry count
4. **Entry cap enforcement:** Entries beyond the cap must go to waitlist
5. **Deadline enforcement:** Registration changes blocked after relevant deadline
6. **Audit before results:** Results should not be published from unaudited ballots
7. **Break requires prelims:** Elimination rounds cannot be created before preliminary results exist

### Format-Specific Invariants

**Debate:**
- Each panel has exactly 2 entries (except byes)
- Each entry has a side (aff/neg) per round
- Win/loss is determined per ballot (majority of judges determines round winner)

**Speech:**
- Each panel (section) has N entries where N is within min/max panel size
- Each entry receives a rank (1-N) from each judge
- No "winner" per section — rankings accumulate across rounds

**Congress:**
- Each panel (chamber) has N entries
- Scoring includes legislation quality, PO performance, and speech quality
- Recency tracking prevents the same students from being in the same chamber repeatedly

---

## Entity Relationship Constraints

### Cascade Delete Rules (from FK constraints in schema)

| Parent | Child | On Delete |
|--------|-------|-----------|
| tourn | campus_log | CASCADE |
| tourn | (most tourn children) | CASCADE (expected) |
| person | permission | (implied, not enforced) |
| round | panel | CASCADE (expected) |
| panel | ballot | CASCADE (expected) |

**Note:** The legacy schema has limited FK constraints. The rebuild should enforce referential integrity more strictly.

### Required Relationships

| Entity | Required Parent(s) |
|--------|-------------------|
| event | tourn, category |
| round | event, timeslot |
| panel | round |
| ballot | panel, entry, judge |
| score | ballot |
| school | tourn, chapter |
| entry | event, school |
| entry_student | entry, student |
| judge | (tournament-scoped, linked to school) |
| jpool_judge | jpool, judge |
| permission | person + exactly one scope entity |

---

## State Storage Patterns

### Implicit State (Current)
Most entity states are **implicit** in Tabroom — inferred from the presence/absence of related records and column values rather than explicit state columns. For example:
- A round is "paired" if it has panels
- A ballot is "complete" if `audit = 1`
- An entry is "dropped" if `active = 0`
- A tournament is "live" if any round has panels with unaudited ballots

### Recommended for Rebuild: Explicit State
Add explicit `status` enum columns to key entities:
- `tournament.status` — draft, setup, registration, live, completed, archived
- `round.status` — created, paired, published, in_progress, complete, results_published
- `ballot.status` — created, started, submitted, audited, corrected
- `entry.status` — registered, waitlisted, active, dropped, withdrawn, eliminated, completed

This makes state transitions explicit, auditable, and queryable without complex joins.

---

## Format Polymorphism Model

The rebuild must handle three fundamentally different competition formats. The recommended approach is **strategy pattern at the service layer**, not table-per-format.

### Shared Structure (All Formats)
- Tournament → Category → Event → Round → Panel → Ballot → Score
- Entry → EntryStudent → Student
- Judge → JPool → JPoolJudge

### Format-Specific Behavior

| Behavior | Debate | Speech | Congress |
|----------|--------|--------|---------|
| Pairing algorithm | Powermatching | Snake paneling | Chamber assignment |
| Panel size | 2 entries | N entries (configurable) | N entries (configurable) |
| Side assignment | Aff/Neg | Speaker order | Seating |
| Scoring | Win/Loss + Points | Ranks | Scores + PO |
| Results | Wins → Speaker Points | Ranks → Reciprocals | Composite scoring |
| Elimination | Bracket | Section breaks | Chamber breaks |
| Special features | Coin flips, side locks | Repeat avoidance | Recency tracking, legislation |

### Recommended Implementation
- Single `event` table with `format` enum (debate, speech, congress)
- Format-specific behavior via strategy services (PairingStrategy, ScoringStrategy, ResultsStrategy)
- Format-specific settings grouped in configuration entities
- Format-specific UI components selected based on event format
