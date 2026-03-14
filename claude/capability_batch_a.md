# Capability Domains: Batch A

## 1. Tournament Creation and Administration

### Tournament Creation and Administration

**Description:** Covers the full lifecycle of creating a tournament, configuring its identity (name, location, webname), managing administrative access and permissions, associating the tournament with circuits, managing tournament logos/files, cloning from previous tournaments, handling payment for Tabroom services, and providing public-facing tournament information pages.

**Primary users:** Tournament directors, tournament owners, tabbers, site administrators, NSDA staff

**Main entry points:**
- Tournament creation wizard: `web/user/tourn/request.mhtml` (creates new tournaments, supports cloning)
- Tournament creation save: `web/user/tourn/save.mhtml`
- Main tournament admin page: `web/setup/tourn/main.mhtml` (name, location, webname, logo, circuits)
- Access/permissions management: `web/setup/tourn/access.mhtml`, `access_add.mhtml`, `access_save.mhtml`
- Tournament payment: `web/setup/tourn/payment.mhtml`
- Tournament notes: `web/setup/tourn/notes.mhtml`
- Tournament messages: `web/setup/tourn/messages.mhtml`
- Tournament import/clone: `web/setup/tourn/import.mhtml`
- Tournament merge (districts): `web/setup/tourn/merge_tourns.mhtml`
- Public tournament index: `web/index/tourn/index.mhtml`
- Public tournament events/schools/judges: `web/index/tourn/events.mhtml`, `schools.mhtml`, `judges.mhtml`

**Major sub-capabilities:**
- Create new tournament (name, dates, location, timezone)
- Clone tournament from a previous edition (copies events, schedules, settings but not registrants/judges)
- Import tournament rules from another tournament (`web/setup/tourn/import.mhtml`)
- Edit tournament name, city, state, country, timezone
- Set and manage webname (public URL: `<webname>.tabroom.com`)
- Upload/delete tournament logo (PNG/JPG only, stored in S3)
- Upload/delete invitation documents and bill packets
- Associate tournament with up to 5 circuits (with approval workflow)
- Manage administrative access with tiered permission levels:
  - **Owner**: full tournament access
  - **Tabber**: full access except ownership changes, contact changes, deleting change logs
  - **Checker**: limited to activation/deactivation, ballot entry, round start/attendance
  - **Category-level tabber/checker**: access limited to a judge category
  - **Event-level tabber/checker**: access limited to specific events
  - **Contact**: public-facing contact listed on tournament website
- Auto-backup follower management (users who receive automated data backups)
- Tournament payment for Tabroom services (separate from registration fees)
- Manage tournament hidden/test mode
- Merge district tournaments
- NSDA district tournament integration (software selection, strikes rules)

**Key entities:**
- `Tab::Tourn` (`web/lib/Tab/Tourn.pm`) - columns: id, name, city, state, country, webname, start, end, reg_start, reg_end, tz, hidden, timestamp
- `Tab::TournSetting` (`web/lib/Tab/TournSetting.pm`) - EAV pattern: id, tourn, tag, value, value_date, value_text, setting, timestamp
- `Tab::Permission` (`web/lib/Tab/Permission.pm`) - columns: id, tag, person, category, event, tourn, chapter, region, district, circuit, created_by, details, timestamp
- `Tab::TournCircuit` (`web/lib/Tab/TournCircuit.pm`) - join table: tourn <-> circuit with approval status
- `Tab::File` (`web/lib/Tab/File.pm`) - tournament files (invitations, bills)
- `Tab::ChangeLog` (`web/lib/Tab/ChangeLog.pm`) - audit trail
- `Tab::Weekend` (`web/lib/Tab/Weekend.pm`) - district tournament weekends: id, name, tourn, site, city, state, start, end, reg_start, reg_end, plus deadline fields

**Key settings/configuration:**
Settings are stored in the `tourn_setting` table via the EAV pattern. The `TournSetting` model supports four value types: plain value, text (blob), date, and json. Known settings from code analysis:

- `logo` - tournament logo filename (stored in S3)
- `nsda_district` - links to NSDA district ID
- `nsda_nats`, `nsda_ms_nats` - NSDA nationals flags
- `ncfl`, `ncfl_codes` - NCFL tournament flags
- `mock_trial_registration` - mock trial mode
- `backup_followers` - JSON array of person IDs for auto-backups
- `chatwoot_enable` - chat widget enable (admin-only)
- `tabroom_purchased`, `tabroom_grant`, `nc_purchased`, `nco_purchased` - payment records
- `original_start`, `original_end` - original dates (for payment change detection)
- `registration_packet` - uploaded registration packet filename
- `invoice_address`, `invoice_message`, `registration_message` - custom text messages

