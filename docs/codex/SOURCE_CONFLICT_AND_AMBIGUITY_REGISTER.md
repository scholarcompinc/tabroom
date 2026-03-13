# Source Conflict and Ambiguity Register

## Purpose

This document records first-pass places where the source corpus is ambiguous, overlapping, incomplete, or likely to produce incorrect assumptions if treated as fully settled.

Its purpose is to prevent the rebuild effort from silently converting uncertainty into architecture.

This is not a bug list. It is a discovery-risk register.

## How To Read This Register

Each item captures:

- the ambiguity or conflict,
- why it matters,
- likely impact on later planning,
- what kind of validation is needed next.

## 1. Documentation Recency and Coverage Gaps

Observed ambiguity:

- the legacy PDF manual is old
- `docs.tabroom.com` is clearly more current for many workflows
- neither source can be assumed to cover all edge cases or specialized modes

Why it matters:

- workflow intent and production behavior may diverge

Likely impact:

- requirements derived only from docs may miss important operational edge cases

Needed validation:

- compare critical workflows against live code and SME usage

## 2. Route Overlap Versus True Capability Duplication

Observed ambiguity:

- similar concepts appear in multiple route areas:
  - judge management
  - results views
  - assignment workflows
  - registration change flows

Why it matters:

- some overlaps are true duplication
- others reflect different actor surfaces or operator fallback paths

Likely impact:

- premature deduplication could delete needed workflows

Needed validation:

- actor-by-actor workflow comparison

## 3. Settings Versus State

Observed ambiguity:

- many `*_setting` tags behave like configuration
- many others behave like operational state, audit markers, or per-tournament overrides

Why it matters:

- a future model that treats all settings uniformly will likely be wrong

Likely impact:

- incorrect service boundaries
- poor validation and poor UX

Needed validation:

- classify high-value tags into configuration, state, audit, and computed/temporary categories

## 4. Which Settings Are Truly Active In Production

Observed ambiguity:

- the tag inventory shows a very large historical surface
- some tags may be residual, rare, or data-only

Why it matters:

- attempting full settings parity may waste substantial effort

Likely impact:

- oversized Release 1 scope

Needed validation:

- SME review plus live-data usage analysis later if available

## 5. Event-Type Scope For The First Product

Observed ambiguity:

- debate and speech are clearly common
- congress is clearly supported and materially different
- WUDC/WSDC and other special formats add even more variation

Why it matters:

- format scope changes data model, workflow, and result design

Likely impact:

- core architecture if scoped incorrectly

Needed validation:

- strategic scope decision before ScholarComp-side mapping

## 6. Exact Meaning Of Some Permission Tags

Observed ambiguity:

- permission tags such as `owner`, `tabber`, `contact`, `checker`, `prefs`, `prefs_only`, `chair`, `wsdc` are visible
- not all semantic differences are fully obvious from first-pass code reading

Why it matters:

- role and access mapping is foundational

Likely impact:

- security and workflow leakage if misunderstood

Needed validation:

- deeper permissions extraction and SME review of real operational roles

## 7. True Tenancy Boundary

Observed ambiguity:

- the data model includes tournaments, chapters, schools, circuits, regions, districts, and site-wide actors
- not all of these are clearly tenant boundaries in the future product sense

Why it matters:

- tenancy affects architecture, auth, and cross-entity visibility

Likely impact:

- ScholarComp-side service and access model

Needed validation:

- explicit product decision plus later fit assessment against ScholarComp tenancy

## 8. Commodity Versus Crown-Jewel Logic In Results

Observed ambiguity:

- some result flows are clearly protocol-driven and specialized
- some display/reporting layers may be simpler or replaceable

Why it matters:

- not all result-adjacent code deserves the same parity investment

Likely impact:

- poor prioritization if everything is treated as equally sacred

Needed validation:

- map result artifacts to operational criticality and usage

## 9. How Much Print Surface Is Actually Mandatory

Observed ambiguity:

- the print/report surface is large
- some outputs are clearly critical
- others may be historical residue or low-frequency specialty use

Why it matters:

- print stack choices can become expensive if over-scoped

Likely impact:

- Release 1 bloat if untriaged

Needed validation:

- identify which artifacts operators still rely on in real tournaments

## 10. Recovery Workflow Frequency

Observed ambiguity:

- the code clearly supports many repair operations
- first-pass source analysis cannot yet tell which ones are everyday versus rare

Why it matters:

- the most common recovery workflows should shape PMC design early

Likely impact:

- operator UX priorities

Needed validation:

- tabber interviews or live-observation evidence

## 11. Financial Scope Ambiguity

Observed ambiguity:

- fees, invoices, fines, concessions, hotels, and purchase-order flows all exist
- source analysis does not yet distinguish common from niche usage

Why it matters:

- financial scope can expand quickly and distort the initial product plan

Likely impact:

- Release 1 scoping and service mapping

Needed validation:

- market strategy plus user interviews

## 12. District / National / Circuit Feature Criticality

Observed ambiguity:

- these features are substantial
- but their commercial importance depends heavily on target market

Why it matters:

- they could either be essential early differentiators or intentional later phases

Likely impact:

- release sequencing

Needed validation:

- strategic market decision, not just code reading

## 13. Public UI Expectations Versus Workflow Equivalence

Observed ambiguity:

- current source analysis supports workflow parity strongly
- it does not by itself determine how much legacy UI pattern compatibility matters socially

Why it matters:

- user adoption may depend on familiar terminology and workflow ordering even if the UI is better

Likely impact:

- Main App and PMC IA decisions later

Needed validation:

- operator and coach feedback on acceptable change level

## 14. Ballot Reliability Expectations On Poor Networks

Observed ambiguity:

- the source evidence strongly suggests reliability matters
- it does not yet quantify what offline/degraded behavior is acceptable

Why it matters:

- modern frontend architecture choices depend on this

Likely impact:

- ballot UX and BFF/client strategy

Needed validation:

- SME validation and field assumptions about venue conditions

## 15. Historical Data / Import Expectations

Observed ambiguity:

- source evidence shows multiple exports and data handoff patterns
- it does not yet show whether first customers will expect historical import from tabroom.com immediately

Why it matters:

- migration and interoperability can become launch blockers if ignored

Likely impact:

- later ScholarComp planning

Needed validation:

- business-side product decision

## Highest-Priority Ambiguities To Resolve Early

These appear most likely to distort later architecture if left vague:

- settings versus state
- event-type scope
- tenancy boundary
- permission semantics
- print/report criticality
- district/national scope
- degraded-network expectations

## Suggested Next Use Of This Register

Use this register to:

- drive SME interviews,
- create explicit decision gates before ScholarComp-side architecture mapping,
- avoid treating low-confidence assumptions as settled requirements.
