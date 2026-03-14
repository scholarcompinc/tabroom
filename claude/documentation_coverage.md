# Documentation Coverage Assessment

## Overview

Documentation quality varies dramatically across Tabroom's capability domains. The strongest documentation exists for user-facing registration and setup workflows (via docs.tabroom.com). The weakest areas are the algorithmically complex backend operations (pairing, tiebreaking, sweepstakes) where behavior is primarily encoded in code with minimal documentation.

## Assessment by Domain

| Domain | Best Sources | Quality | Obvious Gaps | Confidence |
|--------|-------------|---------|-------------|------------|
| User accounts and identity | `web/user/login/`, `web/user/home.mhtml` | Moderate | Password reset flow details, NSDA account linking, session management internals | Medium |
| Tournament creation and administration | docs.tabroom.com "Running a Tournament", `web/setup/tourn/` | Strong | Tournament cloning logic, request/approval workflow internals | High |
| Tournament settings and configuration | docs.tabroom.com settings pages, `web/setup/` | Moderate | Many settings undocumented; EAV tags are not enumerated anywhere; settings interactions unclear | Medium |
| Schedule, rounds, and timeslots | docs.tabroom.com scheduling guide, `web/setup/schedule/` | Moderate | Multi-weekend scheduling, pattern templates, timeslot conflict detection | Medium |
| Sites and rooms | docs.tabroom.com, `web/setup/rooms/` | Strong | Room quality/ADA matching algorithm, online room integration | High |
| Events, divisions, and categories | docs.tabroom.com event setup, `web/setup/events/` | Strong | Category-level format polymorphism details, speech vs debate vs congress behavioral differences | High |
| School/chapter/circuit/region admin | `web/user/chapter/`, `web/user/circuit/` | Moderate | Circuit membership rules, chapter merge logic, cross-circuit permissions | Medium |
| Registration and roster management | docs.tabroom.com registration guide, `web/register/` | Strong | Waitlist logic, hybrid/independent entry handling, entry code validation | High |
| Judges and judge hiring | docs.tabroom.com judge guide, `web/register/judge/`, `web/setup/judges/` | Moderate | Judge hire marketplace, obligation calculation, burden algorithm | Medium |
| Judge preferences, conflicts, and strikes | docs.tabroom.com prefs guide, `web/funclib/category_ratings.mas` | Moderate | Prefs algorithm weights, mutual pref calculation, ordinal vs tiered vs percentage schemes | Medium |
| Pairing, paneling, and schematics | tournament-manual.pdf (partial), `web/panel/` | Weak | Powermatching algorithm, speech paneling algorithm, congress grouping, side assignment rules — all primarily code-only | Low |
| Ballots, scoring, and audits | docs.tabroom.com ballot entry, `web/user/enter/`, `web/tabbing/` | Moderate | Double-entry validation, audit trail, ballot correction flows, online ballot reliability features | Medium |
| Results, breaks, tiebreakers, publication | tournament-manual.pdf tiebreaker math, `web/tabbing/results/`, `web/tabbing/break/` | Weak | Tiebreaker calculation details, break algorithm, advancement pipeline, many undocumented tiebreaker options | Low |
| Districts, qualifications, advancement | `web/register/district/`, `web/user/admin/nsda/` | Weak | Qualification rules, auto-qualification triggers, NSDA reporting integration | Low |
| Sweepstakes and awards | tournament-manual.pdf sweepstakes section, `web/setup/events/` (sweep setup) | Weak | Sweepstakes calculation algorithm, point systems, inclusion/exclusion rules — mostly code-only | Low |
| Financials, invoices, and fines | docs.tabroom.com fees, `web/setup/money/`, `web/register/` | Moderate | Payment processing integration, invoice generation, fine calculation, refund handling | Medium |
| Public pages and discovery | `web/index/` | Moderate | Tournament search/discovery algorithm, SEO considerations, public results display rules | Medium |
| Paradigms and profile content | `web/index/paradigm.mhtml`, `web/user/` | Moderate | Paradigm structure, required vs optional fields, paradigm display rules | Medium |
| Messaging and notifications | `web/inbox/`, `web/funclib/blast_*.mas` | Weak | Email blast system, push notification setup, SMS/text capabilities, notification preferences | Low |
| Reports and exports | `web/register/reports/`, `web/panel/report/`, `web/tabbing/report/` | Weak | Full report catalog, CSV export formats, PDF generation, print layouts — scattered across many directories | Low |
| Online/hybrid tournament support | `web/user/campus/`, campus_log table | Weak | Campus/online room integration, video platform links, online tournament specific workflows, hybrid handling | Low |
| API and automation/admin utilities | `web/api/` (45 files) | Mostly code-only | No API documentation; endpoints appear to be admin scripts rather than a REST API; NSDA reporting endpoints undocumented | Very Low |

## Source Quality Summary

### Strong Sources
- **docs.tabroom.com** — Best source for user-facing workflows (registration, setup, basic operation). Well-organized by topic. Does not cover algorithmic internals.
- **tournament-manual.pdf** (2007) — Valuable for tiebreaker math, sweepstakes formulas, and elimination paneling methods. Dated but foundationally accurate for core business rules.

### Moderate Sources
- **Legacy codebase** (`web/`) — Complete source of truth for all behavior, but requires reading Perl/Mason templates to extract requirements. No inline documentation beyond variable names and occasional comments.
- **ORM models** (`web/lib/Tab/`) — Good for entity relationships and lifecycle methods. Settings methods reveal the EAV dispatch pattern.
- **SQL schema** (`doc/sql/current-schema.sql`) — Definitive for data model, constraints, and relationships. 108 tables well-structured.

### Weak Sources
- **Code comments** — Extremely sparse. Palmer's style is functional code with few comments. Occasional humor (see autohandler XSS comment, Ed Lee easter egg) but no systematic documentation.
- **API endpoints** (`web/api/`) — 45 files with no documentation, no OpenAPI spec, no consistent patterns. Mix of admin utilities, NSDA integrations, and system maintenance scripts.

## Highest-Risk Undocumented Areas

1. **Powermatching algorithm** — Core debate pairing logic; no documentation outside code
2. **Speech paneling algorithm** — Different from debate pairing; undocumented
3. **Congress grouping** — Unique format with chamber assignments; undocumented
4. **Judge assignment optimization** — Constraint satisfaction with prefs, strikes, conflicts, burden; undocumented
5. **Tiebreaker calculation** — tournament-manual.pdf covers basics but many tiebreaker options exist in code that aren't documented
6. **Break/advancement logic** — Multiple methods (bracket, double-flighting, etc.); partially documented in manual
7. **District qualification rules** — NSDA-specific business rules; not publicly documented
8. **Settings interactions** — With 19 EAV tables and hundreds of setting tags, interactions between settings are entirely undocumented

## Recommendations for Catalog Population

1. **Use docs.tabroom.com** as primary source for user-facing workflow descriptions
2. **Use tournament-manual.pdf** for business rule extraction (tiebreakers, sweepstakes, paneling methods)
3. **Use code** as primary source for algorithmic areas, settings inventory, and reports catalog
4. **Flag all algorithmic areas** for domain expert review — code-only documentation is high-risk for misinterpretation
5. **Enumerate EAV setting tags** from code usage (grep for `->setting(` patterns) as a critical inventory task
