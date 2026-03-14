# Source Traceability Index

## Purpose

This document records where the current discovery corpus came from.

Its purpose is to preserve provenance across:

- clean-room requirements extraction,
- later ScholarComp-side mapping,
- future agent work that needs to verify whether a statement came from docs, schema, or code behavior.

This is a research aid, not a product requirements document.

## Source Categories

## 1. Public/User-Facing Documentation

Primary sources:

- `https://docs.tabroom.com/`
- `tabroom-docs` repository structure and markdown content
- `doc/howtos/tournament-manual.pdf`

Best use:

- role language
- workflow sequencing
- user-facing feature descriptions
- operational checklists
- menu/category structure

Known limitations:

- not guaranteed to cover every edge case
- may lag production behavior in specialized or legacy areas

## 2. Legacy Application Route Surface

Primary sources:

- `web/setup/*`
- `web/register/*`
- `web/panel/*`
- `web/tabbing/*`
- `web/user/*`
- `web/index/*`
- `web/api/*`

Best use:

- capability and surface inventory
- role/workflow decomposition
- operator-density workflows
- print/report surface discovery

Known limitations:

- route count overstates unique capabilities
- overlapping surfaces need interpretation rather than simple deduplication

## 3. Domain Models and ORM Layer

Primary sources:

- `web/lib/Tab/*.pm`

Best use:

- entity inventory
- key relationships
- setting scopes
- lifecycle hints
- embedded helper logic

Known limitations:

- business logic is split across models, Mason components, and direct SQL

## 4. Schema and Database Artifacts

Primary sources:

- `doc/sql/current-schema.sql`
- `doc/sql/fk.sql`

Best use:

- canonical table inventory
- actual columns and keys
- triggers and invariants
- result/protocol/settings structures
- local evidence for derived state

Known limitations:

- many important behaviors are settings-driven or query-driven rather than visible from schema alone

## 5. Workflow and Rule Code

Primary sources:

- `web/funclib/*`
- `web/panel/round/*`
- `web/panel/schemat/*`
- `web/tabbing/break/*`
- `web/tabbing/results/*`
- `web/tabbing/entry/*`

Best use:

- pairing logic
- strikes/conflicts/prefs behavior
- break generation
- results generation
- disaster/recovery operations
- ballot handling

Known limitations:

- logic is distributed and sometimes duplicated across specialized paths

## 6. Replacement/Rewrite Repositories

Primary sources:

- `tabroom-docs`
- `indexcards`
- `schemats`

Best use:

- current product terminology
- intended domain decomposition in the rewrite
- identifying what the original author considered important enough to modernize first

Known limitations:

- not the source of truth for full production parity
- should not override observed behavior in the legacy system

## Artifact-to-Source Map

## `FIRST_PASS_EXISTING_PLATFORM_CATALOG.md`

Primary source categories:

- public docs
- route surface
- schema
- domain models

Primary purpose:

- neutral inventory of what exists

## `LIGHTWEIGHT_PARITY_MATRIX.md`

Primary source categories:

- public docs
- route surface
- workflow code

Primary purpose:

- early Release 1 triage

## `GLOSSARY_FIRST_PASS.md`

Primary source categories:

- public docs
- UI labels in route surfaces
- domain model naming

Primary purpose:

- terminology stability

## `FIRST_PASS_WORKFLOW_CATALOG.md`

Primary source categories:

- public docs
- route surface
- workflow code

Primary purpose:

- actor/workflow sequencing

## `FIRST_PASS_CONFIGURATION_SETTINGS_SPEC.md`

Primary source categories:

- schema
- `web/lib/Tab/*Setting.pm`
- setting usage in route/workflow code

Primary purpose:

- make the configuration surface visible as a product domain

## `FIRST_PASS_ROLE_PERMISSION_MATRIX.md`

Primary source categories:

- `web/lib/Tab/Person.pm`
- `web/lib/Tab/Permission.pm`
- `web/autohandler`
- `web/funclib/perms/*`

Primary purpose:

- identify current access scopes and role-like permission tags

## `FIRST_PASS_CAPABILITY_MAP.md`

Primary source categories:

- platform catalog
- workflow catalog
- route surface

Primary purpose:

- normalize the capability landscape

## `FIRST_PASS_NON_FUNCTIONAL_REQUIREMENTS.md`

Primary source categories:

- workflow code
- disaster/recovery surfaces
- print/report surfaces
- live-operations assumptions in docs/manual

Primary purpose:

- preserve operational constraints before architecture design

## `FIRST_PASS_TENANCY_ACCESS_MODEL.md`

Primary source categories:

- permission model
- chapter/school/circuit/region/district relationships
- tournament-local versus durable entity structure

Primary purpose:

- early scoping of isolation and access assumptions

## `FIRST_PASS_DATA_STATE_MODEL.md`

Primary source categories:

- schema
- ORM layer
- route/workflow code for lifecycle hints

Primary purpose:

- identify core entities, relationships, and state transitions

## `FIRST_PASS_BUSINESS_RULES_CATALOG.md`

Primary source categories:

- pairing/break/result code
- ballot entry code
- disaster/recovery code
- protocol/tiebreak structures

Primary purpose:

- inventory the highest-value product logic

## Domain-to-Primary-Source Map

## Identity and access

Primary evidence:

- `person`
- `login`
- `session`
- `permission`
- `web/autohandler`
- `web/funclib/perms/*`

## Tournament setup and configuration

Primary evidence:

- `tourn`, `event`, `round`, `timeslot`, `site`, `room`
- `*_setting` tables
- `web/setup/*`

## Registration

Primary evidence:

- `entry`, `school`, `student`, `judge`, `judge_hire`, `invoice`, `fine`
- `web/register/*`

## Pairing and schematics

Primary evidence:

- `web/panel/round/*`
- `web/panel/schemat/*`
- `panel`, `ballot`, `round`, pool tables

## Ballots and live tabbing

Primary evidence:

- `web/tabbing/entry/*`
- `ballot`
- `score`
- `student_vote`
- `student_ballot`

## Results, breaks, awards

Primary evidence:

- `web/tabbing/results/*`
- `web/tabbing/break/*`
- `protocol`
- `tiebreak`
- `result_*`
- `sweep_*`

## District / qualifier / national specialization

Primary evidence:

- `district`
- `qualifier`
- NSDA-specific routes under `web/user/admin/nsda/*`
- district-specific settings and exports

## Known Confidence Levels

High confidence:

- broad capability inventory
- major entity clusters
- existence of multi-scope permissions
- existence of distributed settings/EAV pattern
- existence of strong event-type polymorphism
- existence of real-time operational and recovery workflows

Medium confidence:

- exact boundaries between some adjacent workflow families
- which niche features are widely used versus legacy residue
- which settings are mandatory versus optional in practice

Lower confidence until deeper extraction:

- exact numeric behavior of pairing heuristics
- all congress-specific rule differences
- all district/national qualification edge cases
- true commonality versus one-off specialization in long-tail settings

## How To Use This Index

When creating later requirements or implementation documents:

- prefer public docs for user intent and labels
- prefer schema for entity truth and invariants
- prefer route/workflow code for actual operational behavior
- when docs and code disagree, record the conflict rather than silently choosing one
- preserve the source category in downstream docs so clean-room provenance stays intact