**Key reports/exports:**
- Public tournament website at `<webname>.tabroom.com`
- Tournament information pages (events, schools, judges, results)
- Change log audit trail

**Integrations/dependencies:**
- S3 storage for logos, files, and documents (`$Tab::s3_url`, `$Tab::s3_bucket`, `$Tab::s3_cmd`)
- NSDA district system (district codes, weekend management)
- Circuit system (approval workflow for circuit association)
- Person/account system for access management
- Indexcards API (`$Tab::indexcards_url`) for AJAX-style permission updates

**Documentation sources:**
- Code comments in `web/setup/tourn/main.mhtml` (inline help text about webnames, logos)
- Permission level descriptions in `web/setup/tourn/access.mhtml` (detailed inline documentation for each role)
- Tournament creation guidance in `web/user/tourn/request.mhtml`
- Clone process documentation in `web/setup/tourn/import.mhtml`
- External: docs.tabroom.com (referenced but not stored in repo)

**Code sources:**
- `web/setup/tourn/` - all tournament admin pages (main.mhtml, access.mhtml, etc.)
- `web/user/tourn/` - tournament creation flow (request.mhtml, save.mhtml, clone_tournament.mas)
- `web/index/tourn/` - public tournament pages
- `web/lib/Tab/Tourn.pm`, `TournSetting.pm`, `Permission.pm`, `TournCircuit.pm`
- `web/funclib/tourn_admins.mas`, `tourn_categories.mas`, `tourn_circuits.mas`, `tourn_contacts.mas`
- `web/setup/tourn/tabbar.mas` - navigation tabs: Name & Info, General, Dates, Access, Messages, Notes, Payment

**Complexity level:** High - The permission system is multi-layered (owner/tabber/checker at tournament, category, and event levels). The cloning system must replicate complex nested structures. District tournament variants add significant conditional logic throughout. Multiple payment integrations exist.

**Feature frequency / criticality:** Every tournament - Tournament creation and access management are prerequisites for all other tournament operations.

**Notes and open questions:**
- The clone/import system in `web/setup/tourn/import.mhtml` includes extensive safeguards (prevents cloning into same tournament, prevents import after tournament starts)
- Date changes are blocked 10+ days after tournament end to prevent misuse
- The permission system uses the `details` column as a JSON blob for extended permission data
- There is a 5-circuit limit per tournament, enforced in the UI
- District tournaments have significantly different UI flows (weekends, merged tournaments, NSDA-specific settings)
- The `setting()` method on Tourn.pm uses a dual getter/setter pattern: call with value to set, without value to get

---

## 2. Tournament Settings and Configuration

### Tournament Settings and Configuration

**Description:** Manages tournament-wide configuration settings that control registration behavior, entry coding, payment processing, school regions, online/hybrid tournament modes, and various operational preferences. These settings are stored in the EAV (Entity-Attribute-Value) `tourn_setting` table and govern how registration, communication, and tournament operations function.

**Primary users:** Tournament directors, tournament owners, tabbers

**Main entry points:**
- General settings page: `web/setup/tourn/settings.mhtml`
- Settings save handler: `web/setup/tourn/settings_save.mhtml`
- Individual setting toggle: `web/setup/tourn/setting_switch.mhtml`
- Dates & deadlines: `web/setup/tourn/dates.mhtml`, `dates_save.mhtml`
- District dates: `web/setup/tourn/district_dates.mhtml`, `district_dates_save.mhtml`
- Messages/invoicing: `web/setup/tourn/messages.mhtml`, `messages_save.mhtml`
- Payment configuration: `web/setup/tourn/settings.mhtml` (sidebar: e-payments section)

**Major sub-capabilities:**
- **Registration settings:**
  - Hidden/test tournament mode (`hidden` column on Tourn)
  - Require adult contact info (`require_adult_contact`)
  - Tabroom account-based coaches & contacts (`account_contacts`)
  - Require second adult contact (`second_adult_contact`)
  - Ask for adult contacts by category (`category_adult_contact`)
  - Closed tournament mode - admin-only registration (`closed_entry`)
  - Tournament-wide entry cap (`overall_cap`)
  - Waitlist double-entry counting (`no_waitlist_double_entry`)
  - Log registration changes (`track_reg_changes`)
  - Limit info in blast messages (`limit_info`)
  - Onsite registration toggle and time window (`onsite_registration`, `onsite_starts`, `onsite_ends`, `onsite_only_paid`)

