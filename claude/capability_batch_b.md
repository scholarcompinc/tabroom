# Capability Domains: Batch B

## 1. Events, Divisions, and Categories

### Events, Divisions, and Categories

**Description:** Manages the competitive events (divisions) within a tournament and their grouping into judge categories. Events define what competitors compete in (e.g., Lincoln-Douglas Debate, Original Oratory). Categories group events that share a common judge pool. This domain handles event creation, configuration of tabulation rules, ballot formats, registration constraints, entry code styles, online competition modes, double-entry/pattern restrictions, and sectioning/pairing parameters.

**Primary users:**
- Tournament directors and tab staff (setup and configuration)
- Site administrators (NSDA-specific overrides)
- Coaches/registrants (view event descriptions and rules during registration)

**Main entry points:**
- `/setup/events/edit.mhtml` - Main event create/edit page
- `/setup/events/tabbing.mhtml` - Tabulation settings per event
- `/setup/events/ballots.mhtml` - Ballot & rules configuration
- `/setup/events/register.mhtml` - Registration settings per event
- `/setup/events/online.mhtml` - Online competition mode settings
- `/setup/events/sectioning.mhtml` - Sectioning/pairing/chamber settings
- `/setup/events/legislation.mhtml` - Congress legislation management
- `/setup/events/double_entry.mhtml` - Double-entry patterns and limits
- `/setup/events/mass_recode.mhtml` - Batch speaker code recoding
- `/setup/events/follow.mhtml` - Update/follower notification settings
- `/setup/events/districts_register.mhtml` - District-specific registration settings
- `/setup/events/nats_register.mhtml` - NSDA Nationals registration settings
- `/setup/events/nsda_events.mhtml` - Automated NSDA event creation for districts

**Major sub-capabilities:**
- **Event CRUD**: Create, edit, delete, merge events within a tournament. Events must have unique names and abbreviations within a tournament. Validation prohibits slashes, percent signs, question marks, and ampersands in names/abbreviations. (`edit.mhtml`, `edit_save.mhtml`, `event_rm.mhtml`, `merge_event.mhtml`)
- **Event types**: speech, debate, congress, wsdc (World Schools), wudc (British Parliamentary), mock_trial, attendee (non-competing). Type determines available settings and ballot formats. (`edit.mhtml` lines 516-553)
- **Event levels**: open/varsity, JV, novice, championship, middle school, Spanish open/novice. (`edit.mhtml` lines 560-631)
- **Entry code styles**: 15+ code formats including numeric, school+number, school+initials, school+names, full names, last names, registrant-supplied codes. (`edit.mhtml` lines 344-465)
- **Judge category assignment**: Each event belongs to one category (judge pool group). Categories contain judges, judge pools, rating tiers, rating subsets, and shifts. (`Category.pm`)
- **NSDA category mapping**: Events map to `nsda_category` IDs for NSDA points posting. Separate table `nsda_category` with id, type, code, name, national columns. (`NSDACategory.pm`, `edit.mhtml` lines 649-700)
- **Qualifier rules binding**: Events can be bound to circuit-level qualifier rule sets via `qualifier_<circuit_id>` and `qualifier_event_<circuit_id>` settings. (`edit.mhtml` lines 693-766)
- **Settings cloning**: New events can clone all settings from an existing event in the same tournament. (`edit_save.mhtml` lines 240-260)
- **Double entry patterns**: Tournament-level Pattern objects group events into scheduling blocks with maximum entry limits and exclusion rules. (`Pattern.pm`: columns name, tourn, type, max, exclude; `double_entry.mhtml`, `pattern_add.mhtml`, `pattern_save.mhtml`)
- **Registration settings**: Per-entry fees, min/max competitors per entry, entry caps, waitlists, drops deadline, hybrid entries. (`register.mhtml`, `register_save.mhtml`)
- **Tabulation settings**: Point scales, point increments, ranks, win/loss, ballots per judge, flighting, speaker award rules, tiebreakers, side constraints, qualification overrides. (`tabbing.mhtml`, `tabbing_save.mhtml`)
- **Ballot & rules**: Bias statements, custom ballot text, rubric ballots, student ballots, ballot templates, custom score labels (aff_label, neg_label). (`ballots.mhtml`, `ballots_save.mhtml`, `rubric_ballot.mhtml`)
- **Online competition modes**: None, sync (external links), async, public Jitsi, NSDA Campus, NSDA Campus with observers. Hybrid online/in-person support. (`online.mhtml`, `online_save.mhtml`)
- **Sectioning/pairing configuration**: Panel size constraints (min/max), school conflict avoidance, geographic constraints for districts. Congress has chamber-specific settings. (`sectioning.mhtml`, `sectioning_save.mhtml`)
- **Congress legislation**: Bill/resolution file uploads (S3), bill category management, draw topics. (`legislation.mhtml`, `legislation_save.mhtml`, `bill_categories.mhtml`, `draw_topics.mhtml`)
- **Event followers/notifications**: Add followers and backup followers for blast updates. (`follow.mhtml`, `follower_add.mhtml`, `backup_add.mhtml`)
- **Fake entries**: For hidden/test tournaments, create fake entries to test pairings. (`fake.mhtml`, `fake_save.mhtml`)
- **Move event**: Site admin can move an event to a different tournament entirely. (`move_tourn.mhtml`)
- **Placard logos**: Upload event-specific logos for printed placards. (`placard_logo_save.mhtml`, `delete_logo.mhtml`)
- **Student ballot / student ballot blast**: Student-facing ballot views and blast notifications. (`student_ballot.mhtml`, `student_ballot_blast.mhtml`)

