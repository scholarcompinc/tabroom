# Tabroom Rebuild Planning Document

## Purpose

This document defines the recommended approach for rebuilding the Tabroom platform as a new product on the ScholarComp V4 stack. It is intended to be reviewed and challenged before major implementation begins.

This plan assumes:

- The end state is a fully independent implementation owned by our team.
- The new product will be built on ScholarComp V4 patterns and constraints.
- Existing Tabroom assets are used as requirements and behavior references, not as the implementation base.
- We want to act as a good community member while preserving the option to commercialize our own implementation.

## Executive Summary

The recommended strategy is a clean-room, requirements-first rebuild on ScholarComp V4.

We should not anchor the program on Chris Palmer's rewrite stack (`indexcards` + `schemats`) or on the legacy Perl codebase. Those systems are valuable inputs, but not the target architecture.

The primary advantage available to us is not just modern infrastructure. It is the ability to:

- extract high-quality requirements from an existing mature product,
- normalize those requirements into agent-ready specifications,
- reuse ScholarComp's mature platform capabilities for commodity concerns,
- focus engineering effort on the domain-specific business rules that make tournament management hard.

This is a substantial undertaking, but it is tractable if we treat requirements extraction as the core enabling asset for AI-agent-driven delivery.

## Planning Assumptions

The items below are planning assumptions, not yet validated commitments.

- Workflow parity is more important than UI parity.
- Release 1 should prioritize the operational core over long-tail legacy breadth.
- A meaningful share of commodity platform capability can be reused from ScholarComp.
- Service boundaries are provisional until requirements and bounded contexts are clearer.
- Time and effort estimates should be treated as tranche-planning outputs, not fixed promises at the program-plan stage.

## Key Design Challenges To Preserve Explicitly

These concerns are important enough that they should not be lost during requirements extraction or early architecture work.

- multi-tenancy and tenant scoping,
- event-type polymorphism across debate, speech, and congress,
- real-time tournament operations and live state propagation,
- print and PDF generation for operational workflows,
- degraded-network tolerance for critical workflows, especially ballot entry,
- operational performance under live tournament conditions,
- adaptation cost of reusing existing ScholarComp services,
- access to domain experts for validation,
- agent coordination across parallel implementation streams.

## Program Goals

### Primary Goals

- Recreate the essential Tabroom product capabilities on ScholarComp V4.
- Preserve the option to monetize the resulting product.
- Achieve functional parity on the highest-value tournament workflows first.
- Use AI-agent-driven execution safely and efficiently by providing strong requirements inputs.

### Secondary Goals

- Reuse ScholarComp services and frontend capabilities wherever the fit is natural.
- Avoid unnecessary coupling to upstream Tabroom rewrite efforts.
- Maintain a credible, constructive posture toward the forensics community and upstream maintainers.

### Non-Goals

- Porting the Perl/Mason application directly.
- Rebuilding the product in Svelte or on the `indexcards`/`schemats` stack.
- Achieving 100% parity before first usable release.
- Reproducing legacy UX one-for-one where ScholarComp patterns can deliver a better result.

## Strategic Position

### Recommended Position

Build a fully independent implementation in ScholarComp while using existing Tabroom materials as:

- product requirements sources,
- workflow references,
- terminology references,
- behavior validation sources,
- parity benchmarks.

### Why Not Depend on the Upstream Rewrite

- We need long-term product ownership and strategic autonomy.
- Monetization is cleaner if the core implementation is our own.
- Upstream rewrite progress is likely constrained by solo-builder bandwidth and legacy support burden.
- Our team can move faster if we front-load requirements and let agents execute against them.
- ScholarComp already provides many of the platform capabilities the product needs.

### How To Still Be a Good Community Member

We can contribute upstream selectively without making our core product dependent on upstream:

- docs fixes and clarifications,
- bug reports with strong repros,
- API contract feedback,
- narrow backend or docs contributions where strategically acceptable,
- interoperability-friendly ideas that help the ecosystem.

We should not treat upstream repos as our primary platform dependency.

## Clean-Room Working Rules

This project should follow a clean-room discipline.

