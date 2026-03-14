# Data and Entity Inventory

## Overview

The Tabroom database consists of **108 tables** in MariaDB, mapped to **107 Perl ORM models** via Class::DBI. The schema includes **19 EAV-style `*_setting` tables** that store flexible key-value configuration across nearly every entity type. The largest tables by auto-increment suggest massive scale: `ballot` (33.6M+), `score` (high), `campus_log` (4.9M+), `autoqueue` (1.1M+).

**Sources:** `doc/sql/current-schema.sql`, `web/lib/Tab/*.pm`

---

## Entity Cluster: Tournament and Event Structure

### tourn
- **Role:** Central entity — a tournament with dates, location, name
- **Major relationships:** Has many events, categories, rounds, schools, permissions, settings
- **Lifecycle:** Created → configured → registration open → live operation → completed → archived
- **Source tables:** `tourn`, `tourn_setting`, `tourn_site`, `tourn_circuit`, `tourn_fee`, `tourn_ignore`
- **Notes:** 200-line ORM model; `tourn_setting` is the largest EAV table by usage

### event
- **Role:** A competitive event within a tournament (e.g., "Lincoln-Douglas Debate", "Original Oratory")
- **Major relationships:** Belongs to tourn and category; has many rounds, entries
- **Lifecycle:** Created during setup → rounds scheduled → paired → results computed
- **Source tables:** `event`, `event_setting`
- **Notes:** Events represent the granular competitive divisions; categories group related events

### category
- **Role:** Groups related events within a tournament (e.g., "Debate", "Speech", "Congress")
- **Major relationships:** Has many events, judge pools, settings; determines format-specific behavior
- **Lifecycle:** Created during setup, persists through tournament
- **Source tables:** `category`, `category_setting`
- **Notes:** 162-line ORM model; categories drive format polymorphism (debate vs speech vs congress)

### round
- **Role:** A round of competition within an event (e.g., "Round 1", "Quarterfinals")
- **Major relationships:** Belongs to event and timeslot; has many panels
- **Lifecycle:** Created → scheduled → paired → started → ballots entered → completed
- **Source tables:** `round`, `round_setting`
- **Notes:** 168-line ORM model; rounds are the primary unit of pairing/scheduling operations

### timeslot
- **Role:** A time block within a tournament for scheduling rounds
- **Major relationships:** Belongs to tourn; has many rounds
- **Source tables:** `timeslot`

### pattern
- **Role:** Scheduling pattern template for round ordering
- **Source tables:** `pattern`

### protocol
- **Role:** Rules/scoring protocol configuration
- **Major relationships:** Has many settings
- **Source tables:** `protocol`, `protocol_setting`
- **Notes:** Protocols define ballot structure, scoring rules, tiebreaker order

---

## Entity Cluster: User Accounts and Identity

### person
- **Role:** A human user of the system (coach, judge, student, admin)
- **Major relationships:** Has many permissions, sessions, settings, students, judges, chapters, conflicts
- **Lifecycle:** Created (registration) → active → last accessed
- **Source tables:** `person`, `person_setting`, `person_quiz`
- **Notes:** 248-line ORM model (largest); contains auth credentials, profile info, site_admin flag

### session
- **Role:** Authentication session
- **Major relationships:** Belongs to person; stores defaults (JSON) and SU reference
- **Source tables:** `session`
- **Notes:** Stores per-session state including current tournament, display preferences

### login
- **Role:** Login attempt/history tracking
- **Source tables:** `login`

---

## Entity Cluster: Participants and Organizations

### chapter
- **Role:** A school or program (organizational unit that competes)
- **Major relationships:** Has many students, chapter_judges, schools (tournament registrations), settings, circuits
- **Lifecycle:** Created → schools register at tournaments → ongoing
- **Source tables:** `chapter`, `chapter_setting`, `chapter_circuit`
- **Notes:** 180-line ORM model; chapters persist across tournaments; schools are tournament-specific instances

### student
- **Role:** A student competitor affiliated with a chapter
- **Major relationships:** Belongs to person and chapter; has many entry_students
- **Source tables:** `student`, `student_setting`, `student_vote`, `student_ballot`
- **Notes:** 140-line model; students link to entries via entry_student junction

