# First-Pass Reporting, Print, and Export Deep Dive

## Purpose

This document captures a deeper first-pass analysis of the reporting, print, and export surface in the current Tabroom platform.

Its purpose is to make explicit:

- which outputs are operationally critical,
- which outputs are public-facing versus operator-facing versus integration-facing,
- where PDF/print behavior is embedded in product workflows,
- where external ecosystem integrations depend on structured exports.

This is a descriptive source-analysis artifact, not a target reporting architecture.

## Source Basis

Primary evidence used:

- `web/panel/report/index.mhtml`
- `web/panel/report/print_ballots.mhtml`
- `web/panel/report/ballot_labels.mhtml`
- `web/tabbing/report/index.mhtml`
- `web/tabbing/results/csv.mhtml`
- `web/tabbing/results/results_csv.mas`
- `web/user/admin/naudl/salesforce_tournament.mhtml`
- `web/user/admin/naudl/sections.mhtml`
- `web/user/admin/naudl/sta.mhtml`
- `web/user/admin/nsda/order_sheets.mhtml`
- broad route inventory under:
  - `web/panel/report/*`
  - `web/tabbing/report/*`
  - `web/tabbing/results/*`
  - `web/user/admin/nsda/*`
  - `web/user/admin/naudl/*`

## First-Pass Conclusions

High-confidence observations:

- Tabroom has a large reporting/print surface because tournament operations still depend on printable and distributable artifacts
- this layer is not just “analytics”; it includes ballots, schematics, postings, labels, ceremony materials, audits, and governing-body exports
- the platform uses multiple output styles: HTML tables, CSV-like exports, and LaTeX-generated PDFs
- some outputs are clearly required for ecosystem interoperability, not just internal convenience

## Output Surface Categories

The current output surface appears to break into six major groups:

1. Live operational printouts
2. Public or semi-public postings
3. Result and standings exports
4. Audit and exception reports
5. Awards and ceremony materials
6. Governing-body / external-system integration exports

## 1. Live Operational Printouts

Primary evidence:

- `panel/report/index.mhtml`
- `panel/report/print_ballots.mhtml`
- `panel/report/ballot_labels.mhtml`
- `panel/report/schematic.mhtml`
- `panel/report/judge_labels.mhtml`
- `panel/report/strike_cards.mhtml`
- `panel/report/cards/*`
- `panel/report/seating.mhtml`

Observed print families:

- schematics
- postings
- ballot printouts
- ballot labels
- strike cards
- seating charts
- placards
- judge labels
- event- and format-specific tab sheets

Implication:

- print is part of core tournament operations, not an optional reporting add-on

## 2. Public and Semi-Public Postings

Observed examples:

- postings views by round/event/timeslot
- slideshow/big posting surfaces
- giant/half/list postings
- public-facing round result views

Observed characteristics:

- these outputs appear tuned for wall posting, projector display, or public consumption
- different variants exist for different physical or display constraints

Implication:

- there is a real distinction between operator-print artifacts and audience-facing posting artifacts

## 3. Results and Standings Exports

Primary evidence:

- `tabbing/results/csv.mhtml`
- `tabbing/results/results_csv.mas`
- `tabbing/results/speakers_csv.mhtml`
- `tabbing/report/event_speakers_csv.mhtml`
- `tabbing/report/last_round_csv.mhtml`
- `panel/report/schemat/round_csv_debate.mhtml`

Observed export families:

- generic result CSV
- speaker award CSV
- round-level pairing CSV
- last-round or event-specific extract formats

Observed use cases:

- operator review
- downstream imports
- external analysis
- archival or reconciliation

Implication:

- a rebuild should assume structured export is a product requirement from early on

## 4. Audit and Exception Reports

Primary evidence:

- `tabbing/report/audit_*`
- `tabbing/report/section_audit.mhtml`
- `tabbing/report/raw_ballots.mhtml`
- `tabbing/report/forfeits.mhtml`
- `tabbing/report/violations.mhtml`
- `panel/report/disasters.mhtml`
- `tabbing/report/print_pending.mhtml`

Observed report purposes:

- checking ballot entry status
- identifying pending or incomplete work
- reviewing violations/forfeits
- surfacing disaster conditions
- auditing section-level issues

Implication:

- reporting is part of operational control and trust, not just retrospective analysis

## 5. Awards and Ceremony Materials

Primary evidence:

- `tabbing/report/awards_*`
- `tabbing/report/awards_ceremony.mhtml`
- `tabbing/report/awards_script.mhtml`
- `tabbing/report/sweep_*`
- `tabbing/report/school_results_print.mhtml`