### Allowed Inputs

- Public documentation from `docs.tabroom.com` / `tabroom-docs`
- Public repo structure and behavior analysis
- Schema analysis
- Workflow observation
- OpenAPI and public API contract analysis
- User manuals and operator docs
- Product terminology and domain concepts

### Restricted Inputs

- Do not transplant source code into ScholarComp services or frontend.
- Do not copy templates, page implementations, or internal code structures.
- Do not treat the existing implementation as a scaffold for our code.

### Required Practice

For each major feature, maintain:

- source references,
- normalized product requirements,
- ScholarComp-specific design notes,
- acceptance criteria,
- parity notes,
- implementation decisions.

This keeps the build defensible, reviewable, and usable by agents.

### Legal Review Requirement

Clean-room discipline reduces risk, but it does not eliminate the need for legal review.

Before Phase 1 begins in earnest, obtain legal review of:

- the intended commercialization posture,
- acceptable use of open-source source materials as requirements inputs,
- any contribution strategy to upstream repos,
- branding, interoperability, and packaging implications.

## Source Inventory

The current source set for requirements extraction is:

1. `tabroom` legacy Perl/Mason repo
2. `tabroom-docs` repo and `docs.tabroom.com`
3. `indexcards` backend rewrite repo
4. `schemats` frontend rewrite repo
5. `tournament-manual.pdf`
6. Live product behavior where observable

### How Each Source Should Be Used

#### Legacy `tabroom`

Use as the deepest source of actual business behavior, hidden edge cases, settings, workflows, and data structures.

Most important value:

- pairing logic,
- prefs and strikes,
- room/judge constraints,
- tabbing rules,
- break logic,
- publication logic,
- permissions,
- reporting/export behaviors.

#### `tabroom-docs`

Use as the strongest source of current user-facing workflows, menu structure, terminology, role framing, and happy-path task guidance.

#### `indexcards`

Use as a signal of domain decomposition, evolving API boundaries, and modernized understanding of the product. It is informative, not authoritative for our implementation.

#### `schemats`

Use as a signal of intended modern UX directions and frontend surface prioritization. It is not our frontend target.

#### `tournament-manual.pdf`

Use as historical workflow and domain-rule context, especially where newer docs are sparse.

## Product Rebuild Principles

### Principle 1: Product First, Stack Second

We are not rewriting a codebase. We are re-specifying and rebuilding a product.

### Principle 2: Use ScholarComp For Commodity Capability

Do not spend early cycles rebuilding benign platform concerns that ScholarComp already handles well.

Likely reuse candidates:

- auth and user identity,
- notifications,
- search,
- school and participant primitives,
- content/public pages,
- audit patterns,
- admin console foundations,
- feature flags,
- BFF and contract patterns.

### Principle 3: Spend Custom Effort On Domain-Specific Value

The hard parts are:

- pairing and schematics,
- judge assignment and optimization,
- judge prefs/conflicts/strikes,
- ballot capture and validation,
- tabulation and standings,
- tiebreak logic,
- break logic,
- qualification and advancement rules,
- tournament operations overrides and recovery.

Do not flatten the domain too early.

Debate, speech, and congress are not just presentation variants. They differ materially in:

- entry structure,
- pairing/grouping logic,
- ballot structure,
- scoring model,
- results computation,
- advancement and awards behavior.

Requirements and architecture work must preserve this polymorphism explicitly.

### Principle 4: Requirements Must Be Agent-Ready

Requirements are not complete unless an AI agent can execute against them with minimal ambiguity.

Each feature spec should include:

- user roles,
- entry points,
- state changes,
- business rules,
- edge cases,
- acceptance criteria,
- API expectations,
- UI expectations,
- observability/testing notes.

### Principle 5: Prefer Fewer Larger Services In Release 1

Do not over-split the tournament domain early.

For Release 1, a new service boundary is justified only when at least one of the following is true:

- it has clearly distinct ownership,
- it has materially different scaling characteristics,
- it needs an independent deployment cadence,
- it creates a significantly simpler domain boundary than keeping it together.

Otherwise, prefer fewer, larger services until the product behavior is validated.

