# High-Risk Ambiguity Shortlist For SME Sessions

## Purpose

This document compresses the larger ambiguity and SME-validation backlog into a short list of the highest-risk unanswered questions.

Its purpose is to support efficient SME sessions before ScholarComp-side mapping begins.

This is a prioritization artifact, not a replacement for the full ambiguity register.

## Source Basis

Primary inputs:

- `SOURCE_CONFLICT_AND_AMBIGUITY_REGISTER.md`
- `SME_VALIDATION_QUESTION_SET.md`
- `EVIDENCE_BASED_RELEASE1_CANDIDATE_SHORTLIST.md`
- results / recovery / settings codex docs

## How To Use This Document

If SME time is limited, start here.

These questions were selected because getting them wrong would distort:

- Release 1 scope
- service boundary choices later
- operator trust assumptions
- the amount of parity work required

## Tier 1: Scope-Defining Questions

## 1. Which event formats are truly required for the first operating release?

Why this is high risk:

- debate, speech, and congress have materially different models
- getting this wrong changes a large amount of downstream design

Minimum answer needed:

- debate only
- debate + speech
- debate + speech + congress
- plus whether any special debate variants are mandatory

## 2. Which recovery workflows are non-negotiable on day one?

Why this is high risk:

- the source shows many recovery flows, but not their real-world frequency

Minimum answer needed:

- which repair actions operators must have to trust a live tournament

## 3. Which print/report artifacts are still operationally mandatory?

Why this is high risk:

- the print surface is large and expensive to overbuild

Minimum answer needed:

- which outputs are absolutely required for live use
- which are nice-to-have or legacy convenience

## 4. Which settings are actively used often enough to preserve early?

Why this is high risk:

- the settings surface is large and historically layered

Minimum answer needed:

- small set of genuinely active, high-value settings for early support

## Tier 2: Trust-Defining Questions

## 5. Which results and advancement behaviors must be exact for operators to trust the system?

Why this is high risk:

- results logic includes many artifact types and policy branches

Minimum answer needed:

- what cannot be approximated or staged in target segments

## 6. How important are paradigms, prefs, and strikes for first-market trust?

Why this is high risk:

- these may be strategically essential in some debate segments but not all

Minimum answer needed:

- whether they are core launch capabilities or later trust accelerators

## 7. How much degraded-network tolerance is required for ballots?

Why this is high risk:

- a modern frontend can fail badly here if this is underestimated

Minimum answer needed:

- what level of mobile/network resilience is expected to be usable

## Tier 3: Market-Defining Questions

## 8. Which affiliation-specific workflows are commercially necessary early?

Why this is high risk:

- district / NSDA / TOC / other program modes are deep and costly

Minimum answer needed:

- which of these matter in the first target market

## 9. Are sweeps and awards core launch value or later differentiation?

Why this is high risk:

- source evidence shows real complexity here, but importance may vary by segment

Minimum answer needed:

- whether a thin awards model is acceptable initially

## 10. Is historical import / interoperability expected at launch?

Why this is high risk:

- if yes, data mapping and artifact authority questions change quickly

Minimum answer needed:

- launch requirement, early follow-up, or later-phase item

## Suggested SME Session Order

If only one session is possible:

1. event format scope
2. recovery must-haves
3. mandatory print/report artifacts
4. active settings subset
5. results/advancement exactness requirements

If two sessions are possible:

- Session 1: operator/trust questions
- Session 2: market/affiliation questions

## Expected Outputs

After using this shortlist, you should be able to write:

- a scoped Release 1 format decision
- a must-have recovery list
- a must-have print/report list
- a high-value settings subset
- a segment-specific affiliation inclusion/exclusion list
