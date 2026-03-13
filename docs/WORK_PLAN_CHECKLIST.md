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

## Status Snapshot

- Branch: `rebuild-planning`
- Current phase: `Planning / early discovery`
- Last updated: `2026-03-13`

## Phase 0: Program Gates

### Strategy and Guardrails

- [ ] Legal review completed and documented
- [ ] Market/package model decided
- [ ] Workflow parity vs UI parity decision made
- [ ] Initial tenancy model decision made
- [ ] Main App vs PMC split approved
- [ ] Configuration philosophy agreed
- [ ] Domain expert access path identified
- [ ] Initial Release 1 scope standard agreed

### Operating Guardrails

- [ ] Clean-room rules accepted
- [ ] Decision log established
- [ ] Testing guardrails established
- [ ] Agent coordination rules acknowledged
- [ ] Requirements artifact template approach agreed

Completion criteria:

- All load-bearing decisions have written outcomes
- Remaining open questions are tracked explicitly, not implied

## Phase 1: Existing Platform Discovery

This phase should stay descriptive and source-grounded.

### Existing Platform Catalog

- [ ] Capability domains populated
- [ ] Role inventory populated
- [ ] Workflow inventory populated
- [ ] Route and surface inventory populated
- [ ] Data and entity inventory populated
- [ ] Settings and configuration inventory populated
- [ ] Reports and exports inventory populated
- [ ] Integrations inventory populated
- [ ] Algorithmically complex areas populated
- [ ] Documentation coverage assessment populated
- [ ] Unknowns and open questions section populated

### Initial Discovery Support Artifacts

- [ ] Lightweight parity matrix started
- [ ] Glossary starter expanded
- [ ] Source traceability conventions applied

Completion criteria:

- Major product areas are inventoried with evidence
- Overlapping legacy surfaces are documented, not collapsed
- Documentation strength is visible per domain

## Phase 2: Requirements Corpus Build

These artifacts convert discovery into normalized product requirements.

### Core Artifacts

- [ ] Existing Platform Catalog is stable enough for downstream use
- [ ] Capability Map created
- [ ] Role and Permission Matrix created
- [ ] Workflow Catalog created
- [ ] Business Rules Catalog created
- [ ] Data and State Model Pack created
- [ ] Tenancy and Access Model created
- [ ] Configuration and Settings Spec created
- [ ] Non-Functional Requirements Catalog created
- [ ] ScholarComp Reuse Fitness Assessment prepared
- [ ] Parity Matrix expanded
- [ ] Glossary and Terminology Map created

Completion criteria:

- Release 1 candidate capabilities are identifiable
- High-risk business rules are explicitly documented
- Debate, speech, and congress differences remain visible

## Phase 3: Release 1 Triage

- [ ] Feature frequency / criticality signals added to scope decisions
- [ ] Every-tournament workflows isolated
- [ ] Most-tournament workflows isolated
- [ ] Some-tournament and rare workflows marked for later
- [ ] Release 1 operating core agreed
- [ ] Release 2 and Release 3 candidate areas grouped

Completion criteria:

- Release 1 scope is based on evidence, not intuition
- Rare or niche features are deliberately deferred, not accidentally omitted

## Phase 4: ScholarComp Handoff Preparation

Do this after the source-analysis corpus is strong enough.

### Handoff Package

- [ ] Existing Platform Catalog ready
- [ ] Capability Map ready
- [ ] Role and Permission Matrix ready
- [ ] Workflow Catalog ready
- [ ] Business Rules Catalog ready
- [ ] Data and State Model Pack ready
- [ ] Tenancy and Access Model ready
- [ ] Configuration and Settings Spec ready
- [ ] Non-Functional Requirements Catalog ready
- [ ] Parity Matrix ready
- [ ] Glossary and Terminology Map ready
- [ ] Source provenance notes included

Completion criteria:

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

- [ ] Finish the top half of the Existing Platform Catalog
- [ ] Create the first lightweight parity matrix
- [ ] Expand the glossary
- [ ] Create the Configuration and Settings Spec
- [ ] Create the Workflow Catalog for Release 1 candidate flows
- [ ] Start the Business Rules Catalog

## Notes

- Keep this file focused on sequencing and status.
- Put detailed acceptance checklists in a separate artifact checklist file.