**Key entities:**
- `Tab::Event` (`web/lib/Tab/Event.pm`) - columns: id, tourn, name, abbr, category, type, level, fee, rating_subset, pattern, timestamp, code_style, nsda_category. Relationships: belongs to Tourn, Category, Pattern, RatingSubset; has many EventSettings, ResultSets, Entries, Rounds, Files.
- `Tab::EventSetting` (`web/lib/Tab/EventSetting.pm`) - columns: id, event, tag, value, value_date, value_text, setting, timestamp. EAV pattern for extensible event configuration.
- `Tab::Category` (`web/lib/Tab/Category.pm`) - columns: id, tourn, name, abbr, timestamp. Has many: judges, jpools, events, hires, rating_tiers, shifts, settings, rating_subsets.
- `Tab::CategorySetting` (`web/lib/Tab/CategorySetting.pm`) - columns: id, category, tag, value, value_date, value_text, setting, timestamp.
- `Tab::Pattern` (`web/lib/Tab/Pattern.pm`) - columns: id, name, tourn, type, max, exclude. Has many events.
- `Tab::NSDACategory` (`web/lib/Tab/NSDACategory.pm`) - columns: id, type, code, name, national, timestamp. Reference table for NSDA event codes.

**Key settings/configuration:**
- EventSetting tags (non-exhaustive): `code_start`, `min_entry`, `max_entry`, `description`, `result_description`, `bowl_description`, `usa_wsdc`, `not_nats`, `topic`, `split_team`, `presplit`, `big_questions`, `aff_label`, `neg_label`, `allow_rank_ties`, `parli_ballot`, `max_points`, `min_points`, `truncate_fill`, `online_mode`, `online_ballots`, `online_hybrid`, `min_panel_size`, `max_panel_size`, `point_increments`, `no_side_constraints`, `ask_for_titles`, `ask_for_authors`, `ask_for_isbn`, `ask_for_bibliography`, `ask_for_topic`, `followers`, `backup_followers`, `bill_categories`, `nsda_qual_override`, `nsda_qual_force`, `supp`, `weekend`, `flip_autopublish`, `flip_before_start`, `allow_repeat_prelim_side`, `code_start`
- CategorySetting tags used for judge management, hiring, and constraint rules
- TournSetting tags: `nsda_district`, `nsda_nats`, `double_entry`, `double_max`, `bias_statement`, `supp_teams`, `supp_online_hybrid`, `mock_trial_registration`

**Key reports/exports:**
- None directly from event setup; events feed into tabulation, results, and sweep tracking elsewhere

**Integrations/dependencies:**
- NSDA points posting via `nsda_category` mapping
- Circuit qualifier rules (stored as JSON in `circuit_setting` tag `qualifiers`)
- NSDA Campus / Jitsi for online rounds
- S3 for legislation file storage
- Debate topics from `funclib/topics.mas`

**Documentation sources:**
- In-page help text and tooltips throughout setup pages
- Code comments in `edit_save.mhtml` and `setting_switch.mhtml`

**Code sources:**
- Primary: `web/setup/events/` (19+ .mhtml files, 3 .mas components)
- ORM models: `web/lib/Tab/Event.pm`, `EventSetting.pm`, `Category.pm`, `CategorySetting.pm`, `Pattern.pm`, `NSDACategory.pm`
- Funclib: `web/funclib/nsda/events.mas` (NSDA district event definitions), `web/funclib/nsda/event_codes.mas`
- Registration side: `web/register/` uses event and category data extensively

**Complexity level:** High
- 15+ code style options, 7 event types, multiple online modes, extensive per-event settings (40+ known tags), district-specific overrides, NSDA national tournament special handling, qualifier rule binding, pattern/double-entry constraint system.

**Feature frequency / criticality:** Every tournament
- Every tournament must create at least one event. Category and event configuration is the foundation for all tabulation, pairing, and results.

**Notes and open questions:**
- The code skips judge codes 69, 420, and 666 in `Category.pm` `next_code()` -- intentional humor/cultural sensitivity.
- District tournaments have read-only event settings (controlled by NSDA), with override for site_admin and nsda_admin.
- Hard-coded circuit IDs appear (e.g., 43 for NDT/CEDA, 228 for TOC, 103 for ADA) -- these should be configurable in rebuild.
- The Pattern system for double-entry is tournament-level, not event-level, but events reference patterns.
- Event "level" includes Spanish-language variants (es-open, es-novice) -- internationalization consideration.
- The `supp` event setting and `supp_teams` tournament setting control supplemental event handling at NSDA Nationals -- complex, tightly coupled logic.


