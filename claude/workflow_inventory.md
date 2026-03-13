# Workflow Inventory

## Overview

This document catalogs the major workflow families in Tabroom, organized by operational phase. Each workflow is described at the level needed for requirements extraction, not implementation detail.

**Key convention:** Workflows marked with format tags indicate format-specific behavior:
- **[D]** = Debate-specific
- **[S]** = Speech-specific
- **[C]** = Congress-specific
- **[All]** = All formats

---

## 1. User Account and Identity Management

### 1.1 Create Account
- **Primary role:** Any user
- **Trigger:** New user visits tabroom.com
- **Steps:** Enter email, name, password → confirm email → complete profile
- **Outputs:** Person record created, session established
- **Key rules:** Email uniqueness required; password requirements enforced; `force_password_change` setting supported
- **Sources:** `web/user/login/`, `web/login.mhtml`

### 1.2 Login / Logout
- **Primary role:** Any registered user
- **Trigger:** User navigates to any authenticated page
- **Steps:** Enter email/password → session created → redirected to home or last tournament
- **Key rules:** Session stores `defaults` (JSON) including current tournament; last_access updated periodically via AJAX to indexcards
- **Sources:** `web/user/login/authenticate.mas`, `web/autohandler`

### 1.3 Link NSDA Account
- **Primary role:** Student, Judge, Coach
- **Trigger:** User wants to connect NSDA membership
- **Steps:** Enter NSDA credentials → verify identity → link person to NSDA records
- **Sources:** `web/user/nsda/`, NSDA API integration

### 1.4 Manage Profile
- **Primary role:** Any user
- **Trigger:** User edits profile settings
- **Steps:** Update name, email, phone, timezone, pronouns, paradigm, notification preferences
- **Sources:** `web/user/login/profile.mhtml`

### 1.5 Switch User (Admin)
- **Primary role:** Site admin
- **Trigger:** Admin needs to diagnose user issue
- **Steps:** Select user → session.su set to admin's person → browse as target user → return to own session
- **Sources:** `web/user/admin/su_return.mhtml`

---

## 2. Tournament Creation and Setup

### 2.1 Request / Create Tournament
- **Primary role:** Tournament director
- **Trigger:** Director wants to host a tournament
- **Steps:** Submit request with dates/name/location → admin approval (or auto-approve for known circuits) → tournament record created
- **Key rules:** Must have `owner` or `tabber` permission on a circuit, or site_admin approval
- **Sources:** `web/user/tourn/request.mhtml`, `web/setup/tourn/main.mhtml`

### 2.2 Clone Tournament
- **Primary role:** Tournament director
- **Trigger:** Director wants to reuse last year's settings
- **Steps:** Select source tournament → clone settings, events, categories, rules → adjust dates and details
- **Sources:** `web/setup/tourn/import.mhtml`

### 2.3 Configure Tournament Settings
- **Primary role:** Tournament director / Owner
- **Trigger:** After creation, before registration opens
- **Steps:** Set dates, deadlines, payment methods, messages, access permissions, contact info, web pages
- **Key settings:** Registration deadlines, judge obligations, payment configuration (Stripe/PayPal/AuthorizeNet), custom messages
- **Sources:** `web/setup/tourn/settings.mhtml`, `web/setup/tourn/settings_save.mhtml`, `web/setup/tourn/dates.mhtml`

### 2.4 Grant Tournament Access
- **Primary role:** Tournament owner
- **Trigger:** Need to add tabbers, limited access users, checkers
- **Steps:** Add person by email → select permission level (owner/tabber/checker/limited) → optionally scope to events/categories
- **Sources:** `web/setup/tourn/access.mhtml`, `web/setup/tourn/access_add.mhtml`

---

## 3. Event and Category Configuration

### 3.1 Create / Edit Events [All]
- **Primary role:** Tournament director
- **Trigger:** Setting up competition divisions
- **Steps:** Create category (Debate/Speech/Congress) → add events under category → configure event-specific settings
- **Sources:** `web/setup/events/edit.mhtml`, `web/setup/events/edit_save.mhtml`