## Recommended Product Surface Split

Use ScholarComp's existing frontend model instead of copying Tabroom's frontend organization.

### Main App

Use for:

- public tournament discovery,
- public tournament pages,
- results and published records,
- coach self-service flows,
- judge self-service flows,
- student self-service flows,
- paradigms and profiles,
- notifications and lightweight dashboards.

### PMC / Admin Console

Use for:

- tournament setup,
- schedule and event configuration,
- registration operations,
- judge pool management,
- room management,
- paneling,
- schematics,
- tabbing,
- audit tools,
- operator overrides,
- disaster recovery workflows,
- exports and reporting.

This split is likely better aligned to the density of the domain than a single-surface UI.

## Recommended Backend Service Strategy

Map cleanly to ScholarComp where possible, and create new services only where the tournament domain truly needs them.

### Likely Existing Service Reuse

- `user-service`
- `competition-service`
- `event-service`
- `school-service`
- `participant-service`
- `team-service`
- `registration-service`
- `notification-service`
- `calendar-service`
- `audit-service`
- `search-service`

### Likely New Domain-Specific Services

These names are provisional and should be confirmed during architecture definition.

- `tournament-operations-service`
- `pairing-service`
- `judge-assignment-service`
- `ballot-service`
- `tabulation-service`
- `results-publication-service`
- `sweepstakes-service`

Alternative: some of these may collapse into fewer services for the first release if service granularity would otherwise create too much coordination overhead.

Release 1 bias:

- prefer fewer larger services,
- bias toward correctness and delivery speed over architectural purity,
- split later when product behavior and operational load justify it.

## Program Phases

### Phase 0: Planning and Guardrails

Goal:

- confirm strategy,
- define clean-room rules,
- define artifact structure,
- decide product boundaries and first-release posture,
- establish testing guardrails,
- answer the load-bearing open questions,
- complete legal review.

Outputs:

- this planning doc,
- clean-room guidance,
- repo/document structure for requirements,
- tranche strategy,
- decision log template,
- testing guardrails,
- initial lightweight parity matrix,
- glossary starter,
- pre-Phase-1 decision outcomes.

### Phase 1: Requirements Corpus Build

Goal:

- extract and normalize product requirements from source materials.

Outputs:

- existing platform catalog,
- capability map,
- role matrix,
- workflow catalog,
- business rules catalog,
- configuration and settings spec,
- data entity dictionary,
- state model definitions,
- tenancy and access model,
- non-functional requirements catalog,
- ScholarComp reuse fitness assessment,
- parity matrix,
- glossary and terminology map,
- source traceability index.

This phase is the key enabler for AI-agent throughput.

### Phase 2: Target Architecture Definition

Goal:

- map the product to ScholarComp frontend and backend boundaries.

Outputs:

- bounded context map,
- frontend surface map,
- service ownership map,
- integration contracts,
- event model outline,
- real-time interaction strategy,
- print and document generation decision,
- degraded-network handling strategy for critical flows,
- performance budgets and operational constraints,
- migration and coexistence assumptions.

### Phase 3: Tranche Planning

Goal:

- define what to build first and in what sequence.

Outputs:

- Release 1 tranche list,
- dependency graph,
- critical-path features,
- risk-based sequencing,
- confidence levels and major assumptions,
- UX/API/test deliverables by tranche.

### Phase 4: Agent-Ready Brief Production

Goal:

- convert tranche items into highly actionable build briefs.

Outputs:

- implementation briefs,
- acceptance scenarios,
- contract expectations,
- testing instructions,
- open questions register,
- agent coordination and integration rules.

### Phase 5: Build and Validate

Goal:

- implement in ScholarComp and validate against requirements.

Outputs:

- services,
- frontend flows,
- tests,
- scenario evidence,
- parity validation notes.

## Phase 0 Guardrails

The following guardrails should be explicit before feature build-out begins.

### Testing Guardrails

Every agent-ready brief should require tests as part of the implementation, not as follow-up work.

Minimum expectations:

- backend: pytest coverage for business rules, API behavior, and edge cases,
- frontend: Vitest or equivalent component/unit coverage where appropriate,
- frontend workflows: Playwright coverage for critical operator and self-service flows,
- acceptance criteria mapped directly to test cases,
- scenario evidence for high-risk tournament workflows.

### Requirements Guardrails

- No build brief without explicit acceptance criteria.
- No major service design before bounded-context review.
- No algorithm implementation before a business-rules spec exists for it.
- No UI work for dense operator workflows without an IA decision on Main App vs PMC.
- No assumption of ScholarComp service reuse without a reuse fitness assessment.
- No flattening of debate, speech, and congress into a single rules model without explicit justification.

### Parity Guardrails

Start parity tracking immediately, even if the first version is lightweight.

Minimum parity matrix columns:

- capability,
- source reference,
- Release 1 required,
- notes/open questions.

## Requirements Document Set

The recommended document set is below.

### 1. Existing Platform Catalog

A source-grounded catalog of what exists today before any ScholarComp mapping begins.

It should inventory:

- user-facing capability areas,
- operator-facing capability areas,
- major workflows,
- route/menu surfaces,
- entity/model surfaces,
- settings/configuration surfaces,
- reports/exports,
- integrations,
- notable algorithmic/business-rule areas,
- known documentation coverage and gaps.

This document should stay descriptive, not prescriptive. Its purpose is to answer "what exists now?" rather than "how should we rebuild it?"

### 2. Capability Map

A top-level product decomposition of all major capabilities and sub-capabilities.

### 3. Role and Permission Matrix

Roles, scopes, privileges, and notable restrictions.

### 4. Workflow Catalog

One document or section per major workflow.

Examples:

- create a tournament,
- configure events and schedule,
- open registration,
- enter schools/entries/judges,
- collect prefs,
- pair rounds,
- assign judges and rooms,
- submit ballots,
- audit ballots,
- tabulate and break,
- publish results.

### 5. Business Rules Catalog

The most important artifact for the hard domain logic.

### 6. Data and State Model Pack

Entities, relationships, lifecycle states, transitions, invariants, and audit requirements.

### 7. Tenancy and Access Model

Define:

- tenant boundary candidates,
- access scopes,
- cross-tournament and cross-organization visibility rules,
- implications for auth and data access,
- unresolved tenancy decisions.

### 8. Configuration and Settings Spec

This should be a first-class artifact.

It should define:

- tournament-level settings,
- event/category-level settings,
- scoring and rules configuration,
- defaults vs overrides,
- typed representations replacing legacy EAV patterns,
- which settings are Release 1 vs later.

### 9. Non-Functional Requirements Catalog

Define non-functional requirements that materially shape the product.

Suggested categories:

- latency and throughput expectations for live operations,
- real-time update requirements,
- degraded-network tolerance,
- mobile usability for judges,
- print/document generation needs,
- reliability and recoverability,
- auditability,
- concurrency/load expectations during publish events.

### 10. ScholarComp Reuse Fitness Assessment

For each candidate reused service, capture:

- current capability,
- required tournament-domain capability,
- fit assessment,
- extension cost/risk,
- recommendation: reuse, extend, wrap, or avoid.

### 11. Frontend IA and Screen Inventory

Map each workflow to Main App or PMC, and define the required screens, route groups, and user states.

### 12. Service Boundary Map

Map each capability to ScholarComp services, including any new tournament-specific services.

### 13. Parity Matrix

Track:

- legacy product feature,
- source evidence,
- required in Release 1 or later,
- target service/frontend owner,
- status.

### 14. Glossary and Terminology Map

Track Tabroom terminology, ScholarComp terminology, and any normalized product language adopted for the rebuild.

## Release Strategy Recommendation

Do not chase complete parity first.

Use a staged release model.

### Release 1 Recommendation

Target a usable operating core:

- tournament creation and setup,
- event/schedule definition,
- school and entry registration,
- judge registration and management,
- core paneling/schematics,
- ballot submission,
- tabulation,
- break handling,
- results publication.

### Release 2 Candidates