---

## 2. School/Chapter/Circuit/Region Administration

### School/Chapter/Circuit/Region Administration

**Description:** Manages the organizational hierarchy outside of individual tournaments: Chapters (schools/teams), Circuits (leagues/associations), Regions (subdivisions of circuits), and their memberships, permissions, and configurations. Chapters maintain student and judge rosters. Circuits organize tournaments, track membership, manage sweepstakes awards, and define qualifier rules. Regions subdivide circuits for geographic organization. This domain is the persistent layer that spans across tournaments.

**Primary users:**
- Coaches/advisors (chapter management, roster, tournament registration)
- Circuit administrators (membership, tournament approvals, qualifier tracking)
- Region administrators (school oversight within a region)
- Site administrators (cross-cutting admin operations)

**Main entry points:**
- **Chapter:** `/user/chapter/index.mhtml` (redirects to tournaments), `/user/chapter/students.mhtml` (roster), `/user/chapter/judges.mhtml` (judge roster), `/user/chapter/settings.mhtml` (school settings), `/user/chapter/circuits.mhtml` (circuit membership), `/user/chapter/tournaments.mhtml` (upcoming/past tournaments), `/user/chapter/nsda.mhtml` (NSDA link/sync), `/user/chapter/history.mhtml` (competition history)
- **Circuit:** `/user/circuit/index.mhtml` (circuit settings), `/user/circuit/chapters.mhtml` (member schools), `/user/circuit/tourns.mhtml` (approved tournaments), `/user/circuit/regions.mhtml` (region management), `/user/circuit/qualifiers.mhtml` (qualifier bid rules), `/user/circuit/awards.mhtml` (sweepstakes/awards), `/user/circuit/emails.mhtml` (circuit emails)
- **Region:** `/user/region/tournaments.mhtml` (region tournament entries), `/user/region/school_entry.mhtml` (qualifier checking)

**Major sub-capabilities:**

*Chapter Management:*
- **Chapter CRUD and settings**: Name, formal name, address (street, city, state, zip, postal, country), level (elementary, middle, high school, university), NSDA membership ID, district assignment. (`Chapter.pm`, `settings.mhtml`)
- **Student roster**: Add, edit, retire, unretire students. Track first/middle/last name, graduation year, NSDA membership, novice status, districts eligibility. Person linking (student to Tabroom account). CSV import. (`students.mhtml`, `student_edit.mhtml`, `student_save.mhtml`, `student_retire.mhtml`, `import_csv_students.mhtml`)
- **Judge roster**: Add, edit, retire judges. Track name, phone, email, ADA needs, diet, gender, NSDA ID, paradigm notes. Person linking. CSV import. Judge history. (`judges.mhtml`, `judge_edit.mhtml`, `judge_save.mhtml`, `import_csv_judges.mhtml`, `judge_history.mhtml`)
- **Chapter access/permissions**: Add/remove coach accounts with permission to manage chapter. (`access.mhtml`, `access_save.mhtml`, `access_rm.mhtml`)
- **Circuit membership**: Join/leave circuits. View circuit memberships. (`circuits.mhtml`, `circuit_join.mhtml`, `circuit_leave.mhtml`)
- **Deduplication**: Merge duplicate students and judges. (`dedupe.mhtml`, `dedupe_save.mhtml`, `dedupe_judges.mhtml`, `dedupe_judges_save.mhtml`)
- **NSDA integration**: Link chapter to NSDA account, sync roster from NSDA API, view NSDA points. (`nsda.mhtml`, `nsda_chapter_sync.mhtml`)
- **Dietary tracking**: Track student/judge dietary restrictions. (`diets.mhtml`, `diet_save.mhtml`)
- **Competition history**: View past tournament participation and results. (`history.mhtml`, `record.mhtml`)
- **Judge preferences**: Set preferences for upcoming tournaments. (`prefs.mhtml`)
- **Tournament registration**: Browse and register for upcoming tournaments. (`tournaments.mhtml`, `tourn_register.mhtml`)
- **Roster paradigms**: View paradigms for all judges on roster. (`roster_paradigms.mhtml`)
- **CSV import/export**: Import students and judges from CSV spreadsheets; export student and judge rosters. (`import_csv.mhtml`, `import_csv_students.mhtml`, `import_csv_judges.mhtml`, `import_csv_template.mhtml`, `student_csv.mhtml`, `judges_csv.mhtml`, `student_download.mhtml`)
- **Practice tracking**: Track practice attendance for NAUDL compliance. (`practice.mhtml`, `practice_add.mhtml`, `practice_attend.mhtml`)
- **Follower management**: Follow/unfollow students for results notifications. (`follow.mhtml`, `unfollow.mhtml`, `follower_switch.mhtml`)