### 3.2 Configure Event Rules / Protocol [All]
- **Primary role:** Tournament director
- **Trigger:** Need to set scoring rules, tiebreakers, ballot format
- **Steps:** Select protocol → configure scoring method, tiebreaker order, ballot fields, point ranges
- **Key rules:** Protocol determines ballot structure; tiebreaker order is configurable per event
- **Sources:** `web/setup/events/ballots.mhtml`, `web/setup/events/tabbing.mhtml`

### 3.3 Configure Registration Settings [All]
- **Primary role:** Tournament director
- **Trigger:** Need to set entry limits, fees, deadlines
- **Steps:** Set entry caps, waitlist settings, fee amounts, judge obligations, registration deadlines
- **Sources:** `web/setup/events/register.mhtml`, `web/setup/events/register_save.mhtml`

### 3.4 Configure Double Entry Rules [All]
- **Primary role:** Tournament director
- **Trigger:** Tournament allows entries in multiple events
- **Steps:** Set double-entry patterns, restrictions, scheduling conflict rules
- **Sources:** `web/setup/events/double_entry.mhtml`, `web/setup/events/double_settings_save.mhtml`

---

## 4. Schedule and Venue Management

### 4.1 Build Schedule / Timeslots
- **Primary role:** Tournament director
- **Trigger:** After events are created
- **Steps:** Create timeslots → assign rounds to timeslots → set round order/patterns
- **Sources:** `web/setup/schedule/`

### 4.2 Configure Sites and Rooms
- **Primary role:** Tournament director
- **Trigger:** Venue logistics
- **Steps:** Create sites → add rooms with capacity/quality/ADA info → create room pools → assign room pools to rounds
- **Sources:** `web/setup/rooms/`

### 4.3 Create Room Pools
- **Primary role:** Tournament director
- **Trigger:** Different events need different rooms
- **Steps:** Create pool → add rooms → assign to rounds
- **Sources:** `web/setup/rooms/` (RPool entities)

---

## 5. Registration and Roster Management

### 5.1 Register School at Tournament
- **Primary role:** Coach / Chapter admin
- **Trigger:** Coach wants to enter students
- **Steps:** Find tournament → register school → enter contact info → begin adding entries
- **Sources:** `web/user/enter/create.mhtml`, `web/user/chapter/tourn_register.mhtml`

### 5.2 Add / Edit / Drop Entries
- **Primary role:** Coach / Chapter admin
- **Trigger:** Managing competition roster
- **Steps:** Select event → add entry (select students) → edit entry details → drop if needed
- **Key rules:** Entry caps, waitlist management, deadline enforcement, hybrid/independent entry support
- **Sources:** `web/user/enter/entry.mhtml`, `web/user/enter/entry_drop.mhtml`, `web/register/entry/`

### 5.3 Manage Waitlist
- **Primary role:** Tournament director
- **Trigger:** Event reaches entry cap
- **Steps:** Review waitlist → admit or remove entries → notify coaches
- **Sources:** `web/register/school/waitlist.mhtml`, `web/register/school/waitlist_admit.mhtml`

### 5.4 Admin-Side Registration Management
- **Primary role:** Tournament director / Tabber
- **Trigger:** Need to manage all schools' registrations
- **Steps:** View all schools → edit entries/judges → override caps/deadlines → manage drops
- **Sources:** `web/register/school/`, `web/register/entry/`

### 5.5 Manage Student Roster
- **Primary role:** Coach / Chapter admin
- **Trigger:** Maintaining school roster
- **Steps:** Add/edit/remove students from chapter → link to NSDA accounts → manage eligibility
- **Sources:** `web/user/enter/students.mhtml`, `web/user/enter/student_save.mhtml`

---

## 6. Judge Management

### 6.1 Register Judges
- **Primary role:** Coach / Chapter admin
- **Trigger:** Meeting judge obligation
- **Steps:** Add judges from chapter roster → set availability/shifts → specify conflicts
- **Key rules:** Judge obligation calculation, burden tracking
- **Sources:** `web/user/enter/judges.mhtml`, `web/user/enter/judge_save.mhtml`

