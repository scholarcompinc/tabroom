# Existing Platform Catalog

## Purpose

This document catalogs what the current Tabroom product appears to do today before any ScholarComp mapping, redesign, or service decomposition is attempted.

This is a descriptive artifact, not a target-architecture artifact.

Questions this document should answer:

- What capability areas exist?
- What roles exist?
- What workflows exist?
- What major data/configuration surfaces exist?
- What reports, exports, and integrations exist?
- Where are the strongest and weakest documentation sources?
- Which areas appear algorithmically or operationally complex?

Questions this document should not answer:

- How should we rebuild this in ScholarComp?
- Which service should own a given capability?
- Which UX should be changed or improved?

Those belong in later artifacts.

## Source Set

Primary sources for this catalog:

- legacy `tabroom` repo
- `tabroom-docs` / `docs.tabroom.com`
- `indexcards`
- `schemats`
- `tournament-manual.pdf`
- observed live behavior where available

## Cataloging Rules

- Prefer observable facts over interpretation.
- Record source references for every major claim.
- Distinguish clearly between:
  - documented behavior,
  - inferred behavior,
  - legacy-only behavior,
  - uncertain/unverified behavior.
- Do not map capabilities to ScholarComp services in this document.
- Do not collapse features just because they seem duplicative; note overlaps explicitly.

## Inventory Summary

This section should become a concise snapshot once the catalog is populated.

Suggested contents:

- major capability domains count
- primary user roles
- number of major workflow families
- major settings/configuration areas
- major reporting/export areas
- major integrations
- high-complexity algorithmic areas
- highest-risk undocumented areas

## Capability Domains

Populate one subsection per domain.

Suggested starting domains:

- User accounts and identity
- Tournament creation and administration
- Tournament settings and configuration
- Schedule, rounds, and timeslots
- Sites and rooms
- Events, divisions, and categories
- School/chapter/circuit/region administration
- Registration and roster management
- Judges and judge hiring
- Judge preferences, conflicts, and strikes
- Pairing, paneling, and schematics
- Ballots, scoring, and audits
- Results, breaks, tiebreakers, and publication
- Districts, qualifications, and advancement pipelines
- Sweepstakes and awards
- Financials, invoices, and fines
- Public pages and discovery
- Paradigms and profile content
- Messaging and notifications
- Reports and exports
- Online/hybrid tournament support
- API and automation/admin utilities

For each domain, capture:

### Domain Template

- Domain name
- Description
- Primary users
- Main entry points
- Major sub-capabilities
- Key entities
- Key settings/configuration
- Key reports/exports
- Integrations/dependencies
- Documentation sources
- Code sources
- Complexity level
- Feature frequency / criticality
- Notes and open questions

## Role Inventory

Catalog the roles that exist in the current product.

Suggested starting roles:

- Public visitor
- Student
- Judge
- Coach
- School/chapter admin
- Tournament director
- Tab room operator / tabber
- Circuit admin
- League/region/district admin
- Site admin / system admin

For each role, capture:

- Role name
- Scope of authority
- Key workflows
- Key restrictions
- Evidence sources
- Notes/open questions

## Workflow Inventory

Catalog workflows before turning them into detailed requirements.

Suggested starting workflow families:

- Create and manage user accounts, profiles, and linked identities
- Request or create a tournament
- Clone/setup a tournament
- Configure events and settings
- Build schedule and rounds
- Register schools, entries, and judges
- Collect or manage prefs/strikes/conflicts
- Build judge and room pools
- Pair a round
- Assign judges and rooms
- Enter or submit ballots
- Audit ballots
- Compute results and breaks
- Publish results
- Generate awards and sweepstakes
- Produce financial reports and invoices
- Manage public pages and paradigms
- Handle mid-tournament corrections and recovery

For each workflow, capture:

- Workflow name
- Primary role
- Trigger/preconditions
- High-level steps
- Outputs/outcomes
- Key rules/settings involved
- Major exceptions/edge cases
- Sources

## Route and Surface Inventory

Document the primary visible product surfaces before redesigning them.

Suggested source breakdown:

- `web/setup`
- `web/register`
- `web/panel`
- `web/tabbing`
- `web/user`
- `web/index`
- `web/api`
- docs site categories

Capture:

- surface area name
- intended users
- purpose
- major pages or routes
- major overlaps with other surfaces
- notes on density/complexity

## Data and Entity Inventory

Catalog major entities without yet designing the new data model.

Suggested groupings:

- tournament/event structure
- user accounts and identity
- participants and organizations
- judging and conflicts
- rounds/panels/rooms
- ballots/scoring/results
- settings/configuration
- finances
- content/publication
- notifications/audit

For each entity or entity cluster, capture:

- entity name
- role in the product
- major relationships
- lifecycle significance
- source tables/models
- notes/open questions

Special note:

- treat legacy `*_setting` / EAV-style entities as a distinct cluster of the product surface, not just as incidental supporting tables; they collectively represent a large share of current configuration behavior.

## Settings and Configuration Inventory

This section should inventory existing configuration surfaces before rationalizing them.

Capture:

- setting family
- scope level:
  - site-wide
  - tournament
  - category/division
  - event
  - round
  - judge/pool
  - registration/financial/publication
- examples
- where documented
- where implemented
- whether it appears Release 1 critical

## Reports and Exports Inventory

Catalog current reporting/output surfaces.

Suggested categories:

- registration reports
- pairing/schematic printouts
- judge reports
- result reports
- award sheets
- invoices and financial reports
- CSV exports
- PDF/print views

For each item, capture:

- report/export name
- user role
- purpose
- output format
- trigger/context
- sources

## Integrations Inventory

Capture external systems and platform dependencies visible in the current product.

Suggested examples:

- email
- SMS/push
- NSDA APIs and reporting
- payment systems
- tabroom.com data import/export and tournament cloning flows
- S3/file storage
- online room/video integrations
- GeoIP/location data
- external identity or SSO features

For each integration, capture:

- integration name
- purpose
- required or optional
- user-visible impact
- source references

## Algorithmically Complex Areas

Identify areas that will require especially careful business-rules extraction.

Suggested starting list:

- debate powermatching
- speech paneling
- congress grouping
- judge assignment optimization
- speaker order assignment
- room quality / ADA matching
- burden calculation
- conflicts/strikes/prefs
- tiebreak logic
- break/advancement logic
- sweepstakes calculation
- ballot validation and audit logic

For each area, capture:

- area name
- why it is complex
- primary source files/docs
- documentation quality
- likely need for expert review

## Documentation Coverage Assessment

For each major domain, assess the current documentation quality:

- strong
- moderate
- weak
- mostly code-only

Capture:

- domain
- best sources
- obvious gaps
- confidence level

## Unknowns and Open Questions

Use this section to record questions discovered during cataloging.

Examples:

- behavior observed in code but not documented
- contradictory docs vs implementation
- domain terms with inconsistent meaning
- settings with unclear effect
- legacy features with unclear current usage

## Suggested Initial Build Order For This Catalog

1. Top-level capability domains
2. Role inventory
3. Workflow inventory
4. Settings/configuration inventory
5. Algorithmically complex areas
6. Reports/exports and integrations
7. Documentation coverage assessment

## Status

- Status: **Complete** — all sections populated across 15 artifacts in `claude/`
- Owner: Jeloni
- Last updated: 2026-03-13
- See `claude/HANDOFF_INDEX.md` for the full artifact inventory