*Circuit Management:*
- **Circuit settings**: Name, abbreviation, state, country, timezone, webname, website URL. Toggle settings: demographics, tourns_no_add, chapters_no_add, autoapprove, regions, ncfl (dioceses), naudl (reporting), naudl_member, naudl_league_code. (`index.mhtml`, `circuit_save.mhtml`)
- **Circuit administrators**: Add/remove admin accounts by email. (`admin_add.mhtml`, `admin_rm.mhtml`)
- **Member school management**: List, search, edit member schools. Set school codes, full membership status, region assignment. Add/remove schools from circuit. (`chapters.mhtml`, `chapter_edit.mhtml`, `chapter_save.mhtml`, `chapter_circuit_add.mhtml`, `chapter_circuit_rm.mhtml`, `codes.mhtml`)
- **School contacts**: View and manage contact information for all member schools. (`contacts.mhtml`)
- **Tournament management**: Approve/deny tournament membership in circuit. View tournament list by year. View tournament admins. (`tourns.mhtml`, `approvals.mhtml`, `approve.mhtml`, `deny.mhtml`, `tourn_admins.mhtml`)
- **Region management**: Create, edit, delete regions within a circuit. Assign schools to regions. Set region codes, areas. Add/remove region administrators. (`regions.mhtml`, `regions_save.mhtml`, `region_add.mhtml`, `region_admin.mhtml`, `region_admin_add.mhtml`, `region_admin_rm.mhtml`, `region_import.mhtml`)
- **Circuit emails**: Compose and send mass emails to circuit members. View sent email history by year. (`emails.mhtml`, `email_compose.mhtml`, `email_send.mhtml`, `email_view.mhtml`)
- **Results tracking**: View tournament final placements. (`tourn_results.mhtml`, `result_sheets.mhtml`)
- **Qualifier bid system**: Define qualifier rule sets (JSON stored in circuit_setting `qualifiers`). Each rule set has a label, optional event codes, and entry/school thresholds. Rule sets can specify number of qualifiers and alternates based on entry count. Post and view qualifying bids. (`qualifiers.mhtml`, `qualifier_add.mhtml`, `qualifier_rm.mhtml`, `qualifier_label.mhtml`, `qualifier_event.mhtml`, `qualifier_rules.mhtml`, `qualifier_report.mhtml`)
- **Sweepstakes awards**: Define cumulative awards across tournaments. Sweep awards have name, description, target, count, min_entries, min_schools, period. Each has multiple sweep_sets (rule sets). (`awards.mhtml`, `award_save.mhtml`, `sweep_rule_save.mhtml`, `sweep_rule_rm.mhtml`, `sweep_set_save.mhtml`)
- **TOC-specific features**: TOC bids display for circuit 228. (`result_sheets.mhtml`, hard-coded circuit ID 228)
- **NDT/CEDA/ADA point management**: Point manager for circuits 43 and 103. (`ndtceda_pt_manager.mhtml`, `ndt_ceda_generator.mhtml`)
- **NCFL diocese management**: Diocese list and quotas, Cooke Award points tracking. (`dioceses.mhtml`, `dioceses_save.mhtml`, `diocese_admin.mhtml`, `cooke_points.mhtml`, `cooke_save.mhtml`)
- **Quiz/certification management**: Manage questionnaires and certifications for circuit members. (`quizzes.mhtml`, `quiz_takers.mhtml`, `quiz_takers_save.mhtml`)
- **NAUDL reporting**: STA attendance, STA pairs, tournament reports, student reports, section reports. Date-range filtered. (`report.mhtml`)
- **BDL export**: All-time student report CSV export for circuit 58. (`bdl_student_export.mhtml`)
- **Chapter-by-tournament view**: See all schools and their participation at a specific tournament. (`chapter_by_tourn.mhtml`)
- **Bid posting**: Post bids for schools at tournaments. (`post_bids.mhtml`)
- **Judge training**: Manage judge training requirements. (`judge_training.mhtml`, `judge_training_save.mhtml`)
- **Permission switches**: Toggle various permissions for schools. (`permissions_switch.mhtml`, `member_switch.mhtml`, `setting_switch.mhtml`)
- **Access management**: Granular access control for circuit features. (`access.mhtml`, `access_add.mhtml`, `access_rm.mhtml`, `access_save.mhtml`)

*Region Management:*
- **Region tournament entries**: View schools and entries within a region for specific tournaments. Check qualifiers for region schools. (`tournaments.mhtml`, `school_entry.mhtml`)
- **Region admin management**: Add/remove region administrators. (`admin_add.mhtml`, `admin_rm.mhtml`)

