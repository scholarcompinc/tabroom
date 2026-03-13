# Tenancy and Access Model

## Purpose

This document describes the tenancy boundaries, access scopes, and visibility rules for the Tabroom rebuild. It captures how data is isolated, shared, and accessed across organizational boundaries — decisions that fundamentally shape the database schema, API design, and authorization middleware.

---

## Tenant Boundary Candidates

### Option A: Tournament as Primary Tenant (Recommended for Release 1)

**Description:** Each tournament is the primary isolation boundary. All operational data (events, rounds, panels, ballots, scores, results) belongs to exactly one tournament.

**Pros:**
- Matches the legacy model exactly
- Natural isolation — tournaments don't share operational data
- Simple query patterns: most queries are scoped by `tourn_id`
- Easy to reason about data lifecycle (tournament created → tournament archived)

**Cons:**
- Cross-tournament features (circuits, season awards, judge history) require cross-tenant queries
- Shared entities (person, chapter, chapter_judge) exist outside tournament scope
- Schema-per-tenant would be overkill at the Tabroom scale

**Implementation:** Single database with `tournament_id` as a standard filter on all tournament-scoped tables. Use row-level security or application-level middleware for enforcement.

### Option B: Organization as Primary Tenant

**Description:** Each organization (chapter/school) is the primary tenant.

**Assessment:** Does not fit. Tournaments involve many organizations, and the primary operational workflows are tournament-centric, not org-centric. Rejected.

### Option C: Circuit as Primary Tenant

**Description:** Each circuit is the primary tenant.

**Assessment:** Partially fits for season-long operations but not for individual tournaments (which can span multiple circuits). Not suitable as primary tenant. Rejected.

### Recommendation

**Tournament as primary operational boundary** with **shared reference data** (persons, chapters, circuits) outside the tournament scope. This matches both the legacy model and the natural operational model.

---

## Access Scope Model

### Scope Levels

```
┌─────────────────────────────────────────────┐
│  GLOBAL (site admin)                         │
│  ┌────────────────────────────────────────┐  │
│  │  CIRCUIT / DISTRICT / REGION           │  │
│  │  ┌─────────────────────────────────┐   │  │
│  │  │  TOURNAMENT                      │   │  │
│  │  │  ┌──────────────────────────┐    │   │  │
│  │  │  │  CATEGORY                 │    │   │  │
│  │  │  │  ┌───────────────────┐    │    │   │  │
│  │  │  │  │  EVENT             │    │    │   │  │
│  │  │  │  │  ┌────────────┐    │    │    │   │  │
│  │  │  │  │  │  ROUND      │    │    │    │   │  │
│  │  │  │  │  └────────────┘    │    │    │   │  │
│  │  │  │  └───────────────────┘    │    │   │  │
│  │  │  └──────────────────────────┘    │   │  │
│  │  │                                   │   │  │
│  │  │  CHAPTER (school at tournament)   │   │  │
│  │  │  ┌──────────────────────────┐    │   │  │
│  │  │  │  ENTRY / JUDGE           │    │   │  │
│  │  │  └──────────────────────────┘    │   │  │
│  │  └─────────────────────────────────┘   │  │
│  └────────────────────────────────────────┘  │
└─────────────────────────────────────────────┘
```

### Scope Definitions

| Scope | Description | Example Entities | Access Pattern |
|-------|-------------|-----------------|----------------|
| **Global** | All data across all scopes | Person, TabroomSetting | Site admin only |
| **Circuit** | Cross-tournament organizational grouping | Circuit, ChapterCircuit, TournCircuit | Circuit admins |
| **District** | NSDA qualification region | District, Qualifier | District chairs |
| **Region** | Sub-circuit geographic area | Region, RegionSetting | Region admins |
| **Tournament** | Single tournament and all its data | Tourn, Event, Round, Panel, Ballot, etc. | Tournament staff |
| **Category** | Event grouping within tournament | Category, JPool, CategorySetting | Scoped tabbers |
| **Event** | Single competitive division | Event, Round, Entry | Scoped tabbers |
| **Round** | Single round of competition | Round, Panel, Ballot | Scoped tabbers |
| **Chapter** | School registration within tournament | School, Entry, Judge | Coaches |
| **Entry/Judge** | Individual competitive or judging unit | Entry, Judge, Ballot | Participants |

---

## Visibility Rules

### Tournament Operational Data

| Data Type | Public | Student | Judge | Coach | Tabber | Owner |
|-----------|--------|---------|-------|-------|--------|-------|
| Tournament exists & info | Y | Y | Y | Y | Y | Y |
| Event list & descriptions | Y | Y | Y | Y | Y | Y |
| Schedule/timeslots | Y | Y | Y | Y | Y | Y |
| Registered schools list | Configurable | Y | Y | Y | Y | Y |
| Entry list | When published | Own | Own panel | Own school | Y | Y |
| Judge list | When published | N | Own schedule | Own school | Y | Y |
| Schematics (pairings) | When published | Own rounds | Own rounds | Own school | Y | Y |
| Ballot scores | When published | Own entry | Own ballots | Own school | Y | Y |
| RFDs/comments | When published | Own entry | Own ballots | Own school | Y | Y |
| Results/standings | When published | Y | Y | Y | Y | Y |
| Speaker awards | When published | Y | Y | Y | Y | Y |
| Invoices/financials | N | N | N | Own school | Y | Y |
| Fines | N | N | N | Own school | Y | Y |

