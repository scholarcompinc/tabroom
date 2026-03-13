# Capability Domains: Batch C

## 1. Registration and Roster Management

### Registration and Roster Management

**Description:** Handles the registration of schools (programs) to tournaments, management of entries (competitors) in events, student rosters, judge registration from the school/coach perspective, contacts, waitlisting, concession purchases, invoicing/payments, and on-site check-in. This is the primary coach-facing registration workflow.

**Primary users:**
- Coaches / program directors (registering their school, entries, and judges)
- Tournament directors / tab staff (managing and overseeing registrations)
- Site admins (NSDA staff for nationals-level operations)

**Main entry points:**
- `/register/index.mhtml` -- registration landing page
- `/register/school/edit.mhtml` -- school general info (address, region, notes, on-site status)
- `/register/school/entries.mhtml` -- view/manage all entries for a school
- `/register/school/judges.mhtml` -- view/manage judges for a school by category
- `/register/school/contacts.mhtml` -- per-category adult contacts
- `/register/school/invoice.mhtml` -- financial invoice for the school
- `/register/school/concessions.mhtml` -- concession/merchandise purchases
- `/register/school/student_roster.mhtml` -- manage the student roster from chapter
- `/register/school/waitlist.mhtml` -- waitlisted entries
- `/register/school/problems.mhtml` -- NSDA Nats compliance checker
- `/register/school/followers.mhtml` -- email followers for the school
- `/register/school/notes.mhtml` -- admin notes on the school
- `/register/school/log.mhtml` -- change log (NSDA Nats)
- `/register/entry/edit.mhtml` -- edit a single entry
- `/register/entry/school.mhtml` -- entry school assignment
- `/register/event/index.mhtml` -- event-level entry management
- `/register/event/roster.mhtml` -- event roster view
- `/register/emails/` -- email composition and blast to registrants

**Major sub-capabilities:**
- School creation and linking to a Chapter (persistent program entity)
- School code assignment (multiple schemes: numeric, state-based, region-based)
- On-site check-in toggle
- Region/district/diocese association for schools
- Entry creation, editing, dropping, undropping, waitlisting, waitlist admission
- Entry code assignment and event switching
- Student-to-entry assignment (via EntryStudent join)
- Student roster management (add/remove students from chapter roster)
- Hybrid/cross-school entries (students from multiple schools on one entry)
- Entry qualification tracking (`qual_save.mhtml`)
- Piece/title management for IE events (`piece_titles.mhtml`)
- Entry seeds and breakout management
- Bulk entry operations (alter, drop, activate)
- Waitlist management with ranked ordering (`waitlist_rank` entry setting)
- Unconfirmed entry workflow (pending entries needing approval, especially NSDA Nats)
- Contact management per category (name, phone, email stored as JSON in school setting `category_contacts`)
- Concession/merchandise ordering
- Invoice generation and payment tracking
- Fine management (add/forgive fines on schools)
- Follower system (email notifications for school updates)
- School notes and admin notes log
- Judge obligation calculation per school (see Domain 2)
- Student form/release form tracking
- ADA accommodations flag on entries
- Registration email blasts with template system
- CSV import/export for entries and judges
- District student tracking
- NSDA membership verification integration

**Key entities:**
- `Tab::School` (`web/lib/Tab/School.pm`) -- columns: id, name, code, onsite, tourn, chapter, state, region, district, created_at, registered_by, timestamp. Has many: entries, judges, fines, invoices, hires, files, strikes, contacts, settings, purchases
- `Tab::SchoolSetting` (`web/lib/Tab/SchoolSetting.pm`) -- EAV settings: id, school, tag, value, value_date, value_text, last_changed, setting, timestamp
- `Tab::Entry` (`web/lib/Tab/Entry.pm`) -- columns: id, code, name, active, dropped, waitlist, unconfirmed, tourn, school, event, registered_by, ada, created_at. Has many: strikes, settings, ballots, changes, ratings, entry_students
- `Tab::EntrySetting` (`web/lib/Tab/EntrySetting.pm`) -- EAV: id, entry, tag, value, value_date, value_text
- `Tab::EntryStudent` (`web/lib/Tab/EntryStudent.pm`) -- join table: id, entry, student, timestamp
- `Tab::Student` (`web/lib/Tab/Student.pm`) -- columns: id, person, first, middle, last, chapter, novice, grad_year, retired, gender, person_request, phonetic, nsda. Has many: entry_students, settings, followers
- `Tab::Chapter` -- persistent program/school entity across tournaments
- `Tab::Contact` (`web/lib/Tab/Contact.pm`) -- columns: id, school, person, official, onsite, email, nsda, book, timestamp, created_at, created_by
- `Tab::Fine` -- fines on schools
- `Tab::Invoice` -- invoices for schools
- `Tab::ConcessionPurchase` / `Tab::ConcessionPurchaseOption` -- merchandise orders
- `Tab::Follower` -- email followers
- `Tab::ChangeLog` -- entry change history
- `Tab::Region` -- regional groupings of schools
- `Tab::District` -- NSDA district groupings

