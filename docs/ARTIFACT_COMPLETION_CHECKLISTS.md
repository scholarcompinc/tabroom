# Artifact Completion Checklists

## Purpose

This document defines what “done enough to use” means for each major artifact in the Tabroom rebuild discovery and planning process.

Use this file when reviewing whether an artifact is ready to:

- inform release scoping,
- support ScholarComp mapping,
- drive agent build briefs.

## 1. Existing Platform Catalog

- [ ] Major capability domains documented
- [ ] Role inventory documented
- [ ] Workflow families documented
- [ ] Route/surface inventory documented
- [ ] Major entity clusters documented
- [ ] Settings/configuration surfaces documented
- [ ] Reports/exports documented
- [ ] Integrations documented
- [ ] Algorithmically complex areas documented
- [ ] Documentation coverage assessed
- [ ] Source references included throughout
- [ ] Unknowns explicitly captured

Ready when:

- A reviewer can answer “what exists today?” without jumping straight into code

## 2. Capability Map

- [ ] Top-level domains defined
- [ ] Key sub-capabilities defined
- [ ] Overlaps noted
- [ ] Release 1 candidate areas visible
- [ ] Frequency/criticality signals included where possible

Ready when:

- A reviewer can see the product shape at a glance

## 3. Role and Permission Matrix

- [ ] Public roles documented
- [ ] Self-service roles documented
- [ ] Operator/admin roles documented
- [ ] Scope restrictions documented
- [ ] Cross-tournament/cross-organization questions flagged

Ready when:

- A reviewer can tell who does what and where boundaries are unclear

## 4. Workflow Catalog

- [ ] Setup workflows documented
- [ ] Registration workflows documented
- [ ] Judge/prefs/conflict workflows documented
- [ ] Pairing/paneling workflows documented
- [ ] Ballot and audit workflows documented
- [ ] Results/breaks/publication workflows documented
- [ ] Recovery/correction workflows documented
- [ ] Debate/speech/congress divergence preserved

Ready when:

- A reviewer can understand end-to-end operational behavior without reading implementation code

## 5. Business Rules Catalog

- [ ] Pairing logic clusters documented
- [ ] Judge assignment logic documented
- [ ] Conflict/strike/prefs logic documented
- [ ] Ballot validation/scoring logic documented
- [ ] Tiebreak logic documented
- [ ] Break/advancement logic documented
- [ ] Sweepstakes logic documented
- [ ] Secondary algorithms documented
- [ ] Ambiguous rules marked for expert review

Ready when:

- High-risk algorithms can be discussed and tested as requirements, not folklore

## 6. Data and State Model Pack

- [ ] Major entities documented
- [ ] Major relationships documented
- [ ] Lifecycle states documented
- [ ] Key transitions documented
- [ ] Invariants documented
- [ ] Settings/EAV cluster documented
- [ ] Format polymorphism reflected

Ready when:

- A reviewer can reason about product state and lifecycle without designing the new schema yet

## 7. Tenancy and Access Model

- [ ] Tenant boundary candidates documented
- [ ] Access scope candidates documented
- [ ] Visibility rules documented
- [ ] Cross-org/cross-tournament access questions documented
- [ ] Assumptions vs confirmed decisions separated

Ready when:

- Architecture work can proceed without hidden tenancy assumptions

## 8. Configuration and Settings Spec

- [ ] Settings grouped by scope
- [ ] Tournament-level settings documented
- [ ] Event/category settings documented
- [ ] Rules/scoring settings documented
- [ ] Registration/financial/publication settings documented
- [ ] Release 1 critical settings marked
- [ ] Simplification candidates clearly separated from must-preserve settings

Ready when:

- The configuration surface is visible enough to constrain API and UI design later

## 9. Non-Functional Requirements Catalog

- [ ] Real-time requirements documented
- [ ] Performance expectations documented
- [ ] Publish/read spike behavior documented
- [ ] Degraded-network considerations documented
- [ ] Ballot-entry reliability expectations documented
- [ ] Print/PDF requirements documented
- [ ] Auditability/recoverability expectations documented

Ready when:

- The product’s live-operations constraints are explicit

## 10. ScholarComp Reuse Fitness Assessment

- [ ] Candidate reused services listed
- [ ] Current capability summarized
- [ ] Tournament-domain needs summarized
- [ ] Fit/gap analysis recorded
- [ ] Recommendation made: reuse, extend, wrap, or avoid

Ready when:

- “Existing service” no longer gets mistaken for “good fit”

## 11. Parity Matrix

- [ ] Major capabilities listed
- [ ] Source references included
- [ ] Release 1 required flags included
- [ ] Notes/open questions included
- [ ] Later-phase candidates marked

Ready when:

- Release triage can be evidence-based

## 12. Glossary and Terminology Map

- [ ] Core tournament terms included
- [ ] Role terms included
- [ ] Pairing/results terms included
- [ ] Ambiguous terms flagged
- [ ] ScholarComp-oriented language noted where relevant

Ready when:

- Humans and agents can speak consistently about the domain

## 13. ScholarComp Handoff Package

- [ ] Discovery artifacts assembled
- [ ] Provenance preserved
- [ ] Descriptive vs prescriptive artifacts clearly labeled
- [ ] Handoff scope documented

Ready when:

- The ScholarComp repo can take over architecture and build planning without redoing discovery

## 14. Agent-Ready Build Briefs

- [ ] Scope clearly bounded
- [ ] Contracts explicit
- [ ] Rules explicit
- [ ] UI expectations explicit
- [ ] Tests required
- [ ] Acceptance criteria explicit
- [ ] Integration dependencies explicit

Ready when:

- An implementation agent can execute without guessing about adjacent behavior
