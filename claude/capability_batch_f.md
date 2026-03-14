# Capability Domains: Batch F

## 1. Financials, Invoices, and Fines

### Financials, Invoices, and Fines

**Description:** Manages all monetary aspects of tournaments: entry fees per event, per-school standing fees, per-student/person fees, judge hire fees, nuisance fines (drop/add/no-show), concessions (merchandise/tickets/transit), hotel block surcharges, invoice generation, payment tracking, and NSDA billing code integration. Also handles Tabroom.com platform fees and NSDA Campus room purchases.

**Primary users:** Tournament directors, school coaches (viewing invoices), site admins (payment overrides), NSDA staff (billing codes)

**Main entry points:**
- `/setup/money/edit.mhtml` -- Financials landing page
- `/setup/money/entry.mhtml` -- Entry fees per event
- `/setup/money/schools.mhtml` -- Per-school and per-student fees, standing fees, one-time universal fees
- `/setup/money/fines.mhtml` -- Nuisance fine configuration (drop, add, judge no-show)
- `/setup/money/hires.mhtml` -- Judge hire fees (per-judge, per-entry, per-round, missing judge)
- `/setup/money/concessions.mhtml` -- Concessions/merchandise management
- `/setup/money/hotel.mhtml` -- Hotel block configuration with surcharges
- `/setup/money/message.mhtml` -- Invoice address, message, and judge fine notification text
- `/setup/money/nsda.mhtml` -- NSDA billing codes (entries, judges, bonds, fines)
- `/setup/tourn/payment.mhtml` -- Tabroom.com platform fee calculator and NSDA Campus room purchases
- `/register/reports/finance_report.mhtml` -- Financial report
- `/register/reports/finance_csv.mhtml` -- Financial CSV export
- `/register/reports/school_balances.mhtml` -- School balance summary
- `/register/reports/invoice_all.mhtml` -- Bulk invoice generation
- `/register/reports/fines.mhtml` -- Fines report
- `/register/reports/payments.mhtml` -- Payments report
- `/register/reports/concessions.mhtml` -- Concession orders report
- `/register/reports/concessions_totals.mhtml` -- Concession totals
- `/register/reports/concessions_print.mhtml` -- Printable concession orders
- `/api/school_invoice.mhtml` -- API invoice endpoint
- `/api/update_invoices.mhtml` -- Batch invoice update
- `/api/payment_confirm.mhtml` -- Payment confirmation callback

**Major sub-capabilities:**
- Per-event entry fees with alternate state pricing (`fees_alternate_state`, `alt_state_fee`)
- Configurable currency symbol (`currency` setting)
- Per-school standing fees with date ranges (TournFee model with start/end dates)
- Per-student and per-person fees (`per_student_fee`, `per_person_fee`)
- One-time universal fee levy across all registered schools (`levy_fine_save.mhtml`)
- Judge hire fees: per-judge, per-uncovered-entry, per-round, and missing-judge fine
- Hired judge availability pools with auto-accept thresholds
- Nuisance fines: drop fine, add fine, judge forfeit fines (prelim/elim), first-forfeit multiplier
- Fine deadline configuration linked to tournament dates
- Coach notification on judge fines (`forfeit_notify_coaches`)
- Concessions system: items with name, price, cap, school cap, deadline, billing code, description, sizes/types (ConcessionType/ConcessionOption)
- Concession invoice style: separate page or combined with main invoice
- Customizable concession label (`concession_name`)
- Hotel block management with fee multipliers and entry surcharges
- Hotel confirmation number requirement option (`require_hotel_confirmation`)
- Invoice address and message customization
- Judge fine notification message customization
- Balance calculation engine (`/funclib/balances.mas`) aggregating entries, judges, fines, concessions, hires, discounts
- School-level fee discounts: `entry_fee_discount`, `all_fee_discount`, `concession_fee_discount`
- Judge surcharge per school (`judge_surcharge`)
- Waitlist handling: include on invoices (`invoice_waitlist`), count toward judge obligations (`judges_waitlist`)
- NSDA billing code mapping for entries, hired judges, bonds, fines, and concessions
- BluSynergy invoice integration (Invoice model has `blusynergy`, `blu_number` columns)
- Tabroom.com platform fees: entry-based pricing with free threshold, grants, Campus discounts
- NSDA Campus room day purchases and observer room purchases
- Payment via NSDA Store checkout (`payment_register.mhtml`)
- Admin override for purchased/requested quantities

**Key entities:**
- `Tab::Invoice` (`web/lib/Tab/Invoice.pm`) -- columns: id, blusynergy, blu_number, paid, total, details, school, timestamp; has_a school, has_many payments (Fine)
- `Tab::Fine` (`web/lib/Tab/Fine.pm`) -- columns: id, reason, amount, payment, deleted, deleted_at, deleted_by, levied_at, levied_by, tourn, school, region, judge, person, parent, invoice, timestamp
- `Tab::TournFee` (`web/lib/Tab/TournFee.pm`) -- columns: id, tourn, amount, reason, start, end, timestamp
- `Tab::Concession` (`web/lib/Tab/Concession.pm`) -- columns: id, name, price, tourn, deadline, description, cap, school_cap, billing_code
- `Tab::ConcessionPurchase` (`web/lib/Tab/ConcessionPurchase.pm`) -- columns: id, concession, quantity, school, placed, fulfilled, invoice
- `Tab::ConcessionType` (`web/lib/Tab/ConcessionType.pm`)
- `Tab::ConcessionOption` (`web/lib/Tab/ConcessionOption.pm`)
- `Tab::ConcessionPurchaseOption` (`web/lib/Tab/ConcessionPurchaseOption.pm`)

**Key settings/configuration:**
- Tourn settings: `drop_fine`, `add_fine`, `forfeit_judge_fine`, `forfeit_judge_fine_elim`, `first_forfeit_multiplier`, `forfeit_notify_coaches`, `fine_deadline`, `per_student_fee`, `per_person_fee`, `currency`, `fees_alternate_state`, `invoice_waitlist`, `judges_waitlist`, `invoice_address`, `invoice_message`, `judge_fine_message`, `concession_name`, `concession_invoice`, `hotel_message`, `require_hotel_confirmation`, `nsda_billing`, `nsda_nats`, `nsda_ms_nats`, `nsda_billing_entries`, `nsda_billing_judges`, `nsda_billing_bonds`, `nsda_billing_fines`, `tabroom_purchased`, `tabroom_requested`, `tabroom_grant`, `nc_purchased`, `nc_requested`, `nco_purchased`, `nco_requested`
- Category settings: `hired_fee`, `uncovered_entry_fee`, `missing_judge_fee`, `round_hire_fee`, `hired_jpool`, `hired_rounds`, `rounds_per`
- Event settings: `alt_state_fee`, `adjust_judges_fees`
- School settings: `balance`, `entry_fee_discount`, `all_fee_discount`, `concession_fee_discount`, `individuals`, `judge_surcharge`, `no_judge_burden`, `hotel`
- TabroomSetting: `pricing` (JSON with `tabroom_entry`, `tabroom_free_threshold`, `campus_room`, `campus_room_observers`)