### school
- **Role:** A chapter's registration at a specific tournament
- **Major relationships:** Belongs to tourn and chapter; has many entries, judges
- **Lifecycle:** Created when chapter registers → entries/judges added → tournament completes
- **Source tables:** `school`, `school_setting`
- **Notes:** 168-line model; school is the tournament-specific projection of a chapter

### entry
- **Role:** A competitive entry in a tournament event (individual or team)
- **Major relationships:** Belongs to event and school; has many entry_students, ballots
- **Lifecycle:** Registered → active → dropped/withdrawn or completed
- **Source tables:** `entry`, `entry_setting`, `entry_student`
- **Notes:** 177-line model; entries can be individuals (1 student) or teams (2+ students)

### circuit
- **Role:** An organizational grouping of chapters (league, conference, state)
- **Major relationships:** Has many chapters (via chapter_circuit), tournaments (via tourn_circuit), settings
- **Source tables:** `circuit`, `circuit_setting`, `circuit_membership`
- **Notes:** 148-line model; circuits provide cross-tournament organizational structure

### region
- **Role:** A geographic or organizational subdivision within a circuit or district
- **Source tables:** `region`, `region_setting`, `region_fine`

### district
- **Role:** An NSDA district for qualification/advancement purposes
- **Source tables:** `district`
- **Notes:** Districts are NSDA-specific; they drive qualification pipelines to nationals

### diocese
- **Role:** Unclear — possibly a legacy or alternate organizational unit (Catholic diocese?)
- **Source tables:** `diocese`
- **Notes:** Minimal presence in codebase; `web/user/diocese/` exists as a route family

---

## Entity Cluster: Judging and Conflicts

### chapter_judge
- **Role:** A judge's persistent identity within a chapter (cross-tournament)
- **Major relationships:** Belongs to person and chapter; has many tournament-specific judges
- **Source tables:** `chapter_judge`

### judge
- **Role:** A judge's tournament-specific instance
- **Major relationships:** Belongs to school (and by extension chapter_judge/person); has many ballots, shifts
- **Lifecycle:** Registered → assigned to pools → assigned to panels → ballots entered
- **Source tables:** `judge`, `judge_setting`, `judge_hire`
- **Notes:** 155-line model; judge hiring (`judge_hire`) handles cross-school judge sharing

### jpool (Judge Pool)
- **Role:** A grouping of judges available for specific rounds
- **Major relationships:** Has many judges (via jpool_judge), rounds (via jpool_round), settings
- **Source tables:** `jpool`, `jpool_judge`, `jpool_round`, `jpool_setting`
- **Notes:** 132-line model; judge pools constrain which judges can be assigned to which rounds

### shift
- **Role:** Judge availability time blocks
- **Source tables:** `shift`

### strike
- **Role:** A constraint preventing a judge from judging a specific entry/school
- **Major relationships:** Links judge to entry, school, event, or other entities
- **Source tables:** `strike`
- **Notes:** Strikes can be mutual conflicts, coach strikes, or administrative strikes

### conflict
- **Role:** Personal conflict between two people (bidirectional)
- **Major relationships:** Links two person records
- **Source tables:** `conflict`
- **Notes:** Distinct from strikes; conflicts are person-level, strikes are judge-entry level

### rating
- **Role:** Judge preference rating assigned by a school/entry to a judge
- **Major relationships:** Part of the prefs system
- **Source tables:** `rating`, `rating_subset`, `rating_tier`
- **Notes:** Rating system supports ordinal, tiered, and percentage-based preference schemes

---

## Entity Cluster: Rounds, Panels, and Rooms

### panel
- **Role:** A competition unit within a round (a "section" or "flight")
- **Major relationships:** Belongs to round and room; has many ballots
- **Source tables:** `panel`, `panel_setting`
- **Notes:** 137-line model; "panel" is the generic term for what might be called a "section" in speech or a "round room" in debate

### room
- **Role:** A physical or virtual room where competition occurs
- **Major relationships:** Belongs to site; has many room_strikes
- **Source tables:** `room`, `room_strike`

### site
- **Role:** A physical venue hosting competition
- **Major relationships:** Has many rooms; linked to tourn via tourn_site
- **Source tables:** `site`, `tourn_site`