- **School coding:**
  - School code format (`school_codes` - options include 'incremental')
  - First school code number (`first_school_code`)
  - NCFL diocese codes (`ncfl_codes`)

- **Regions:**
  - Enable school regions (`regions`)
  - Use circuit standard regions (`region_circuit`)
  - Custom region label (`region_label`)
  - Custom reference label (`reference_label`)

- **Dates & deadlines (stored as date settings):**
  - Tournament start/end (columns on `tourn` table)
  - Registration open/close (`reg_start`, `reg_end` on tourn table)
  - Fees & obligations freeze (`freeze_deadline`)
  - Drops & name changes due (`drop_deadline`)
  - Nuisance fines apply after (`fine_deadline`)
  - Judge registration due (`judge_deadline`)
  - Script info uploads due (`script_deadline`)
  - Entry release forms due (`release_deadline`)
  - NSDA-specific: legislation due (`bill_deadline`), 50% fees due (`fifty_percent_deadline`), 100% fees due (`hundred_percent_deadline`), supps due (`supp_deadline`), refund info due (`refund_deadline`)

- **E-payment configuration:**
  - Tournament Money (`tmoney_enable`, `tmoney_url`, `tmoney_require_epayments`, `tmoney_staging`)
  - PayPal (`paypal_enable`, `paypal_merchant_id`, `paypal_client_id`, `paypal_email`, `paypal_url`)
  - Authorize.Net (`authorizenet_enable`, `authorizenet_api_login`, `authorizenet_transaction_key`, `authorizenet_client_key`, `authorizenet_ach_enable`, `authorizenet_cc_fee`, `authorizenet_ach_fee`)

- **Messages/Communication:**
  - Invoice address/payable-to text (`invoice_address`)
  - Invoice message (`invoice_message`)
  - Registration front-page message (`registration_message`)
  - Onsite registration notes (`onsite_notes`)
  - Require hotel form (`require_hotel_form`)

- **Credit/billing:**
  - Enable credit system (`enable_credit`)

**Key entities:**
- `Tab::Tourn` - core date columns: start, end, reg_start, reg_end, hidden, tz
- `Tab::TournSetting` - EAV settings store: tag, value, value_date, value_text
- `Tab::Setting` / `Tab::SettingLabel` - setting metadata/labels (referenced by TournSetting FK)

**Key settings/configuration:**
The settings_save.mhtml handler processes three categories of settings:
1. **Plain settings** (stored as value): `track_reg_changes`, `overall_cap`, `school_codes`, `first_school_code`, `ncfl_codes`, `no_waitlist_double_entry`, `regions`, `onsite_registration`, `onsite_only_paid`, `require_hotel_form`, `enable_credit`, `region_circuit`, `region_label`, `reference_label`
2. **Date settings** (stored as value="date", value_date=datetime): `onsite_ends`, `onsite_starts`
3. **Text settings** (stored as value="text", value_text=blob): `onsite_notes`

Boolean toggle settings use `web/funclib/bool_switch.mas` which makes AJAX calls to `setting_switch.mhtml` for immediate save without form submission.

**Key reports/exports:**
- No direct reports; settings control behavior of registration, billing, and operations

**Integrations/dependencies:**
- Tournament Money platform (`tmoney_url` endpoint)
- PayPal payment processing
- Authorize.Net payment processing (with ACH support)
- S3 for registration packet upload
- `web/funclib/bool_switch.mas` - reusable toggle component
- `web/funclib/log.mas` - change logging for text settings

**Documentation sources:**
- Inline help text throughout `web/setup/tourn/settings.mhtml`
- Tooltip text on form elements
- Deadlines explanation sidebar: `web/setup/tourn/deadlines.mas`
- Date change restrictions documented in `web/setup/tourn/dates.mhtml` (10-day post-tournament lock)