**Key reports/exports:**
- Finance report (`/register/reports/finance_report.mhtml`)
- Finance CSV (`/register/reports/finance_csv.mhtml`)
- School balances (`/register/reports/school_balances.mhtml`)
- Invoice all (`/register/reports/invoice_all.mhtml`)
- Fines report (`/register/reports/fines.mhtml`)
- Payments report (`/register/reports/payments.mhtml`)
- Concession reports (orders, totals, printable)
- NCFL fines reports (`/register/reports/ncfl_fines.mhtml`, `ncfl_fines_csv.mhtml`, `ncfl_fines_print.mhtml`)
- Diocese finance (`/register/reports/diocese_finance.mhtml`)
- Refund report (`/register/reports/refund_report.mhtml`, `refund_report_csv.mhtml`)
- Packet invoices (`/register/reports/packet_invoices.mhtml`)
- Hotel counts (`/register/reports/hotel_counts.mhtml`)
- Shipping report (`/register/reports/shipping_report.mhtml`)

**Integrations/dependencies:**
- BluSynergy invoicing (Invoice.blusynergy, Invoice.blu_number columns)
- NSDA Store checkout (payment_register.mhtml redirects to NSDA)
- NSDA billing code system for national tournaments
- `/funclib/balances.mas` -- central balance calculation engine
- `/funclib/school_fees.mas` -- per-school fee calculation

**Documentation sources:** Code comments in `web/setup/money/menu.mas` (fee explanation sidebar), inline help text in forms

**Code sources:**
- `web/setup/money/` -- all financial configuration pages
- `web/setup/tourn/payment.mhtml` -- Tabroom platform fees
- `web/register/reports/` -- financial reports (finance_report, finance_csv, school_balances, invoice_all, fines, payments, concessions_*)
- `web/funclib/balances.mas` -- balance calculation engine
- `web/funclib/school_fees.mas` -- per-school fee breakdown
- `web/lib/Tab/Invoice.pm`, `Fine.pm`, `TournFee.pm`, `Concession.pm`, `ConcessionPurchase.pm`, `ConcessionType.pm`, `ConcessionOption.pm`, `ConcessionPurchaseOption.pm`
- `web/api/school_invoice.mhtml`, `update_invoices.mhtml`, `payment_confirm.mhtml`

**Complexity level:** High

**Feature frequency / criticality:** Every tournament (fees and invoices are universal); concessions and hotel blocks are used by some tournaments; NSDA billing is specific to national-level events

**Notes and open questions:**
- BluSynergy integration details are unclear from code alone; the Invoice model has columns for it but the integration flow is not fully visible in the Mason code
- Payment confirmation flow (`payment_confirm.mhtml`) likely receives callbacks from the NSDA Store but the exact handshake protocol needs investigation
- The `parent` column on Fine suggests hierarchical fine relationships (possibly linking refunds to original fines)
- Comment in `balances.mas` references "duplicate invoice mess going on with Blu" suggesting historical issues with the BluSynergy integration
- Mock trial tournaments have special pricing rules (no free threshold)

---

## 2. Public Pages and Discovery

### Public Pages and Discovery

**Description:** The unauthenticated or lightly-authenticated public-facing pages of Tabroom.com. Provides tournament discovery (listing, searching, filtering), circuit browsing, tournament detail pages (info, events, schools, judges, paradigms, results, postings/schematics), public results archives, and static content pages (about, help). Also includes the public tournament registration portal.

**Primary users:** General public, prospective competitors, coaches browsing tournaments, parents/observers, researchers

**Main entry points:**
- `/index/index.mhtml` -- Homepage / tournament listing with filters (circuit, year, state, country)
- `/index/search.mhtml` -- Tournament name search
- `/index/circuits.mhtml` -- Circuit directory with country/state filtering
- `/index/circuit/` -- Individual circuit pages (index, calendar)
- `/index/schedule.mhtml` -- Circuit schedule view
- `/index/about.mhtml` -- About page (content from Webpage model, slug='about', sitewide=1)
- `/index/help.mhtml` -- Help page
- `/index/tourn/index.mhtml` -- Public tournament info page
- `/index/tourn/events.mhtml` -- Tournament event list
- `/index/tourn/schools.mhtml` -- Registered schools
- `/index/tourn/judges.mhtml` -- Judge list
- `/index/tourn/paradigms.mhtml` -- Tournament paradigms
- `/index/tourn/codes.mhtml` -- Entry codes
- `/index/tourn/fields.mhtml` -- Fields of competition
- `/index/tourn/emails.mhtml` -- Tournament email archive
- `/index/tourn/past.mhtml` -- Past results for this tournament series
- `/index/tourn/judge_signups.mhtml` -- Public judge volunteer/hire signup
- `/index/tourn/book.mhtml` -- Tournament book/program
- `/index/tourn/tournament_money.mhtml` -- Public financial summary
- `/index/tourn/postings/` -- Public schematics: round postings, entry records, judge assignments, congress chambers, jpool lists
- `/index/tourn/results/` -- Public results: event results, brackets, round results, prelim tables, cumesheets, ranked lists
- `/index/tourn/updates/` -- Tournament updates: school/entry/judge following
- `/index/results/` -- Historical results archive: circuit stats, debate stats, TOC bids, NDT/CEDA points, speaker rankings, RPI, team records, qualifier lists
- `/index/register.mhtml` -- Tournament registration landing
- `/index/paradigm.mhtml` -- Global paradigm search
- `/index/manifest.mhtml` -- PWA manifest
- `/index/wsdc_calendar.mhtml` -- WSDC calendar

**Major sub-capabilities:**
- Tournament listing with caching (10-minute cache on production, keyed by country/year/state/circuit)
- Filtering by circuit, school year, US state/province, country
- Display of registration status (open/due/closed with dates and timezone)
- Online/hybrid/in-person mode indicators per tournament
- Public judge signup availability display
- Tournament search by name (limited to 150 results, sorted reverse chronologically)
- Circuit directory with tournament counts per circuit
- Circuit calendar/schedule view with academic year scoping
- Weekend/multi-site tournament support in listings
- District tournament special display and Speechwire registration note
- Tournament detail pages with sidebar navigation (tabbar)
- Public schematic/posting views (round, entry record, judge, congress, jpool)
- Public results display (event results, brackets, round-by-round, cumesheets)
- Historical cross-tournament results (circuit stats, TOC bids, speaker rankings, team records)
- Tournament updates/following system (entry, judge, school follows)
- CMS-driven about page via Webpage model
- Static help page
- PWA manifest for mobile installation

