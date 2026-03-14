# Capability Map

## Purpose

This document defines the top-level capability domains and their key sub-capabilities, organized to show the product shape at a glance. It converts the Phase 1 inventory into a structured map suitable for release scoping, service boundary discussion, and ScholarComp mapping.

---

## Domain 1: Identity and Access

### 1.1 User Accounts
- Account creation and email verification
- Login/logout and session management
- Password management and forced reset
- Profile management (name, contact, timezone, pronouns)
- NSDA account linking

### 1.2 Permissions and Authorization
- Tournament-scoped roles (owner, tabber, limited, checker, contact)
- Entity-scoped permissions (event, category, chapter, circuit, region, district)
- Site administrator and switch-user (SU) capability
- Permission cascade and inheritance rules

### 1.3 Organization Membership
- Chapter (school) management and roster
- Circuit membership and administration
- Region and district affiliation
- Cross-organization permissions

**Release 1 candidate:** Yes (core)
**Criticality:** Every tournament
**ScholarComp reuse:** Full (Auth Service, User Service, RBAC Service)
**Overlaps:** Permission model is tournament-specific; RBAC needs domain extension

---

## Domain 2: Tournament Lifecycle

### 2.1 Tournament Creation
- Tournament request and approval
- Tournament cloning from templates
- Circuit association
- Tournament type designation (regular, district, nationals, NCFL)

### 2.2 Tournament Configuration
- General settings (dates, location, contacts)
- Registration settings (caps, deadlines, waitlist)
- Financial settings (fees, payment methods, fines)
- Display settings (codes, labels, branding)
- Communication settings (messages, disclaimers, instructions)

### 2.3 Event and Category Setup
- Category creation (Debate/Speech/Congress)
- Event creation with type and format
- Protocol/rules configuration (scoring, tiebreakers, ballot format)
- Double-entry rules
- Registration parameters per event

### 2.4 Schedule Management
- Timeslot creation and management
- Round scheduling and ordering
- Pattern templates
- Multi-weekend support
- Schedule conflict detection

### 2.5 Venue Management
- Site and room CRUD
- Room pools and round assignment
- Room quality and ADA attributes
- Room strikes and constraints

### 2.6 Tournament Access Control
- Grant/revoke tournament permissions
- Scope permissions to events/categories
- Tournament contact designation

**Release 1 candidate:** Yes (core)
**Criticality:** Every tournament
**Complexity:** High (193 tournament settings, 231 event settings, 145 category settings)
**ScholarComp reuse:** Partial (Competition Service for structure; new for tournament-specific config)

---

## Domain 3: Registration

### 3.1 School Registration
- School-at-tournament registration (chapter → school)
- Contact and address management
- School code assignment

### 3.2 Entry Management
- Add/edit/drop entries
- Student assignment to entries (individual or team)
- Entry code generation (15+ styles)
- Entry status management (active, waitlisted, dropped)
- Hybrid and independent entries
- TBA (placeholder) entries
- Maverick (incomplete team) entries

### 3.3 Waitlist Management
- Waitlist enforcement based on caps
- Priority ranking
- Admission and notification
- Waitlist-to-active transitions

### 3.4 Student Roster
- Chapter-level student roster
- Student-to-person linking
- NSDA membership verification
- Eligibility tracking

### 3.5 Registration Deadlines
- Multi-deadline enforcement (registration, freeze, drop, supplemental)
- Percentage-based fee deadlines
- Per-event deadline overrides

**Release 1 candidate:** Yes (core)
**Criticality:** Every tournament
**ScholarComp reuse:** Partial (Registration Service, Participant Service)

---

## Domain 4: Judging

### 4.1 Judge Registration
- Judge-at-tournament registration (chapter_judge → judge)
- Judge pool assignment
- Judge availability/shifts

### 4.2 Judge Obligation and Burden
- Obligation calculation from entry count (10+ adjustment factors)
- Burden tracking across rounds
- Obligation fulfillment monitoring

### 4.3 Judge Hiring Marketplace
- Entry-based hiring (school offers judges)
- Judge-based hiring (judges offer availability)
- Round-based hiring
- Exchange/swap system
- Public judge signups

### 4.4 Judge Conflicts and Strikes
- Person-level conflicts (bidirectional, auto-propagating)
- Tournament-level strikes (7+ types: entry, school, time, event, region, etc.)
- Strike card management
- Conflict detection and enforcement

### 4.5 Judge Preferences (Prefs)
- Ordinal ranking system
- Tiered rating system
- Percentage-based system
- Mutual Judge Preference (MJP) calculation
- Pref completion tracking and reporting
- Tab director quality ratings

