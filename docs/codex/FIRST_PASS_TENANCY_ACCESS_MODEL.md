# First-Pass Tenancy and Access Model

## Purpose

This document captures the first-pass access and scope model visible in the current Tabroom product before any ScholarComp tenancy design begins.

This is not yet a SaaS tenancy architecture. It is a descriptive view of how access and authority appear to be partitioned in the current system.

## Source Basis

Primary evidence used:

- `web/lib/Tab/Person.pm`
- `web/lib/Tab/Permission.pm`
- `web/autohandler`
- `web/funclib/perms/*`
- role-specific route surfaces

## First-Pass Model

The current product appears to organize authority through overlapping scopes rather than an explicit modern tenant abstraction.

Observed scopes:

- platform/global
- tournament
- event
- category
- chapter
- circuit
- region
- district

Observed implication:

- access is scoped by organizational or tournament context rather than by a clearly explicit commercial tenant entity

## Scope Observations

## Global Scope

- site admins exist
- site admins effectively inherit owner-level tournament access
- admin-host checks restrict some surfaces to site admins

## Tournament Scope

- central operational scope
- permissions can be broad (`owner`, `tabber`, `contact`, `checker`) or limited via event/category
- most live operational flows appear tournament-scoped

## Organizational Scope

- chapters, circuits, regions, and districts all appear as separate access scopes
- these likely represent persistent organizational authority that cuts across tournaments

## Event / Category Scope

- current system supports event- and category-limited tournament access
- suggests some users can operate only on slices of a tournament rather than full tournament control

## Current-Product Access Questions Relevant To Later Tenancy Design

- Is the effective “tenant” concept closer to chapter, circuit, tournament, or some combination?
- Which roles need cross-tournament visibility?
- Which roles need cross-organization visibility?
- Which organizational scopes are operationally mandatory versus legacy administrative structure?
- Which data is effectively shared/public versus scoped/private?

## High-Confidence Takeaways

- the current product is multi-scope, not single-scope
- tournament is the primary operational boundary
- organizational entities above the tournament matter materially
- later SaaS tenancy design should not assume “one tournament equals one tenant” without deliberate analysis

## Follow-On Work

1. Cross-reference this with the role matrix and workflow catalog
2. Determine which current scopes are true business requirements versus legacy structure
3. Resolve how Release 1 should treat:
   - cross-tournament organization views
   - district/circuit authority
   - school/chapter ownership boundaries