**Key entities:**
- `Tab::Chapter` (`web/lib/Tab/Chapter.pm`) - columns: id, name, formal, street, city, state, zip, postal, country, level, naudl, nsda, district, timestamp. Has many: schools, students, chapter_judges, chapter_circuits, chapter_settings, permissions. Belongs to District.
- `Tab::ChapterSetting` (`web/lib/Tab/ChapterSetting.pm`) - EAV settings: id, chapter, tag, value, value_date, value_text, setting, timestamp. Known tags include `coaches`.
- `Tab::ChapterCircuit` (`web/lib/Tab/ChapterCircuit.pm`) - Junction table: id, circuit, chapter, code, full_member, circuit_membership, region. Links chapters to circuits with membership metadata.
- `Tab::ChapterJudge` (`web/lib/Tab/ChapterJudge.pm`) - columns: id, first, middle, last, ada, retired, phone, email, diet, notes, notes_timestamp, gender, nsda, chapter, person, person_request, timestamp. Has many judges (tournament-specific judge records).
- `Tab::Circuit` (`web/lib/Tab/Circuit.pm`) - columns: id, name, abbr, tz, active, state, country, webname, timestamp. Has many: sites, regions, webpages, permissions, quizzes, settings, tourn_circuits, chapter_circuits, awards.
- `Tab::CircuitSetting` (`web/lib/Tab/CircuitSetting.pm`) - EAV: id, circuit, tag, value, value_date, value_text, setting, timestamp. Key tags: `qualifiers` (JSON), `url`, `demographics`, `tourns_no_add`, `chapters_no_add`, `autoapprove`, `regions`, `ncfl`, `naudl`, `naudl_member`, `naudl_league_code`, `full_members`, `tourn_only`, `track_bids`, `judge_demographics`.
- `Tab::Region` (`web/lib/Tab/Region.pm`) - columns: id, name, code, circuit, tourn, timestamp. Belongs to Circuit and Tourn. Has many: schools, permissions, admins, chapters (via ChapterCircuit).
- `Tab::RegionSetting` (`web/lib/Tab/RegionSetting.pm`) - EAV: id, region, tag, value, value_date, value_text, event, setting, timestamp. Notable: has `event` foreign key, allowing per-event-per-region settings.
- `Tab::SweepAward` (`web/lib/Tab/SweepAward.pm`) - columns: id, name, description, circuit, target, count, min_entries, min_schools, period, timestamp. Has many sweep_sets and result_sets.

**Key settings/configuration:**
- Chapter-level: `coaches`, various chapter_settings via EAV
- Circuit-level: `qualifiers` (JSON blob defining qualifier rule sets with labels, events, and threshold rules), `url`, `demographics`, `tourns_no_add`, `chapters_no_add`, `autoapprove`, `regions`, `ncfl`, `naudl`, `naudl_member`, `naudl_league_code`
- Region-level: `area`, per-event settings via event FK on RegionSetting
- ChapterCircuit: `code` (school code within circuit), `full_member`, `region`

**Key reports/exports:**
- Qualifier report: Lists all qualifiers for a circuit by event code and year, showing student names, tournament, placement (`qualifier_report.mhtml`)
- NAUDL reports: STA attendance, STA pairs, tournaments, students, sections (date-range filtered)
- BDL all-time student export (CSV)
- Cooke Award cumulative points (NCFL-specific)
- TOC bids display
- School contacts export
- Student/judge roster CSV export
- NDT/CEDA point reports

**Integrations/dependencies:**
- NSDA API: Chapter sync (`nsda_chapter_sync.mhtml`), roster import (`funclib/nsda/chapter_sync.mas`, `school_roster.mas`, `user_import.mas`)
- NAUDL reporting system
- Tournament registration system (chapters register for tournaments as schools)
- Permission system (`Tab::Permission`) for access control across chapters, circuits, regions
- S3 for file storage

**Documentation sources:**
- In-page labels and tooltips
- Code comments (e.g., `Circuit.pm` qualifier system, `Chapter.pm` helper methods)
- NCFL/NAUDL-specific inline documentation

**Code sources:**
- `web/user/chapter/` (70+ files) - Chapter admin interface
- `web/user/circuit/` (75+ files) - Circuit admin interface
- `web/user/region/` (4 files) - Region admin interface
- ORM: `web/lib/Tab/Chapter.pm`, `ChapterSetting.pm`, `ChapterCircuit.pm`, `ChapterJudge.pm`, `Circuit.pm`, `CircuitSetting.pm`, `Region.pm`, `RegionSetting.pm`, `SweepAward.pm`
- Funclib: `web/funclib/chapter_regions.mas`, `web/funclib/person_chapters.mas`

**Complexity level:** High
- Multi-level organizational hierarchy (Chapter -> Circuit -> Region), extensive membership tracking, complex qualifier bid rule system (JSON), integration with NSDA/NAUDL/NCFL external systems, 70+ chapter admin pages, 75+ circuit admin pages.

**Feature frequency / criticality:** Every tournament
- Chapters and circuits are fundamental to the platform. Every school must have a chapter to register for tournaments. Circuits organize the competitive calendar. Qualifier tracking is critical for the TOC and state/national qualification pipelines.