### 6.2 Judge Hiring / Marketplace
- **Primary role:** Coach, Tournament director
- **Trigger:** School needs more judges or has excess
- **Steps:** Post judge availability → other schools request hire → confirm/deny → exchange tracking
- **Sources:** `web/register/school/judge_hires.mhtml`, `web/user/enter/hire_*.mhtml`

### 6.3 Configure Judge Pools
- **Primary role:** Tournament director
- **Trigger:** Need to organize judges for pairing
- **Steps:** Create judge pools → assign judges to pools → assign pools to rounds
- **Sources:** `web/setup/judges/`, JPool entities

### 6.4 Manage Judge Conflicts and Strikes
- **Primary role:** Tournament director, Coach
- **Trigger:** Need to prevent biased judging
- **Steps:** Add conflicts (personal) → add strikes (school/entry-level) → review conflict matrix
- **Sources:** `web/register/judge/conflicts.mhtml`, `web/register/judge/strikes.mhtml`

---

## 7. Judge Preferences and Ratings

### 7.1 Set Tab Ratings
- **Primary role:** Tournament director
- **Trigger:** Director rates judge quality
- **Steps:** Assign quality ratings to judges → set rating tiers
- **Sources:** `web/register/judge/tab_ratings.mhtml`

### 7.2 Coach Prefs Entry
- **Primary role:** Coach
- **Trigger:** Pref sheets open before tournament
- **Steps:** View judge list → rank/tier/rate judges → submit preferences
- **Key rules:** Ordinal, tiered, or percentage-based schemes; deadlines enforced
- **Sources:** `web/user/enter/strike_cards.mhtml`, `web/user/enter/ratings/`

### 7.3 Collect / Manage Prefs (Admin)
- **Primary role:** Tournament director
- **Trigger:** Review pref submission status
- **Steps:** Monitor pref completion → fix missing prefs → generate pref reports
- **Sources:** `web/register/judge/prefs.mhtml`, `web/register/judge/pref_report.mhtml`

---

## 8. Pairing and Paneling

### 8.1 Pair a Debate Round [D]
- **Primary role:** Tabber
- **Trigger:** Round is ready to be paired
- **Steps:** Select round → run pairing algorithm (preset/power/bracket) → review → publish
- **Key rules:** Powermatching (win-loss, speaker points), side assignment (aff/neg balance), pullup handling, bye handling
- **Sources:** `web/panel/round/pair_debate.mas`, `web/panel/round/pair_powered.mas`, `web/panel/round/pair_preset.mas`, `web/panel/round/pair_bracket.mhtml`

### 8.2 Panel a Speech Round [S]
- **Primary role:** Tabber
- **Trigger:** Round is ready to be paneled
- **Steps:** Select round → run paneling algorithm (snake) → review sections → publish
- **Key rules:** Minimize repeat judges, minimize repeat competitors in same section, speaker order assignment
- **Sources:** `web/panel/round/pair_speech.mas`, `web/panel/round/snake_speech.mas`

### 8.3 Create Congress Chambers [C]
- **Primary role:** Tabber
- **Trigger:** Congress session needs chambers assigned
- **Steps:** Select round → run chamber assignment → assign presiding officers → publish
- **Key rules:** Recency tracking, chamber balance, legislation assignment
- **Sources:** `web/panel/round/pair_congress.mas`, `web/panel/round/congress_chambers.mhtml`, `web/panel/round/congress_recency.mhtml`

### 8.4 WUDC Pairing [D]
- **Primary role:** Tabber
- **Trigger:** World Universities format tournament
- **Steps:** Specialized 4-team pairing format
- **Sources:** `web/panel/round/pair_wudc.mas`