### rpool (Room Pool)
- **Role:** A grouping of rooms available for specific rounds
- **Major relationships:** Has many rooms (via rpool_room), rounds (via rpool_round), settings
- **Source tables:** `rpool`, `rpool_room`, `rpool_round`, `rpool_setting`
- **Notes:** 129-line model; parallels jpool structure for rooms

---

## Entity Cluster: Ballots, Scoring, and Results

### ballot
- **Role:** Links an entry to a judge in a panel — the core competition record
- **Major relationships:** Belongs to panel, judge, entry; has many scores
- **Lifecycle:** Created during pairing → started → scores entered → audited
- **Source tables:** `ballot`
- **Key columns:** `side`, `speakerorder`, `seat`, `chair`, `bye`, `forfeit`, `audit`, `judge_started`, `entered_by`, `audited_by`
- **Notes:** Uniquely constrained on (entry, judge, panel). Auto-increment at 33.6M+ indicates massive historical data.

### score
- **Role:** Individual scoring values on a ballot
- **Major relationships:** Belongs to ballot
- **Source tables:** `score`
- **Notes:** Score structure varies by format (debate wins/points, speech ranks, congress scores)

### result / result_set / result_key / result_value
- **Role:** Computed aggregated results for entries across rounds
- **Major relationships:** Result_set groups results; result_key defines metrics; result_value stores computed values
- **Source tables:** `result`, `result_set`, `result_key`, `result_value`
- **Notes:** This is the denormalized results cache, computed from ballot/score data

### tiebreak
- **Role:** Configuration for tiebreaker order and rules
- **Major relationships:** Belongs to protocol
- **Source tables:** `tiebreak`
- **Notes:** Tiebreaker logic is one of the most complex algorithmic areas

---

## Entity Cluster: Sweepstakes and Awards

### sweep_set
- **Role:** A sweepstakes competition configuration
- **Source tables:** `sweep_set`

### sweep_rule
- **Role:** Rules for calculating sweepstakes points
- **Source tables:** `sweep_rule`

### sweep_event
- **Role:** Events included in sweepstakes calculation
- **Source tables:** `sweep_event`

### sweep_include
- **Role:** Inclusion criteria for sweepstakes
- **Source tables:** `sweep_include`

### sweep_award
- **Role:** A sweepstakes award definition
- **Source tables:** `sweep_award`, `sweep_award_event`

---

## Entity Cluster: Settings and Configuration (EAV)

Tabroom uses **19 EAV-style `*_setting` tables** that share a common pattern:

| Setting Table | Parent Entity | ORM Model |
|--------------|--------------|-----------|
| `tourn_setting` | `tourn` | `TournSetting.pm` |
| `category_setting` | `category` | `CategorySetting.pm` |
| `event_setting` | `event` | `EventSetting.pm` |
| `round_setting` | `round` | `RoundSetting.pm` |
| `panel_setting` | `panel` | `PanelSetting.pm` |
| `person_setting` | `person` | `PersonSetting.pm` |
| `chapter_setting` | `chapter` | `ChapterSetting.pm` |
| `circuit_setting` | `circuit` | `CircuitSetting.pm` |
| `region_setting` | `region` | `RegionSetting.pm` |
| `school_setting` | `school` | `SchoolSetting.pm` |
| `entry_setting` | `entry` | `EntrySetting.pm` |
| `student_setting` | `student` | `StudentSetting.pm` |
| `judge_setting` | `judge` | `JudgeSetting.pm` |
| `jpool_setting` | `jpool` | `JPoolSetting.pm` |
| `rpool_setting` | `rpool` | `RPoolSetting.pm` |
| `protocol_setting` | `protocol` | `ProtocolSetting.pm` |
| `tabroom_setting` | (global) | `TabroomSetting.pm` |
| `setting` | (global labels) | `Setting.pm` |
| `setting_label` | (label metadata) | `SettingLabel.pm` |

**Common EAV schema pattern:**
```sql
CREATE TABLE `*_setting` (
  id int PRIMARY KEY,
  [parent_entity] int NOT NULL,   -- FK to parent
  tag varchar(100) NOT NULL,      -- setting name
  value varchar(255),             -- setting value (or type indicator)
  value_date datetime,            -- for date-type values
  value_text mediumtext,          -- for text/JSON values (may be compressed)
  timestamp ...
)
```