### 4.6 Judge Pools
- Pool creation and membership
- Pool-to-round assignment
- Pool-level settings

**Release 1 candidate:** Yes (core — except hiring marketplace R2)
**Criticality:** Every tournament (prefs for most debate tournaments)
**Complexity:** Very High (Strike entity overloaded; prefs algorithms complex)
**ScholarComp reuse:** New (entirely domain-specific)

---

## Domain 5: Pairing and Paneling

### 5.1 Debate Pairing
- Powermatching (bracket-based with penalty system)
- Preset pairing (pre-determined matchups)
- Round robin pairing
- Bracket/elimination pairing
- WUDC 4-team format pairing

### 5.2 Speech Paneling
- Snake algorithm for section assignment
- Section size balancing
- Repeat competitor avoidance
- Repeat judge avoidance

### 5.3 Congress Chambering
- Chamber assignment and balancing
- Recency tracking across sessions
- Legislation assignment
- Presiding Officer designation

### 5.4 Side Assignment (Debate)
- Serpentine side assignment
- Random assignment
- Manual assignment
- Side lock in eliminations
- Coin flip management

### 5.5 Judge Assignment
- Constraint satisfaction (prefs, strikes, conflicts, burden)
- Multi-dimensional scoring (15+ factors)
- Chair designation
- 30-iteration random-start optimization
- Manual override and adjustment

### 5.6 Room Assignment
- Pool-based allocation
- Quality/ADA matching
- Manual override

### 5.7 Panel Manipulation
- Entry swaps between panels
- Judge replacement
- Room changes
- Bye management
- Flight assignment

### 5.8 Schematic Publication
- Publish/unpublish controls
- Blast notification triggers
- Blind mode (hide judge names)
- Public posting pages

**Release 1 candidate:** Yes (core — except WUDC R3+)
**Criticality:** Every tournament
**Complexity:** Very High (largest algorithmic surface)
**ScholarComp reuse:** New (entirely domain-specific)

---

## Domain 6: Ballots and Scoring

### 6.1 Ballot Entry
- Online judge ballot entry (format-specific UI)
- Tab room staff entry
- Ballot start/complete lifecycle
- Format-specific scoring (debate: win/loss + points; speech: ranks; congress: scores + PO)

### 6.2 Ballot Validation
- Score range enforcement
- Low-point win detection
- Rank consistency checks
- Required field enforcement

### 6.3 Audit
- Double-entry verification
- Audit trail (entered_by, audited_by, timestamps)
- Score correction with change logging
- Audit status dashboard

### 6.4 Real-Time Status
- Round completion monitoring
- Outstanding ballot tracking
- Judge start/submission tracking
- Auto-queue for scheduled operations

**Release 1 candidate:** Yes (core)
**Criticality:** Every tournament
**Complexity:** High
**ScholarComp reuse:** New

---

## Domain 7: Results and Publication

### 7.1 Results Computation
- Tiebreaker engine (30+ metrics, composite protocols)
- High/low dropping and truncation
- Format-specific calculation (debate/speech/congress)
- Result set generation (8 types)

### 7.2 Break/Advancement
- Debate elimination brackets (single/double)
- Speech section-based breaks
- Congress chamber-based breaks
- Seed placement and bracket construction

### 7.3 Results Publication
- Three-tier visibility (primary/secondary/feedback)
- Per-result-set publication control
- Public results pages
- Anonymous/blind publication options

### 7.4 Speaker Awards
- Individual performance rankings
- Outstanding speaker determination
- Top novice awards
- Honorable mentions

### 7.5 Sweepstakes
- Sweep rule configuration (15+ rule types)
- School-level calculation
- Individual-level calculation
- Recursive sweep set composition
- Circuit-level season awards

**Release 1 candidate:** Yes (core — sweepstakes R1?)
**Criticality:** Every tournament
**Complexity:** Very High (tiebreaker engine is 4,069 lines)
**ScholarComp reuse:** New

---

## Domain 8: Financial Operations

### 8.1 Fee Configuration
- Entry fees, judge fees, late fees, drop fees
- Per-person and per-student fees
- Fee schedules with deadlines

### 8.2 Invoice Management
- Automatic invoice generation
- Manual payment recording
- Discount application
- Invoice printing

### 8.3 Fine Management
- Judge obligation fines
- Late drop fines
- Forfeit fines with multipliers
- Fine forgiveness