### Configurable Visibility Controls

| Control | Setting | Effect |
|---------|---------|--------|
| Blind mode | `blind_mode` (event) | Judge names hidden from public postings |
| Anonymous results | `anonymous_public` (event) | Entry identities hidden in public results |
| Hide codes | `hide_codes` (tourn) | Entry codes hidden from public |
| Hide deadlines | `hide_deadlines` (tourn) | Deadlines hidden from registration |
| No opponent results | `no_opponent_results` (event) | Opponent records hidden from results |
| Publish schools | `publish_schools` (tourn) | School list visibility |

### Cross-Tournament Data Visibility

| Data Type | Visibility Rule |
|-----------|----------------|
| Person profile | Visible to self; visible to chapter admins of their chapters; visible to tournament staff at tournaments where they participate |
| Judge paradigm | Public (searchable) |
| Judge record | Public (aggregated across tournaments) |
| Student record | Visible to coaches/chapter admins of their chapter |
| Chapter roster | Visible to chapter admins |
| Circuit tournaments | Public (listing) |
| Circuit results | Public (aggregated) |
| NSDA qualification status | Visible to district admins and the student's chapter |

---

## Cross-Organization Access Patterns

### Pattern 1: Multi-Tournament Person
A single `person` can have roles across many tournaments simultaneously:
- Judge at Tournament A
- Coach at Tournament B
- Owner at Tournament C
- Student at Tournament D (rare but possible)

**Implication:** Person and session are outside tournament scope. The home screen must aggregate across all tournament relationships.

### Pattern 2: Multi-Chapter Person
A single `person` can admin multiple chapters:
- Coach at School X and School Y
- This is common for coaches who move schools or help at multiple programs

**Implication:** Chapter permission is per-chapter, not one-to-one.

### Pattern 3: Judge Across Schools
A judge registered by School X can be "hired" by School Y:
- The `judge_hire` system handles this cross-school sharing
- The judge's person record stays linked to their original chapter

**Implication:** Judge-to-school relationships are many-to-many within a tournament context.

### Pattern 4: Circuit-Level Aggregation
Circuit admins need to see aggregated data across all tournaments in their circuit:
- Season-long results
- Qualification standings
- Tournament administration history

**Implication:** Circuit-scoped queries must join across tournaments. This is a performance consideration.

### Pattern 5: District Qualification Pipeline
District operations span multiple tournaments and feed into nationals:
- District tournament results → qualification
- Qualification status → nationals registration
- Cross-district at-large allocations

**Implication:** Qualification tracking requires a cross-tournament entity that persists across the qualification season.

---

## Data Isolation Requirements

### Hard Isolation (Must Enforce)

1. **Ballot scores are only visible to authorized roles** until published
2. **Unpublished results are only visible to tabbers/owners**
3. **Financial data (invoices, fines, payments) is scoped to school** — coaches see only their own
4. **Judge prefs/strikes are not visible across schools** — each coach sees only their own ratings
5. **Personal data (email, phone, address) is not publicly visible**
6. **Admin operations (SU, user management) are restricted to site_admin**

### Soft Isolation (Application-Level)

1. **Tournament A's operations don't affect Tournament B** — but shared person/chapter data is inherently shared
2. **Circuit admin cannot modify tournament operations** — view access only
3. **Chapter admin at School X cannot see School Y's entries** at the same tournament
4. **Judge can only enter ballots for their own assigned panels**

### Shared Data (No Isolation)

1. **Person records** — shared across all contexts
2. **Chapter records** — shared across all tournaments
3. **Circuit records** — shared across all members
4. **Published results** — public
5. **Paradigms** — public

---

## Assumptions vs Confirmed Decisions

### Confirmed (from legacy code analysis)
- Tournament is the primary operational boundary ✓
- Permission model is flat, not hierarchical ✓
- Person/chapter exist outside tournament scope ✓
- Multiple people can have the same role at the same tournament ✓
- A person can have different roles at different tournaments ✓
- Site admin bypasses all checks ✓
- Published data is public ✓

### Assumptions (need confirmation)
- **Multi-tenancy strategy:** Single database with tenant filters (not schema-per-tenant or DB-per-tenant)
- **Session scope:** A session is associated with one person and optionally one "active" tournament (from legacy `defaults.tourn`)
- **API authorization:** Every API endpoint must validate both authentication and tournament-scoped authorization
- **Data retention:** Tournament data is retained indefinitely (no TTL or archival policy visible in legacy)
- **Cross-tournament queries:** Acceptable for circuit/district operations; performance SLA TBD
- **PII handling:** Student PII (under FERPA considerations) requires access controls; no explicit FERPA compliance visible in legacy

### Open Decisions (for architecture phase)
1. **Row-level security vs application middleware?** PostgreSQL RLS could enforce tournament scoping at the DB level.
2. **API key auth for external integrations?** Legacy has basic API key auth; rebuild needs a proper API gateway strategy.
3. **Multi-region deployment?** Legacy is single-region; ScholarComp is Azure-based. Multi-region adds complexity but improves latency for geographically distributed tournaments.
4. **Data residency requirements?** Student data may have jurisdiction-specific requirements.
5. **Service-to-service auth?** With 28+ microservices, internal auth strategy matters. ScholarComp likely has this solved.