**Key entities:**
- `Tab::Tourn` -- Tournament with webname, name, dates, location, timezone, hidden flag
- `Tab::Circuit` -- Circuit with abbr, name, state, country, active flag
- `Tab::TournCircuit` -- Tournament-circuit association with approved flag
- `Tab::Weekend` -- Multi-weekend tournament support
- `Tab::Webpage` (`web/lib/Tab/Webpage.pm`) -- CMS content: title, slug, content, sidebar, published, sitewide, special, page_order, tourn, parent
- `Tab::File` (`web/lib/Tab/File.pm`) -- File attachments: label, filename, tag, published, tourn, school, entry, event, district, circuit, webpage
- `Tab::Follower` (`web/lib/Tab/Follower.pm`) -- Following subscriptions: type, person, tourn, judge, entry, school, student
- `Tab::Schedule` -- Circuit schedule entries
- `Tab::Site` -- Tournament venue sites

**Key settings/configuration:**
- Tourn settings: `closed_entry`, `nsda_nats`, `nsda_ms_nats`, `ncfl`, `nsda_district`, `nsda_district_questions`, `nsda_tabbing_software`
- Event settings: `online_mode`, `online_hybrid`
- Category settings: `public_signups`, `public_signups_deadline`, `private_signup_link`
- `tourn.hidden` -- Controls visibility in public listings

**Key reports/exports:** N/A (this domain is the public display layer, not report generation)

**Integrations/dependencies:**
- Mason caching (`$m->cache_self`) for homepage performance
- Webpage CMS model for about page content
- Follower model for tournament update subscriptions
- Links to Speechwire for districts using that platform

**Documentation sources:** Inline sidebar help text in `web/index/index.mhtml`

**Code sources:**
- `web/index/` -- All public-facing pages
- `web/index/tourn/` -- Tournament detail pages
- `web/index/tourn/postings/` -- Public schematics
- `web/index/tourn/results/` -- Public results pages
- `web/index/tourn/updates/` -- Tournament following/updates
- `web/index/results/` -- Historical results archive
- `web/index/circuit/` -- Circuit pages
- `web/lib/Tab/Webpage.pm`, `File.pm`, `Follower.pm`

**Complexity level:** Medium

**Feature frequency / criticality:** Every tournament (public pages are the primary discovery and transparency layer)

**Notes and open questions:**
- The historical results system (`web/index/results/`) is extensive with debate-specific analytics (RPI, speaker rankings, TOC bids, NDT/CEDA points) -- unclear how this data is aggregated
- The Webpage CMS is minimal (title, slug, content, sidebar) but supports parent-child hierarchy
- File attachments can be associated with tournaments, schools, entries, events, circuits, districts, webpages
- The tournament listing page has a 256-item hard limit for current/upcoming display
- Weekend model complicates the public listing logic significantly (district tournaments)

---

## 3. Paradigms and Profile Content

### Paradigms and Profile Content

**Description:** Manages judge paradigm content (free-text philosophical statements about judging preferences), paradigm search/display, judge certification/quiz display, judging record display, and user profile information. Paradigms are stored as PersonSetting entries and are publicly searchable. The system includes periodic paradigm review cycles with deadlines.

**Primary users:** Judges (authoring paradigms), coaches/competitors (searching/reading paradigms), tournament admins (paradigm reports), site admins (setting review cycles)

**Main entry points:**
- `/index/paradigm.mhtml` -- Public paradigm search and display (requires login)
- `/index/paradigm.mas` -- Paradigm content rendering component
- `/index/tourn/paradigm.mhtml` -- Tournament-specific paradigm view
- `/index/tourn/paradigms.mhtml` -- All paradigms for a tournament
- `/user/judge/paradigm_approve.mhtml` -- Paradigm review/reconfirmation flow
- `/user/login/profile.mhtml` -- User profile editing
- `/user/login/name_check.mhtml` -- Edit entry/judge record names and phonetic guides
- `/register/reports/paradigms.mhtml` -- Tournament paradigm report

**Major sub-capabilities:**
- Paradigm search by first/last name with chapter affiliation display
- Paradigm content display (stored as `paradigm` tag in PersonSetting, value type "text")
- Paradigm review cycle: `paradigm_review_start` and `paradigm_review_cutoff` TabroomSettings trigger forced reconfirmation
- Auto-redirect to paradigm approval page when paradigm is outdated (checked on login via `home.mhtml`)
- Paradigm count display (`paradigm_count` TabroomSetting)
- Configurable main text and judge-specific text on paradigm pages (`paradigm_main_text`, `paradigm_judge_text`)
- Judge certification/quiz display (PersonQuiz model with badges, labels, descriptions)
- Quiz answer viewing when `show_answers` is enabled
- Judging record tab showing historical judging assignments
- Past ratings/preferences viewing (`/user/tourn/show_past_prefs.mhtml`)
- Banned account filtering (accounts with `banned` PersonSetting excluded from search)
- Unconfirmed email filtering (accounts with `email_unconfirmed` excluded from paradigm display)
- User profile management: name, email, phone, pronouns, address, timezone, country, state, ZIP
- Pronoun disclosure to section participants via blast notifications
- "No Emails" opt-out (`no_email` on Person model)
- NSDA membership display on profile (degrees, diamonds, points, member ID)
- NSDA sync and NSDA Learn sync from profile
- Active session management (view, end remote sessions, stop push notifications)
- Password change with strength meter (jquery.complexify)
- Account deletion (`user_remove.mhtml`)
- API key display on profile for users who have one

**Key entities:**
- `Tab::Person` (`web/lib/Tab/Person.pm`) -- columns: id, email, first, middle, last, phone, site_admin, street, city, state, zip, country, postal, pronoun, no_email, tz, nsda, password, accesses, last_access, pass_timestamp
- `Tab::PersonSetting` (`web/lib/Tab/PersonSetting.pm`) -- columns: id, person, tag, value, value_date, value_text, setting; used for paradigm text, API keys, banned status, email confirmation, etc.
- `Tab::PersonQuiz` -- Quiz/certification answers linked to person
- `Tab::Quiz` -- Certification/quiz definitions with tag, label, description, badge, show_answers, circuit

**Key settings/configuration:**
- PersonSetting tags: `paradigm` (value="text", paradigm content in value_text), `banned`, `email_unconfirmed`, `api_key`, `push_notify`, `inbox_accessed`, `last_attempt`, `last_attempt_ip`, `force_password_change`, `pw_token`
- TabroomSettings: `paradigm_review_start`, `paradigm_review_cutoff`, `paradigm_count`, `paradigm_main_text`, `paradigm_judge_text`

**Key reports/exports:**
- Tournament paradigm listing (`/register/reports/paradigms.mhtml`)
- NSDA elim judge bios (`/register/reports/nsda_elim_judge_bios.mhtml`, print version)

**Integrations/dependencies:**
- NSDA API for membership data, degree/diamond display, points sync
- NSDA Learn API for course completion sync
- PersonQuiz/Quiz system for judge certifications
- OneSignal for push notification management from profile

**Documentation sources:** Inline help text in profile page sidebar, pronoun disclosure explanation, privacy notice

**Code sources:**
- `web/index/paradigm.mhtml`, `web/index/paradigm.mas` -- public paradigm search/display
- `web/index/tourn/paradigm.mhtml`, `paradigms.mhtml` -- tournament paradigm views
- `web/user/judge/paradigm_approve.mhtml` -- review cycle
- `web/user/login/profile.mhtml` -- profile editing
- `web/user/login/profile_save.mhtml` -- profile save handler
- `web/user/login/name_check.mhtml` -- name editing
- `web/lib/Tab/Person.pm`, `PersonSetting.pm`