### 8.4 Payment Processing
- Stripe integration (target — replacing AuthorizeNet + PayPal)
- NSDA TMoney integration
- Payment confirmation and recording

### 8.5 Concessions
- Item catalog management
- Order placement and tracking
- Concession invoicing

**Release 1 candidate:** Yes (core fees/invoicing; concessions R2)
**Criticality:** Most tournaments
**ScholarComp reuse:** Full (Payment Service), Partial (Billing Service)

---

## Domain 9: Communication and Notifications

### 9.1 Email Blasts
- Recipient selection (all, coaches, judges, specific groups)
- Template composition
- Delivery tracking

### 9.2 Push Notifications
- Round posting notifications
- Result publication notifications
- Follower notifications

### 9.3 Tournament Following
- Follow/unfollow tournaments
- Entry-level following
- Notification preferences

### 9.4 Internal Messaging
- Inbox system
- Person-to-person messaging

**Release 1 candidate:** R1 (email blasts), R1? (push), R2 (following, messaging)
**Criticality:** Most tournaments
**ScholarComp reuse:** Partial (Notification Service)

---

## Domain 10: Public Discovery and Content

### 10.1 Tournament Discovery
- Search by name, date, location, circuit
- Tournament listing with filters
- Schedule/calendar views

### 10.2 Tournament Public Pages
- Tournament info, events, schedule
- Registered schools and entries
- Published schematics/postings
- Published results

### 10.3 Paradigms
- Judge paradigm authoring
- Paradigm search and display
- Judging record display

### 10.4 Circuit Directory
- Circuit listing and search
- Circuit tournament history
- Circuit results aggregation

**Release 1 candidate:** R1 (search, tournament pages, results), R2 (paradigms, circuit directory)
**Criticality:** Every tournament (public pages), Some (paradigms, circuits)
**ScholarComp reuse:** Partial (Search Service, Public API)

---

## Domain 11: Reports and Exports

### 11.1 Registration Reports
- Entry lists, school rosters, contact sheets
- Financial summaries
- On-site registration packets

### 11.2 Pairing/Schematic Reports
- Round postings (print-optimized)
- Printed ballots (format-specific)
- Tab cards and operational reports

### 11.3 Results Reports
- Final standings
- Speaker awards
- Award certificates
- CSV exports

### 11.4 Administrative Reports
- Judge utilization and burden
- Pref reports
- Audit reports

### 11.5 PDF Generation
- LaTeX-based PDF generation
- Print-optimized layouts

**Release 1 candidate:** R1 (core reports), R1? (PDF generation strategy)
**Criticality:** Every tournament
**ScholarComp reuse:** New
**Note:** 190+ report surfaces in legacy; prioritize most-used for R1

---

## Domain 12: Online/Hybrid Tournament Support

### 12.1 Online Mode Configuration
- Per-event online mode settings
- Video platform integration
- Online room allocation

### 12.2 Hybrid Support
- Per-entry online/in-person designation
- Mixed-mode pairing constraints

### 12.3 Online Monitoring
- Room activity tracking
- Attendance verification
- Observer access control

**Release 1 candidate:** R2
**Criticality:** Some tournaments (growing post-COVID)
**ScholarComp reuse:** New

---

## Domain 13: District and Qualification (NSDA-Specific)

### 13.1 District Tournament Operations
- District tournament setup wizard
- Qualification tracking
- Auto-qualification triggers

### 13.2 NSDA Points and Reporting
- Points calculation
- Qualifier reporting
- Member verification

### 13.3 NSDA Integration API
- 46 funclib files
- Bidirectional data flow
- Member/roster synchronization

**Release 1 candidate:** Defer (NSDA-specific; evaluate for Release 2+)
**Criticality:** Critical for NSDA tournaments, N/A for independent use
**ScholarComp reuse:** New

---

## Cross-Cutting Concerns

| Concern | Applies To | Release | Notes |
|---------|-----------|---------|-------|
| Format polymorphism (debate/speech/congress) | Domains 5, 6, 7 | R1 | Must be a first-class architectural concept |
| EAV settings migration | Domains 2, 3, 4 | R1 | 570+ tags → strongly typed |
| Audit trail | Domains 3, 6, 7 | R1 | Change tracking across entities |
| Real-time updates | Domains 5, 6 | R1? | Live ballot status, schematic updates |
| Print/PDF | Domain 11 | R1 | Critical for day-of operations |
| Multi-tenancy | All | R1 | Tournament isolation, cross-org access |
| Offline resilience | Domains 5, 6 | R2 | Degraded network at tournament venues |