**Key settings/configuration:**
- School settings (SchoolSetting tags): `notes`, `notes_log`, `rejected`, `judge_surcharge`, `category_contacts`, `release_forms`, `original_school`
- Entry settings (EntrySetting tags): `waitlist_rank`, `rejected_by`, `rejected_at`, `accepted_by`, `accepted_at`
- Tourn settings: `nsda_nats`, `nsda_ms_nats`, `ncfl`, `hide_codes`, `school_codes`, `regions`, `no_school_judges`, `mock_trial_registration`, `no_registration_fees`, `student_form_confirm`, `student_form_label`, `category_adult_contact`, `refund_address`, `judges_waitlist`, `currency`, `concession_name`, `region_circuit`
- Event settings: `no_judge_burden`, `supp`, `online_mode`, `adjust_judge_rounds_owed`, `adjust_judges_owed`

**Key reports/exports:**
- `/register/reports/` -- extensive report suite including:
  - `entries_csv.mhtml`, `judges_csv.mhtml` -- CSV exports
  - `finance_report.mhtml`, `finance_csv.mhtml` -- financial reports
  - `school_list.mhtml`, `school_list_csv.mhtml` -- school listings
  - `contact_list.mhtml`, `contact_sheets.mhtml` -- contact reports
  - `student_cards.mhtml`, `judge_cards.mhtml` -- printed cards
  - `school_headcount.mhtml` -- headcount reports
  - `stats.mhtml` -- registration statistics
  - `fines.mhtml` -- fine reports
  - `prefs.mhtml` -- preference report
  - `strikes.mhtml` -- strike report
  - `paradigms.mhtml` -- paradigm report
  - `payments.mhtml` -- payment report
  - `ada.mhtml` -- ADA accommodation report
  - `diets.mhtml` -- dietary restriction report
  - NSDA-specific: `nsda_school_status.mhtml`, `nsda_ribbons.mhtml`, `nats_book.mhtml`, etc.
  - NCFL-specific: `ncfl_reports.mhtml`, `ncfl_fines.mhtml`, `ncfl_cards.mhtml`, etc.

**Integrations/dependencies:**
- NSDA membership API (`/funclib/nsda/` components, `supp_api.mhtml`)
- Chapter system (persistent school/student/judge records across tournaments)
- Judge obligation calculation system (`/funclib/judgemath/`)
- Email system for registration blasts
- Payment processing (invoice/payment_save)

**Documentation sources:**
- Code comments (sparse; one notable comment in `School.pm` judges list: "Somewhat Hideous Code replaced Even More Hideous Code")
- Inline help text in HTML templates (e.g., hire request explanations in `judges.mhtml`)

**Code sources:**
- `web/register/school/` -- school-level registration (60+ files)
- `web/register/entry/` -- entry-level management (30+ files)
- `web/register/event/` -- event-level entry management (45+ files)
- `web/register/emails/` -- email blast system
- `web/register/reports/` -- registration reports (110+ files)
- `web/register/menubar.mas` -- registration navigation with school selector
- `web/register/judge/` -- judge registration (tab-side, 80+ files)
- `web/lib/Tab/School.pm`, `SchoolSetting.pm`, `Entry.pm`, `EntrySetting.pm`, `EntryStudent.pm`, `Student.pm`, `Contact.pm`
- `web/funclib/school_*.mas` -- school utility functions
- `web/funclib/judgemath/` -- judge obligation calculations

**Complexity level:** High -- The registration system is the largest user-facing domain, with extensive special-casing for NSDA Nationals, NCFL, district tournaments, and mock trial. The school-entry-student relationship is mediated through Chapter for persistence across tournaments, and the obligation calculation system is deeply intertwined with judge management.

**Feature frequency / criticality:** Every tournament -- Registration is the fundamental starting point for all tournaments. Every tournament uses school and entry registration. Judge registration from the school side is used by most tournaments.

**Notes and open questions:**
- The menubar.mas file acts as both navigation and a data aggregation point, running judge obligation checks and NSDA status checks on every page load, which could be a performance concern
- The School entity is tournament-scoped (linked to a Tourn) while Chapter is the persistent cross-tournament entity; this dual-layer creates complexity
- There is significant NSDA Nationals-specific logic interleaved throughout (release forms, supplemental entries, membership checks, pending/unconfirmed workflow)
- The `Contact` model has both ORM-level contacts AND a JSON blob in SchoolSetting `category_contacts` -- two overlapping systems
- Waitlist management uses both the `waitlist` column on Entry and a `waitlist_rank` EntrySetting
- Hybrid entries (cross-school) have their own special handling throughout


---

## 2. Judges and Judge Hiring

### Judges and Judge Hiring

