# First-Pass Specialized Governing-Body and Program Modes

## Purpose

This document captures a first-pass analysis of the specialized governing-body, district, circuit, and program modes embedded in the current Tabroom platform.

Its purpose is to make explicit:

- which special organizational modes materially change product behavior,
- where those modes affect core workflows rather than just branding or exports,
- which specialized capabilities likely need separate parity treatment.

This is a descriptive source-analysis artifact, not a target support matrix.

## Source Basis

Primary evidence used:

- `web/user/admin/nsda/*`
- `web/register/district/*`
- `web/tabbing/results/nsda_qualifiers.mhtml`
- `web/tabbing/results/nsda_points.mhtml`
- `web/tabbing/results/nsda_sweepstakes.mhtml`
- `web/tabbing/report/nsda/*`
- `web/user/admin/naudl/*`
- `web/funclib/nsda/*`
- `web/funclib/ncfl/*`
- `web/funclib/district_*`
- `web/funclib/judgemath/nats_*`
- earlier pairing, break, and result-generation sources that branch on NSDA/NCFL/NAUDL flags

## First-Pass Conclusions

High-confidence observations:

- specialized governing-body modes are not peripheral features
- NSDA district/nationals, NCFL, and NAUDL logic alter core registration, pairing, judging, results, reporting, and export behavior
- these modes are better understood as product variants layered onto the base tournament engine

## Major Specialized Modes Visible In The Repo

The clearest specialized modes are:

- NSDA district tournaments
- NSDA nationals / nats-adjacent workflows
- NCFL-specific operations
- NAUDL-specific exports and syncs
- TOC / special points-reporting variants
- WSDC / worlds-school related special event handling

## 1. NSDA District Mode

Primary evidence:

- `tourn_setting.nsda_district`
- `user/admin/nsda/district_forms.mhtml`
- `register/district/*`
- `funclib/district_*`
- district branches in pairing, break, and results code

Observed behaviors:

- district registration has its own surface
- district openings and deadlines are centrally controlled
- district-specific consent, email, and ballot-header content exists
- district qualifiers and alternates are generated as explicit result artifacts
- district-specific tiebreakers, awards, and judging logic exist
- district permissions include specialized `chair` and related roles

Implication:

- district mode is a real product slice, not just a reporting overlay

## 2. NSDA Nationals / National Pipeline Mode

Primary evidence:

- `nsda_nats` branches across pairing, breaking, judging, reporting, and exports
- `funclib/nsda/nats_*`
- `tabbing/report/nsda/*`
- `tabbing/results/nsda_qualifiers.mhtml`

Observed behaviors:

- nationals-related entry limits and judging checks exist
- burden/judging rules have special calculations
- finals/placement/autoqualifier reporting exists
- event ordering and qualification treatment differ
- national categories and appearances are explicit data concepts

Implication:

- nationals support changes core competition rules and qualification outputs

## 3. NCFL Mode

Primary evidence:

- `ncfl` tournament setting
- `funclib/ncfl/*`
- branches in pairing, rooming, and judging logic

Observed behaviors:

- NCFL changes region/diocese-related constraints
- NCFL-specific fees, registration printouts, invoices, and cards exist
- judging obligation logic has NCFL-specific variants
- speaker and sweeps flows have NCFL-specific outputs
- some public/posting displays vary under NCFL

Implication:

- NCFL is a domain variant that affects both logistics and competition logic

## 4. NAUDL Mode

Primary evidence:

- `circuit_setting.tag = 'naudl'`
- `user/admin/naudl/*`
- `naudl_*` reporting/export surfaces

Observed behaviors:

- NAUDL logic identifies schools through circuit/region relationships
- specialized exports target Salesforce and STA-style downstream systems
- tournament, chapter, section, and student data are reshaped for external systems
- event and speaker result data are repackaged into NAUDL-specific formats

Implication:

- NAUDL support is heavily integration-driven and likely required for ecosystem interoperability where used

