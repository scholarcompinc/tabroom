# Must-Preserve Workflows and Rules

## Purpose

This document captures the first-pass set of workflows and rule areas that appear too central, too risky, or too trust-sensitive to casually simplify during a rebuild.

It is not a final “copy exactly” mandate. Its purpose is to identify what must be preserved in substance even if implementation and UI change.

## Source Basis

Primary inputs:

- `EVIDENCE_BASED_RELEASE1_CANDIDATE_SHORTLIST.md`
- `FIRST_PASS_WORKFLOW_CATALOG.md`
- `FIRST_PASS_BUSINESS_RULES_CATALOG.md`
- `FIRST_PASS_PAIRING_AND_ROUND_GENERATION_DEEP_DIVE.md`
- `FIRST_PASS_RESULTS_AND_ADVANCEMENT_DEEP_DIVE.md`
- `FIRST_PASS_JUDGE_ROOM_ASSIGNMENT_DEEP_DIVE.md`
- `FIRST_PASS_NON_FUNCTIONAL_REQUIREMENTS.md`
- `SOURCE_CONFLICT_AND_AMBIGUITY_REGISTER.md`

## How To Read This Document

“Must preserve” means one of:

- the workflow must exist in Release 1,
- the rule effect must remain materially true,
- the operator trust signal must still be present,
- the product will likely fail live adoption if it is missing or wrong.

## Must-Preserve Workflow Families

## 1. Tournament Setup To Runnable State

Must preserve:

- tournament creation
- event/category setup
- schedule/timeslot creation
- site and room setup
- enough configuration to make rounds legally runnable

Why:

- everything else depends on it

## 2. Registration With Late-Change Handling

Must preserve:

- school/chapter participation
- entry registration
- student linkage
- judge registration
- drops / waitlist / key late changes

Why:

- live tournaments routinely require changes after initial registration

## 3. Judge Preference / Strike / Conflict Handling

Must preserve:

- explicit conflicts and strikes
- category/event-specific preference behavior
- judge pool interaction where required
- meaningful visibility into judge quality/fit

Why:

- this is core to ecosystem trust, especially in debate-heavy use

## 4. Automatic Pairing With Manual Recovery

Must preserve:

- debate pairing
- speech sectioning
- congress chambering if congress is in scope
- operator correction after automatic generation

Why:

- source evidence shows automatic generation alone is not enough

## 5. Judge Assignment With Constraint Awareness

Must preserve:

- same-school restrictions
- strike/conflict restrictions
- time-overlap restrictions
- pool/category fit
- ability to add/remove/swap judges manually

Why:

- this is central operational logic and a high-trust area

## 6. Room Assignment With Constraint Awareness

Must preserve:

- site/time overlap handling
- room strikes
- room suitability awareness
- manual move/reassign flows

Why:

- unusable rooming breaks the tournament even if pairing is correct

## 7. Ballot Collection, Fallback Entry, and Audit

Must preserve:

- judge-facing ballot submission
- tab-room fallback entry path
- audit/re-entry/correction path
- bye/forfeit/no-show handling

Why:

- results trust depends on this flow more than on the UI style

## 8. Results Computation And Advancement

Must preserve:

- protocol-driven standings
- tiebreak-aware ordering
- event-type-specific result handling
- break generation
- elimination-round progression

Why:

- this is the other crown-jewel domain after pairing/assignment

## 9. Publication Control

Must preserve:

- operator control over when pairings become visible
- operator control over what result detail becomes visible
- distinction between pairings publication and results publication

Why:

- the current product clearly treats publication as staged and audience-sensitive

## 10. Disaster / Recovery Workflows

Must preserve:

- disaster validation
- judge/room reassignment
- movement and correction workflows during live operations

Why:

- recovery is normal product behavior, not a rare admin edge case

## Must-Preserve Rule Effects

## 1. Entry Activity And Eligibility Separation

Must preserve the effect that:

- an entry can exist but not be active
- an entry can be active but ineligible for some downstream outcomes

Examples:

- waitlist
- unconfirmed
- dropped
- DQ
- no-elims
- sweeps exclusion

## 2. Event-Type Polymorphism

Must preserve the effect that:

- debate, speech, congress, and special formats are not one generic workflow with different labels

Why:

- ballots, pairing, results, and advancement all diverge materially

## 3. Protocol-Driven Result Logic

Must preserve the effect that:

- standings are computed from configurable rule sets and tiebreak sequences rather than one hard-coded ranking formula

## 4. Fairness Constraints In Pairing

Must preserve the effect of:

- side fairness where relevant
- repeat-hit avoidance
- same-school avoidance
- region/district constraints where active
- byes handled with fairness considerations

## 5. Fairness Constraints In Speech/Congress Placement

Must preserve the effect of:

- speaker order fairness
- section/chamber composition fairness
- special positional fairness in WUDC/WSDC-like formats

## 6. Judge Eligibility And Assignment Integrity

Must preserve the effect of:

- no illegal judge-entry pairings
- no obvious conflict/strike violations
- no impossible simultaneous assignments

## 7. Publication Gating By Detail Level

Must preserve the effect that:

- not all result data becomes visible at the same time
- primary, secondary, and feedback-level publication can differ

## 8. Auditability

Must preserve the effect that:

- critical changes and sensitive tournament operations are attributable and reviewable

## Must-Preserve Trust Signals

The following are product-trust signals that appear non-negotiable:

- visible audit/correction path for ballots
- visible disaster/conflict validation
- reliable regeneration of standings and breaks
- manual override capability for operators
- print/export equivalents for key operational artifacts

## Areas That Can Likely Change Safely If Substance Is Preserved

These can likely change in implementation or UI as long as the core effect remains:

- exact menu layout
- exact page decomposition
- exact settings-storage strategy
- exact document-generation technology
- exact report visual styling

## Areas That Should Not Be Simplified Casually

These appear dangerous to simplify without validation:

- judge prefs/strikes/conflicts
- pairing fairness logic
- result/tiebreak protocols
- publication staging
- recovery workflows
- print/report outputs used live

## Suggested Next Use Of This Document

Use this list later to:

- define non-negotiable acceptance criteria,
- anchor Release 1 implementation briefs,
- test simplification ideas against real operator needs,
- keep AI-agent execution focused on preserving product trust rather than copying legacy structure.
