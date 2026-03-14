# Evidence-Based Release 1 Candidate Shortlist

## Purpose

This document is the first-pass Release 1 candidate shortlist derived from the current source-analysis corpus.

It is meant to answer:

- what appears necessary for a first operational ScholarComp-based tournament product,
- what appears important but deferable,
- what appears specialized enough to keep out of the first operating core unless strategy demands otherwise.

This is not a final release commitment. It is a source-grounded triage artifact.

## Source Basis

Primary inputs:

- `LIGHTWEIGHT_PARITY_MATRIX.md`
- `FIRST_PASS_CAPABILITY_MAP.md`
- `FIRST_PASS_WORKFLOW_CATALOG.md`
- `FIRST_PASS_NON_FUNCTIONAL_REQUIREMENTS.md`
- `FIRST_PASS_BUSINESS_RULES_CATALOG.md`
- `FIRST_PASS_PAIRING_AND_ROUND_GENERATION_DEEP_DIVE.md`
- `FIRST_PASS_RESULTS_AND_ADVANCEMENT_DEEP_DIVE.md`
- `FIRST_PASS_JUDGE_ROOM_ASSIGNMENT_DEEP_DIVE.md`
- `FIRST_PASS_REPORTING_PRINT_EXPORT_DEEP_DIVE.md`
- `FIRST_PASS_SPECIALIZED_GOV_BODY_AND_PROGRAM_MODES.md`

## Release 1 Framing

For this shortlist, “Release 1” means:

- a product capable of running real tournaments end to end,
- on core formats and common workflows,
- with sufficient operator trust to use live,
- without needing long-tail affiliation-specific parity on day one.

This is workflow parity, not UI parity.

## Inclusion Criteria

A capability is a Release 1 candidate if it appears to satisfy most of these:

- used by every or most tournaments
- load-bearing for live operations
- prerequisite for later tournament-day workflows
- required for operator trust
- difficult to retrofit later without architectural distortion

## Exclusion / Defer Criteria

A capability is a likely later-scope candidate if it appears to be:

- specialized to a subset of tournaments or affiliations
- mostly integration/reporting oriented rather than core operating flow
- valuable but not required to run the event
- high-cost and low-frequency relative to initial market value

## Release 1 Core Candidates

## 1. Identity, Login, and Access Control

Why it belongs:

- foundational across all roles
- the rest of the product cannot function without role-aware access

Minimum expected scope:

- login
- role-scoped access
- tournament-scoped permissions
- basic account/profile flows needed for operators, coaches, judges

## 2. Tournament Creation and Base Administration

Why it belongs:

- every workflow depends on tournament existence and baseline metadata

Minimum expected scope:

- create tournament
- core tournament metadata
- cloning or templating if it materially accelerates setup
- ownership/contact assignment

## 3. Core Configuration Surface

Why it belongs:

- Tabroom is configuration-driven
- event, round, and publication behavior depend on settings

Minimum expected scope:

- only the settings needed to operate Release 1 workflows
- typed, explicit configuration for core behaviors instead of attempting full legacy settings parity immediately

## 4. Events, Categories, Schedule, Timeslots, Sites, and Rooms

Why it belongs:

- competition structure and round execution depend on this entire cluster

Minimum expected scope:

- event/category setup
- round schedule
- timeslots
- site selection
- room inventory with basic suitability data

## 5. Registration: Schools, Entries, Students, Judges

Why it belongs:

- every tournament requires registration and staffing

Minimum expected scope:

- school/chapter participation
- entry registration
- student linkage
- judge registration
- drops / waitlist / key late-change handling

## 6. Judge Ecosystem Core

Why it belongs:

- judging supply, conflicts, and preferences are central to tournament operations

Minimum expected scope:

- judge obligations / availability basics
- judge pools
- preferences / strikes / conflicts
- paradigm visibility only if strategically necessary in first cut

## 7. Pairing / Sectioning / Chambering

Why it belongs:

- this is one of the crown-jewel business domains
- no real tournament can operate without it

Minimum expected scope:

- debate pairing
- speech sectioning
- congress chamber assignment if congress is in Release 1 scope
- manual override workflows for corrections

## 8. Judge Assignment and Room Assignment

Why it belongs:

- pairing without usable judge/room assignment is not tournament-ready

Minimum expected scope:

- constraint-aware judge assignment
- constraint-aware room assignment
- operator add/remove/swap/move flows
- basic disaster validation

## 9. Ballots, Ballot Entry, and Audit

Why it belongs:

- live competition depends on it
- trust in results depends on audit/correction paths

Minimum expected scope:

- judge-facing ballot submission
- tab-room fallback/correction entry
- audit/verification path
- support for byes/forfeits/basic anomalies

## 10. Results, Standings, and Advancement

Why it belongs:

- the product has no value without correct standings and breaks

Minimum expected scope:

- protocol-driven standings
- event-type-aware round results
- break generation
- elimination-round progression
- publication of results to appropriate audiences

## 11. Essential Reporting, Print, and Export

Why it belongs:

- live tournament operations still depend on artifacts beyond on-screen UIs

Minimum expected scope:

- schematics/postings
- ballot printing or equivalent operational output
- core standings/result exports
- key audit/status reports

## 12. Recovery and Operator Override Workflows

Why it belongs:

- source evidence shows recovery is normal, not exceptional

Minimum expected scope:

- reassign judge
- reassign room
- move panel / move entry where necessary
- disaster check / validity view

## Likely Release 1 If Target Market Demands It

These are strong candidates but depend on initial market and format scope:

## 1. Public tournament pages

Why conditional:

- useful and expected
- but can be thinner than the operator core initially if necessary

## 2. Judge paradigms / philosophy

Why conditional:

- important in debate ecosystem trust and judge selection culture
- not required to compute pairings or results

## 3. Financial basics

Why conditional:

- fees and invoices matter for many tournaments
- but scope can balloon quickly

Suggested first-cut scope if included:

- fee setup
- invoice summary
- basic surcharge/fine handling only where operationally necessary

## 4. Sweepstakes / basic awards

Why conditional:

- common enough to matter
- but full sweep logic is a secondary rule engine

Suggested first-cut scope if included:

- only the most common school/entry award outputs

## Likely Release 2+ Candidates

## 1. Full district / qualifier administration

Why later:

- high complexity
- affiliation-specific
- changes core rules enough to deserve dedicated design

## 2. Full NSDA nationals support

Why later:

- specialized judging, qualification, packet, and output logic

## 3. Full NCFL specialty workflows

Why later:

- real product value, but specialized and operationally distinct

## 4. NAUDL / Salesforce / STA interoperability

Why later:

- important where needed
- but integration-heavy and not core to generic tournament operation

## 5. Concessions, housing, and long-tail logistics

Why later:

- clearly useful
- not part of the minimal operating competition core

## 6. Practice rounds and long-tail support features

Why later:

- lower operational centrality than the tournament-day core

## Recommended Release 1 Format Strategy

A plausible evidence-based first cut is:

- debate
- speech
- optionally congress if the target market requires it immediately

Reason:

- debate and speech cover the broadest common workflows
- congress adds real complexity in chambering, PO logic, and result structure

This is not a final recommendation, but the source corpus suggests congress should be an explicit scope decision rather than assumed.

## Minimum Trust Features For Release 1

Regardless of specific scope, the following appear mandatory for operator trust:

- visible auditability
- manual correction paths
- publish/unpublish control
- reliable standings regeneration
- disaster/conflict checking
- usable print/export equivalents for key artifacts

## Notable Dependencies

The source corpus strongly suggests these dependencies:

- registration before pairing
- schedule/timeslots before valid round results
- judge ecosystem before viable judge assignment
- pairing before room/judge assignment
- ballot trust before results trust
- result trust before break trust

## Biggest Release 1 Risks

The highest-risk areas to under-scope are:

- settings/configuration depth
- judge prefs/conflicts/assignment quality
- operator recovery flows
- print/report requirements
- event-type polymorphism in ballots and results

## Decision Questions Before Locking Release 1

- Is congress in scope for the first operating release?
- Are paradigms required for initial market trust?
- Are financials required for first customers, or can they be deferred?
- Is district/qualification support strategically required early?
- Which print artifacts are mandatory for launch?

## Suggested Next Use Of This Document

This shortlist should be used later in the ScholarComp repo to:

- anchor the reuse-fitness assessment,
- define the first bounded-context mapping,
- establish the first tranche plan,
- create agent-ready Release 1 implementation briefs.