**Code sources:**
- `web/setup/tourn/settings.mhtml` - main settings UI (very large file, ~660+ lines)
- `web/setup/tourn/settings_save.mhtml` - settings persistence handler
- `web/setup/tourn/setting_switch.mhtml` - individual boolean toggle handler
- `web/setup/tourn/dates.mhtml` - dates & deadlines UI
- `web/setup/tourn/dates_save.mhtml` - dates persistence
- `web/setup/tourn/messages.mhtml`, `messages_save.mhtml` - messaging config
- `web/setup/tourn/payment.mhtml` - payment overview (room/judge counts for billing)
- `web/setup/tourn/paypal_save.mhtml`, `authorizenet_save.mhtml` - payment provider saves
- `web/lib/Tab/TournSetting.pm`, `Setting.pm`, `SettingLabel.pm`
- `web/funclib/bool_switch.mas` - toggle widget used extensively

**Complexity level:** High - The EAV pattern means settings are not schema-enforced; any string tag can be stored. The settings page is very large with extensive conditional display logic based on tournament type (regular vs. NSDA district vs. NSDA nationals vs. NCFL vs. mock trial). Payment integrations add three distinct configuration paths. Multiple date types with timezone conversion add complexity.

**Feature frequency / criticality:** Every tournament - Settings must be configured for registration to function properly. Incorrect deadline settings can lock out or open registration unintentionally.

**Notes and open questions:**
- The EAV `setting()` method on Tourn.pm supports four value types: plain string, "text" (blob in value_text), "date" (datetime in value_date), "json" (JSON-encoded in value_text). Setting value to "delete", "", or "0" removes the setting.
- There is no schema-level catalog of valid settings. Settings are created on the fly by any code that calls `$tourn->setting("tag_name", value)`. A complete inventory of all setting tags used across the codebase would require a full grep of all files.
- The `all_settings()` method loads all settings into a hash for efficient access (used as `$tourn_settings` throughout the UI)
- Payment provider credentials (Authorize.Net) are noted as "auto-deleted post tournament"
- The settings page has significant JavaScript for conditional show/hide of sections based on toggle states
- District tournaments, NSDA nationals, NCFL, and mock trial tournaments each suppress or modify different settings
- The `hidden` flag on the Tourn table itself (not a setting) controls public visibility

---

## 3. Schedule, Rounds, and Timeslots

### Schedule, Rounds, and Timeslots

**Description:** Manages the temporal structure of a tournament: timeslots define when things happen, rounds define what competitive activity occurs in each timeslot per event, and the schedule view ties them together. This domain covers creating/editing timeslots, assigning rounds to timeslots, setting round types (prelim, elim, final, etc.), assigning tiebreak protocols, managing flights, detecting schedule conflicts, and printing the master schedule.

**Primary users:** Tournament directors, tabbers, event-level tabbers

**Main entry points:**
- Schedule overview (by day): `web/setup/schedule/sked.mhtml`
- Schedule by event: `web/setup/schedule/event.mhtml`
- Save schedule changes: `web/setup/schedule/save.mhtml`
- Save event schedule: `web/setup/schedule/event_save.mhtml`
- Create timeslot: `web/setup/schedule/create.mhtml`
- Delete timeslot: `web/setup/schedule/delete.mhtml`
- Clone schedule: `web/setup/schedule/clone_schedule.mhtml`
- Move all timeslots on a day: `web/setup/schedule/move_day.mhtml`
- Print master schedule: `web/setup/schedule/print.mhtml`
- Print event schedule: `web/setup/schedule/print_event.mhtml`
- Navigation: `web/setup/schedule/menu.mas`

**Major sub-capabilities:**
- **Timeslot management:**
  - Create timeslots with name, start time, end time
  - Edit timeslot names and times (bulk save per day)
  - Delete timeslots (cascading: deletes all rounds and results in the timeslot)
  - Move all timeslots from one day to another
  - Multi-day support with day selector
  - Async event support (timeslots can span multiple days with separate end-day selector)