**Description:** Manages the lifecycle of judges at tournaments: registration of judges to categories, assignment to schools, obligation tracking (per-judge or per-round), judge hiring/exchange marketplace, judge pools (JPools), time shifts/availability, questionnaires, paradigms, and the administrative setup of judge categories. This domain covers both the setup side (tournament director configuring judge categories) and the registration side (coaches adding judges, requesting hires).

**Primary users:**
- Tournament directors / tab staff (setting up categories, pools, managing hires)
- Coaches / program directors (registering judges, managing obligations, requesting hires)
- Judges themselves (offering hired rounds, managing availability via `/user/judge/`)
- Site admins (NSDA-level judge management)

**Main entry points:**
- **Setup (tournament director):**
  - `/setup/judges/edit.mhtml` -- category registration settings (codes, deadlines, linking requirements)
  - `/setup/judges/hires.mhtml` -- hire/exchange/signup configuration
  - `/setup/judges/tabbing.mhtml` -- tabbing settings
  - `/setup/judges/ratings.mhtml` -- prefs/ratings system configuration
  - `/setup/judges/tiers.mhtml` -- MPJ tier scale definition
  - `/setup/judges/coach_tiers.mhtml` -- coach/self rating tiers
  - `/setup/judges/shifts.mhtml` -- time shift definition
  - `/setup/judges/pools.mhtml` -- judge pool (JPool) setup
  - `/setup/judges/nsda_pools.mhtml` -- NSDA Nats-specific pool setup
  - `/setup/judges/messages.mhtml` -- judge category messaging
  - `/setup/judges/fake.mhtml` -- fake judge creation (for hidden/test tournaments)
- **Registration (tab-side judge management):**
  - `/register/judge/index.mhtml` -- judge roster for a category
  - `/register/judge/add.mhtml` -- add judge from chapter roster or create new
  - `/register/judge/edit.mhtml` -- edit individual judge details
  - `/register/judge/save.mhtml` -- save judge changes
  - `/register/judge/drop.mhtml` -- remove a judge
  - `/register/judge/rounds.mhtml` -- judge round assignments
  - `/register/judge/hire_requests.mhtml` -- manage hire requests
  - `/register/judge/signups.mhtml` -- public signup management
  - `/register/judge/csv.mhtml` -- CSV import
  - `/register/judge/bulk_alter.mhtml` -- bulk judge operations
  - `/register/judge/qualifications.mhtml` -- judge qualification tracking
  - `/register/judge/questionnaire.mhtml` -- judge questionnaire management
- **School-side (coach view):**
  - `/register/school/judges.mhtml` -- school's judge roster by category
  - `/register/school/judge_hires.mhtml` -- hire request management
  - `/register/school/judge_details.mhtml` -- judge detail editing
  - `/register/school/judge_shifts.mhtml` -- judge availability/shift selection
- **Judge-facing:**
  - `/user/judge/index.mhtml` -- judge dashboard
  - `/user/judge/hire.mhtml` -- offer hired rounds on exchange
  - `/user/judge/hire_save.mhtml`, `hire_edit.mhtml`, `hire_cancel.mhtml` -- manage hire offers
  - `/user/judge/paradigm.mhtml` -- edit paradigm
  - `/user/judge/panels.mhtml` -- view assignments
  - `/user/judge/training.mhtml` -- training/certification tracking
  - `/user/judge/certifications.mhtml` -- certification status
  - `/user/judge/quiz_take.mhtml` -- take required questionnaires

**Major sub-capabilities:**
- **Category management:** Judge categories group judges and events; each category has its own settings for obligations, prefs, hiring, etc.
- **Judge registration:** Adding judges from ChapterJudge roster or creating new ones; linking to Tabroom person accounts
- **Obligation calculation:** Two models -- "judge_per" (1 judge per N entries) and "rounds_per" (N rounds owed per entry). Calculated in `/funclib/judgemath/judges_needed_by_category.mas` with support for:
  - Per-event adjustments (`adjust_judge_rounds_owed`, `adjust_judges_owed`)
  - Free entries deduction (`free` category setting)
  - Min/max burden caps (`min_burden`, `max_burden`)
  - Large school bump (`commitment_bump_after`, `commitment_bump_unit`)
  - Custom rounds-per tables (`custom_rounds_per`)
  - School surcharge adjustments (`judge_surcharge`)
  - Regional adjustments (`regional_judge_adjustments`)
  - Drops-don't-count option (`drops_no_burden`)
- **Judge hiring system:** Multiple hiring models:
  - Entry-based hiring (`uncovered_entry_fee`) -- schools request coverage for N uncovered entries
  - Judge-based hiring (`hired_fee`) -- schools request N hired judges
  - Round-based hiring (`round_hire_fee`) -- schools request N rounds of coverage
  - Judge exchange (`exchange`) -- judges offer rounds; schools hire them directly
  - Public signups (`public_signups`) -- open registration for unaffiliated judges
  - Hire request workflow: request -> accept/reduce/delete with optional email notification