## 5. WSDC / Worlds-School / Special Debate Variants

Primary evidence:

- `usa_wsdc`
- `worlds_schools`
- WSDC/WUDC branches in pairing, ballot, and results code

Observed behaviors:

- speaker and reply/refute handling differ
- qualification and event eligibility may differ
- score and publication behavior have special cases

Implication:

- these are format-level variants with governing-body adjacency, not just labels on ordinary debate

## Specialized Permissions and Administration

Observed specialized permission concepts:

- `chair`
- `wsdc`
- district scope in `permission`
- circuit scope
- region scope

Observed specialized admin surfaces:

- district chair/admin tools
- district attendance and committee tools
- NSDA-specific topic, quiz, and paradigm admin
- NAUDL chapter/tournament sync and exports

Implication:

- special modes affect who can do work, not just what gets printed

## Specialized Registration and Eligibility

Observed behaviors:

- district entry and registration flows are separate from ordinary tournament registration
- NSDA membership and advisor-access helpers exist
- school status and roster sync logic exists
- entry limits and supplementary eligibility logic exist
- district school and committee context influences participation

Implication:

- specialized modes can alter the registration model itself

## Specialized Pairing and Break Logic

Observed from earlier deep dives:

- NSDA district and nationals flags change speech/debate/congress pairing penalties and constraints
- special qualifier counts influence break generation
- district/national modes change judging requirements and chamber behavior
- some formats or divisions are excluded from normal weekend or district handling

Implication:

- specialized organizational modes modify the competition engine, not just metadata

## Specialized Results and Qualification Outputs

Primary evidence:

- `nsda_qualifiers.mhtml`
- `nsda_points.mhtml`
- `nsda_sweepstakes.mhtml`
- district awards and autoqualifier reports

Observed behaviors:

- “district qualifiers” and “final places” are explicit output modes
- alternates, qualifiers, audited results, and weekend-specific event handling matter
- some event categories have special qualification behavior

Implication:

- qualification is a product domain layered on top of results, not just a post-hoc view

## Specialized Print and Document Outputs

Observed examples:

- district forms
- NSDA order sheets
- district ballot header language
- full packet and awards reports
- NAUDL exports and external-data handoff files

Implication:

- many specialized modes are operationalized through documents and structured handoffs

## Specialized Data and Sync Behavior

Observed evidence:

- NSDA API/client and sync helpers
- chapter/school/person linking helpers
- import workflows for districts, users, and schools
- posting of points and tournament/event data outward

Implication:

- some specialized modes include system-to-system synchronization, not just manual exports

## Product-Planning Implications

These specialized modes should likely be treated in different parity tiers:

## Release-1-core candidates only if market requires them immediately

- generic tournament operation
- maybe a limited subset of district qualification if strategically necessary

## Likely Release-2+ specialized slices

- full NSDA district administration
- full NSDA nationals support
- NCFL specialty workflows
- NAUDL/Salesforce interoperability
- full awards/governing-body packet generation

This is not a final release recommendation, but the current source evidence suggests these modes are substantial enough to isolate explicitly.

## Highest-Risk Rebuild Requirements

The most load-bearing specialized-mode requirements to preserve or consciously defer are:

- district qualification logic
- national judging/burden rules where relevant
- specialized permission/admin roles
- organization-specific exports and document packages
- external sync/import/export assumptions
- event-type variants tied to governing-body rules

## Recommended Next Extraction Areas

The next deeper source-analysis passes should focus on:

- district registration workflow details
- qualifier and alternate generation rules
- NSDA sync/import/export contract surfaces
- NCFL-specific judging and fee rules
- NAUDL export field inventories

## Open Questions For Later Passes

- which specialized modes are commercially necessary in your target market?
- which specialized exports are contractual or ecosystem-critical versus optional?
- how much of the specialized-mode surface is actively used today versus retained legacy support?
- which specialized modes can be isolated cleanly from the core tournament engine?