**Value type dispatch** (from `Person.pm:setting()` method, representative of all entities):
- `value = "text"` → actual value in `value_text`
- `value = "date"` → actual value in `value_date`
- `value = "json"` → compressed JSON in `value_text`
- Otherwise → `value` column is the actual value

**Notes:** This EAV pattern represents a massive share of the product's configuration surface. The rebuild plan explicitly calls for eliminating this pattern in favor of strongly-typed alternatives.

---

## Entity Cluster: Finances

### invoice
- **Role:** Tournament registration invoice for a school
- **Source tables:** `invoice`

### fine
- **Role:** Penalty charges (judge obligation fines, etc.)
- **Source tables:** `fine`, `region_fine`

### tourn_fee
- **Role:** Fee schedule configuration for tournaments
- **Source tables:** `tourn_fee`

### concession / concession_type / concession_option / concession_purchase / concession_purchase_option
- **Role:** Concession sales system (food, merchandise at tournaments)
- **Source tables:** `concession`, `concession_type`, `concession_option`, `concession_purchase`, `concession_purchase_option`
- **Notes:** A complete mini e-commerce system embedded in the tournament platform

---

## Entity Cluster: Content and Publication

### webpage
- **Role:** Tournament-specific web pages (announcements, information)
- **Source tables:** `webpage`

### file
- **Role:** Uploaded file storage references
- **Source tables:** `file`

### topic
- **Role:** Debate topics / resolutions
- **Source tables:** `topic`

### caselist
- **Role:** Debate caselist tracking (arguments used by entries)
- **Source tables:** `caselist`

### ad
- **Role:** Advertisement management
- **Source tables:** `ad`

---

## Entity Cluster: Notifications and Audit

### email
- **Role:** Email message records
- **Source tables:** `email`

### follower
- **Role:** Users following tournaments/entries for notifications
- **Source tables:** `follower`

### change_log
- **Role:** Audit trail for changes
- **Source tables:** `change_log`

### campus_log
- **Role:** Online tournament ("Campus") activity logging
- **Source tables:** `campus_log`
- **Notes:** 4.9M+ rows; heavily used for online tournament monitoring

### autoqueue
- **Role:** Scheduled/queued operations (blast emails, auto-pair, etc.)
- **Source tables:** `autoqueue`
- **Notes:** 1.1M+ rows; used for deferred/scheduled tournament operations

---

## Entity Cluster: Miscellaneous

### qualifier
- **Role:** District qualification tracking
- **Source tables:** `qualifier`

### weekend
- **Role:** Multi-weekend tournament support
- **Source tables:** `weekend`

### hotel / housing / housing_slots
- **Role:** Tournament housing management
- **Source tables:** `hotel`, `housing`, `housing_slots`

### practice / practice_student
- **Role:** Practice round tracking
- **Source tables:** `practice`, `practice_student`

### quiz / person_quiz
- **Role:** Assessment/quiz system (possibly for judge training)
- **Source tables:** `quiz`, `person_quiz`

### stats
- **Role:** Aggregated statistics
- **Source tables:** `stats`

### nsda_category
- **Role:** NSDA event category mapping
- **Source tables:** `nsda_category`

---

## Entity Relationship Summary

```
person ──┬── permission ──┬── tourn
         │                ├── chapter
         │                ├── circuit
         │                ├── region
         │                └── district
         ├── chapter_judge ── judge ──── ballot ── score
         ├── student ── entry_student ── entry ──┘
         └── conflict

tourn ──┬── category ── event ── round ── panel ── ballot
        ├── school ──── entry                       │
        ├── site ────── room ───────────────────────┘
        ├── timeslot ── round
        ├── jpool ──── jpool_judge / jpool_round
        └── rpool ──── rpool_room / rpool_round
```

## Open Questions

- The `diocese` table's purpose and relationship to districts/regions is unclear
- Whether `student_ballot` and `student_vote` are used for congress-specific scoring needs verification
- The `circuit_membership` table's relationship to `chapter_circuit` needs clarification
- The `stats` table's contents and update mechanism are unknown
- The `weekend` entity's relationship to multi-day vs multi-weekend tournaments needs clarification
- The `housing` system's current usage level is unknown — may be rarely used
- Whether `practice` and `practice_student` are actively used or legacy features is unclear
