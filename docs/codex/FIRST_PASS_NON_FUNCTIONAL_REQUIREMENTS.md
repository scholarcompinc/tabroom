# First-Pass Non-Functional Requirements Catalog

## Purpose

This document captures the first-pass non-functional requirements implied by the current Tabroom product shape and usage patterns.

These are not final performance budgets. They are early constraints that should shape later architecture and UX decisions.

## Source Basis

Primary evidence used:

- live-operations workflow structure in `panel`, `tabbing`, and coach/judge dashboards
- manual sections on paneling, printouts, disaster checking, ballot entry, auditing, and breaks
- explicit publication and ballot-status logic in user/operator surfaces
- route density in report and print-oriented areas
- room accessibility and operational report handling

## 1. Live Operations Latency

Observed implication:

- operators appear to run pairings, judge assignment, room assignment, ballot audits, and break calculations in live tournament windows

First-pass requirement:

- critical live tournament operations must complete quickly enough to avoid delaying rounds

Applies especially to:

- pairing a round
- assigning judges
- assigning rooms
- loading schematic/manipulation views
- computing breaks and results

## 2. Real-Time or Near-Real-Time Visibility

Observed implication:

- coaches, judges, and operators watch pairings, ballot status, and publication state closely
- coach dashboards and operational status screens suggest multi-viewer live state consumption

First-pass requirement:

- the product needs timely propagation of round/pairing/ballot state changes to multiple viewer types

Likely critical surfaces:

- judge assignment/ballot views
- coach entry dashboards
- tab room status views
- published pairings/results

## 3. Publication Read Spikes

Observed implication:

- publication of pairings/results likely causes many viewers to load or refresh the same data at once

First-pass requirement:

- publish events should tolerate short bursts of high concurrent read activity without degrading operational control surfaces

## 4. Ballot Entry Reliability

Observed implication:

- judge ballot entry is central and likely often mobile
- tab room correction/audit flows exist, implying ballot submission failures or inconsistencies matter

First-pass requirement:

- ballot entry must minimize the risk of data loss and support recovery when submission is interrupted or incomplete

## 5. Degraded-Network Tolerance

Observed implication:

- tournaments occur in school buildings and physical venues where connectivity can be uneven
- legacy server-rendered behavior may have hidden some frontend fragility that a modern client-heavy UI would expose

First-pass requirement:

- critical workflows should degrade gracefully under unreliable or slow connections

Most sensitive likely workflows:

- judge ballot entry
- coach pairing/result visibility
- operator correction flows

## 6. Print and Document Operational Support

Observed implication:

- reports/printouts are spread throughout registration, panel, and tabbing surfaces
- the manual explicitly calls out printouts as part of live operations

First-pass requirement:

- the product must support operationally useful printable/exportable artifacts, not just on-screen views

Likely mandatory outputs:

- postings/pairings
- ballots
- registration packets/coversheets
- result/award sheets
- invoices and key reports

## 7. Accessibility and Accommodation Support

Observed implication:

- ADA-related fields and reports are explicitly present across entries, judges, rooms, room assignment, and disaster checks

First-pass requirement:

- accessibility and accommodation constraints must be treated as functional and operational requirements, not decorative metadata

## 8. Recoverability and Correctability

Observed implication:

- disaster checks, manual manipulation screens, and drop/change workflows are pervasive

First-pass requirement:

- the system must support correction and recovery during live tournament operations without forcing full rebuilds of state

## 9. Auditability

Observed implication:

- explicit audit flags, change tracking, dropped-by/dropped-at settings, and change-log-like entities are visible

First-pass requirement:

- critical operational state changes should be attributable and reviewable

Most important areas:

- ballot audit/corrections
- drops and registration changes
- publication actions
- manual operator overrides

## 10. Role-Appropriate Surface Performance

Observed implication:

- operator screens are dense and likely high-frequency
- coach/judge/public screens are simpler but latency-sensitive

First-pass requirement:

- performance expectations differ by user surface:
  - PMC/operator flows need efficient high-density data handling
  - judge/coach/public flows need fast load and refresh for time-sensitive views

## 11. Format-Aware Behavior

Observed implication:

- debate, speech, and congress appear to have different ballot, pairing, and result semantics

First-pass requirement:

- non-functional design should avoid assuming one interaction pattern fits all formats

Examples:

- ballot UX and validation may differ
- results computation cost may differ
- publication timing/visibility may differ

## 12. Reporting and Export Usability

Observed implication:

- large report surface likely exists because operators depend on exports and printed artifacts

First-pass requirement:

- exports and reports should be operationally useful, not just technically available

## Open Questions

- Which print outputs are absolutely mandatory for Release 1?
- Which workflows need near-real-time updates versus ordinary refresh?
- Is judge ballot entry expected to work well on poor mobile connections from day one?
- What performance thresholds do operators consider acceptable in live use?
- Which reports are mission-critical versus nice-to-have?

## Follow-On Work

1. Tie non-functional expectations to specific workflows in the workflow catalog
2. Add rough performance targets once domain-expert input exists
3. Separate Release 1 non-functional minimums from later hardening goals