- **Round management (per event):**
  - Assign rounds to timeslots via checkbox grid
  - Round types: prelim, highlow (hi/lo pairing), highhigh, snaked_prelim, elim, final, runoff
  - Round labels (custom display names vs. auto-numbered "Round N")
  - Assign tiebreak protocol per round (from tournament's protocol list)
  - Assign site per round (when multiple sites exist)
  - Set flights per round (1-5)
  - Move rounds between timeslots
  - Clone schedule from one event to another
  - Published/paired status tracking

- **Schedule validation:**
  - Warns if timeslot start equals end (zero-duration)
  - Warns if timeslot duration exceeds 4 hours (except congress and async events)
  - Warns if end time precedes start time
  - Detects overlapping rounds within the same event
  - District tournament: warns if rounds start after 9:30 PM
  - Warns if round has no tiebreak protocol set
  - Warns if round site is not attached to tournament

- **Schedule printing:**
  - Master schedule PDF via LaTeX (`web/setup/schedule/print.mhtml`)
  - Event-specific schedule PDF (`web/setup/schedule/print_event.mhtml`)

- **Congress-specific:**
  - Rounds display as "Session N" instead of "Round N"
  - Flight selection hidden for congress events

**Key entities:**
- `Tab::Timeslot` (`web/lib/Tab/Timeslot.pm`) - columns: id, tourn, name, start, end, timestamp. Has many rounds. Methods: `done()` (checks if all ballots are complete), `span()` (returns DateTime::Span), `entries()` (entries competing in this timeslot)
- `Tab::Round` (`web/lib/Tab/Round.pm`) - columns: id, type, name, label, flighted, published, post_primary, post_secondary, post_feedback, paired_at, start_time, site, event, runoff, protocol, timeslot, timestamp. Has many: jpools, rpools, autoqueues, settings, panels, results. Methods: `realname()`, `shortname()`, `parents()`, `setting()`, `all_settings()`
- `Tab::RoundSetting` (`web/lib/Tab/RoundSetting.pm`) - EAV: id, round, tag, value, value_date, value_text, setting, timestamp
- `Tab::Protocol` (`web/lib/Tab/Protocol.pm`) - tiebreak protocols assigned to rounds

**Key settings/configuration:**
- Round types (stored in `round.type`): `prelim`, `highlow`, `highhigh`, `snaked_prelim`, `elim`, `final`, `runoff`
- Round settings (EAV in `round_setting`):
  - `use_for_breakout` - marks round as a breakout round (excluded from overlap detection)
- Event settings that affect scheduling:
  - `online_mode` - values: `sync`, `async`, `public_jitsi`, `public_jitsi_observers`, `nsda_campus`, `nsda_campus_observers`
  - `online_hybrid` - marks event as hybrid online/in-person
  - `weekend` - assigns event to a specific district weekend (or "nope" for not held)
  - `round_robin` - limits round type options
  - `supp` - supplemental event flag
- The `round.flighted` column controls how many flights a round has (1-5)
- The `round.published` column tracks publication state
- The `round.protocol` FK links to the tiebreak protocol

**Key reports/exports:**
- Master Schedule PDF: `web/setup/schedule/print.mhtml` - LaTeX-generated PDF showing all timeslots with their rounds, organized by time
- Event Schedule PDF: `web/setup/schedule/print_event.mhtml`

**Integrations/dependencies:**
- `Tab::Event` - rounds belong to events; schedule view groups by event
- `Tab::Protocol` - tiebreak protocols must exist before rounds can be properly configured
- `Tab::Site` - rounds are assigned to sites
- `Tab::Panel` - sections/panels belong to rounds (rounds with panels cannot be deleted via schedule screen)
- `Tab::Weekend` - district events are filtered to weekend timeslots
- Permission system: schedule editing requires owner or tabber level; event-level permissions filter which events are shown
- `web/funclib/perms/timeslots.mas` - filters visible timeslots by permissions
- `web/funclib/perms/events.mas` - filters visible events by permissions
- `web/funclib/tourn_days.mas` - calculates tournament days for day selector
- `web/funclib/printout.mas` - LaTeX PDF generation framework

**Documentation sources:**
- Inline help text in `web/setup/schedule/sked.mhtml`:
  - Explains how to delete timeslots (use orange trash button)
  - Warning about backup before deletion
  - Explains that deleting timeslots deletes all rounds and results
- Inline warnings in `web/setup/schedule/event.mhtml`:
  - Multiple rounds in same timeslot warning
  - Missing tiebreak protocol warning ("DANGER WILL ROBINSON!")
  - Missing site assignment warning
  - Overlap detection with explanation

**Code sources:**
- `web/setup/schedule/` - all schedule UI files
  - `sked.mhtml` - main timeslot editor (day view)
  - `event.mhtml` - event round assignment (primary round configuration UI)
  - `event_save.mhtml` - saves round assignments
  - `save.mhtml` - saves timeslot edits
  - `create.mhtml` - creates new timeslots
  - `delete.mhtml` - deletes timeslots
  - `clone_schedule.mhtml` - clones rounds from one event to another
  - `move_day.mhtml` - bulk moves timeslots between days
  - `print.mhtml`, `print_event.mhtml` - PDF generation
  - `menu.mas` - left sidebar with event list and day selector
- `web/lib/Tab/Round.pm`, `RoundSetting.pm`, `Timeslot.pm`
- `web/funclib/tourn_days.mas`, `tourn_rounds.mas`, `category_rounds.mas`, `category_timeslots.mas`, `event_timeslots.mas`

**Complexity level:** High - The schedule is a multi-dimensional grid (days x timeslots x events x rounds) with extensive validation logic. The event.mhtml file handles round creation, updating, moving, type changes, site assignment, protocol assignment, and flight configuration in a single form. Overlap detection uses custom SQL. District weekends add another layer of timeslot filtering. Async events alter the timeslot model to support multi-day spans.

**Feature frequency / criticality:** Every tournament - Without timeslots and rounds, no pairing or tabulation can occur. Schedule errors (overlapping rounds, missing protocols) cascade into pairing and judging failures.

**Notes and open questions:**
- Rounds with existing panels (sections) cannot be deleted from the schedule screen; they are locked and shown with a green check. Users must go to the schematic to manage existing rounds.
- The `runoff` column on Round is a self-referential FK (a runoff round points to the round it is a runoff of)
- The `post_primary`, `post_secondary`, `post_feedback` columns on Round appear to control what results are published but their exact semantics need expert review
- The `paired_at` and `start_time` columns are datetimes - `paired_at` tracks when the round was paired, `start_time` appears to be an actual start time vs. the scheduled timeslot start
- The clone_schedule feature only appears when an event has zero rounds
- Event types "debate", "speech", and "congress" each have different available round types:
  - Debate: prelim, highlow, highhigh, elim, final, runoff
  - Speech: prelim, snaked_prelim, elim, final, runoff
  - Congress: prelim, elim, final, runoff (no flight selector)
  - Round-robin events: only prelim, elim, final, runoff (no hi-lo options)

---

## 4. Sites and Rooms

### Sites and Rooms

**Description:** Manages the physical and virtual locations where tournament rounds take place. Sites represent locations (schools, convention centers, or online platforms). Rooms are individual spaces within sites. This domain covers site creation, room management (add/edit/import/delete), room pools (RPools) for assigning subsets of rooms to specific rounds, room time blocks/strikes for constraining room availability, and NSDA Campus online room provisioning.

**Primary users:** Tournament directors, tabbers, site hosts

**Main entry points:**
- Site management: `web/setup/rooms/manage_sites.mhtml`
- Site settings/edit: `web/setup/rooms/site_edit.mhtml`, `site_save.mhtml`
- Add existing site: `web/setup/rooms/site_add.mhtml`
- Create new site: `web/setup/rooms/site_new.mhtml`
- Remove site: `web/setup/rooms/site_rm.mhtml`
- Room list/edit: `web/setup/rooms/list.mhtml`
- Add rooms: `web/setup/rooms/site_rooms_add.mhtml`
- Save room changes: `web/setup/rooms/site_rooms_save.mhtml`
- Import rooms from CSV: `web/setup/rooms/import.mhtml`, `import_save.mhtml`
- Room time blocks: `web/setup/rooms/blocks.mhtml`, `block_add.mhtml`, `block_save.mhtml`, `block_rm.mhtml`
- Activate/deactivate all rooms: `web/setup/rooms/activate_rooms.mhtml`
- Room property toggle: `web/setup/rooms/room_switch.mhtml`
- Print rooms: `web/setup/rooms/print_rooms.mhtml`
- NSDA Campus rooms: `web/setup/rooms/nsda_campus.mhtml`
- NSDA Campus admin: `web/setup/rooms/campus_admin.mhtml`
- Auto-create online rooms: `web/setup/rooms/site_online_autocreate.mhtml`
- Navigation: `web/setup/rooms/menu.mas`, `tabbar.mas`

**Major sub-capabilities:**
- **Site management:**
  - Add existing site from user's known sites (sites associated with tournaments the user administers)
  - Create brand-new site
  - Remove site from tournament
  - Edit site details: name, online flag, ballot dropoff location, site host, directions/GPS
  - Sites can be marked as "online" to enable URL/passcode fields on rooms
  - Sites belong to circuits and can be shared across tournaments

- **Room management (in-person sites):**
  - Add rooms manually (10 at a time via form)
  - Edit room properties: name, quality rating (lower=better), capacity, notes, map URL, ADA accessibility
  - Soft-delete rooms (set `deleted=1`, not hard delete)
  - Activate/deactivate individual rooms or bulk all
  - Import rooms from CSV file (format: Name, Quality, Capacity, Notes, Accessible, Map URL)

- **Room management (online sites):**
  - Add rooms with: name, entry URL, entry passcode, judge URL, judge passcode, CC API ID
  - Auto-create online rooms in bulk (with prefix, starting number, count, optional URL append)
  - Import online rooms from CSV (format: Name, Entry URL, Entry Password, Judge URL, Judge Password, CC API ID)

- **Room pools (RPools):**
  - Create named room pools that group rooms together
  - Assign rooms to pools
  - Assign rounds to pools (determines which rooms are available for which rounds)
  - Pool settings via EAV

- **Room time blocks/strikes (RoomStrike):**
  - Time-based blocks: room unavailable during specific time window
  - Event-based blocks: room excluded from specific events ("No rounds of [Event]")
  - Judge-based strikes: room excluded for specific judges
  - Entry-based strikes: room excluded for specific entries
  - Block display organized by day with add/remove interface

- **NSDA Campus integration:**
  - Day-by-day room count allocation
  - Observer room allocation
  - Automated room provisioning from NSDA Campus service
  - Separate from regular site/room management

- **Printing:**
  - Room list PDF via LaTeX (room name, quality, capacity, notes, ADA, active status)

**Key entities:**
- `Tab::Site` (`web/lib/Tab/Site.pm`) - columns: id, name, circuit, timestamp; Others: host, directions, circuit, dropoff, online. Has many rooms and rounds. Methods: `tourns()`, `events()`, `panels()`
- `Tab::Room` (`web/lib/Tab/Room.pm`) - columns: id, name, site, inactive; Others: quality, timestamp, capacity, notes, ada, rowcount, seats, deleted, url, password, judge_url, judge_password, api. TEMP: score. Has many: panels, strikes, blocks, rpools
- `Tab::TournSite` (`web/lib/Tab/TournSite.pm`) - join table: id, tourn, site, timestamp
- `Tab::RPool` (`web/lib/Tab/RPool.pm`) - room pools: id, name, tourn, timestamp. Has many: room_links (RPoolRoom), round_links (RPoolRound). Methods: `setting()`, `all_settings()`
- `Tab::RPoolRoom` (`web/lib/Tab/RPoolRoom.pm`) - join: id, room, rpool, timestamp
- `Tab::RPoolRound` (`web/lib/Tab/RPoolRound.pm`) - join: id, round, rpool, timestamp
- `Tab::RPoolSetting` (`web/lib/Tab/RPoolSetting.pm`) - EAV for room pool settings
- `Tab::RoomStrike` (`web/lib/Tab/RoomStrike.pm`) - columns: id, room, type, event, judge, tourn, entry, start, end, timestamp. Types: "time", "event", "judge", "entry". Method: `name()` generates human-readable description

**Key settings/configuration:**
- `Site.online` - boolean flag distinguishing physical vs. online sites
- `Site.dropoff` - ballot dropoff location text
- `Site.directions` - directions/GPS text
- `Site.host` - FK to Person (site host)
- `Room.quality` - numeric quality rating (lower = better, used first in room assignment)
- `Room.capacity` - numeric capacity (informational only, not used automatically)
- `Room.ada` - ADA accessibility flag
- `Room.inactive` - room deactivation flag
- `Room.deleted` - soft delete flag (rooms with deleted=1 are excluded from queries)
- Tournament settings for NSDA Campus:
  - `nsda_campus_days` - JSON object with per-day room count allocations and timezone
  - `nsda_campus_observer_days` - JSON object with per-day observer room allocations
  - `nc_purchased` - purchased room count limit
  - `nco_purchased` - purchased observer room count limit

**Key reports/exports:**
- Room list PDF: `web/setup/rooms/print_rooms.mhtml` - LaTeX-generated PDF with room details per site

**Integrations/dependencies:**
- `Tab::Round` - rounds are assigned to sites; room pools link rooms to rounds
- `Tab::Panel` - panels (sections) are assigned to rooms
- `Tab::Event` - event-based room strikes exclude rooms from specific events
- `Tab::Judge` - judge-based room strikes
- `Tab::Entry` - entry-based room strikes
- `Tab::Weekend` - district weekends have site assignments
- Circuit system - sites belong to circuits and are shared across circuit tournaments
- NSDA Campus API - for automated online room provisioning
- `web/funclib/clean_rooms.mas`, `clean_rooms_hash.mas` - room data preparation utilities
- `web/funclib/docshare_rooms.mas` - room sharing for document distribution
- `web/funclib/online_room.mas` - online room URL handling
- `web/funclib/room_panels.mas`, `room_rpools.mas`, `room_strikes.mas` - room data queries
- `web/funclib/round_rooms.mas`, `round_clear_rooms.mas`, `round_entry_rooms.mas` - round-room relationship queries
- `web/funclib/rpool_rooms.mas`, `rpool_sites.mas` - room pool queries
- `web/funclib/site_room_strikes.mas` - site-level room strike queries
- `web/funclib/tourn_sites.mas` - tournament site queries
- `web/funclib/printout.mas` - LaTeX PDF generation framework

**Documentation sources:**
- Inline help in `web/setup/rooms/manage_sites.mhtml`:
  - Explains when to use multiple sites vs. one site
  - Online tournament guidance (NSDA Campus vs. custom URLs)
  - District site assignment instructions
- Inline help in `web/setup/rooms/import.mhtml`:
  - Detailed CSV format specification for both online and in-person rooms
  - Field ordering requirements
- Inline help in `web/setup/rooms/list.mhtml`:
  - Online room passcode/URL usage explanation

**Code sources:**
- `web/setup/rooms/` - all room/site management pages
  - `manage_sites.mhtml` - tournament site list and add/create
  - `site_edit.mhtml` - site settings (name, online, dropoff, host, directions)
  - `list.mhtml` - room list/edit with mode switching (online vs. in-person)
  - `blocks.mhtml` - room time block management grid (rooms x days)
  - `block_add.mhtml` - add time block or event block to room
  - `import.mhtml`, `import_save.mhtml` - CSV room import
  - `nsda_campus.mhtml` - NSDA Campus room provisioning
  - `campus_admin.mhtml` - NSDA Campus admin functions
  - `activate_rooms.mhtml` - bulk activate/deactivate
  - `print_rooms.mhtml` - PDF room list
  - `menu.mas` - left sidebar with site list and NSDA Campus link
  - `tabbar.mas` - tabs: Edit Rooms, Add New, Import from File, Time Blocks, Site Settings
- `web/lib/Tab/Site.pm`, `Room.pm`, `TournSite.pm`, `RPool.pm`, `RPoolRoom.pm`, `RPoolRound.pm`, `RPoolSetting.pm`, `RoomStrike.pm`
- `web/funclib/clean_rooms.mas`, `docshare_rooms.mas`, `online_room.mas`, `room_panels.mas`, `room_rpools.mas`, `room_strikes.mas`, `round_rooms.mas`, `rpool_rooms.mas`, `rpool_sites.mas`, `site_room_strikes.mas`, `tourn_sites.mas`

**Complexity level:** Medium-High - The dual mode (online vs. in-person) for sites and rooms doubles the UI and data model surface area. Room pools (RPools) add an additional layer of indirection between rooms and rounds. The room strike/block system supports four different constraint types. NSDA Campus integration adds a third room provisioning path. The soft-delete pattern (deleted flag) requires careful filtering throughout queries.

**Feature frequency / criticality:** Every tournament (in-person or online) - Rooms must exist for pairing to assign sections to physical/virtual spaces. Tournaments without rooms configured cannot complete round pairing. Room pools are used by most larger tournaments.

**Notes and open questions:**
- Sites are shared across tournaments via circuits. A site created for one tournament can be reused by any tournament whose admin has access. This means room data persists between tournaments.
- The `Room.rowcount` and `Room.seats` columns exist in the schema but are not visible in the current room editing UI -- their purpose is unclear and may be legacy.
- Room quality rating is "lower is better" -- rooms with lower quality numbers are used first in automatic room assignment.
- Room capacity is stated as "solely for your information and is not automatically used by Tabroom" in the import help text.
- The RPool (room pool) system is architecturally similar to the JPool (judge pool) system -- both use a pool -> pool_room/pool_judge -> pool_round three-table pattern.
- Online rooms have separate URL/password fields for entries and judges, allowing judges to be room hosts on different links.
- The `Room.api` field stores a "CCID" (Classrooms.cloud ID) for integration with that platform.
- Room strikes can be created for time blocks, specific events, specific judges, or specific entries -- the same `room_strike` table handles all four types via the `type` column.
- When a round's site is not attached to the tournament, the system auto-corrects to the weekend site (for districts) or the default site, logging a warning.