**Complexity level:** Medium

**Feature frequency / criticality:** Most tournaments (paradigms are central to the debate community); profile management is used by every user

**Notes and open questions:**
- Paradigm review cycle mechanics: when `paradigm_review_start > ps.timestamp`, the user is forced to reconfirm their paradigm on next login
- The 1-hour cache on paradigm pages means updates are not instantly visible
- Search is limited to 75 results to prevent large result sets
- PersonSetting is a generic key-value store used for many purposes beyond paradigms (API keys, banning, email confirmation, etc.)

---

## 4. Messaging and Notifications

### Messaging and Notifications

**Description:** Handles all outbound communication from Tabroom.com including email delivery, web push notifications (OneSignal), the internal inbox system, round/pairing blasts, tournament announcements, scheduled blasts, and follower notifications. Supports email with attachments, HTML formatting, BCC batching, and configurable reply-to addresses.

**Primary users:** Tournament directors (sending blasts), judges/competitors/coaches (receiving notifications), site admins (system emails)

**Main entry points:**
- `/inbox/index.mhtml` -- Internal message inbox (AJAX-driven, polls indexcards service)
- `/funclib/send_email.mas` -- Legacy email sending via SMTP (Email::Stuffer)
- `/funclib/send_notify.mas` -- Modern notification dispatch via indexcards REST API
- `/funclib/round_blast.mas` -- Round pairing blast data collection (entries, judges, followers)
- `/funclib/blast_tabbers.mas` -- Blast to tournament tabbers
- `/funclib/blast_flips.mas` -- Coin flip notification blast
- `/funclib/blast_results.mas` -- Results notification blast
- `/funclib/push_notifications.mas` -- OneSignal web push integration
- `/funclib/push_login.mas` -- Push notification login tracking
- `/funclib/email_confirm.mas` -- Email confirmation workflow
- `/funclib/judge_reg_email.mas` -- Judge registration email
- `/api/scheduled_blasts.mhtml` -- Scheduled blast processing
- `/api/day_emails.mhtml` -- Daily email digest
- `/api/district_notices.mhtml` -- District tournament notices
- `/api/district_notices_warning.mhtml` -- District notice warnings

**Major sub-capabilities:**
- Internal inbox system polling indexcards microservice (`$Tab::indexcards_url`)
  - List messages (`/user/inbox/list` GET)
  - Mark read (`/user/inbox/markRead` POST)
  - Mark all read (`/user/inbox/markAllRead` POST)
  - Delete message (`/user/inbox/markDeleted` POST)
  - 30-second auto-refresh polling
  - Messages auto-deleted 1 week after tournament ends or 1 month if not tournament-bound
- Email delivery via SMTP (Email::Stuffer with configurable `$Tab::smtp_server` and `$Tab::smtp_port`)
  - BCC batching (10 recipients per batch)
  - HTML and plain text dual format
  - Attachment support (files, JSON)
  - Reply-to header configuration
  - "No Emails" opt-out respect (`person.no_email`)
  - Override capability for mandatory emails
  - NSDA-branded sender addresses for NSDA/district communications
  - Footer with opt-out instructions
  - Production-only sending (hostname check)
- Modern notification via indexcards REST service
  - POST to `/ext/mason/blast` with Basic auth
  - Supports: person IDs, HTML/text body, subject, sender, reply-to, tournament association, URL, CC
  - Flags: `noWeb`, `noEmail`, `noInbox`, `ignoreNoEmail`
  - Attachment support
  - Email record tracking (Tab::Email model)
- Web push notifications via OneSignal
  - OneSignal SDK integration (`OneSignalSDK.page.js`)
  - Per-session push subscription tracking (`session.push_notify`)
  - Push enable/disable via indexcards (`/user/push/enable/{id}`, `/user/push/{id}/false`)
  - Notification button on pages
  - Logout/unsubscribe flow
- Round/pairing blasts collecting emails for:
  - Entry students (via person link)
  - Judges (via person link)
  - Entry followers
  - Judge followers
  - School followers
- Tournament announcement emails
- Scheduled blast processing
- District notice system
- Email confirmation workflow with confirmation codes

**Key entities:**
- `Tab::Email` (`web/lib/Tab/Email.pm`) -- columns: id, sender, person, sender_raw, content, metadata, subject, sent_at, sent_to, hidden, circuit, tourn, timestamp
- `Tab::Follower` (`web/lib/Tab/Follower.pm`) -- subscription: type, person, tourn, judge, entry, school, student
- `Tab::Session` (`web/lib/Tab/Session.pm`) -- push_notify, push_active columns for web push

**Key settings/configuration:**
- Person: `no_email` flag, `email_unconfirmed` PersonSetting
- PersonSetting: `push_notify`, `inbox_accessed`
- Session: `push_notify` (OneSignal subscription ID), `push_active`
- Global: `$Tab::smtp_server`, `$Tab::smtp_port`, `$Tab::indexcards_url`, `$Tab::indexcards_user`, `$Tab::indexcards_key`, `$Tab::onesignal_app_id`, `$Tab::onesignal_safari_id`

**Key reports/exports:** N/A (messaging is a delivery mechanism, not a report generator)

**Integrations/dependencies:**
- **Indexcards microservice** -- REST API for modern notification dispatch and inbox management (Node.js service, separate codebase)
- **OneSignal** -- Web push notification service (OneSignalSDK.page.js)
- **SMTP** -- Email delivery via configured SMTP server (Email::Stuffer, MIME::Lite)
- **Email::Sender::Transport::SMTP** -- SMTP transport layer

**Documentation sources:** Inline welcome text in inbox ("This feature is VERY new!"), code comments in send_email.mas

**Code sources:**
- `web/inbox/index.mhtml` -- Internal inbox UI
- `web/funclib/send_email.mas` -- Legacy SMTP email
- `web/funclib/send_notify.mas` -- Indexcards REST notification
- `web/funclib/round_blast.mas` -- Round blast data
- `web/funclib/blast_tabbers.mas`, `blast_flips.mas`, `blast_results.mas` -- Specialized blasts
- `web/funclib/push_notifications.mas` -- OneSignal integration
- `web/funclib/push_login.mas` -- Push login tracking
- `web/funclib/email_confirm.mas` -- Email confirmation
- `web/api/scheduled_blasts.mhtml`, `day_emails.mhtml` -- Batch processing
- `web/lib/Tab/Email.pm`, `Follower.pm`

**Complexity level:** High

**Feature frequency / criticality:** Every tournament (round blasts are critical for operations); inbox is used by all active users

