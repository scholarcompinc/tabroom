# Must-Not-Simplify Parity Checklist

## Purpose

This document captures the first-pass checklist of product behaviors and operational capabilities that source analysis suggests should not be casually simplified, flattened, or omitted during downstream planning.

Its purpose is to function as a parity guardrail, not as a release commitment.

This is a source-backed warning checklist.

## How To Use This Document

Use this checklist when:

- defining Release 1 scope
- normalizing requirements into ScholarComp language
- deciding whether a workflow is “just legacy complexity”
- drafting agent implementation briefs

If an item here is intentionally simplified or deferred, that decision should be explicit.

## 1. Live Recovery And Correction Workflows

Do not simplify away:

- judge add/remove/swap flows
- room reassignment flows
- entry/panel movement flows
- side/speaker-order/seating corrections
- ballot/score correction paths
- publication rollback and republish flows
- disaster-check / validation-assisted repair

Why:

- the source strongly suggests these are normal operational tools, not rare admin-only extras

## 2. Layered Publication And Visibility

Do not simplify away:

- distinction between round publication and result-set publication
- distinction between public and coach-only visibility
- layered round result thresholds such as primary/secondary/feedback posting
- generation versus publication as separate actions

Why:

- the source clearly models visibility as multi-stage and audience-sensitive

## 3. Format-Specific Results Behavior

Do not simplify away:

- debate versus speech versus congress result differences
- debate side labels and positional semantics
- speech section/rank/point behavior
- congress chamber/chair/speech-value behavior
- special debate variants such as WUDC/WSDC/mock-trial handling where in scope

Why:

- the results domain is polymorphic, not one generic scoring pipeline

## 4. Result Artifact Families

Do not collapse into one generic “results page”:

- round results
- standings / prelim seeds
- final places
- speaker awards
- prelims table
- brackets
- sweeps outputs
- qualifier outputs

Why:

- these artifacts have different lifecycle, audience, timing, and downstream dependency roles

## 5. Final Places As A Distinct Official Artifact

Do not treat as a cosmetic label on standings:

- `Final Places` appears to function as a canonical upstream artifact for later official workflows

Why:

- district qualifiers, NSDA-related flows, and other outputs appear to depend on it explicitly

## 6. Speaker Awards As A Separate Recognition Lane

Do not reduce to a column on entry standings:

- speaker awards have their own protocol, ranking logic, outputs, and distribution implications

Why:

- the source models them as a distinct result-set family

## 7. Sweep Rules As A Policy Engine

Do not simplify to “sum some points”:

- novice-only rules
- exclude-breakouts logic
- ignore-round logic
- manual sweep points
- entry-size multipliers
- coachover/walkover treatment

Why:

- the source shows a configurable rule engine, not a fixed aggregation

## 8. Qualification Outputs As Policy-Driven

Do not simplify to “top N qualify”:

- qualifier counts vary by affiliation/event/threshold
- eligibility matters
- vacancies and alternates matter
- ties matter

Why:

- qualification is a policy outcome layered on top of placement, not placement itself

## 9. Print / Packet / Ceremony Surface

Do not assume all result/report artifacts can be replaced by simple web views:

- seeding printouts
- result table printouts
- ceremony scripts
- packets
- audit PDFs
- pickup/distribution views

Why:

- the source shows a real operational print/document layer

## 10. Awards Distribution Operations

Do not simplify away:

- school-level award pickup tracking
- ceremony/distribution separation
- school-specific award detail flows

Why:

- recognition continues after winners are computed; distribution itself is operationally modeled

## 11. Settings As Policy Inputs

Do not treat all `*_setting` behavior as optional UX tuning:

- many settings materially alter ranking, qualification, awards, breakouts, and publication behavior

Why:

- the source shows settings acting as policy controls and state markers, not just defaults

## 12. Mid-Tournament “As Of Round” Behavior

Do not simplify away:

- standings and award calculations as of a selected round
- mid-tournament result generation before final completion

Why:

- many operator workflows depend on intermediate authoritative states

## 13. Recovery Before Perfection

Do not optimize only for happy-path automation:

- operators need to repair, override, and re-run

Why:

- live tournament trust depends on controllability under bad conditions, not only on ideal automation

## 14. Segment-Specific External Posting

Do not assume ecosystem posting is just export formatting:

- NSDA, TOC, district, and similar flows contain policy and readiness logic

Why:

- these are domain workflows, not thin integrations

## Checklist Summary

If a later design proposes simplifying something in this list, ask:

1. Is it genuinely low-frequency, or just hard to understand from source?
2. Does something else depend on it downstream?
3. Is it a trust-critical operator capability?
4. Is it a market-segment-critical requirement?
5. Has an SME explicitly validated that simplification as safe?