- **Judge pools (JPools):** Grouping judges into pools for round assignment; pools can be linked to rounds, sites, and have parent/child hierarchy
- **Time shifts:** Named time periods with start/end; judges can be struck from shifts to indicate unavailability; shifts can carry fines and `no_hires` flags
- **Judge codes:** Auto-incrementing within category; skips 69, 420, 666; configurable start number
- **Judge linking:** Connecting tournament judges to Tabroom person accounts; requirements for linked accounts, phone numbers, NSDA Campus access
- **Departure times:** Optional collection of judge departure information
- **Questionnaires/quizzes:** Required questionnaires judges must complete
- **Paradigms:** Judge philosophy statements linked via person_setting
- **Judge details deadlines:** Separate deadline for judge info completion
- **Covering/alt categories:** Judges can cover obligations in one category while judging in another (`covers`, `alt_category`)
- **Free strikes / first-year judges:** Judges marked as free strikes don't count toward obligations
- **Tab ratings:** Internal ratings set by tab staff
- **Standby judges:** Judges on standby status
- **Judge activation/deactivation:** Active flag controls inclusion in panels

**Key entities:**
- `Tab::Judge` (`web/lib/Tab/Judge.pm`) -- columns: id, school, first, middle, last, code, active, ada, category, person, chapter_judge, alt_category, covers, obligation, hired, person_request, registered_by, created_at. Has many: ratings, strikes, ballots, settings, hires, jpools
- `Tab::JudgeSetting` (`web/lib/Tab/JudgeSetting.pm`) -- EAV: id, judge, tag, value, value_date, value_text, setting, conditional, timestamp
- `Tab::JudgeHire` (`web/lib/Tab/JudgeHire.pm`) -- columns: id, entries_requested, entries_accepted, rounds_requested, rounds_accepted, requested_at, judge, tourn, category, school, region, requestor, timestamp
- `Tab::Category` (`web/lib/Tab/Category.pm`) -- columns: id, tourn, name, abbr. Has many: judges, jpools, events, hires, rating_tiers, shifts, settings, rating_subsets
- `Tab::CategorySetting` -- EAV settings for categories
- `Tab::JPool` (`web/lib/Tab/JPool.pm`) -- columns: id, name, category, site, parent. Has many: children (self-referencing), settings, pool_judges, judges, rounds
- `Tab::JPoolJudge` (`web/lib/Tab/JPoolJudge.pm`) -- join: id, jpool, judge
- `Tab::JPoolSetting` (`web/lib/Tab/JPoolSetting.pm`) -- EAV for pools
- `Tab::JPoolRound` -- join between pools and rounds
- `Tab::JudgeShift` (`web/lib/Tab/JudgeShift.pm`) -- table name `shift`; columns: id, name, type, fine, start, end, category, no_hires. Availability is tracked via Strike records with type "shift"
- `Tab::ChapterJudge` (`web/lib/Tab/ChapterJudge.pm`) -- persistent judge record across tournaments; columns: id, first, middle, last, ada, retired, phone, email, diet, notes, gender, nsda, chapter, person, person_request

**Key settings/configuration:**
- Category settings (via CategorySetting): `judge_per`, `rounds_per`, `custom_rounds_per`, `free`, `min_burden`, `max_burden`, `commitment_bump_after`, `commitment_bump_unit`, `drops_no_burden`, `no_codes`, `code_start`, `field_report`, `linked_only`, `link_phone_required`, `link_campus_required`, `departure_times`, `details_deadline`, `track_judge_hires`, `exchange`, `public_signups`, `public_signups_open`, `public_signups_deadline`, `hired_jpool`, `hired_rounds`, `hired_fee`, `uncovered_entry_fee`, `round_hire_fee`, `auto_conflict_hires`, `hired_deadline`, `missing_judge_fee_is_hired`, `minimum_supplied`, `min_rounds`, `min_judges`, `minimum_supplied_fine`, `paradigm`, `nats_category`, `nsda_category`, `weekend`, `double_entry`, `regional_judge_adjustments`, `open_switcheroo`, `close_switcheroo`, `signup_message`, `signup_url`, `signup_url_message`, `required_quizzes`, `publish_paradigms`
- Judge settings (via JudgeSetting): `tab_rating`, `neutral`, `diversity`, `chief_adjudicator`, `parli`, `free_strike`, `first_year`, `original_school`, `standby`
- Tourn settings: `judge_deadline`, `nsda_nats`, `ncfl`, `hide_codes`, `no_school_judges`, `mock_trial_registration`