**Notes and open questions:**
- Two parallel email systems exist: legacy `send_email.mas` (direct SMTP) and modern `send_notify.mas` (indexcards REST). The codebase is mid-migration.
- The inbox system is described as "VERY new" and "a beta" in the UI text, suggesting ongoing development
- Phone/SMS text delivery is described as "no longer works" in inbox welcome text
- The indexcards service URL and credentials are configured in global Tab settings but the service itself is a separate codebase
- Follower model supports granular subscriptions (entry, judge, school, student, tourn) for targeted notifications
- Cell phone domain mapping (`cell_domains.mas` referenced but not deeply explored) was likely for SMS gateway

---

## 5. Reports and Exports (Cross-Cutting)

### Reports and Exports

**Description:** A comprehensive cross-cutting report generation system spanning registration, paneling, tabulation, and results. Reports are primarily HTML-rendered pages with print-friendly formatting, CSV exports, and PDF-like printable layouts. Reports cover financial summaries, school/entry/judge lists, ballots, schematics, results, awards, and specialized national/NCFL/district reports.

**Primary users:** Tournament directors, tab staff, registration staff, coaches, NSDA officials, NCFL officials

**Main entry points:**
- `/register/reports/` -- Registration reports (100+ reports)
- `/panel/report/` -- Paneling/schematic reports
- `/tabbing/report/` -- Tabulation/results reports
- `/panel/report/ballot/` -- Ballot printing
- `/panel/report/cards/` -- Judge/student cards
- `/panel/report/schemat/` -- Schematic views
- `/tabbing/report/ncfl/` -- NCFL-specific reports
- `/tabbing/report/ndca/` -- NDCA-specific reports
- `/tabbing/report/nsda/` -- NSDA-specific reports
- `/tabbing/report/toc/` -- TOC-specific reports

**Major sub-capabilities:**

*Registration Reports (`web/register/reports/`):*
- Financial: finance_report, finance_csv, school_balances, invoice_all, fines, payments, refund_report, refund_report_csv, diocese_finance
- School lists: school_list, school_list_csv, school_events, school_headcount (with print), school_labels, school_forms, school_memberships
- Entry lists: entries_csv, contact_list, contact_sheets, student_cards, student_contacts, student_status, multiple_entries, multiple_totals
- Judge lists: judges_csv, judge_cards, judge_card_picker, category_card
- Concession reports: concessions, concessions_totals, concessions_print
- Special event reports: hotel_counts, ada, diets, video_links, file_upload_report, release_forms
- NSDA-specific: nsda_school_status, nsda_book_count, nsda_ribbons, nsda_final_judges, nsda_supp_only, nsda_student_years, nsda_elim_judge_bios, nsda_memberships_save
- NCFL-specific: ncfl_book_data, ncfl_cards, ncfl_codes, ncfl_contact, ncfl_entry_cards, ncfl_entry_reports, ncfl_judge_cards, ncfl_judge_reports, ncfl_reports, ncfl_fines (+ csv, print)
- District: district_students, four_year_qualifiers, nats_attended, nats_bond_check, nats_book, nats_congress, nats_coachlist, nats_po, nats_nametags, nats_district_print, nats_judge_required
- Administrative: stats, timeslots, site_attendance, prefs, strikes, paradigms, questionnaire, bulk_questionnaire, problem_children, problem_contacts, the_coachless, the_codeless, shenanigans, unlinked_students, check_burdens
- Health: vaccine_check, vaccine_csv, vaccine_import, vaccine_schools, vaccine_switch
- Shipping: shipping_report, packet_assignments, packet_count, packet_invoices, packet_registrations, purchase_switch

*Panel Reports (`web/panel/report/`):*
- Schematics: postings, bigass_posting, giant_postings, half_postings, list_postings
- Ballots: print_ballots, ballot_labels, ballot_table (with subfolder for specific ballot formats)
- Judge sheets: judge_chart, judge_labels, judge_points, strike_cards, pref_experience
- Room management: rooms_master, round_report
- Congress: chamber_report, chamber_roster, congress_scoresheet, seating
- Print aids: placards, adjust_labels, pick_labels
- Online: slideshow, tabs
- Specialized: preset_draw, double_entry, nats_elim_bios
- NCFL subfolder: ncfl-specific panel reports

*Tabulation Reports (`web/tabbing/report/`):*
- Results: audit tables (audit_csv.mas, audit_print.mas, audit_table.mas), print_audit, print_pending
- Awards: awards_ceremony, awards_pickup, awards_refresh, awards_school, awards_script
- Entries: code_list, code_print, codebreaker, sweep_entries
- Sweepstakes: sweep_schools, sweep_schools_print, sweep_students, sweep_students_print, sweep_post
- Statistics: stats, score_report, event_speakers, event_speakers_csv, violations, forfeits
- Judge work: judge_work
- Congress: congress_scores, po_report
- School results: school results_print
- Specialized: round_robin_script, room_cleaning, reading, readingjudges
- Export: naudl_student_export, naudl_tourn_export, last_round_csv, raw_ballots
- NSDA/NDCA/TOC/NCFL subfolders for organization-specific reports
- Packet/pickup management: packet, pickup, pickup_switch

**Key entities:** Reports reference virtually all ORM models; key ones include Entry, Judge, School, Panel, Round, Ballot, Event, Category, Fine, Concession, Person

**Key settings/configuration:** Reports are primarily query-driven; they reference tourn_settings, event_settings, category_settings for formatting and filtering decisions

**Key reports/exports:** (This IS the reports domain -- see sub-capabilities above)

**Integrations/dependencies:**
- `/funclib/tablesorter.mas` -- Client-side table sorting/filtering
- Various `/funclib/*.mas` data gathering components
- Print CSS for formatted output
- CSV generation inline in Mason templates

**Documentation sources:** Report names are self-descriptive; some have inline help text

**Code sources:**
- `web/register/reports/` -- ~120 registration report files
- `web/panel/report/` -- ~40 panel/schematic report files plus subfolders
- `web/tabbing/report/` -- ~50 tabulation report files plus subfolders
- `web/panel/report/ballot/` -- Ballot format templates
- `web/panel/report/cards/` -- Card printing templates
- `web/panel/report/schemat/` -- Schematic views

**Complexity level:** High (due to sheer volume and variety)

**Feature frequency / criticality:** Every tournament uses some reports; financial and schematic reports are used at every tournament; specialized reports (NSDA, NCFL, NDCA, TOC) are used at specific national/circuit events

**Notes and open questions:**
- Reports are primarily server-rendered HTML with print CSS, not dedicated PDF generation
- CSV exports are generated inline rather than via a shared CSV library
- Many reports are organization-specific (NSDA nationals, NCFL, NDCA) and may contain hardcoded assumptions
- The `lcq_shit_for_burdt.mhtml` filename in registration reports suggests ad-hoc report creation for specific requests
- Audit trail reports (`audit_csv.mas`, `audit_print.mas`, `audit_table.mas`) are shared Mason components used across tabbing
- The distinction between public results (`/index/tourn/results/`) and admin results (`/tabbing/report/`) is primarily access control

---

## 6. Online/Hybrid Tournament Support

### Online/Hybrid Tournament Support

