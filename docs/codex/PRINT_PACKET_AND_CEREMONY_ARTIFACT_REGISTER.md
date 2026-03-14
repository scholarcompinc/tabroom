# Print, Packet, and Ceremony Artifact Register

## Purpose

This document captures the first-pass register of print-oriented, packet-oriented, and ceremony-oriented artifacts visible in the current Tabroom platform.

Its purpose is to distinguish these operational artifacts from ordinary on-screen result viewing.

This is a descriptive source-analysis artifact, not a future document-generation design.

## Source Basis

Primary evidence used:

- `web/tabbing/report/index.mhtml`
- `web/tabbing/report/prelims_order.mhtml`
- `web/tabbing/report/results_table_print.mas`
- `web/tabbing/report/awards_ceremony.mhtml`
- `web/tabbing/report/awards_script.mhtml`
- `web/tabbing/report/packet.mhtml`
- `web/tabbing/report/event_speakers.mhtml`
- `web/tabbing/report/score_report.mhtml`
- `web/tabbing/report/print_audit.mhtml`
- `web/funclib/printout.mas`

## First-Pass Conclusions

High-confidence observations:

- the platform has a substantial print/report layer beyond public web results
- many tournament operations still rely on PDF/LaTeX-generated artifacts
- ceremony, audit, packet, and posting workflows have dedicated print-oriented outputs

## Artifact Families

## 1. Seeding and Standings Printouts

Observed examples:

- `prelims_order.mhtml`
- `results_table_print.mas`
- event speaker printouts

Observed purpose:

- print standings/seeding for operator reference, posting, or room distribution

## 2. Awards and Ceremony Artifacts

Observed examples:

- `awards_ceremony.mhtml`
- `awards_script.mhtml`
- school awards sheets / awards-school views

Observed purpose:

- support formal awards presentation
- produce scripts and ceremony-ready ordered outputs

## 3. Packet Artifacts

Observed examples:

- `packet.mhtml`
- NSDA full packet assembly
- audit / section-audit print flows

Observed purpose:

- bundle multiple result/report artifacts into one operational handoff package
- support tab room, judges, or governing-body packet preparation

## 4. Audit and Verification Artifacts

Observed examples:

- `print_audit.mhtml`
- `section_audit.mhtml`
- score reports

Observed purpose:

- verify correctness
- support operator review and troubleshooting
- produce paper or PDF audit references

## 5. Specialized Governing-Body Printouts

Observed examples:

- TOC bid report
- NSDA full packet
- final wins / final cumes
- sweep print variants

Observed purpose:

- satisfy external reporting or ceremony obligations
- produce formal, distributable documents

## Register

| Artifact Family | Primary Sources | Primary Users | Notes |
|---|---|---|---|
| Seeding / prelim order printouts | `prelims_order.mhtml`, `results_table_print.mas` | Operators / coaches / posting staff | Print form of standings-like outputs |
| Event speaker reports | `event_speakers.mhtml`, speaker CSV/print paths | Operators / awards staff | Speaker-award-specific print lane |
| Awards ceremony reports | `awards_ceremony.mhtml`, `awards_script.mhtml` | Ceremony staff / operators | Ceremony-focused ordered outputs |
| Awards school sheets | `awards_school.mhtml`, related pickup flows | School reps / awards desk | Supports distribution of awards to schools |
| Audit/section audit PDFs | `print_audit.mhtml`, `section_audit.mhtml` | Operators / judges / reviewers | Verification and troubleshooting artifacts |
| Full packets | `packet.mhtml`, `nsda/full_packet.mhtml` | Operators / external recipients | Composite bundles of multiple reports |
| Score reports | `score_report.mhtml` | Operators / governing-body workflows | Dense scoring reference document |
| Sweep printouts | `sweep_students_print.mhtml`, `sweep_schools_print.mhtml` | Awards/ceremony staff | Printed award summaries |

## Delivery Mechanism

Observed mechanism:

- many artifacts use `printout.mas`
- print generation relies on LaTeX / PDF workflows
- some flows merge PDFs or redirect to generated files in `/tmp`

Implication:

- document generation is a significant operational subsystem, not a thin export helper

## Why These Artifacts Matter

The source suggests these artifacts support:

- wall posting
- tab room reference
- ceremony scripting
- school pickup/distribution
- audit and verification
- external packet delivery

This is a broader operational footprint than “download final standings PDF.”

## Important Distinctions To Preserve Later

- screen-oriented result view vs print-oriented operational artifact
- single artifact report vs packet/bundle
- ceremony script vs ranking table
- audit report vs public result publication

## Open Questions For Later Validation

- which of these print artifacts are still operationally mandatory in modern tournaments
- which are mostly legacy convenience outputs
- which packet artifacts are expected by external organizations or tournament staff