- advanced reports and exports,
- specialized formats,
- housing,
- deep finance workflows,
- niche league features,
- secondary admin utilities,
- rare override tools,
- migration and interoperability tooling,
- import/export adapters for historical data,
- stronger degraded-network/offline affordances if not fully addressed in Release 1.

### Release 3 Candidates

- low-frequency edge workflows,
- advanced historical analytics,
- legacy compatibility affordances,
- long-tail configuration features.

## Key Risks

### Risk 1: Underestimating Domain Complexity

Mitigation:

- treat business-rules extraction as first-class work,
- use parity matrices,
- validate with operators early.

### Risk 2: Over-Splitting Services Too Early

Mitigation:

- optimize for product throughput and coherence first,
- defer service fragmentation where not clearly valuable.

### Risk 3: Requirements Ambiguity For Agents

Mitigation:

- standardize feature brief format,
- require edge cases and acceptance criteria,
- maintain traceability to evidence.

### Risk 4: UX Complexity In High-Density Operator Flows

Mitigation:

- put dense workflows in PMC,
- explicitly design operator tooling rather than adapting consumer UX patterns.

### Risk 5: Clean-Room Drift

Mitigation:

- keep source references at requirement level,
- avoid implementation borrowing,
- review artifacts for provenance discipline.

### Risk 6: Building Too Much Commodity Functionality

Mitigation:

- reuse ScholarComp primitives aggressively where the fit is acceptable,
- reserve bespoke effort for tournament-specific logic.

### Risk 7: Licensing or Commercialization Assumptions Are Wrong

Mitigation:

- require explicit legal review before Phase 1,
- maintain source provenance in requirements artifacts,
- keep implementation clean-room discipline visible and auditable.

### Risk 8: Multi-Tenancy Assumptions Are Wrong

Mitigation:

- define tenancy and access model in Phase 1,
- make tenant boundaries explicit before service contracts harden,
- validate auth and visibility rules early.

### Risk 9: Reuse Assumptions Hide Large Adaptation Costs

Mitigation:

- complete a ScholarComp reuse fitness assessment before architecture hardens,
- avoid assuming “service exists” means “service fits.”

### Risk 10: Lack of Domain Expert Access Slows Validation

Mitigation:

- identify validation channels early,
- secure operator review paths before high-risk algorithms are implemented,
- treat missing domain access as a program risk, not a later inconvenience.

## Community Contribution Posture

Recommended stance:

- be helpful upstream where doing so creates goodwill and ecosystem value,
- do not anchor our product roadmap on upstream delivery,
- do not give away core differentiating implementation assets.

Safe contribution areas:

- documentation improvements,
- bug reports,
- API feedback,
- narrow fixes,
- test cases and clarifications,
- interoperability discussions.

Protected areas:

- our frontend experience,
- our service architecture,
- our agent-driven delivery methods,
- our core business-rule implementation,
- our product strategy and release sequencing.

## Decision Gates Before Major Build-Out

The following decisions should be explicitly reviewed and approved.

### Gate 0: Legal Review

Confirm that legal review has been completed for the intended commercialization and clean-room posture.

### Gate 1: Strategy Approval

Confirm that the program is a full independent rebuild on ScholarComp, not an integration-led dependency strategy.

### Gate 2: Clean-Room Discipline

Confirm acceptable input sources and working rules.

### Gate 3: Market and Packaging Model

Confirm the intended commercialization model and product packaging.

Questions to resolve:

- SaaS, hosted enterprise, open-core, or another model?
- ScholarComp-branded module or standalone product identity?
- single product surface or separate product/subdomain strategy?

### Gate 4: Tenancy Model

Confirm the initial tenancy and access model assumptions.

Questions to resolve:

- what is the effective tenant boundary,
- which roles can see across tournaments or organizations,
- how tenancy interacts with ScholarComp identity and access patterns.

### Gate 5: Parity Standard

Confirm workflow parity vs UI parity as the governing standard.

### Gate 6: Frontend Surface Split

Confirm Main App vs PMC responsibilities.

### Gate 7: Configuration Strategy

Confirm configuration philosophy.

Questions to resolve:

- which legacy settings are essential,
- where opinionated defaults are acceptable,
- where full configurability is non-negotiable.