**Description:** Supports running tournaments in online, hybrid (mixed online/in-person), and asynchronous modes. Integrates with NSDA Campus (private Jitsi-based video conferencing) for live online rounds, supports pre-created room URLs for third-party platforms, and async video link submission for recorded events. Manages online room allocation, entry/judge display modes, observer access, and status tracking.

**Primary users:** Tournament directors (configuring online modes), tab staff (managing rooms), judges (entering online rooms), competitors (entering online rooms, submitting video links), coaches (observing via Campus with Observers)

**Main entry points:**
- `/setup/events/online.mhtml` -- Per-event online mode configuration
- `/setup/events/online_save.mhtml` -- Save online settings
- `/setup/rooms/nsda_campus.mhtml` -- NSDA Campus room day allocation
- `/setup/rooms/campus_admin.mhtml` -- Campus administration
- `/setup/tourn/payment.mhtml` -- Campus room purchase/pricing
- `/setup/schedule/event.mhtml` -- Event scheduling with online awareness
- `/setup/judges/tabbing.mhtml` -- Judge settings with online options
- `/index/tourn/postings/` -- Public schematics with online room links

**Major sub-capabilities:**
- Per-event online mode selection:
  - `none` -- Not online (in-person)
  - `nsda_campus` -- NSDA Campus (private Jitsi servers)
  - `nsda_campus_observers` -- NSDA Campus with observer access (not available for Congress)
  - `async` -- Asynchronous video link submission
  - `sync` -- Synchronous via pre-created room URLs (Zoom, Google Meet, etc.)
- NSDA Campus features:
  - Room usage limit per event (`campus_room_limit`)
  - Cross-section room sharing between events (`online_event_match`)
  - Entry display mode in rooms: code, code+first name, code+name, name (`online_entry_display`)
  - Judge display mode in rooms: name, code+first, code+name, code (`online_judge_display`)
  - Role prepending in room names (Judge/Entry/Tab) (`online_prepend_role`)
  - Campus room day allocation per tournament day
  - Room pricing: standard rooms vs. observer-enabled rooms (different per-day rates)
- Observer support:
  - Coaches designate observers per entry
  - Observer limits: 2 per entry in debate, 1 per entry in IE/speech
  - Not available for Congress events
- Async video mode:
  - Video link submission by entries
  - Show async links on judge ballots (`show_async_links`)
  - Show async links to other entries (`show_async_to_entries`)
  - Schedule judges synchronously for async rounds (`dumb_half_async_thing`)
- Online/in-person hybrid mode (`online_hybrid` event setting)
  - Per-entry online/in-person designation (`online_hybrid` entry setting)
  - Supported for debate, WSDC, mock trial, and speech events
- Status screen tracking:
  - Track entries or individuals (`status_by_entry`)
  - Poke judges only option (`dont_poke_entries`)
- Public schematic display of room URLs (`online_public`)
- Online support contact email and instructions per event (`online_support`, `online_instructions`)
  - Scoping: per-event, per-category, per-type, or tournament-wide
- Tournament listing indicators for online/hybrid/in-person modes
- Room count estimation based on entry counts and panel sizes

**Key entities:**
- Event settings: `online_mode`, `online_hybrid`, `online_public`, `show_async_links`, `show_async_to_entries`, `dumb_half_async_thing`, `status_by_entry`, `dont_poke_entries`, `campus_room_limit`, `online_event_match`, `online_prepend_role`, `online_entry_display`, `online_judge_display`, `online_support`, `online_instructions`
- Entry settings: `online_hybrid` (marks individual entries as online in hybrid events)
- Tourn settings: `nc_purchased`, `nco_purchased`, `nc_requested`, `nco_requested`, `tabroom_purchased`, `tabroom_requested`, `tabroom_grant`, `supp_online_hybrid`
- TabroomSetting: `pricing` (JSON with `campus_room`, `campus_room_observers`, `tabroom_entry`, `tabroom_free_threshold`)

**Key settings/configuration:** See entities above; all stored in event_setting, entry_setting, tourn_setting, or tabroom_setting tables

**Key reports/exports:**
- `/api/campus_count.mhtml` -- Campus room usage counting
- `/register/reports/video_links.mhtml` -- Video link report for async events

**Integrations/dependencies:**
- **NSDA Campus** -- Private Jitsi-based video conferencing hosted on NSDA servers
- **NSDA Store** -- Payment checkout for Campus room purchases
- **Indexcards service** -- Status screen tracking and poke notifications
- Note: Public Jitsi integration was deprecated ("Jitsi servers now require a github, Google, or Facebook login")

**Documentation sources:** Inline help text in `online.mhtml`, sidebar notes in `payment.mhtml`, deprecation notice about public Jitsi

**Code sources:**
- `web/setup/events/online.mhtml` -- Online mode configuration UI
- `web/setup/events/online_save.mhtml` -- Save handler
- `web/setup/events/setting_switch.mhtml` -- AJAX setting toggle
- `web/setup/rooms/nsda_campus.mhtml` -- Campus room allocation
- `web/setup/rooms/campus_admin.mhtml` -- Campus admin
- `web/setup/tourn/payment.mhtml` -- Payment/pricing page
- `web/index/tourn/postings/` -- Public display with room links

**Complexity level:** High

**Feature frequency / criticality:** Some tournaments (growing since COVID era); critical for those that use it

**Notes and open questions:**
- NSDA Campus is built on Jitsi but hosted privately by the NSDA, distinguishing it from the deprecated public Jitsi integration
- The `public_jitsi` and `public_jitsi_observers` modes still appear in code but are deprecated per the UI notice
- Room pricing is stored in a global `pricing` TabroomSetting as JSON, making price changes a site-admin operation
- The `dumb_half_async_thing` setting name suggests developer frustration with a specific feature request
- Hybrid mode adds significant complexity: entries can individually opt in/out of online participation
- Campus room day allocation is separate from Campus room purchase -- directors must both buy and allocate

---

## 7. API and Automation/Admin Utilities

### API and Automation/Admin Utilities

**Description:** Provides data export/import APIs, administrative automation scripts, cron-job endpoints, bulk data operations, and site-admin utilities. The API layer supports both cookie-based and API-key-based authentication. Most endpoints are administrative tools rather than a formal REST API.

**Primary users:** Site admins, NSDA staff, automated cron jobs, external integrations (NDCA, TOC, CEDA), tournament directors (data export)

**Main entry points:**
- `/api/index.mhtml` -- API landing page (mostly empty placeholder)
- `/api/login_api.mas` -- API authentication (cookie, username/password, or API key)
- `/api/download_data.mhtml` -- Tournament data export (JSON, modular: whole tournament, event, round, school, category)
- `/api/upload_data.mhtml` -- Tournament data import (JSON)
- `/api/round_csv.mhtml` -- Round data as CSV (API key auth)
- `/api/timeslot_csv.mhtml` -- Timeslot data as CSV
- `/api/timeslots.mhtml` -- Timeslot listing
- `/api/school_list.mhtml` -- School listing
- `/api/school_invoice.mhtml` -- School invoice generation
- `/api/update_invoices.mhtml` -- Batch invoice update
- `/api/payment_confirm.mhtml` -- Payment confirmation callback
- `/api/payment_settings_clear.mhtml` -- Clear payment settings