Observed use cases:

- ceremony scripts
- award pickup and ceremony logistics
- school/entry/individual sweeps outputs
- public and print-friendly school result views

Implication:

- the output layer supports event-day ceremony execution, not just after-the-fact reporting

## 6. Governing-Body and External-System Exports

Primary evidence:

- `user/admin/naudl/salesforce_tournament.mhtml`
- `user/admin/naudl/salesforce_chapter.mhtml`
- `user/admin/naudl/sections.mhtml`
- `user/admin/naudl/sta.mhtml`
- `user/admin/naudl/sta_pairs.mhtml`
- `user/admin/nsda/order_sheets.mhtml`
- `user/admin/nsda/district_forms.mhtml`
- `tabbing/report/naudl_*`

Observed use cases:

- NAUDL / Salesforce data exchange
- STA and pair exports
- NSDA district ordering and ballot/fees workflows
- district-specific document generation

Implication:

- the platform participates in a broader ecosystem of organizations and downstream systems

## PDF / Document Generation Technology Signals

Primary evidence:

- `ballot_labels.mhtml`
- `print_ballots.mhtml`

Observed technology signals:

- LaTeX is used directly for some output generation
- files are written to temp paths and compiled into PDFs
- margin/spacing/label settings are configurable at tournament level

Observed specific document concerns:

- label layout and printer offsets
- multi-column page layout
- ballot formatting differences by event type
- incorporation of motions, resolutions, rubrics, points scales, and signatures

Implication:

- document generation is not incidental HTML printing; it is a specialized part of the product

## Ballot Printing As A Product Workflow

Primary source:

- `print_ballots.mhtml`

Observed behaviors:

- printing ballots marks tournament state (`printed_ballots`)
- ballot rendering is heavily parameterized by event and round settings
- event-specific ballot templates exist (`debate`, `speech`, `congress_student`, `wsdc`, `wudc`, `combined`)
- printed ballots include room, judge, event, round, side/order, speech times, motions, point scales, and other configured display details

Implication:

- printed ballots are a first-class operational artifact and interact with configuration deeply

## Label and Posting Generation

Primary source:

- `ballot_labels.mhtml`

Observed behaviors:

- labels are generated from live panel/judge/entry/room state
- tournament-level print calibration settings are used (`top_margin`, `left_margin`, `row_space`, `col_space`)
- label content varies by event type and special tournament mode such as NCFL

Implication:

- report/print settings are also part of the tournament configuration surface

## Reporting Menus Reflect Product Breadth

Primary sources:

- `panel/report/index.mhtml`
- `tabbing/report/index.mhtml`

Observed signal:

- the sheer breadth of menu options suggests reporting/printing is a major capability area
- reports are segmented by operational need rather than by one generic “reports” module

Implication:

- later parity planning should classify reports by usage criticality rather than treat them as one backlog bucket

## Exports As Integration Contracts

Primary evidence:

- NAUDL Salesforce exports
- STA / pairs outputs
- NSDA order sheets and district forms
- CSV outputs across results and round data

Observed pattern:

- some exports clearly exist because outside organizations or downstream tools expect a particular shape
- those shapes encode institutional relationships and workflows

Implication:

- these are proto-contracts and should be treated as integration requirements, not just convenience exports

## Print and Reporting Criticality Tiers

First-pass criticality view:

## Highest criticality

- printed ballots
- schematics/postings
- judge labels / ballot labels
- audit/violation/disaster reports
- result CSV and key standings exports

## Medium criticality

- ceremony and awards scripts
- sweeps and school result printouts
- seating charts and placards

## Specialized but important

- NAUDL/Salesforce exports
- NSDA district forms/order sheets
- specialty circuit/governing-body reports

## Highest-Risk Rebuild Requirements

The most load-bearing print/report/export requirements to preserve are:

- operational ballot printing by event type
- postings/schematics suitable for live tournament use
- audit/disaster visibility
- structured results exports
- governing-body exports where they drive real workflows
- configurable print layout controls where paper artifacts are still expected

## Recommended Next Extraction Areas

The next deeper source-analysis passes should focus on:

- exact artifact inventory by menu option
- which printed outputs are still widely used versus historical residue
- LaTeX/document-generation dependencies and shared templates
- governing-body export shapes and field mappings

## Open Questions For Later Passes

- which print artifacts are mandatory for Release 1 versus later?
- which exports represent contractual integration obligations versus optional conveniences?
- how much of the current print layer can be collapsed without harming real tournament operations?
- which organizations depend on exact field names or ordering in exported files?
