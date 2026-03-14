# Awards and Qualification Policy Variation Register

## Purpose

This document captures the first-pass register of policy variation visible in award, sweeps, and qualification logic across the current Tabroom platform.

Its purpose is to highlight where behavior varies because of affiliation, event category, participant counts, or special-case rules.

This is a descriptive source-analysis artifact, not a future policy model.

## Source Basis

Primary evidence used:

- `web/funclib/nsda/qualifier_count.mas`
- `web/funclib/district_qualifiers.mas`
- `web/tabbing/results/top_novice.mas`
- `web/tabbing/results/sweep_tourn.mas`
- `web/tabbing/publish/generate_sweeps.mhtml`
- `web/tabbing/report/toc/post_bids.mhtml`

## First-Pass Conclusions

High-confidence observations:

- this area contains substantial policy variation
- many rules depend on event type, event abbreviation, district level, entry counts, or affiliation flags
- some variation appears historical or era-specific, but it still exists in the current source

## Variation Families

## 1. Event-Category-Specific Qualification Rules

Observed examples:

- special qualifier count logic for `BQ`
- special qualifier count logic for `SEN`
- house-event special cases for `HOU` / `HSE`

Implication:

- qualification count is not one global formula across all event categories

## 2. District-Level / Affiliation Variation

Observed examples:

- district level affects qualifier counts
- district level can be forced by setting
- district versus nationals mode changes output semantics and print/report labels

Implication:

- affiliation mode is a real policy branch, not a small presentation tweak

## 3. Entry-Count Threshold Variation

Observed examples:

- qualifier counts increase at specific entry-count thresholds
- alternate counts derive from entry counts
- some overrides apply only above certain thresholds

Implication:

- many outputs are population-sensitive, not fixed-size

## 4. Era / Exceptional-Condition Variation

Observed example:

- “worst year ever” branch in qualifier-count logic

Implication:

- the source contains exceptional policy eras that may or may not remain current, but they cannot be ignored in source analysis

## 5. Novice / Honorable Mention / Eligibility Variation

Observed examples:

- top novice can exclude elim/final participants depending on configuration
- speaker awards can depend on minimum-speeches thresholds
- district qualifiers screen marks ineligible or vacated entries

Implication:

- recognition outputs are shaped by eligibility policy, not just raw rank

## 6. Sweepstakes Policy Variation

Observed examples:

- novice-only sweeps
- multiply by entry size
- skip/ignore selected rounds
- exclude breakout rounds
- manually added sweeps points
- special treatment of coachovers/walkovers

Implication:

- sweeps is a broad configurable policy framework, not a single formula

## 7. Circuit / Bid Variation

Observed examples:

- TOC bid round
- bid limit
- silver-bid and ghost-bid logic in some debate cases

Implication:

- bid outputs are shaped by circuit-specific policy, not generic tournament advancement

## Variation Register

| Variation Area | Primary Drivers | Notes |
|---|---|---|
| District qualifier counts | event code, district level, entry counts, overrides, special periods | Policy-heavy, affiliation-dependent |
| District qualifier membership/advancement | NSDA eligibility, vacates, ties, alternates | Qualification is more than top-N placement |
| Top novice awards | novice eligibility + elim/final inclusion mode | Award-specific policy overlay |
| Speaker awards | protocol config + minimum-speeches + breakout/student filters | Student-level recognition path |
| Sweepstakes | sweep-rule set + exclusions + novice and breakout logic + manual points | Separate award policy engine |
| TOC bid outputs | bid round, bid limit, event type, silver/ghost logic | Circuit-specific recognition/reporting |

## Why This Matters

The source corpus suggests that awards and qualification cannot be reduced to:

- “run standings”
- “take top N”
- “show trophy outputs”

Instead, they depend on:

- affiliation context
- configurable policy rules
- eligibility and vacancy state
- event-category and format variation

## Important Distinctions To Preserve Later

- competitive ordering policy vs recognition policy
- qualification capacity policy vs eligibility policy
- award-specific policy vs general result publication
- exceptional historical branch vs stable evergreen rule

## Open Questions For Later Validation

- which of these variation branches are still actively used
- which affiliation-specific policies are mandatory for the intended market
- whether some historical special cases should be documented as legacy-only during later normalization