**Administrative Utilities:**
- `/api/auto_flip.mhtml` -- Automated coin flip processing
- `/api/auto_queue.mhtml` -- Automated queue processing
- `/api/autopost.mhtml` -- Automated round posting
- `/api/flip_monitor.mhtml` -- Coin flip monitoring
- `/api/scheduled_blasts.mhtml` -- Process scheduled email blasts
- `/api/day_emails.mhtml` -- Daily email processing
- `/api/current_round.mhtml` -- Current round status
- `/api/this_weekend.mhtml` -- Tournaments happening this weekend
- `/api/person_tourn.mhtml` -- Person-tournament associations
- `/api/check_judges.mhtml` -- Judge validation
- `/api/side_streaks.mhtml` -- Side assignment streak analysis
- `/api/cc_check.mhtml` -- Credit card check
- `/api/show_env.mhtml` -- Show environment (debug)

**NSDA/Organization-Specific:**
- `/api/award_ndca.mhtml` -- NDCA award calculation
- `/api/award_toc.mhtml` -- TOC award/bid calculation
- `/api/ceda_points.mhtml` -- CEDA points calculation
- `/api/diamonds.mhtml` -- NSDA diamond coach calculation
- `/api/block_diamonds.mhtml` -- Block diamond processing
- `/api/dedupe_nsda.mhtml` -- NSDA member deduplication
- `/api/emma_contacts.mhtml` -- EMMA contact export
- `/api/districts_import.mhtml` -- District tournament import
- `/api/district_dates.mhtml` -- District date management
- `/api/district_notices.mhtml` -- District notice sending
- `/api/district_notices_warning.mhtml` -- District notice warnings
- `/api/districtize_chapters.mhtml` -- Chapter district assignment
- `/api/convert_district.mhtml` -- District conversion

**Data Maintenance:**
- `/api/clean_settings.mhtml` -- Clean orphaned settings
- `/api/compress_comments.mhtml` -- Compress ballot comments
- `/api/convert_followers.mhtml` -- Convert follower records
- `/api/convert_permissions.mhtml` -- Convert permission records
- `/api/coach_persons.mhtml` -- Coach person linking
- `/api/strike_card_process.mhtml` -- Strike card processing

**Major sub-capabilities:**
- JSON data export of full tournament or subsets (events, rounds, schools, categories)
- JSON data import with round/school/category scoping
- API key authentication: stored as `api_key` PersonSetting, requires owner/tabber permission or site_admin
- Cookie-based authentication reusing web session
- Username/password authentication for API access
- CSV export for rounds and timeslots
- Automated round management (auto-flip, auto-queue, autopost)
- Scheduled blast processing for timed notifications
- Organization-specific award/points calculations (NDCA, TOC, CEDA, NSDA diamonds)
- District tournament import and management
- Data cleanup and maintenance utilities
- Environment debugging

**Key entities:**
- PersonSetting: `api_key` -- Per-user API key for authenticated access
- Permission: tag in ('owner', 'tabber') -- Required for API access to a tournament
- All tournament entities are accessible via download_data.mhtml

**Key settings/configuration:**
- PersonSetting `api_key` -- API key per user
- `$Tab::indexcards_url`, `$Tab::indexcards_user`, `$Tab::indexcards_key` -- Indexcards service credentials
- `$Tab::discourse_secret` -- Discourse SSO secret

**Key reports/exports:**
- Full tournament JSON export (`download_data.mhtml`)
- Round CSV export (`round_csv.mhtml`)
- Timeslot CSV export (`timeslot_csv.mhtml`)
- School list (`school_list.mhtml`)

**Integrations/dependencies:**
- NSDA API (district import, member dedup, diamonds, chapter management)
- NDCA award system
- TOC bid tracking
- CEDA points system
- EMMA contact system
- BluSynergy (invoice updates)
- Discourse SSO (`web/user/login/sso.mhtml` -- HMAC SHA-256 payload validation)

**Documentation sources:** Code comments in `download_data.mhtml` describing export modularity; `login_api.mas` documents auth methods

**Code sources:**
- `web/api/` -- All API and admin utility endpoints (~45 files)
- `web/api/login_api.mas` -- Authentication component
- `web/user/login/sso.mhtml` -- Discourse SSO integration

**Complexity level:** High

**Feature frequency / criticality:** API export/import used by some tournaments and integrations; admin utilities used by site admins; automated endpoints (auto_flip, autopost, scheduled_blasts) run continuously during tournaments

**Notes and open questions:**
- The API is not a formal REST API; it is a collection of Mason endpoints with mixed authentication
- `download_data.mhtml` is described as "the first exporter...that is sufficiently modular" suggesting earlier, less organized exports existed
- API key auth requires both the key and the person_id, plus owner/tabber permission on the target tournament
- The `upload_data.mhtml` import endpoint supports round-level granularity for importing results
- Many admin utilities appear to be one-off or periodic maintenance scripts (dedupe_nsda, clean_settings, compress_comments)
- The `show_env.mhtml` endpoint may be a security concern in production

---

## 8. User Accounts and Identity

### User Accounts and Identity

**Description:** Manages user registration, authentication, session management, password handling, email confirmation, NSDA account import/linking, SSO (Discourse), permission/authorization model, chapter/school associations, and the "su" (switch user) capability for site admins. The identity model is Person-centric with role-based permissions at tournament, circuit, chapter, region, and district levels.

**Primary users:** All Tabroom.com users (registration, login, profile), site admins (user management, su), tournament directors (permission management), coaches (chapter access)

**Main entry points:**
- `/user/login/login.mhtml` -- Login form
- `/user/login/login_save.mhtml` -- Login processing
- `/user/login/new_user.mhtml` -- New account registration
- `/user/login/new_user_save.mhtml` -- Account creation handler
- `/user/login/authenticate.mas` -- Cookie-based session authentication
- `/user/login/logout.mhtml` -- Logout
- `/user/login/profile.mhtml` -- Profile editing
- `/user/login/profile_save.mhtml` -- Profile save handler
- `/user/login/passwd.mhtml` -- Password change handler
- `/user/login/forgot.mhtml` -- Forgot password form
- `/user/login/forgot_send.mhtml` -- Send reset email
- `/user/login/forgot_save.mhtml` -- Process reset
- `/user/login/forgot_change.mhtml` -- Change password via reset
- `/user/login/confirm.mhtml` -- Email confirmation
- `/user/login/import_account.mhtml` -- Import from NSDA account
- `/user/login/import_chapter.mhtml` -- Import chapter from NSDA
- `/user/login/nsda.mhtml` -- NSDA sync from profile
- `/user/login/sso.mhtml` -- Discourse SSO endpoint
- `/user/login/ldap_sync.mhtml` -- LDAP sync
- `/user/login/session_rm.mhtml` -- Remote session termination
- `/user/login/setting_switch.mhtml` -- AJAX setting toggle
- `/user/login/delink.mhtml` -- Account delinking
- `/user/login/user_remove.mhtml` -- Account deletion
- `/user/login/tourn_confirm.mhtml` -- Tournament-specific confirmation
- `/user/login/name_check.mhtml` -- Name editing
- `/user/login/console.mhtml` -- Admin console
- `/user/login/test_notify.mhtml` -- Push notification test
- `/user/home.mhtml` -- Smart home redirect (judges -> panels, students -> student page, coaches -> chapter, etc.)

