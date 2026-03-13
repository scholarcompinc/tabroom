# Tabroom Rebuild Work Plan Checklist

## Purpose

This document turns the planning documents into a practical execution checklist.

Use this file to track:

- sequencing,
- gate completion,
- current phase status,
- artifact production order,
- handoff readiness for the ScholarComp repo.

Primary references:

- [TABROOM_REBUILD_PLAN.md](/home/jeloni/tabroom/TABROOM_REBUILD_PLAN.md)
- [EXISTING_PLATFORM_CATALOG.md](/home/jeloni/tabroom/docs/EXISTING_PLATFORM_CATALOG.md)
- [Handoff Package Index](/home/jeloni/tabroom/claude/HANDOFF_INDEX.md)

## Status Snapshot

- Branch: `rebuild-planning`
- Current phase: `Phase 4 complete — ready for ScholarComp-side mapping`
- Last updated: `2026-03-13`

## Phase 0: Program Gates

### Strategy and Guardrails

- [ ] Legal review completed and documented
- [ ] Market/package model decided
- [x] Workflow parity vs UI parity decision made — workflow parity chosen
- [x] Initial tenancy model decision made — tournament-as-primary-tenant
- [ ] Main App vs PMC split approved
- [x] Configuration philosophy agreed — no EAV, typed columns with schema validation
- [ ] Domain expert access path identified
- [x] Initial Release 1 scope standard agreed — 78 capabilities, debate + speech

### Operating Guardrails

- [x] Clean-room rules accepted
- [ ] Decision log established
- [ ] Testing guardrails established
- [x] Agent coordination rules acknowledged
- [x] Requirements artifact template approach agreed

Completion criteria:

- All load-bearing decisions have written outcomes
- Remaining open questions are tracked explicitly, not implied

## Phase 1: Existing Platform Discovery ✅

This phase should stay descriptive and source-grounded.

### Existing Platform Catalog

- [x] Capability domains populated → `capability_batch_a.md` through `capability_batch_f.md`
- [x] Role inventory populated → `role_inventory.md`
- [x] Workflow inventory populated → `workflow_inventory.md`
- [x] Route and surface inventory populated → covered in capability batches
- [x] Data and entity inventory populated → `data_entity_inventory.md`
- [x] Settings and configuration inventory populated → `settings_configuration_inventory.md`
- [x] Reports and exports inventory populated → `reports_and_integrations.md`
- [x] Integrations inventory populated → `reports_and_integrations.md`
- [x] Algorithmically complex areas populated → `algorithmically_complex_areas.md`
- [x] Documentation coverage assessment populated → `documentation_coverage.md`
- [x] Unknowns and open questions section populated → `unknowns_and_open_questions.md`

### Initial Discovery Support Artifacts

- [x] Lightweight parity matrix started → `parity_matrix.md`
- [x] Glossary starter expanded → `glossary.md`
- [x] Source traceability conventions applied

Completion criteria: ✅

- Major product areas are inventoried with evidence
- Overlapping legacy surfaces are documented, not collapsed
- Documentation strength is visible per domain

## Phase 2: Requirements Corpus Build ✅

These artifacts convert discovery into normalized product requirements.

### Core Artifacts

- [x] Existing Platform Catalog is stable enough for downstream use
- [x] Capability Map created → `capability_map.md`
- [x] Role and Permission Matrix created → `role_permission_matrix.md`
- [x] Workflow Catalog created → `workflow_catalog_detailed.md`
- [x] Business Rules Catalog created → `business_rules_catalog.md`
- [x] Data and State Model Pack created → `data_state_model.md`
- [x] Tenancy and Access Model created → `tenancy_access_model.md`
- [x] Configuration and Settings Spec created → `configuration_settings_spec.md`
- [x] Non-Functional Requirements Catalog created → `nonfunctional_requirements.md`
- [x] ScholarComp Reuse Fitness Assessment prepared → `scholarcomp_reuse_assessment.md`
- [x] Parity Matrix expanded → `parity_matrix.md` (103 capabilities)
- [x] Glossary and Terminology Map created → `glossary.md`

Completion criteria: ✅

- Release 1 candidate capabilities are identifiable
- High-risk business rules are explicitly documented
- Debate, speech, and congress differences remain visible

## Phase 3: Release 1 Triage ✅

- [x] Feature frequency / criticality signals added to scope decisions
- [x] Every-tournament workflows isolated
- [x] Most-tournament workflows isolated
- [x] Some-tournament and rare workflows marked for later
- [x] Release 1 operating core agreed → 78 capabilities (70%)
- [x] Release 2 and Release 3 candidate areas grouped

Artifact: `release1_triage.md` — 12 concrete scope decisions

Completion criteria: ✅

- Release 1 scope is based on evidence, not intuition
- Rare or niche features are deliberately deferred, not accidentally omitted

## Phase 4: ScholarComp Handoff Preparation ✅

### Handoff Package

- [x] Existing Platform Catalog ready
- [x] Capability Map ready
- [x] Role and Permission Matrix ready
- [x] Workflow Catalog ready
- [x] Business Rules Catalog ready
- [x] Data and State Model Pack ready
- [x] Tenancy and Access Model ready
- [x] Configuration and Settings Spec ready
- [x] Non-Functional Requirements Catalog ready
- [x] Parity Matrix ready
- [x] Glossary and Terminology Map ready
- [x] Source provenance notes included → `HANDOFF_INDEX.md`

Artifact: `HANDOFF_INDEX.md` — master index with provenance and critical findings

Completion criteria: ✅

- Artifacts are clean-room descriptive
- ScholarComp mapping can begin without reopening basic source discovery

## Phase 5: ScholarComp-Side Mapping

This work belongs in the ScholarComp repo.

- [ ] ScholarComp reuse fitness assessment completed there
- [ ] Frontend IA and screen inventory completed there
- [ ] Service boundary map completed there
- [ ] Release 1 tranche plan completed there
- [ ] Contract planning completed there
- [ ] Agent-ready build briefs completed there

Completion criteria:

- Each Release 1 capability has an owner
- Each reused service has an explicit fit/gap decision
- Main App vs PMC ownership is explicit by workflow

## Immediate Next Actions

- [x] ~~Finish the top half of the Existing Platform Catalog~~
- [x] ~~Create the first lightweight parity matrix~~
- [x] ~~Expand the glossary~~
- [x] ~~Create the Configuration and Settings Spec~~
- [x] ~~Create the Workflow Catalog for Release 1 candidate flows~~
- [x] ~~Start the Business Rules Catalog~~
- [ ] Begin Phase 5 in ScholarComp repo — service boundary mapping
- [ ] Resolve remaining Phase 0 gates (legal review, market model, PMC split, domain expert access)

## Notes

- Keep this file focused on sequencing and status.
- Put detailed acceptance checklists in a separate artifact checklist file.
- All Phase 1–4 artifacts are in `/home/jeloni/tabroom/claude/`. See `HANDOFF_INDEX.md` for the complete inventory.