**Notes and open questions:**
- Hard-coded circuit IDs throughout: 228 (TOC), 43 (NDT/CEDA), 103 (ADA), 58 (BDL), 15, 2. These should be configurable or feature-flagged in a rebuild.
- The qualifier system stores rules as JSON in a single circuit_setting row -- complex nested structure that could benefit from normalization.
- `RegionSetting` has an `event` FK, making it unique among setting tables -- allows per-event quotas or configuration at the region level.
- ChapterCircuit serves triple duty: membership tracking, school coding, and region assignment.
- The `full_member` field on ChapterCircuit and related commented-out UI code in circuit settings suggests full membership tracking was partially implemented but may be dormant.
- Practice tracking (`practice.mhtml`) appears to be NAUDL-specific for compliance reporting.
- The NCFL diocese system is a circuit-specific feature that adds a layer of organization (diocese) between circuit and chapter.
- Student `districts_eligible` tag on student_setting tracks whether students can compete at NSDA districts.
- The `person` vs `person_request` pattern on both Student and ChapterJudge allows linking to a Tabroom account that the person has confirmed vs one that has been requested but not yet confirmed.


---

## 3. Districts, Qualifications, and Advancement Pipelines

### Districts, Qualifications, and Advancement Pipelines

**Description:** Manages the NSDA district tournament system, including district organization, district tournament creation and configuration, qualification counts, advancement to NSDA Nationals, district committee governance, sweepstakes awards, and the pipeline from district competition to national participation. Also encompasses the broader circuit-level qualification/bid system used by TOC, state associations, and other circuits to track qualifying performances and manage advancement. This domain is deeply intertwined with NSDA-specific business logic.

**Primary users:**
- District chairs and committee members (district tournament setup, entry approval, nationals registration)
- NSDA administrators (`nsda_admin` person setting)
- Site administrators
- Coaches/advisors (viewing district information, submitting entries)
- Circuit administrators (managing qualifier rules and bids)

**Main entry points:**
- **District admin (NSDA):** `/user/nsda/district.mhtml` - Main district dashboard
- **District tournament creation:** `/user/nsda/district_tournament_create.mhtml` (step 1), `/user/nsda/district_tournament_create_events.mhtml` (step 2: events), `/user/nsda/district_tournament_dates.mhtml` (step 3: dates)
- **District registration view:** `/register/district/index.mhtml` - View district schools, entries, judges, congress legislation
- **Tournament district dates:** `/setup/tourn/district_dates.mhtml` - Configure district weekend dates and event assignments
- **District survey/updates:** `/user/nsda/district_survey.mhtml`, `/user/nsda/district_update.mhtml`
- **Circuit qualifiers:** `/user/circuit/qualifiers.mhtml` - Define qualifier rule sets
- **Qualifier report:** `/user/circuit/qualifier_report.mhtml` - View qualified entries

**Major sub-capabilities:**

*NSDA District System:*
- **District organization**: Districts have name, code, location, level, realm, and belong to a Region. Districts contain chapters (schools). Governed by a committee with chair, member, and alternate roles. (`District.pm`, `funclib/district_committee.mas`)
- **District tournament creation wizard**: Multi-step process: (1) basic info and software choice, (2) select which NSDA events to offer (standardized list: HSE, SEN, CX, LD, PF, BQ, DI, DUO, HI, INF, IX, OO, POI, USX), (3) set dates/weekends. Creates a full tournament with pre-configured events. (`district_tournament_create.mhtml`, `district_tournament_create_events.mhtml`, `district_tournament_dates.mhtml`)
- **Weekend management**: District tournaments span multiple weekends. Events are assigned to specific weekends. The `Tab::Weekend` model tracks start/end dates per weekend. (`district_dates.mhtml`, `district_weekend_events.mhtml`)
- **Qualifier count calculation**: Automatic computation of how many entries qualify from each event based on number of competitors and schools. Uses `POSIX::ceil(entries/2)` with cap of 14 alternates. Supports manual override via `nsda_qual_force` and `nsda_qual_override` settings. COVID-era special handling (2020-2021). (`funclib/nsda/qualifier_count.mas`)
- **District registration view**: Tabbed interface showing General info, Schools, Entries, Judges, and Congress legislation for a district. District notes (freeform text). Software tracking (Tabroom vs SpeechWire). (`register/district/index.mhtml`)
- **Entry status tracking**: Entries have states: PENDING (unconfirmed), ACCEPTED (active), REJECTED (rejected_by setting), DROPPED. District-specific entry management. (`register/district/index.mhtml` lines 366-383)
- **District awards**: Standardized awards including Coach of the Year, Student of the Year, Volunteer of the Year, Alum of the Year, Communicator of the Year, Administrator of the Year, New Coach of the Year, Assistant Coach of the Year. (`funclib/nsda/district_awards.mas`)
- **District sweepstakes**: Aggregate sweepstakes scoring for districts. (`district_sweepstakes_save.mhtml`)
- **District notes**: Free-form notes stored as `nsda_notes` tournament setting. (`register/district/district_notes.mhtml`)
- **District questions/metadata**: Stored as JSON in tournament setting `nsda_district_questions`, including tabbing software choice, extemp topics preference, and award nominations. (`district_tournament_create.mhtml`)
- **NSDA Nationals registration pipeline**: District qualifiers advance to nationals. Entry confirmation/rejection workflow. Judge nominations from districts. Worlds Schools team management. (`district_nats.mas`, `district_judge_noms.mhtml`, `wsdc_team_save.mhtml`, `wsdc_judge_edit.mhtml`)
- **District committee management**: Chair, member, and alternate roles. Chairs have elevated permissions including tournament creation and judge nomination. (`funclib/district_committee.mas`, `show_district.mas`)
- **Congress legislation for districts**: Upload and manage legislation files per district, stored in S3. (`register/district/index.mhtml` Congress tab, `/user/nsda/upload_legislation.mhtml`, `legislation_rm.mhtml`)
- **NSDA chapter import/sync**: Import chapters from NSDA into districts. Sync rosters from NSDA API. (`import_chapter.mhtml`, `import_nsda_roster.mhtml`, `sync_roster.mhtml`)
- **Online ballots for districts**: District-specific online ballot management and printing. (`online_ballots.mhtml`, `online_ballots_print.mhtml`)
- **District done/completion**: Mark district tournament as complete. (`district_done.mhtml`)
- **Student linking**: Link students between NSDA records and Tabroom accounts. (`student_link.mhtml`, `funclib/nsda/student_link.mas`)