**Major sub-capabilities:**
- **Registration:** Email, first/middle/last name, phone, state, country, timezone, password with complexity meter (jquery.complexify). NSDA Terms & Conditions and Code of Honor consent required.
- **NSDA Account Import:** Login with NSDA credentials to create Tabroom account with same email/password. Advisors get school roster, co-coaches, and competitor import.
- **Authentication:** Cookie-based with `TabroomToken` cookie. Session key = `crypt(session_id + $Tab::string, userkey)`. Cookie attributes: httponly, secure (HTTPS), domain-scoped.
- **Session Management:**
  - Session table: id, person, userkey, ip, su, defaults (JSON), created_at, agent_data (JSON), geoip (JSON), push_notify, push_active, last_access
  - Multiple active sessions visible on profile page
  - Remote session termination
  - Agent detection (browser, OS) and GeoIP location tracking
  - ISP identification
  - Push notification per-session management
- **Password Security:**
  - SHA-512 crypt hashing (`crypt($password, '$6$'.$salt)`)
  - Complexity enforcement via jquery.complexify with banlist
  - CSRF protection via SHA-checked hidden fields (ip + day_of_year + hour + server_string)
  - Rate limiting via `last_attempt` and `last_attempt_ip` tracking
  - Force password change flag (`force_password_change`)
  - Password timestamp tracking
- **Email Confirmation:** Mandatory for coach/school contact roles. Unconfirmed users can still judge/compete but cannot register schools or send tournament communications.
- **Switch User (SU):** Site admins can impersonate other users. Session.su stores the original admin person. SU sessions prevent password changes.
- **Permission Model (Tab::Permission):**
  - Scoped to: tourn, event, category, chapter, region, district, circuit
  - Tags: `owner`, `tabber`, `checker`, `contact`, plus limited event/category-specific roles
  - Hierarchical: site_admin -> owner -> tabber -> limited (event/category specific)
  - Tournament-level: full tournament access (owner/tabber) or limited to specific events/categories
  - Non-tournament: circuit, chapter, region, district permissions
  - Created_by tracking for audit
  - Details field (JSON) for additional permission metadata
- **Smart Home Redirect:** `home.mhtml` checks in priority order:
  1. Outdated paradigm -> paradigm review
  2. Active judging panels (online ballots) -> judge panels
  3. Active student entries -> student page
  4. Chapter permissions -> chapter page
  5. Upcoming judging -> judge page
  6. Student records -> student page
  7. Fallback -> setup page
- **SSO:** Discourse SSO via HMAC SHA-256 payload validation, returning nonce with user identity
- **NSDA Integration:** Person.nsda field links to NSDA member ID. Periodic advisor access refresh (14-day interval).

**Key entities:**
- `Tab::Person` (`web/lib/Tab/Person.pm`) -- Primary identity model: id, email, first, middle, last, phone, site_admin, street, city, state, zip, country, postal, pronoun, no_email, tz, nsda, password, accesses, last_access, pass_timestamp. Methods: `setting()`, `all_settings()`, `all_permissions()`
- `Tab::Session` (`web/lib/Tab/Session.pm`) -- Session management: id, person, userkey, timestamp, ip, su, defaults (JSON), created_at, agent_data (JSON), geoip (JSON), push_notify, push_active, last_access. Methods: `default()`, `location()`, `agent()`
- `Tab::Permission` (`web/lib/Tab/Permission.pm`) -- Authorization: id, tag, person, category, event, tourn, chapter, region, district, circuit, created_by, details (JSON). Methods: `get_details()`, `set_details()`
- `Tab::PersonSetting` (`web/lib/Tab/PersonSetting.pm`) -- Generic key-value settings: id, person, tag, value, value_date, value_text, setting

**Key settings/configuration:**
- Person columns: `site_admin` (boolean), `no_email`, `nsda` (NSDA member ID), `password` (SHA-512 hash), `accesses` (login count), `tz`
- PersonSettings: `paradigm`, `banned`, `email_unconfirmed`, `api_key`, `push_notify`, `inbox_accessed`, `last_attempt`, `last_attempt_ip`, `force_password_change`, `pw_token`
- Global: `$Tab::cookie_name` (default 'TabroomToken'), `$Tab::cookie_domain`, `$Tab::url_prefix`, `$Tab::string` (server secret), `$Tab::hostname`, `$Tab::discourse_secret`, `$Tab::admin_email`

**Key reports/exports:** N/A (identity is infrastructure, not reporting)

**Integrations/dependencies:**
- **NSDA API** -- Account import, advisor access, membership data, roster sync, NSDA Learn sync
- **Discourse** -- SSO integration via HMAC SHA-256 for support.tabroom.com
- **LDAP** -- Sync capability (`ldap_sync.mhtml`)
- **OneSignal** -- Push notification subscription management per session
- **Indexcards service** -- Push notification enable/disable
- **GeoIP service** -- Session location tracking (`/funclib/session_location.mas`)
- **User-Agent parsing** -- Browser/OS detection (`/funclib/session_agent.mas`)

**Documentation sources:** Inline help text in login/registration pages, privacy notice in profile sidebar, password sharing warning, GDPR cookie consent notice

**Code sources:**
- `web/user/login/` -- All authentication, registration, profile files (~28 files)
- `web/user/home.mhtml` -- Smart home redirect logic
- `web/user/menu.mas` -- User navigation menu
- `web/lib/Tab/Person.pm` -- Person ORM with settings and permissions methods
- `web/lib/Tab/Session.pm` -- Session ORM with JSON methods
- `web/lib/Tab/Permission.pm` -- Permission ORM with details methods
- `web/lib/Tab/PersonSetting.pm` -- Generic person settings

**Complexity level:** High

**Feature frequency / criticality:** Every user (authentication is foundational); permissions are checked on every authenticated request

**Notes and open questions:**
- The CSRF protection is time-based (ip + day_of_year + hour) which means tokens are valid for up to an hour and shared across all forms on the same page
- Password hashing uses SHA-512 crypt (`$6$` prefix) which is strong but not bcrypt/argon2
- The `su` (switch user) feature stores the original admin in the session, allowing impersonation -- this is powerful but has proper safeguards (blocks password changes)
- Multiple cookie-clearing strategies in `authenticate.mas` suggest historical domain/cookie-name migration issues
- The smart home redirect in `home.mhtml` is sophisticated but creates complex redirect chains
- LDAP sync exists but its usage/scope is unclear
- Person.nsda field creates a direct coupling to the NSDA member system
- The permission model supports very granular access (per-event, per-category) but the UI for managing these is spread across tournament setup pages
