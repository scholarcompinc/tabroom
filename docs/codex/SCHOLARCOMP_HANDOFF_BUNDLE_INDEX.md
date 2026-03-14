# ScholarComp Handoff Bundle Index

## Purpose

This document defines the recommended bundle of source-analysis artifacts to move into the ScholarComp repo when target-side mapping begins.

Its purpose is to keep the handoff focused, traceable, and small enough to work from productively.

This is not the target architecture pack. It is the source-analysis transfer pack.

## Handoff Principles

The handoff bundle should:

- preserve provenance
- include the minimum set needed for target-side planning
- separate descriptive source truth from inferential prioritization
- avoid dragging the entire archive over if a smaller set will do

## Bundle Structure

## 1. Orientation Bundle

Purpose:

- establish vocabulary and provenance

Recommended docs:

- `SOURCE_TRACEABILITY_INDEX.md`
- `FIRST_PASS_EXISTING_PLATFORM_CATALOG.md`
- `GLOSSARY_FIRST_PASS.md`
- `CORPUS_MANIFEST_AND_RECOMMENDED_READING_ORDER.md`

## 2. Core Product Bundle

Purpose:

- describe what the platform does and how people use it

Recommended docs:

- `FIRST_PASS_CAPABILITY_MAP.md`
- `FIRST_PASS_WORKFLOW_CATALOG.md`
- `FIRST_PASS_ROLE_PERMISSION_MATRIX.md`
- `FIRST_PASS_CONFIGURATION_SETTINGS_SPEC.md`
- `FIRST_PASS_DATA_STATE_MODEL.md`
- `FIRST_PASS_BUSINESS_RULES_CATALOG.md`

## 3. Critical Domain Bundle

Purpose:

- preserve the highest-value tournament logic

Recommended docs:

- `FIRST_PASS_PAIRING_AND_ROUND_GENERATION_DEEP_DIVE.md`
- `FIRST_PASS_RESULTS_AND_ADVANCEMENT_DEEP_DIVE.md`
- `FIRST_PASS_JUDGE_ROOM_ASSIGNMENT_DEEP_DIVE.md`
- `MUST_NOT_SIMPLIFY_PARITY_CHECKLIST.md`

## 4. Results And Recognition Bundle

Purpose:

- support later results/awards/reporting mapping without rereading the legacy repo

Recommended docs:

- `RESULTS_PUBLICATION_AND_AUDIENCE_GATING_MODEL.md`
- `RESULTS_FORMAT_VARIATION_MATRIX.md`
- `RESULTS_GENERATION_REGENERATION_AND_MUTABILITY_MODEL.md`
- `RESULTS_EXCEPTION_AND_OVERRIDE_REGISTER.md`
- `RESULTS_INPUT_DEPENDENCY_MAP.md`
- `RESULTS_STAGE_AND_TIMING_MODEL.md`
- `RESULT_SET_RENDERING_AND_TYPE_MODEL.md`
- `RESULT_AND_RECOGNITION_SOURCE_OF_TRUTH_REGISTER.md`
- `OFFICIAL_RECOGNITION_AND_QUALIFICATION_OUTPUT_MAP.md`
- `GOVERNING_BODY_RESULTS_POSTING_DEPENDENCY_REGISTER.md`

## 5. Reporting And Operations Bundle

Purpose:

- preserve the non-obvious operational footprint

Recommended docs:

- `FIRST_PASS_REPORTING_PRINT_EXPORT_DEEP_DIVE.md`
- `PRINT_PACKET_AND_CEREMONY_ARTIFACT_REGISTER.md`
- `AWARDS_DISTRIBUTION_AND_PICKUP_OPERATION_MODEL.md`
- `RECOVERY_AND_CORRECTION_WORKFLOW_INVENTORY.md`

## 6. Planning Input Bundle

Purpose:

- help target-side scoping and interviews start fast

Recommended docs:

- `LIGHTWEIGHT_PARITY_MATRIX.md`
- `EVIDENCE_BASED_RELEASE1_CANDIDATE_SHORTLIST.md`
- `HIGH_RISK_AMBIGUITY_SHORTLIST_FOR_SME_SESSIONS.md`
- `SOURCE_CONFLICT_AND_AMBIGUITY_REGISTER.md`
- `SME_VALIDATION_QUESTION_SET.md`

## 7. Optional Hypothesis Bundle

Purpose:

- provide source-backed heuristics, while keeping them clearly separate from descriptive truth

Recommended docs:

- `RESULT_REPORT_AND_PRINT_ARTIFACT_FREQUENCY_HYPOTHESIS.md`
- `AWARDS_AND_QUALIFIER_FREQUENCY_HYPOTHESIS.md`
- `SOURCE_BACKED_RECOVERY_FREQUENCY_SEVERITY_RANKING.md`

## Recommended Import Order In ScholarComp Repo

1. Orientation Bundle
2. Core Product Bundle
3. Critical Domain Bundle
4. Planning Input Bundle
5. Results And Recognition Bundle
6. Reporting And Operations Bundle
7. Optional Hypothesis Bundle

## Suggested Folder Structure In ScholarComp Repo

Recommended target shape:

- `docs/tabroom-rebuild/source-analysis/orientation/`
- `docs/tabroom-rebuild/source-analysis/core/`
- `docs/tabroom-rebuild/source-analysis/domains/`
- `docs/tabroom-rebuild/source-analysis/results/`
- `docs/tabroom-rebuild/source-analysis/operations/`
- `docs/tabroom-rebuild/source-analysis/triage/`

## Provenance Guidance

Each imported file should retain:

- original source repo path
- original branch or commit reference if useful
- note that the doc is descriptive source analysis
- date of transfer into ScholarComp repo

## What Should Not Be Carried Over As-Is

Do not treat imported docs as:

- service designs
- frontend decomposition
- final Release 1 commitments
- implementation briefs

Those belong to ScholarComp-side planning after the handoff.