**Key reports/exports:**
- `/register/reports/judges_csv.mhtml` -- judge CSV export
- `/register/reports/judge_cards.mhtml` -- printable judge cards
- `/register/reports/paradigms.mhtml` -- paradigm report
- `/register/reports/nats_judge_required.mhtml` -- NSDA required judge report
- `/register/reports/nsda_elim_judge_bios.mhtml` -- elim judge bios
- `/register/reports/nsda_final_judges.mhtml` -- final round judge report
- `/register/judge/print.mhtml` -- judge print view
- `/register/judge/print_contacts.mhtml` -- judge contact list
- `/register/judge/print_sheet.mhtml` -- judge sheet
- `/register/judge/hired_judge_report.mhtml` -- hired judge report
- `/register/judge/pref_report.mhtml` -- preference averages report with CSV output
- `/register/judge/csv.mhtml` -- judge CSV import/export
- `/register/judge/roster_phonelist.mhtml` -- phone list
- `/register/judge/seasonal_round_counts.mhtml` -- seasonal round tracking
- `/register/judge/decision_times.mhtml` -- decision timing report

**Integrations/dependencies:**
- Chapter system (ChapterJudge persistence)
- Person/account system (linking judges to Tabroom accounts)
- NSDA membership system (nsda field on ChapterJudge)
- Email notification system (hire acceptance notifications)
- Prefs/ratings system (Domain 3)
- Pairing/paneling system (judge assignment to panels)
- NSDA Campus integration (link_campus_required)
- Quiz/questionnaire system

**Documentation sources:**
- Inline HTML explanations in templates (especially `/user/judge/hire.mhtml` explaining the exchange system)
- Code comment in Judge.pm: "Wow, that's a lot." regarding TEMP columns
- Category.pm next_code() skips codes 69, 420, 666

**Code sources:**
- `web/setup/judges/` -- 43 files for category setup
- `web/register/judge/` -- 80+ files for tab-side judge management
- `web/register/school/judges.mhtml` and related files -- coach-side judge registration
- `web/user/judge/` -- 60+ files for judge-facing functionality
- `web/lib/Tab/Judge.pm`, `JudgeSetting.pm`, `JudgeHire.pm`, `JPool.pm`, `JPoolJudge.pm`, `JPoolSetting.pm`, `JPoolRound.pm`, `JudgeShift.pm`, `Category.pm`, `CategorySetting.pm`, `ChapterJudge.pm`
- `web/funclib/judgemath/` -- 8 files for obligation calculations
- `web/funclib/category_judges.mas` -- complex judge listing with prefs/ratings joins
- `web/funclib/chapter_judges_free.mas` -- finding available chapter judges
- `web/funclib/clean_judges.mas`, `clean_to_judge.mas` -- judge cleanup utilities
- `web/funclib/exchange_judges.mas` -- exchange marketplace
- `web/funclib/judge_*.mas` -- 25+ judge utility functions

**Complexity level:** High -- The obligation calculation alone has 10+ adjustment factors. The hiring system supports 4+ distinct models that can be combined. Judge pools add another layer of complexity with hierarchical pools, round associations, and site bindings. The Category entity is a central configuration hub with 40+ settings controlling judge behavior.

**Feature frequency / criticality:** Every tournament -- Every tournament with judges needs category setup and judge registration. Hiring is used by most larger tournaments. JPools are common for multi-day or multi-site tournaments. The exchange system is used at some tournaments. NSDA Nats has its own entirely separate pool management system.

**Notes and open questions:**
- The distinction between `Judge` (tournament-scoped) and `ChapterJudge` (persistent) mirrors the `School`/`Chapter` split for entries
- The `covers` and `alt_category` fields on Judge allow complex cross-category judging arrangements that add significant edge cases
- JudgeShift availability is tracked as Strike records (type="shift") rather than a dedicated availability table -- this overloads the Strike entity
- The `obligation` and `hired` fields on Judge are separate from the JudgeHire records -- potential for inconsistency
- `category_judges.mas` contains three entirely different SQL queries depending on the prefs type (tiered vs ordinals vs none), each with complex joins
- The "fake judges" feature (`/setup/judges/fake.mhtml`) for test tournaments suggests the system is also used for practice/training scenarios


---

## 3. Judge Preferences, Conflicts, and Strikes

### Judge Preferences, Conflicts, and Strikes

**Description:** Manages the complex system by which entries (competitors) express preferences for or against judges, and by which conflicts of interest are tracked and enforced. This encompasses mutual preference judging (MPJ) with tiered or ordinal ratings, entry-level and school-level strikes, automated conflict detection from person-level conflicts, tab ratings, coach ratings, and the constraint system that prevents conflicted judges from judging specific entries. The Strike entity serves as a multi-purpose constraint record covering true strikes, conflicts, time unavailability, event restrictions, and more.

**Primary users:**
- Coaches / program directors (entering prefs/strikes for their entries)
- Competitors/entries (entering their own preferences in some configurations)
- Judges (managing personal conflicts via `/user/judge/conflicts.mhtml`)
- Tournament directors / tab staff (configuring pref systems, viewing pref reports, managing tab ratings)
- The automated pairing system (consuming strikes/prefs to assign judges)

