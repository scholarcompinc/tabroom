# Results Exception and Override Register

## Purpose

This document captures the first-pass register of exception cases, exclusions, and override-like conditions that affect results behavior in the current Tabroom platform.

Its purpose is to keep parity work from treating result calculation and publication as a clean happy-path pipeline.

This is a descriptive source-analysis artifact, not a future exception-handling design.

## Source Basis

Primary evidence used:

- `web/tabbing/results/order_entries.mas`
- `web/tabbing/results/speakers_csv.mhtml`
- `web/tabbing/publish/generate_results.mhtml`
- `web/tabbing/break/index.mhtml`
- `web/tabbing/break/break_debate.mhtml`
- `web/tabbing/break/break_speech.mhtml`
- `web/tabbing/break/break_congress.mhtml`
- `web/funclib/results_debate.mas`
- `web/funclib/results_speech.mas`
- `web/funclib/results_congress.mas`
- `web/funclib/results_wudc.mas`
- `web/funclib/entry_byes.mas`
- `web/funclib/region_results.mas`
- prior codex results/break/settings docs

## First-Pass Conclusions

High-confidence observations:

- results behavior contains many exclusion and exception rules
- some exceptions affect calculation, others publication, others advancement
- byes, forfeits, ignored rounds, breakout filters, and specialized labels are native cases, not afterthoughts

## Exception Families

## 1. Bye and Forfeit Handling

Observed behavior:

- byes and forfeits are treated explicitly in round-result views
- speech/congress/WUDC round displays typically skip bye/forfeit rows
- debate displays synthesize textual outcomes such as `BYE`, `FFT`, and `advances`
- break logic checks for unresolved byes or coachovers before allowing advancement

Implication:

- bye/forfeit handling is format-sensitive and affects both display and downstream advancement

## 2. Ignored Rounds

Observed behavior:

- round setting `ignore_results` excludes rounds from at least some standings and speaker computations
- operator results surfaces explicitly inventory ignored rounds

Implication:

- “round exists” does not imply “round contributes to results”

## 3. Breakout-Limited Results

Observed behavior:

- generation and break flows can be limited to a breakout
- breakout labels can rename result artifacts
- breakout-specific student and entry filtering exists
- some rules explicitly exclude breakout rounds from sweeps or other aggregations

Implication:

- one event can have multiple overlapping result universes depending on breakout scope

## 4. Specialized Placement Labels

Observed behavior:

- `Prelim`
- elim round labels
- ordinal places
- `Co-Champion`
- tied place suffixes such as `-T`

Implication:

- place strings are semantically meaningful outputs, not mere formatting

## 5. Qualification / Governing-Body Exceptions

Observed behavior:

- qualifier flows look for `Final Places` and `District Qualifiers`
- region helpers allow `published = 1 or coach = 1`
- NSDA and other governing-body views introduce extra ordering and export behavior

Implication:

- qualification-oriented results can diverge from generic published standings

## 6. Novice / Honorable Mention / Speaker Minimum Variants

Observed behavior:

- top novice and honorable-mention settings shape result variants
- speaker awards can depend on `speaker_min_speeches`

Implication:

- award outputs may branch based on event configuration even when the underlying standings stay constant

## 7. Tiebreak / Runoff / Self-Reference Failure Cases

Observed behavior:

- missing tiebreak sets block ranking and break generation
- malformed tiebreak configurations are explicitly rejected
- runoff self-reference and missing runoff tiebreakers are treated as failure states

Implication:

- some results failures are configuration failures, not scoring failures

## 8. Format-Specific Exceptions

Observed examples:

- congress chair ordering and penalties
- mock-trial special display behavior
- WUDC-specific bracket and novice-bracket variants
- elimination/final handling diverging from ordinary prelim standings

Implication:

- format variance includes exception logic, not just different labels

## Exception Matrix

| Exception / Override Area | Affects | Notes |
|---|---|---|
| Byes | round display, standings, breaks, sweeps | Can be visible as explicit outcomes or filtered from displays |
| Forfeits | round display, standings, breaks | `forfeits_never_break` appears as a standings/break rule |
| `ignore_results` | standings, speaker awards, exports | Round can exist operationally but be excluded competitively |
| Breakout filters | standings generation, speaker generation, break flows, sweeps | Event can be partitioned into sub-result populations |
| Top novice / honorable mentions | awards variants | Conditional additional outputs |
| Speaker minimum speeches | speaker awards eligibility/display | Alters awards interpretation |
| Coachovers / unresolved bye winners | break readiness | Can block advancement |
| Tied placements | display labels and final result semantics | Ties propagate into place strings and some advancement behavior |
| Student vote labels / special rounds | final-place generation | Certain labels are skipped in some final calculations |
| Region / coach-visible helper paths | downstream consumption | Visibility can broaden outside ordinary public pages |

## Operational Meaning

The source suggests that results correctness depends on more than ranked scores:

- configuration may disqualify or include data
- round types may or may not count
- special outcome types change advancement logic
- auxiliary visibility states change which consumers can access a result

## Important Distinctions To Preserve Later

- exception in computation vs exception in publication
- filtered-out round vs missing round
- special award variant vs core standings
- special label semantics vs plain text decoration
- audience override vs public visibility

## Open Questions For Later Validation

- which exception paths are common enough to be Release-1 critical versus long-tail compatibility work
- which governing-body exceptions can be normalized and which are operationally non-negotiable
- how often operators intentionally use `ignore_results` and similar exclusion mechanisms in live tournaments
