# Governing-Body Results Posting Dependency Register

## Purpose

This document captures the first-pass register of dependencies and special conditions visible in governing-body or circuit-related result-posting flows.

Its purpose is to isolate the ecosystem-facing obligations from ordinary local tournament result publication.

This is a descriptive source-analysis artifact, not a future integration strategy.

## Source Basis

Primary evidence used:

- `web/funclib/nsda/qualifier_count.mas`
- `web/funclib/district_qualifiers.mas`
- `web/funclib/nsda/post_points.mas`
- `web/tabbing/results/nsda_qualifiers.mhtml`
- `web/tabbing/report/toc/post_bids.mhtml`
- `web/tabbing/publish/index.mhtml`
- `web/tabbing/publish/generate_results.mhtml`

## First-Pass Conclusions

High-confidence observations:

- ecosystem-facing result posting depends on more than event standings
- governing-body flows often add eligibility, category, district-level, or count-policy logic
- some postings are blocked or made meaningless unless upstream result artifacts already exist

## Dependency Areas

## 1. Upstream Result Artifact Dependency

Observed examples:

- district qualifier posting depends on generated `Final Places`
- NSDA point posting references `Final Places`
- TOC bids depend on bid-round configuration and event ordering

Implication:

- many ecosystem flows do not compute from scratch; they rely on prior tournament result products

## 2. Event Classification Dependency

Observed examples:

- NSDA category resolution
- special-case category handling for congress / WSDC / guessed category codes
- category code failure can block or flag posting

Implication:

- event taxonomy mapping is part of external posting readiness

## 3. Eligibility and Vacancy Dependency

Observed examples:

- district qualifiers check student NSDA identifiers
- entries can be vacated
- alternates/promotions depend on vacancy and tie conditions

Implication:

- qualification outputs are not equivalent to “top N placements”

## 4. District / Level / Override Dependency

Observed examples:

- qualifier count depends on district level
- level can be forced by tournament setting
- qualifier counts can be overridden or forced at the event level
- event-size thresholds and special event abbreviations alter counts

Implication:

- external advancement count is policy-driven and highly configurable

## 5. Participation / Included-Round Dependency

Observed examples:

- NSDA point posting excludes disqualified entries
- ignored rounds are excluded
- competed-once logic affects qualifier count in some cases

Implication:

- simply having registered entries is not enough for some external postings

## Register

| Posting / Output Family | Visible Dependencies | Notes |
|---|---|---|
| District Qualifiers | `Final Places`, qualifier count rules, NSDA eligibility, vacates, ties, alternates | Policy-heavy qualification artifact |
| NSDA Points | event category, included rounds, result state, posting flag, non-DQ status | External posting with idempotence-like state (`nsda_points_posted`) |
| TOC Bids | bid round config, bid limit, event type, hidden/start state, silver bid logic in some debate cases | Circuit-specific reporting/output path |
| NSDA / district awards-related outputs | district context, award definitions, sometimes existing result context | Recognition/compliance-adjacent |

## Special-Case Logic Visible In Source

Examples:

- “worst year ever” pandemic-era branch in qualifier count logic
- event-abbreviation-specific qualifier counts such as `BQ`, `SEN`, `HOU`, `HSE`
- team-event detection affecting counts
- point-posting category guessing and flagging for manual review
- silver-bid and ghost-bid handling in TOC reporting

Implication:

- these flows are full policy engines, not thin exports

## Readiness Signals

The source suggests a governing-body posting flow is only “ready” when:

- required upstream result artifacts exist
- event metadata maps to external category expectations
- participant eligibility identifiers are present
- district/circuit configuration is complete
- rounds included in the posting are valid and not ignored

## Important Distinctions To Preserve Later

- local result publication vs external posting readiness
- placement rank vs eligibility-qualified advancement
- event type/name vs external category code
- tournament completion vs postable ecosystem state

## Open Questions For Later Validation

- which governing-body posting flows are routine for the intended market versus niche extensions
- whether operators expect retry/idempotence behavior beyond the visible posted-flag pattern
- which special-case policy branches still reflect current real-world rules versus legacy residue