**Main entry points:**
- **Setup (tournament director):**
  - `/setup/judges/ratings.mhtml` -- configure pref type, deadlines, conflict settings
  - `/setup/judges/ratings_save.mhtml` -- saves: prefs type, deadlines, strikes, conflicts, coach ratings, etc.
  - `/setup/judges/tiers.mhtml` -- define MPJ tier scale (tier names, min/max percentages, strike tier)
  - `/setup/judges/coach_tiers.mhtml` -- define coach/self rating tiers
- **Tab-side management:**
  - `/register/judge/strikes.mhtml` -- view all strikes for a category
  - `/register/judge/conflicts.mhtml` -- view all conflicts/strikes with detail (entered_by, type, school/entry targets)
  - `/register/judge/prefs.mhtml` -- view/edit prefs for a specific judge
  - `/register/judge/pref_report.mhtml` -- pref averages report across all judges
  - `/register/judge/tab_ratings.mhtml` -- manage tab-assigned ratings
  - `/register/judge/judge_strikes.mhtml` -- per-judge strike view
  - `/register/judge/strike_save.mhtml` -- add a strike
  - `/register/judge/strike_rm.mhtml` -- remove a strike
  - `/register/judge/strike_switch.mhtml` -- toggle strike properties
  - `/register/judge/pools_and_conflicts.mhtml` -- combined pool/conflict view
- **Entry-side (for coaches entering prefs):**
  - `/register/entry/prefs.mhtml` -- enter prefs for a specific entry (tiered, ordinal, or side-based)
  - `/register/entry/prefs_save.mhtml` -- save preference ratings
  - `/register/entry/prefs_export.mhtml` -- export prefs
  - `/register/entry/prefs_fix.mhtml` -- fix/repair prefs
  - `/register/entry/strikes.mhtml` -- view/manage strikes for an entry
  - `/register/entry/strike_switch.mhtml` -- toggle entry strike
- **Judge-facing:**
  - `/user/judge/conflicts.mhtml` -- judge's personal conflict management (add person or chapter conflicts)
  - `/user/judge/conflict_add.mhtml` -- add a conflict
  - `/user/judge/conflict_rm.mhtml` -- remove a conflict
  - `/user/judge/judge_conflicts.mhtml` -- view tournament-specific conflicts
  - `/user/judge/conflict_tourn_rm.mhtml` -- remove tournament conflict

**Major sub-capabilities:**
- **Preference types** (controlled by category setting `prefs`):
  - `none` -- no preferences
  - `tiered` -- MPJ with named tiers (e.g., 1-6 scale); entries rate judges into tiers; mutual pref = average of both sides
  - `tiered_round` -- tiered with round-based considerations
  - `caps` -- tiered with capacity caps per tier (e.g., max 10% in tier 1)
  - `ordinals` -- ordinal ranking (1st, 2nd, 3rd...); tracked via `percentile` and `ordinal` fields on Rating
  - Side-based prefs (`side_based_prefs` setting) -- separate ratings for aff/neg sides
  - Cumulative MPJ (`cumulate_mjp`) -- using fewer high-tier slots allows more mid-tier
- **Rating tiers:**
  - Defined per category via RatingTier; each tier has name, min, max (percentage caps), description
  - Special tier properties: `strike` (tier acts as a strike), `conflict` (tier acts as a conflict), `start` (starting tier for default)
  - Rating subsets allow different pref scales for different events within a category
- **Strike types** (multiplexed on the Strike entity `type` field):
  - `entry` -- strike on a specific entry (entry cannot have this judge)
  - `school` -- strike on an entire school (judge cannot judge any entry from school)
  - `event` -- judge cannot judge in a specific event
  - `elim` -- judge only available for elims (struck from prelims) in an event
  - `conflict` -- automated conflict (from person-level Conflict records)
  - `time` / `departure` / `timeslot` -- time-based unavailability
  - `region` -- judge cannot judge entries from a region
  - `district` -- judge cannot judge entries from a district
  - Additional flags on Strike: `registrant` (entered by registrant vs tab), `conflict` (is this a conflict rather than a preference strike), `conflictee` (auto-generated from conflict system)
- **Conflict system** (two layers):
  - `Tab::Conflict` -- person-level permanent conflicts (person A conflicts with person B, or person with a chapter); stored globally, not tournament-specific
  - Tournament-level conflicts -- auto-generated as Strike records when a conflicted person registers as a judge; implemented in `/funclib/person_conflict.mas` which creates Strike records for both school-level and entry-level conflicts
  - Conflict auto-propagation: when a judge with conflicts registers, the system creates Strike records for all schools whose chapter matches a Conflict record, and for all entries containing students who are conflicted persons