*Circuit-Level Qualification System:*
- **Qualifier rule sets**: Circuits define named rule sets (e.g., "State Speech Rules", "TOC Octos Bid") stored as JSON in circuit_setting `qualifiers`. Each rule set contains a label, optional event code mapping, and threshold-based rules. (`qualifiers.mhtml`, `qualifier_rules.mhtml`)
- **Threshold rules**: Rules specify qualifiers and alternates based on entry count and school count. Multiple threshold tiers per rule set (e.g., 10+ entries from 5+ schools = 2 qualifiers). (`qualifier_rules.mhtml`)
- **Event code mapping**: Qualifier rule sets can map to specific NSDA category codes and circuit-specific event codes. Events in tournaments bind to these rule sets via `qualifier_<circuit_id>` settings. (`qualifier_event.mhtml`)
- **Qualifier report**: Displays all qualifying entries for a circuit by event code and school year, with student names, tournament names, and placements. Uses result_sets with label format `<circuit_abbr> Qualification`. (`qualifier_report.mhtml`)
- **Bid posting**: Circuit admins can post bids for specific entries. (`post_bids.mhtml`)
- **TOC bid tracking**: Specialized bid display for TOC circuit (hard-coded circuit 228). (`result_sheets.mhtml`)
- **Result set integration**: Qualifications are tracked through `Tab::ResultSet` with labels and sweep_award linkage. Published or coach-visible results feed the qualifier report.

*NSDA Points and Membership:*
- **Points posting**: Automatic posting of NSDA points based on tournament results and event `nsda_category` mapping. (`funclib/nsda/post_points.mas`)
- **Membership checking**: Verify NSDA membership status for students and schools. (`funclib/nsda/membership.mas`, `funclib/nsda/status_check.mas`)
- **Entry eligibility**: Check first-year-out status, supplemental event eligibility, districts eligibility. (`funclib/nsda/check_fyo.mas`, `funclib/nsda/supp_eligible.mas`, `funclib/nsda/entry_check.mas`)
- **Entry limits**: District-level entry limits per event. (`funclib/nsda/entry_limits.mas`)
- **School status**: Track school registration/payment status. (`funclib/nsda/school_status.mas`, `school_status_data.mas`)

**Key entities:**
- `Tab::District` (`web/lib/Tab/District.pm`) - columns: id, name, code, location, level, realm, region, timestamp. Belongs to Region. Has many: chapters, schools, permissions, admins.
- `Tab::Region` (`web/lib/Tab/Region.pm`) - columns: id, name, code, circuit, tourn, timestamp. Belongs to Circuit and Tourn. Has many: schools, permissions, admins, chapters.
- `Tab::RegionSetting` (`web/lib/Tab/RegionSetting.pm`) - columns: id, region, tag, value, value_date, value_text, event, setting, timestamp. Per-event-per-region settings capability.
- `Tab::NSDACategory` (`web/lib/Tab/NSDACategory.pm`) - columns: id, type, code, name, national, timestamp. Types: debate (103-CX, 102-LD, 104-PF, 108-BQ), speech (202-IX/USX, 203-OO, 204-DI, 205-HI, 206-DUO, 207-POI, 208-INF), congress (301).
- `Tab::SweepAward` (`web/lib/Tab/SweepAward.pm`) - Used for both sweepstakes and qualification tracking via result_sets.
- `Tab::Weekend` (referenced but not in lib listing) - Manages weekend scheduling for district tournament series.
- Key tournament settings: `nsda_district` (marks tournament as district), `nsda_nats` (marks as NSDA Nationals), `nsda_district_questions` (JSON metadata)

**Key settings/configuration:**
- Tournament-level: `nsda_district` (district tournament flag), `nsda_nats` (nationals flag), `nsda_notes` (district notes), `nsda_district_questions` (JSON: tabbing software, extemp topics, award nominations, max_step)
- Event-level: `nsda_qual_force` (override qualifier count), `nsda_qual_override` (threshold override as "quals,count_threshold"), `weekend` (which weekend the event is held on)
- Circuit-level: `qualifiers` (JSON blob of all qualifier rule sets)
- Global: `nsda_district_open` (TabroomSetting: when district setup opens), `nsda_district_deadline` (TabroomSetting: when district setup closes)
- Student-level: `districts_eligible` (student_setting), `nsda_points` (student_setting)
- Person-level: `nsda_admin` (person_setting for NSDA staff)