### 8.5 Assign Judges to Panels
- **Primary role:** Tabber
- **Trigger:** After pairing is created
- **Steps:** Run judge assignment algorithm → review assignments → manual adjustments → publish
- **Key rules:** Respect prefs/strikes/conflicts, burden balancing, chair assignment, mutual pref optimization
- **Sources:** `web/panel/round/debate_judge_assign.mhtml`, `web/panel/round/judges.mhtml`, `web/panel/round/manual_judges.mhtml`

### 8.6 Assign Rooms
- **Primary role:** Tabber
- **Trigger:** After pairing and judge assignment
- **Steps:** Run room assignment → review → adjust for ADA/quality needs
- **Sources:** `web/panel/round/rooms.mhtml`, `web/panel/round/manual_rooms.mhtml`

### 8.7 Manual Panel Adjustments
- **Primary role:** Tabber
- **Trigger:** Need to swap entries, judges, or rooms after pairing
- **Steps:** Move entry between panels, swap judges, change rooms, add/remove byes
- **Sources:** `web/panel/manipulate/`, `web/panel/schemat/` (move/swap operations)

### 8.8 Publish Schematics
- **Primary role:** Tabber
- **Trigger:** Pairing is finalized
- **Steps:** Publish round → blast notifications → schematics visible to coaches/judges/public
- **Key rules:** Blind mode option hides judge names; postings timing
- **Sources:** `web/panel/publish/`, `web/panel/schemat/blast.mhtml`

---

## 9. Ballot Entry and Scoring

### 9.1 Online Ballot Entry (Judge) [All]
- **Primary role:** Judge
- **Trigger:** Judge assigned to panel, round started
- **Steps:** View ballot → enter scores/ranks/decisions → confirm → submit
- **Key rules:** Format-specific ballot structure (debate: win/loss + points; speech: ranks; congress: scores + legislation votes)
- **Sources:** `web/user/enter/` (judge ballot entry path)

### 9.2 Manual Ballot Entry (Tab Room) [All]
- **Primary role:** Tabber
- **Trigger:** Paper ballots need to be entered
- **Steps:** Select panel → enter ballot data → optionally double-enter for verification → audit
- **Key rules:** Double-entry verification, audit trail (entered_by, audited_by, timestamps)
- **Sources:** `web/tabbing/` (ballot entry and audit paths)

### 9.3 Audit Ballots
- **Primary role:** Tabber
- **Trigger:** Ballots entered, need verification
- **Steps:** Review entered ballots → verify against paper → mark audited → resolve discrepancies
- **Key rules:** Audit flag on ballot, audited_by tracking
- **Sources:** `web/tabbing/status/`

### 9.4 Coin Flip Entry [D]
- **Primary role:** Judge or Checker
- **Trigger:** Debate round with coin flip for side
- **Steps:** Enter flip result → update side assignments
- **Sources:** `web/panel/schemat/flips.mhtml`, `web/user/enter/flip.mhtml`

---

## 10. Results and Breaks

### 10.1 Compute Results [All]
- **Primary role:** Tabber (system-computed)
- **Trigger:** Round ballots are complete
- **Steps:** Aggregate scores → apply tiebreakers → generate standings
- **Key rules:** Tiebreaker order is configurable; different calculation for debate (wins, speaker points) vs speech (ranks, reciprocals) vs congress
- **Sources:** `web/tabbing/results/`, `web/panel/schemat/debate_results.mas`, `web/panel/schemat/speech_results.mas`

### 10.2 Run Break Round (Elimination) [D/S]
- **Primary role:** Tabber
- **Trigger:** Prelims complete, ready for elimination rounds
- **Steps:** Set break level → apply break algorithm → generate bracket → pair elim rounds
- **Key rules:** Multiple break methods, seed placement, bye handling in brackets
- **Sources:** `web/tabbing/break/break_debate.mhtml`, `web/tabbing/break/break_speech.mhtml`, `web/tabbing/break/break_congress.mhtml`

### 10.3 Publish Results
- **Primary role:** Tabber
- **Trigger:** Results finalized
- **Steps:** Review results → publish to public → blast notifications
- **Sources:** `web/tabbing/publish/`, `web/panel/publish/`

