# First-Pass Capability Map

## Purpose

This document is the first normalized capability map for the current Tabroom platform.

It is derived from the existing platform catalog, workflow catalog, and parity matrix. It is meant to answer:

- what the product does at a capability level,
- which capabilities appear central versus peripheral,
- where the densest parts of the product are.

It is not yet a ScholarComp mapping.

## Top-Level Capability Domains

## 1. Identity and Access

Sub-capabilities:

- account creation and login
- profile management
- linked student/judge identities
- session and impersonation/su behavior
- permission-scoped access

Criticality:

- foundational

## 2. Tournament Administration

Sub-capabilities:

- tournament creation
- tournament cloning
- tournament ownership and contacts
- tournament metadata
- broad tournament setup

Criticality:

- foundational

## 3. Configuration and Rules

Sub-capabilities:

- tournament settings
- category/event settings
- round/pool/document settings
- scoring and publication configuration
- financial and registration configuration
- specialized district/nationals settings

Criticality:

- foundational and cross-cutting

## 4. Schedule and Competition Structure

Sub-capabilities:

- timeslots
- rounds
- flights
- sites and rooms
- room pools
- event/category structure

Criticality:

- foundational

## 5. Organizations and Affiliations

Sub-capabilities:

- schools and chapters
- circuits
- regions
- districts
- tournament affiliation structures

Criticality:

- high

## 6. Registration and Change Management

Sub-capabilities:

- school registration
- roster management
- entry registration
- judge registration
- waitlists
- drops/adds/changes
- data import/export

Criticality:

- foundational

## 7. Judge Ecosystem

Sub-capabilities:

- judge profiles and paradigms
- judge hiring and obligations
- judge pools
- prefs/conflicts/strikes
- burden and availability
- judge qualification metadata

Criticality:

- very high

## 8. Pairing, Paneling, and Assignment

Sub-capabilities:

- debate pairing
- speech paneling
- congress grouping
- judge assignment
- room assignment
- speaker order
- schematic inspection and manipulation
- validation/disaster checking

Criticality:

- very high

## 9. Ballots and Live Round Operations

Sub-capabilities:

- judge ballot submission
- tab room ballot entry
- ballot correction
- ballot audit
- outstanding ballot monitoring
- byes/forfeits/no-shows

Criticality:

- very high

## 10. Results and Advancement

Sub-capabilities:

- standings/results computation
- tiebreak execution
- break calculation
- elimination round progression
- award/sweepstakes downstream data

Criticality:

- very high

## 11. Publication and Public Consumption

Sub-capabilities:

- pairings publication
- results publication
- public tournament pages
- public results pages
- paradigm visibility
- public schedule visibility

Criticality:

- high

## 12. Reporting and Print Outputs

Sub-capabilities:

- registration reports
- operational reports
- financial reports
- CSV exports
- printouts/postings/ballots
- labels/coversheets/packets

Criticality:

- high

## 13. Financial Operations

Sub-capabilities:

- fee setup
- invoices
- fines
- surcharges/discounts
- concession-related billing

Criticality:

- medium-high

## 14. Specialized Programs and Affiliations

Sub-capabilities:

- district workflows
- qualification pipelines
- nationals/NSDA-specific processes
- bid tracking and specialized reports

Criticality:

- medium overall, high for affected tournaments

## 15. Online / Hybrid Support

Sub-capabilities:

- online room handling
- hybrid event settings
- online ballots
- campus/room usage support

Criticality:

- variable by tournament

## Capability Density and Importance

The densest and highest-risk domains appear to be:

1. configuration and rules
2. judge ecosystem
3. pairing/paneling/assignment
4. ballots and live round operations
5. results and advancement

These are the areas most likely to dominate later requirements and implementation effort.

## Release 1 Candidate Core

Based on first-pass evidence, the likely operating core is:

- identity/access
- tournament administration
- configuration and rules
- schedule and competition structure
- registration and change management
- judge ecosystem
- pairing/paneling/assignment
- ballots and live round operations
- results and advancement
- publication
- essential reporting/print outputs

Likely later-scope or tranche-dependent:

- specialized district/nationals workflows
- some online/hybrid specialization
- concessions and niche financial features
- long-tail reports and print layout tuning

## Notes

- This capability map should later be reconciled against feature frequency and parity scoring.
- It should also later be connected to the workflow catalog rather than being treated as a separate truth source.