**Key reports/exports:**
- District registration sheet: Printable registration overview for a district (`/register/reports/district_registration_print.mhtml`)
- Qualifier report: All qualifying entries by circuit, event code, and year (`qualifier_report.mhtml`)
- TOC bid sheets: Bid display for TOC qualification (`result_sheets.mhtml`)
- NSDA points reports (via student settings)
- District sweepstakes reports

**Integrations/dependencies:**
- **NSDA API**: Core integration for chapter/student import, membership verification, points posting, school status. API client at `funclib/nsda/api_client.mas`. Endpoints for rosters, person data, school data.
- **SpeechWire**: Alternative tabbing software; district system checks tabbing software choice and adjusts qualifier count calculation accordingly.
- **S3**: Legislation file storage for district congress events.
- **Tournament system**: District tournaments are full `Tab::Tourn` objects with special flags and restrictions.
- **Result system**: Qualifications feed through `ResultSet` with circuit-specific labels.
- **Permission system**: District committee roles (chair, member, alternate) stored as `Tab::Permission` entries.

**Documentation sources:**
- In-page help text in district creation wizard
- Code comment in `district_awards.mas`: "I refuse to put this nonsense into my database. Even though it probably should live there."
- Inline documentation in `qualifier_count.mas` explaining the qualification formula

**Code sources:**
- `web/user/nsda/` (37 files) - NSDA/district admin interface
- `web/register/district/` (3 files) - District registration view
- `web/setup/tourn/district_dates.mhtml`, `district_dates_save.mhtml`, `district_weekend_events.mhtml` - Tournament-side district setup
- `web/setup/events/districts_register.mhtml`, `nsda_events.mhtml` - Event-side district configuration
- `web/funclib/nsda/` (46 .mas files) - NSDA-specific business logic library
- `web/funclib/district*.mas` (8 files) - District utility functions: `district_tourns.mas`, `district_judges.mas`, `district_qualifiers.mas`, `district_committee.mas`, `district_tiebreakers.mas`, `district_entry.mas`, `district_registration.mas`, `district_select.mas`
- `web/user/circuit/qualifiers.mhtml`, `qualifier_*.mhtml` (6 files) - Circuit qualifier management
- ORM: `web/lib/Tab/District.pm`, `Region.pm`, `RegionSetting.pm`, `NSDACategory.pm`

**Complexity level:** Very High
- Deeply intertwined with NSDA-specific business rules. Multi-step tournament creation wizard. Complex qualifier count formula with COVID-era exceptions and multiple override mechanisms. External API integration with NSDA. District committee governance model. Worlds Schools team management layer. Multiple hard-coded special cases (circuit IDs, NSDA category codes). The qualification pipeline spans district -> regional -> national levels.

**Feature frequency / criticality:** Some tournaments (but critical for those)
- District tournaments are a yearly cycle affecting thousands of schools. The qualifier system is critical for TOC and state-level competition. NSDA Nationals registration depends entirely on this pipeline. However, many tournaments do not use district features at all.

**Notes and open questions:**
- The District model has no `DistrictSetting` table -- district-level settings are stored on the associated tournament's settings (`nsda_district_questions` JSON, `nsda_notes`). This is an unusual pattern.
- District awards are hard-coded in `district_awards.mas` rather than stored in a database table. The developer's comment explicitly acknowledges this should be in the database.
- The `qualifier_count.mas` has special handling for the 2020-2021 COVID year with explicit date bounds -- this kind of temporal business logic is fragile.
- SpeechWire integration creates a branching path in qualifier count logic -- entries from SpeechWire-tabbed tournaments may not have ballots to verify participation.
- The `realm` column on District is present but its purpose is unclear from the code examined.
- Hard-coded NSDA event codes (HSE, SEN, CX, LD, PF, BQ, DI, DUO, HI, INF, IX, OO, POI, USX) are in `funclib/nsda/events.mas` -- changes to NSDA event lineup require code changes.
- The distinction between `nsda_qual_force` and `nsda_qual_override` is subtle: force sets an absolute count, while override provides a threshold-based formula.
- The `Tab::Weekend` model is used extensively in district tournament management but its ORM definition was not found in the standard location -- may need investigation.
- Region's `tourn` FK is noteworthy -- regions can be both circuit-level (persistent) and tournament-specific, suggesting some overlap in how regions are used across the platform.
- The circuit qualifier rule system stores everything in a single JSON blob, which means concurrent editing by multiple admins could cause data loss. A normalized schema would be safer.
- Worlds Schools (WSDC) team management at districts (`wsdc_team_save.mhtml`, `wsdc_judge_edit.mhtml`) adds another layer of complexity for selecting and managing national team representatives.