### 10.4 NSDA Points / Qualification Reporting
- **Primary role:** Tabber / System
- **Trigger:** Tournament results are final
- **Steps:** Calculate NSDA points → report qualifications → submit to NSDA
- **Sources:** `web/tabbing/results/nsda_points.mhtml`, `web/tabbing/results/nsda_qualifiers.mhtml`

---

## 11. Sweepstakes and Awards

### 11.1 Configure Sweepstakes
- **Primary role:** Tournament director
- **Trigger:** Tournament setup
- **Steps:** Create sweep sets → define rules (point systems) → include events → set award levels
- **Sources:** `web/setup/events/` (sweep configuration)

### 11.2 Calculate Sweepstakes
- **Primary role:** Tabber
- **Trigger:** Results complete
- **Steps:** Run calculation → generate school and individual standings → determine awards
- **Sources:** `web/tabbing/results/sweep_schools.mas`, `web/tabbing/results/sweep_students.mas`, `web/tabbing/results/sweep_tourn.mas`

---

## 12. Financial Operations

### 12.1 Configure Fees
- **Primary role:** Tournament director
- **Trigger:** Tournament setup
- **Steps:** Set entry fees, judge fees, late fees, drop fees, concession items
- **Sources:** `web/setup/money/`

### 12.2 Manage Invoices
- **Primary role:** Tournament director, Coach
- **Trigger:** Registration activity
- **Steps:** Generate invoices → track payments → apply discounts → record payments
- **Sources:** `web/register/school/invoice.mhtml`, `web/user/enter/fees.mhtml`

### 12.3 Manage Fines
- **Primary role:** Tournament director
- **Trigger:** Judge obligation violations, late drops, etc.
- **Steps:** Assess fines → notify coaches → track payment/forgiveness
- **Sources:** `web/register/school/fine_add.mhtml`, `web/register/school/fine_forgive.mhtml`

### 12.4 Process Payments
- **Primary role:** Coach
- **Trigger:** Invoice due
- **Steps:** Select payment method → process via Stripe/PayPal/AuthorizeNet → record payment
- **Sources:** `web/setup/tourn/payment.mhtml`, `web/user/enter/authorizenet.mas`, `web/user/enter/paypal.mas`

### 12.5 Manage Concessions
- **Primary role:** Coach, Tournament director
- **Trigger:** Tournament offers food/merchandise
- **Steps:** Configure items → coaches place orders → track purchases
- **Sources:** `web/user/enter/concessions.mhtml`, `web/register/school/concessions.mhtml`

---

## 13. Public Pages and Communication

### 13.1 Browse / Search Tournaments
- **Primary role:** Public visitor
- **Trigger:** Looking for tournaments
- **Steps:** Search by name/date/location → view tournament info → view schedule → view results
- **Sources:** `web/index/search.mhtml`, `web/index/index.mhtml`

### 13.2 View Public Results
- **Primary role:** Public visitor
- **Trigger:** Tournament results published
- **Steps:** Find tournament → view round results → view final standings → view speaker awards
- **Sources:** `web/index/results/`, `web/index/tourn/`

### 13.3 Manage Tournament Web Pages
- **Primary role:** Tournament director
- **Trigger:** Need to publish tournament info
- **Steps:** Create/edit web pages → upload documents → set visibility
- **Sources:** `web/setup/web/`

### 13.4 Send Blast Communications
- **Primary role:** Tournament director
- **Trigger:** Need to notify participants
- **Steps:** Compose message → select recipients (all, coaches, judges, specific groups) → send via email/push
- **Sources:** `web/register/emails/`, `web/panel/schemat/blast.mhtml`

---

## 14. Mid-Tournament Corrections and Recovery

### 14.1 Swap / Move Entries Between Panels
- **Primary role:** Tabber
- **Trigger:** Error discovered, entry needs to move
- **Steps:** Select entry → move to different panel → adjust affected ballots/scores
- **Sources:** `web/panel/schemat/move_panel.mhtml`, `web/panel/schemat/move_speech.mhtml`, `web/panel/schemat/move_debate.mhtml`