- **Coach ratings:** Coaches rate judges in their category (`coach_ratings` setting); stored as Rating records with `type = 'coach'`; separate tier scale via coach_tiers
- **Tab ratings:** Tab staff assign internal ratings (`tab_rating` judge setting); used for judge quality tracking
- **Free strikes:** Judges marked as `free_strike` can be struck without counting against obligations; first-year judges (`first_year` setting) can also be auto-free-struck (`fyo_free_strikes`)
- **Pref deadlines:** Separate deadlines for prelim prefs (`strike_start`/`strike_end`) and elim prefs (`elim_strike_start`/`elim_strike_end`); plus general judge deadline
- **Pref JPool:** Optionally limit pref entry to judges in a specific pool (`pref_jpool`)
- **Default rating settings:** For WUDC-style tournaments, predefined rating distributions based on field size (`/funclib/default_rating_settings.mas`)
- **Diversity tracking:** `diversity` and `neutral` judge settings for diversity-conscious judge placement
- **WSDC/WUDC special modes:** Cap/repel settings, chief adjudicator flags, parli flags

**Key entities:**
- `Tab::Strike` (`web/lib/Tab/Strike.pm`) -- THE central constraint record. Columns: id, type, start, end, registrant, conflict, conflictee, tourn, judge, event, entry, school, district, region, timeslot, shift, entered_by, created_at, timestamp. This single table handles strikes, conflicts, time unavailability, event restrictions, regional blocks, and more.
- `Tab::Conflict` (`web/lib/Tab/Conflict.pm`) -- person-level permanent conflicts. Columns: id, person, conflicted, chapter, added_by, timestamp. Bidirectional: queries check both `person` and `conflicted` fields.
- `Tab::Rating` (`web/lib/Tab/Rating.pm`) -- preference rating records. Columns: id, entry, type, rating_tier, judge, rating_subset, ordinal, percentile, side, entered, timestamp. Types include 'entry' (competitor prefs), 'coach' (coach ratings), 'elim' (elimination prefs).
- `Tab::RatingSubset` (`web/lib/Tab/RatingSubset.pm`) -- grouping of events within a category that share a pref scale. Columns: id, name, category. Has many: events, ratings, rating_tiers.
- `Tab::RatingTier` (`web/lib/Tab/RatingTier.pm`) -- individual tier in a pref scale. Columns: id, name, rating_subset, category, description, strike, type, max, min, conflict, start. The `strike` flag makes the tier act as an auto-strike; `conflict` makes it an auto-conflict.

**Key settings/configuration:**
- Category settings controlling prefs: `prefs` (none/tiered/caps/ordinals/tiered_round), `side_based_prefs`, `cumulate_mjp`, `pref_jpool`, `obligation_before_strikes`, `entry_strikes`, `school_strikes`, `elim_only_ratings`, `coach_ratings`, `tab_ratings`, `self_ratings`, `conflicts`, `conflict_denominator`, `fyo_free_strikes`, `free_strikes_dont_count`, `free_strikes_no_pref`, `diversity_selfie`, `ask_paradigm`
- Category deadline settings: `deadline`, `strike_start`, `strike_end`, `elim_strike_start`, `elim_strike_end`
- Judge settings: `tab_rating`, `free_strike`, `first_year`, `neutral`, `diversity`, `chief_adjudicator`, `parli`, `standby`
- RatingTier properties: `name`, `min`, `max` (percentage caps), `strike` (boolean), `conflict` (boolean), `start` (default tier), `description`

**Key reports/exports:**
- `/register/judge/pref_report.mhtml` -- pref average/stddev report for all judges; supports CSV output
- `/register/judge/prefs.mhtml` -- per-judge pref view showing all entries and their ratings
- `/register/judge/conflicts.mhtml` -- comprehensive conflict view with entered_by tracking
- `/register/judge/strikes.mhtml` -- all strikes in a category organized by judge
- `/register/reports/prefs.mhtml` -- registration-side pref report
- `/register/reports/strikes.mhtml` -- registration-side strike report
- `/register/entry/prefs_export.mhtml` -- export entry prefs

**Integrations/dependencies:**
- Pairing/paneling engine -- consumes strikes and ratings to assign judges; this is the primary consumer of all pref/conflict data
- Person/Conflict system -- permanent conflicts propagate into tournament strikes automatically
- Judge registration -- free strikes and first-year flags affect obligation calculations (Domain 2)
- Ballot system -- judges assigned based on prefs/constraints
- NSDA/WUDC/WSDC -- specialized pref/rating modes for international debate formats

**Documentation sources:**
- `/setup/judges/ratings_explained.mhtml` -- explanation of the ratings system (appears to be a help page)
- Inline HTML in tiers.mhtml: "If you want to use cumulative prefs (Fewer 3s permit more 2s, etc) you must use numbers for MPJ tier names"
- `/funclib/default_rating_settings.mas` -- hardcoded WUDC rating distribution tables
- Code comments in `person_conflict.mas` explaining the bidirectional conflict propagation logic

