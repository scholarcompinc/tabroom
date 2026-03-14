# Result and Recognition Source-of-Truth Register

## Purpose

This document captures the first-pass register of which result or recognition artifacts appear to function as the authoritative source for specific downstream purposes in the current Tabroom platform.

Its purpose is to distinguish:

- informative display artifacts,
- operational decision-support artifacts,
- canonical placement artifacts,
- downstream recognition / qualification authorities.

This is a descriptive source-analysis register, not a future data-governance design.

## Source Basis

Primary evidence used:

- `web/tabbing/publish/index.mhtml`
- `web/tabbing/publish/generate_results.mhtml`
- `web/tabbing/publish/generate_sweeps.mhtml`
- `web/tabbing/results/nsda_qualifiers.mhtml`
- `web/funclib/district_qualifiers.mas`
- `web/funclib/nsda/post_points.mas`
- `web/funclib/results_table.mas`
- prior codex results/recognition docs

## First-Pass Conclusions

High-confidence observations:

- different artifacts are authoritative for different jobs
- round results are authoritative for round transparency, not for final qualification
- standings and prelim seeds are authoritative for some mid-tournament decisions
- `Final Places` appears to become the canonical upstream artifact for several official downstream flows
- sweeps and speaker awards have their own separate authority lanes

## Register

| Domain Question | Apparent Source-of-Truth Artifact | Why |
|---|---|---|
| “What happened in this round?” | published round results | Round-result pages are purpose-built for one-round visibility |
| “How are entries currently ordered?” | standings / prelim-seed style result computation | Operator and result-generation flows compute ordering from ranking protocols |
| “Who advances into elim/break structure?” | break-ready standings + bracket generation outputs | Break workflows consume ordered standings and then create advancement artifacts |
| “What are the official final placements?” | `Final Places` result set | Multiple downstream flows explicitly depend on it |
| “Who are the speaker award winners?” | `Speaker Awards` result set | Separate student-level award authority |
| “Who won sweeps?” | sweep-set-backed result set for the relevant scope | Sweeps is a separate configured award engine |
| “Who qualifies / is alternate / is vacated?” | district qualifier output layered on `Final Places` plus eligibility rules | Qualification is not raw placement alone |
| “What gets posted to external ecosystems?” | specialized governing-body posting flows using official result artifacts | NSDA/TOC flows add policy and metadata dependencies |

## Authority Layers

## 1. Informational Round Layer

Authoritative for:

- one-round public visibility
- round-specific competitive transparency

Not authoritative for:

- final placements
- qualification status
- sweep winners

## 2. Ordering / Standings Layer

Authoritative for:

- current ordering
- break preparation
- prelim seed generation
- operator review

Not always authoritative for:

- final placements after elims/finals
- district qualification after eligibility adjustments

## 3. Official Placement Layer

Authoritative for:

- final event placement
- many downstream official uses

Observed primary artifact:

- `Final Places`

## 4. Recognition Layer

Authoritative for:

- speaker awards
- sweep winners
- novice/honorable mention variants where configured

Observed primary artifacts:

- `Speaker Awards`
- sweep-set-generated result sets

## 5. Qualification / External Posting Layer

Authoritative for:

- district qualifiers
- posted points / bids
- affiliated ecosystem reporting

Observed shape:

- external-facing outputs are derived from official placement artifacts plus policy logic

## Source-of-Truth Notes By Artifact Family

### Round Results

Strength:

- best authority for “what was posted for this round”

Limitation:

- not sufficient as the canonical final result for many downstream workflows

### Prelim Seeds / Standings

Strength:

- best authority for current ordering before finals and breaks

Limitation:

- may be superseded by later elimination or final-round outcomes

### Final Places

Strength:

- appears to be the most canonical event-level placement artifact
- is explicitly referenced by downstream qualifier/posting flows

Limitation:

- still generated and replaceable, not immutable by construction

### Speaker Awards

Strength:

- explicit separate authority for student-level recognition

Limitation:

- not a substitute for entry/team placement

### Sweeps Result Sets

Strength:

- explicit authority for award aggregation outputs

Limitation:

- authority is scoped to a specific sweep-set ruleset, not generic tournament standing

### District Qualifiers / TOC / NSDA Outputs

Strength:

- authoritative for external recognition/reporting use once generated under the right policy context

Limitation:

- depend on upstream artifacts and policy/eligibility logic that may change independently of raw placement

## Important Distinctions To Preserve Later

- informative truth vs official truth
- current-order truth vs final-placement truth
- placement truth vs recognition truth
- local tournament truth vs external-posting truth

## Open Questions For Later Validation

- whether operators conceptually treat `Final Places` as the canonical lock point for an event
- whether any tournament segments use prelim seeds or bracket state as the primary authority longer than the source suggests
- whether some external postings derive from direct computation rather than generated result sets in edge cases
