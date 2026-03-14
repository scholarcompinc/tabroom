# ScholarComp Handoff Package Index

## Purpose

This index assembles all discovery, requirements, and triage artifacts produced during Phases 1–3 into a single reference for ScholarComp-side architecture and implementation work (Phase 5). Every artifact listed here is clean-room descriptive — derived from code analysis and domain observation, not copied implementation.

---

## How to Use This Package

1. **Start with the Release 1 Triage** to understand what's in scope
2. **Read the Capability Map and Parity Matrix** for the full feature landscape
3. **Dive into Business Rules and Workflows** for implementation-critical details
4. **Use the ScholarComp Reuse Assessment** to plan service boundaries
5. **Reference Data/State Model, Settings Spec, and NFRs** during schema and API design

---

## Artifact Inventory

### Phase 1: Existing Platform Discovery

These artifacts describe what Tabroom does today. They are source-grounded and descriptive.

| Artifact | File | Lines | Description |
|----------|------|-------|-------------|
| Capability Batch A | `capability_batch_a.md` | 533 | Tournament admin, settings, schedule, sites/rooms |
| Capability Batch B | `capability_batch_b.md` | 353 | Events/categories, org hierarchy, districts |
| Capability Batch C | `capability_batch_c.md` | 463 | Registration, judges, prefs/strikes/conflicts |
| Capability Batch D | `capability_batch_d.md` | 518 | Pairing/paneling, ballots/scoring |
| Capability Batch E | `capability_batch_e.md` | 264 | Results/breaks/tiebreakers, sweepstakes |
| Capability Batch F | `capability_batch_f.md` | 867 | Financials, public pages, paradigms, messaging, reports, online, API, accounts |
| Role Inventory | `role_inventory.md` | 176 | 11 roles with permission tags and access control architecture |
| Workflow Inventory | `workflow_inventory.md` | 522 | 16 workflow families, ~50 specific workflows |
| Settings Inventory | `settings_configuration_inventory.md` | 453 | 570+ EAV settings across 19 scope tables |
| Algorithmic Areas | `algorithmically_complex_areas.md` | 743 | 12 areas requiring careful extraction and expert review |
| Reports & Integrations | `reports_and_integrations.md` | 683 | 190+ report surfaces, 18 integration points |
| Data Entity Inventory | `data_entity_inventory.md` | 434 | 108 tables in 10 entity clusters |
| Documentation Coverage | `documentation_coverage.md` | 66 | Per-domain documentation quality assessment |
| Glossary | `glossary.md` | 171 | 100+ terms across 8 categories |
| Unknowns | `unknowns_and_open_questions.md` | 207 | 28 open questions, prioritized |

### Phase 2: Requirements Corpus

These artifacts convert discovery into normalized requirements suitable for architecture and implementation planning.

| Artifact | File | Lines | Description |
|----------|------|-------|-------------|
| Capability Map | `capability_map.md` | 481 | 13 domains with sub-capabilities, release classification, reuse potential |
| Parity Matrix | `parity_matrix.md` | 265 | 103 capabilities scored: 65 R1, 13 R1?, 20 R2, 3 R3+, 2 Defer |
| Business Rules Catalog | `business_rules_catalog.md` | 1,054 | 20+ rules with algorithmic details, penalty weights, formulas |
| Workflow Catalog (Detailed) | `workflow_catalog_detailed.md` | 1,606 | 10 critical workflows fully specified with steps, rules, edge cases |
| Configuration Settings Spec | `configuration_settings_spec.md` | 303 | 570+ EAV tags rationalized into 10 typed groups, ~80-90 R1-critical |
| Data and State Model | `data_state_model.md` | 299 | Entity lifecycles, state machines, invariants, format polymorphism |
| Role/Permission Matrix | `role_permission_matrix.md` | 200 | 14 roles × all capability areas with rebuild recommendations |
| Tenancy/Access Model | `tenancy_access_model.md` | 234 | Tournament-as-tenant, 10 scope levels, visibility rules, isolation |
| Non-Functional Requirements | `nonfunctional_requirements.md` | 780 | 10 NFR areas: performance, scale, security, reliability |
| ScholarComp Reuse Assessment | `scholarcomp_reuse_assessment.md` | 656 | 15 services assessed, 11 new services needed, ~25-30% effort savings |

### Phase 3: Release 1 Triage

| Artifact | File | Lines | Description |
|----------|------|-------|-------------|
| Release 1 Triage | `release1_triage.md` | 232 | 12 scope decisions, 78 R1 capabilities (70%), risk assessment |

---

## Key Numbers

| Metric | Value |
|--------|-------|
| Total artifacts | 26 |
| Total lines | ~8,100+ |
| Legacy tables analyzed | 108 |
| Legacy settings enumerated | 570+ |
| Capabilities cataloged | 103 |
| Release 1 capabilities | 78 (70%) |
| Critical algorithms | 9 (must implement for R1) |
| R1-critical settings | ~80-90 |
| ScholarComp services to reuse | 14 (3 as-is, 6 extend, 4 wrap, 1 replace) |
| New domain services needed | 11 |

---

## Critical Findings for Architecture

### 1. Algorithm Risk
The four highest-risk algorithms (powermatching, tiebreakers, judge assignment, snake paneling) total ~9,700 lines of legacy Perl with zero external documentation. Each requires domain expert review. See `algorithmically_complex_areas.md` and `business_rules_catalog.md`.

### 2. Settings Surface
570+ EAV settings across 19 tables must be converted to typed schema. The `configuration_settings_spec.md` proposes 10 functional groups. ~80-90 are R1-critical. Inverted boolean logic (`no_waitlist`, `closed_entry`) must be normalized.

### 3. Format Polymorphism
Debate, Speech, and Congress require entirely different algorithms for pairing, scoring, results, and breaks. Strategy pattern at the service layer is recommended. See `data_state_model.md` (Format Polymorphism Model).

### 4. Permission Model
Flat permission model (not hierarchical) with 13 tags. Owner and tabber are functionally identical. Limited is derived, not stored. Contact is metadata, not access control. See `role_permission_matrix.md` (Rebuild Recommendations).

### 5. State is Implicit
Tournament, round, ballot, and entry states are all inferred from related records — no explicit status columns. The rebuild should add explicit `status` enums. See `data_state_model.md` (State Storage Patterns).

### 6. Cross-Tenant Entities
Person and chapter records exist outside tournament scope. A person can have different roles at different tournaments simultaneously. See `tenancy_access_model.md` (Cross-Organization Access Patterns).

---

## Provenance

All artifacts were produced by analyzing the legacy `tabroom` repository (branch `master`, commit range up to `316357874`). No code was copied. Analysis methods:

- **Code reading**: Direct analysis of Perl/Mason source, SQL schema, ORM models
- **Pattern extraction**: Grep/glob across 2,400+ templates and 417 funclib files
- **Schema analysis**: 108-table schema with FK constraints and EAV patterns
- **Documentation cross-reference**: docs.tabroom.com, tournament-manual.pdf, in-code comments

Artifacts are clean-room descriptive: they document observable behavior, not implementation details suitable for direct porting.

---

## What Comes Next (Phase 5 — ScholarComp Repo)

The following work belongs in the ScholarComp repository, not here:

1. **Service boundary map** — assign each R1 capability to a ScholarComp service
2. **Frontend IA and screen inventory** — design the Next.js page structure
3. **Schema design** — convert data/state model to PostgreSQL DDL
4. **API contract planning** — define endpoints from workflow catalog
5. **Release 1 tranche plan** — sequence the 78 capabilities into buildable increments
6. **Agent-ready build briefs** — per-service implementation specs