**Code sources:**
- `web/lib/Tab/Strike.pm` -- the core constraint entity
- `web/lib/Tab/Conflict.pm` -- person-level conflicts
- `web/lib/Tab/Rating.pm`, `RatingSubset.pm`, `RatingTier.pm` -- preference data model
- `web/setup/judges/ratings.mhtml`, `ratings_save.mhtml` -- pref system configuration
- `web/setup/judges/tiers.mhtml`, `tier_mpj_save.mhtml`, `tier_rm.mhtml` -- tier management
- `web/setup/judges/coach_tiers.mhtml`, `coach_tier_save.mhtml`, `coach_tier_rm.mhtml` -- coach rating tiers
- `web/register/judge/strikes.mhtml`, `conflicts.mhtml`, `prefs.mhtml`, `pref_report.mhtml`, `strike_save.mhtml`, `strike_rm.mhtml`, `strike_switch.mhtml`, `tab_ratings.mhtml`, `tab_ratings_save.mhtml`, `rating_save.mhtml` -- tab-side strike/pref management
- `web/register/entry/prefs.mhtml`, `prefs_save.mhtml`, `prefs_export.mhtml`, `prefs_fix.mhtml`, `strikes.mhtml`, `strike_switch.mhtml` -- entry-side pref/strike management
- `web/user/judge/conflicts.mhtml`, `conflict_add.mhtml`, `conflict_rm.mhtml`, `judge_conflicts.mhtml` -- judge-facing conflict management
- `web/funclib/category_strikes.mas` -- loads all strikes for a category with round-level resolution
- `web/funclib/category_ratings.mas` -- loads all ratings for a category/event with subset handling
- `web/funclib/entry_conflicts.mas` -- entry-level conflict queries
- `web/funclib/school_conflicts.mas` -- school-level conflict queries
- `web/funclib/person_conflict.mas` -- auto-propagation of person conflicts to tournament strikes
- `web/funclib/free_strikes.mas` -- free strike identification
- `web/funclib/default_rating_settings.mas` -- WUDC default distributions
- `web/funclib/judge_rating.mas` -- judge rating display
- `web/funclib/judge_averages.mas` -- average pref calculations
- `web/funclib/judges_by_pref.mas` -- sorting judges by preference data
- `web/funclib/event_judgeprefs.mas` -- event-level judge prefs
- `web/funclib/trpc_category_strikes.mas` -- TRPC-format strikes
- `web/funclib/event_selfstrike.mas` -- self-strike for events
- `web/funclib/strike_name.mas` -- display name for a strike
- `web/funclib/strike_judges.mas` -- judges affected by strikes

**Complexity level:** High -- This is one of the most complex domains in the system. The Strike entity is heavily overloaded, serving as the universal constraint record with 7+ type values and multiple boolean flags (registrant, conflict, conflictee) that modify semantics. The preference system supports 5+ modes (none, tiered, caps, ordinals, tiered_round) each with different data paths. The two-layer conflict system (permanent Conflict records auto-propagating to tournament Strike records) involves complex bidirectional queries. Rating subsets add another dimension, allowing different events in the same category to have different pref scales.

**Feature frequency / criticality:** Most tournaments -- Most competitive debate/speech tournaments use some form of judge preferences. Strikes are nearly universal. The conflict system runs automatically whenever judges register. Ordinal prefs and WUDC modes are used at a smaller subset of tournaments. Coach ratings and tab ratings are optional features used by some tournaments.

**Notes and open questions:**
- The Strike table is the most overloaded entity in the system, combining what should arguably be 4-5 separate tables (strikes, conflicts, time availability, event restrictions, regional blocks) into one with a `type` discriminator
- The `registrant` flag on Strike distinguishes between "entered by the registrant/coach" vs "entered by tab staff" -- this affects visibility and editability
- The `conflict` and `conflictee` flags on Strike create a distinction between user-initiated conflicts and system-auto-generated conflicts that is tracked but could easily become inconsistent
- `person_conflict.mas` runs complex bidirectional queries (checking both `conflict.person` and `conflict.conflicted`) against entries, students, and schools -- this is a performance-sensitive operation that runs at registration time
- The interaction between the `strike` flag on RatingTier (a tier that acts as a strike) and actual Strike records is not immediately obvious -- rating a judge in a "strike tier" should create an implicit constraint, but whether that creates a Strike record or is handled at pairing time needs investigation
- The `side` field on Rating enables side-based prefs (separate aff/neg ratings) but this adds complexity to the pref display and calculation
- There is a `conflict_denominator` category setting whose exact purpose is not immediately clear from the code -- it may control how conflicts are counted in pref averages
- The `cumulate_mjp` setting enables a point-budget system where using fewer high-tier slots allows more mid-tier -- this is a sophisticated MPJ variant that requires careful UI and validation
- The `pref_jpool` setting allows limiting which judges are visible for pref entry to those in a specific pool -- this is important for large tournaments with specialized pools
- `obligation_before_strikes` is a setting that appears to control whether judge obligations must be met before strikes can be entered -- this creates a registration-flow dependency between Domains 2 and 3