### Gate 8: Reuse Strategy

Confirm that candidate ScholarComp service reuse has been assessed for fitness rather than assumed.

### Gate 9: Service Boundary Strategy

Confirm which existing ScholarComp services are reused and which new tournament services are introduced.

### Gate 10: Domain Validation Access

Confirm how domain experts, operators, or community reviewers will validate high-risk workflows.

### Gate 11: Release 1 Scope

Confirm which capabilities are truly first-release mandatory.

## Recommended Immediate Next Steps

1. Approve or revise this planning approach.
2. Create the requirements artifact structure and templates.
3. Produce the Existing Platform Catalog before any ScholarComp mapping work.
4. Produce the first lightweight parity matrix from `tabroom-docs`, the legacy repo, and the manual.
5. Produce the top-level capability map, role matrix, and glossary starter.
6. Add the Configuration and Settings Spec, Tenancy and Access Model, and Non-Functional Requirements Catalog to the immediate requirements work queue.
7. Complete the first-pass ScholarComp reuse fitness assessment.
8. Identify the Release 1 core workflows.
9. Start converting those workflows into agent-ready briefs.

## Open Questions

These questions should be resolved in Phase 0 and converted into decision outcomes before Phase 1 begins.

- What exact market and monetization model is envisioned?
- Is workflow parity enough, or are there areas where UI parity is strategically necessary?
- What is the acceptable degree of backward compatibility for terminology, workflows, data imports, and exports?
- Should tournament-specific functionality live under the ScholarComp brand architecture or as a product-branded subdomain/app?
- Which rare but mission-critical workflows must be in Release 1?
- How much configuration flexibility is required versus opinionated defaults?
- What is the initial multi-tenancy and access model?
- Which real-time behaviors are Release 1 requirements versus later enhancements?
- Which print/PDF outputs are operationally mandatory in Release 1?
- What degraded-network tolerance is required for ballot entry and other critical flows?
- Which ScholarComp services are genuinely reusable versus superficially adjacent?
- What operator or domain-expert validation access is available to the program?

## Legacy Scope Reference

These metrics are useful for planning and triage, but they should not be mistaken for one-to-one build targets.

- legacy UI pages: approximately 1,800+
- funclib components: approximately 400+
- ORM models: approximately 100+
- business-rule-heavy utility code: tens of thousands of lines
- configuration/settings footprint: very large and distributed across many legacy settings constructs

The modern rebuild should consolidate and normalize this significantly.

## Glossary Starter

This is a seed only. The full glossary should live in the requirements corpus.

| Tabroom Term | ScholarComp-Oriented Term | Notes |
|---|---|---|
| Tournament / Tourn | Competition / Tournament | Keep final naming decision explicit |
| Category | Division / Category | May differ by workflow context |
| Entry | Team / Participant Entry | Needs precise modeling by event type |
| Panel / Section | Section / Pairing Unit | Legacy meaning varies by format |
| Schemat / Schematic | Pairing / Bracket / Assignment View | Likely a UI term more than a domain entity |
| Break | Advancement / Cut | Preserve legacy meaning in docs where needed |
| Paradigm | Judge Philosophy | User-facing wording may stay legacy |
| JPool | Judge Pool | Direct mapping is likely fine |
| RPool | Room Pool | Direct mapping is likely fine |
| Sweepstakes | Team Awards / Sweepstakes | Keep both terms in glossary |

## Agent Coordination Notes

Because this program depends on parallel agent execution, coordination rules should be explicit once build briefs begin.

At minimum:

- backend and frontend briefs should align to explicit contracts, not implicit expectations,
- cross-service work should define ownership of the contract source of truth,
- integration validation should be part of tranche completion, not deferred,
- adjacent completed briefs should be checked for compatibility before close-out.

## Recommendation

Proceed with a clean-room, requirements-first rebuild on ScholarComp V4.

Use upstream Tabroom assets as requirements inputs and ecosystem references, not as the implementation platform. Build the requirements corpus before significant coding. Treat the business rules catalog and parity matrix as the most important assets in the program.