### 14.2 Replace / Swap Judges
- **Primary role:** Tabber
- **Trigger:** Judge no-show or conflict discovered
- **Steps:** Remove judge from panel → assign replacement → update ballots
- **Sources:** `web/panel/schemat/judge_rm.mhtml`, `web/panel/schemat/judge_add.mhtml`, `web/panel/schemat/judge_push.mhtml`

### 14.3 Drop / Withdraw Entry Mid-Tournament
- **Primary role:** Tabber, Coach
- **Trigger:** Entry cannot continue competing
- **Steps:** Drop entry → handle current round ballot → update standings
- **Sources:** `web/register/entry/drop.mhtml`, `web/tabbing/entry/`

### 14.4 Score Correction
- **Primary role:** Tabber
- **Trigger:** Scoring error discovered
- **Steps:** Locate ballot → correct scores → re-audit → recalculate results
- **Sources:** `web/panel/schemat/score_remove.mhtml`

### 14.5 Round Disaster Check
- **Primary role:** Tabber
- **Trigger:** Something went wrong with a round
- **Steps:** Run disaster check → identify problems → apply fixes
- **Sources:** `web/panel/schemat/disaster_check.mhtml`

### 14.6 Backup and Restore
- **Primary role:** Tournament director
- **Trigger:** Need to save/restore tournament state
- **Steps:** Create backup → download → restore from backup if needed
- **Sources:** `web/setup/tourn/backups.mhtml`, `web/panel/schemat/upload_backup.mhtml`

---

## 15. District and Qualification Operations

### 15.1 Run District Tournament [All]
- **Primary role:** District admin
- **Trigger:** NSDA district qualification tournament
- **Steps:** Standard tournament flow + qualification tracking + district-specific reporting
- **Sources:** `web/register/district/`, `web/user/admin/nsda/`

### 15.2 Track Qualifications
- **Primary role:** District admin
- **Trigger:** Results finalized
- **Steps:** Determine qualifiers → report to NSDA → track advancement
- **Sources:** `web/tabbing/results/nsda_qualifiers.mhtml`

---

## 16. Online / Hybrid Tournament Support

### 16.1 Configure Online Tournament
- **Primary role:** Tournament director
- **Trigger:** Tournament will be online/hybrid
- **Steps:** Enable online mode → configure campus/video settings → set up online rooms
- **Sources:** `web/setup/events/online.mhtml`, `web/user/campus/`

### 16.2 Monitor Online Rooms
- **Primary role:** Tabber
- **Trigger:** Online tournament in progress
- **Steps:** Monitor room activity → check attendance → intervene if issues
- **Sources:** `web/user/campus/`, `campus_log` table

---

## Workflow Coverage Notes

### High-Frequency (Every Tournament)
- Tournament setup and configuration
- Event and category configuration
- School registration and entry management
- Judge registration and pool setup
- Pairing/paneling rounds
- Ballot entry and audit
- Results computation and publication

### Medium-Frequency (Most Tournaments)
- Sweepstakes calculation
- Blast communications
- Financial invoicing
- Manual panel adjustments
- Mid-tournament corrections

### Lower-Frequency (Some Tournaments)
- Judge hiring marketplace
- Concessions management
- District operations
- Online/hybrid configuration
- WUDC-format pairing
- Legislation management (Congress)

### Open Questions
- Full extent of the runoff workflow (`web/panel/round/runoff.mhtml`)
- Speaker order improvement algorithm (`web/panel/round/speaker_order_improve.mhtml`)
- Mass creation workflow (`web/panel/round/mass_create.mhtml`) — what does this batch-create?
- Seating chart workflow (`web/panel/schemat/seating_*.mhtml`) — appears to be Congress-specific
- Preset pairing workflow details and how it differs from powermatching
- The "pascal" component (`web/panel/schemat/pascal.mas`) — name suggests mathematical distribution
- TBA (to be announced) entry workflow — entries without assigned competitors
- Release upload workflow — what releases are being uploaded?
