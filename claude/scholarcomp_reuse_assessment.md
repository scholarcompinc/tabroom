# ScholarComp Reuse Fitness Assessment

## Purpose

This document evaluates each ScholarComp V4 service for reuse in the Tabroom rebuild. For each service, it assesses the current capability, what the tournament domain requires, the fit/gap, and a recommendation. The goal is to maximize leverage of existing infrastructure while identifying where new domain-specific work is needed.

---

## Service Assessments

---

### 1. Authentication / Identity Service

**ScholarComp Service:** `user-service` (port 8010, schema `user_service`)

**Current Capability Summary:**
- JWT-based authentication with bcrypt password hashing (passlib)
- Access tokens (30 min TTL) and refresh tokens (7 day TTL)
- Token contains `sub` (ULID), `email`, `roles[]`, `exp`, `type`, `iat`, `jti`
- OAuth exchange flow: Google, Facebook, LinkedIn token exchange via `POST /v1/auth/oauth/exchange`
- Rate limiting via `slowapi`
- Frontend integration via NextAuth 4.x (main app) and NextAuth 5.x beta (PMC)
- Identity is JWT-derived only (no user_id in request bodies)

**Tournament Domain Needs:**
- Account creation with email verification
- Login/logout and session management
- Password management and forced reset
- NSDA account linking (deferred, but architecture must not preclude it)
- Admin switch-user (SU) capability for support workflows

**Fit/Gap Analysis:**

| Aspect | Fit | Gap |
|--------|-----|-----|
| Account creation & login | Full fit | None |
| JWT token flow | Full fit | None |
| OAuth providers | Full fit (Google, Facebook) | LinkedIn less relevant for debate; may need Apple |
| Password management | Full fit | Forced reset flow may need extension |
| Session management (NextAuth) | Full fit | None |
| NSDA account linking | No existing analog | New concept: external identity federation with NSDA API |
| Switch-user (SU) | No existing analog | Requires impersonation token minting with audit trail |

**Recommendation: Extend**

The authentication core (JWT issuance, password hashing, OAuth exchange, session management) is fully reusable. Two gaps need development: (1) NSDA account linking (deferred to R2+, but schema should accommodate external identity links), and (2) admin switch-user capability (R2). Neither gap undermines the core service; both are additive features.

---

### 2. User / Profile Service

**ScholarComp Service:** `user-service` (combined with auth), `persona-profile-service` (port 8033), `persona-affiliation-service` (port 8034)

**Current Capability Summary:**
- User CRUD with ULID identifiers
- Profile data management (persona-profile-service)
- Organization affiliation tracking (persona-affiliation-service)
- Activity history tracking (activity-history-service, port 8035)
- User-generated content management (user-content-service, port 8041)

**Tournament Domain Needs:**
- Profile management (name, contact info, timezone, pronouns)
- Chapter (school) roster membership
- Student-to-person linking
- Judge paradigm authoring (R2)
- Judging record/history display

**Fit/Gap Analysis:**

| Aspect | Fit | Gap |
|--------|-----|-----|
| Core profile CRUD | Full fit | None |
| Contact info, preferences | Full fit (persona-profile) | May need tournament-specific fields (pronouns, dietary, etc.) |
| Org affiliation | Good fit (persona-affiliation) | Chapter/circuit/region model is more hierarchical than ScholarComp's flat org model |
| Activity history | Partial fit | Tournament participation history is domain-specific (rounds judged, tournaments attended) |
| Judge paradigms | No analog | New concept: rich-text competitive philosophy documents |

**Recommendation: Extend**

The persona-profile and persona-affiliation services provide a solid foundation for user identity and organizational relationships. The core profile model needs extension for tournament-specific attributes. Judge paradigm authoring is a new feature (R2) that could be built as an extension or a thin domain layer on top.

---

### 3. RBAC / Authorization Service

**ScholarComp Service:** `user-service` RoleManager (part of user-service)

**Current Capability Summary:**
- Role hierarchy: user -> student -> parent -> coach -> organizer -> school_administrator -> application_administrator/admin -> editor -> taxonomy.admin
- Service subjects: `svc:service-name` format
- Bearer token extraction with role-based route protection
- PMC middleware enforces admin/application_administrator roles
- Roles stored in JWT claims as `roles[]`

**Tournament Domain Needs:**
- Tournament-scoped roles: owner, tabber, limited, checker, contact
- Entity-scoped permissions: event, category, chapter, circuit, region, district
- Permission cascade and inheritance (e.g., tournament owner inherits all event permissions)
- Site administrator capability
- 13+ permission tags per tournament

**Fit/Gap Analysis:**

| Aspect | Fit | Gap |
|--------|-----|-----|
| Global roles (admin, user) | Full fit | None |
| Role hierarchy pattern | Partial fit | Tournament roles are scoped to a resource, not global |
| JWT-based identity | Full fit | None |
| Route-level authorization | Full fit (middleware pattern) | Need domain-specific policy checks |
| Tournament-scoped roles | Not present | ScholarComp roles are global; Tabroom needs per-tournament role assignment |
| Entity-scoped permissions | Not present | Tabroom needs "user X is tabber for event Y in tournament Z" |
| Permission cascade | Not present | Inheritance rules are domain-specific |

**Recommendation: Wrap**

The authentication middleware, JWT infrastructure, and global role management are fully reusable. However, Tabroom's permission model is fundamentally different: it is resource-scoped (per tournament, per event, per category) rather than globally hierarchical. A tournament-authorization layer must be built on top that resolves "can user X perform action Y on resource Z in tournament T?" while leveraging the existing JWT and middleware infrastructure underneath.

---

### 4. Organization / Institution Service

**ScholarComp Service:** `managing-org-service` (port 8003), `school-service` (port 8006)

**Current Capability Summary:**
- managing-org-service: Organization management, CRUD operations, organization hierarchy
- school-service: Institution data management, school profiles
- Event publishing enabled (CloudEvents for managing-org changes)

**Tournament Domain Needs:**
- Chapter (school) management: roster, contact info, address, NSDA membership
- Circuit membership and administration
- Region and district affiliation
- Cross-organization permissions (e.g., a coach from school A helping at school B's tournament)
- School-at-tournament entity (a school's registration instance at a specific tournament)

**Fit/Gap Analysis:**

| Aspect | Fit | Gap |
|--------|-----|-----|
| Organization CRUD | Full fit | None |
| School/institution profiles | Full fit (school-service) | May need debate-specific fields (NSDA chapter ID, district assignment) |
| Org hierarchy | Partial fit | Circuit -> region -> district hierarchy is debate-specific |
| School-at-tournament | No analog | This is a registration-time entity (school registers for tournament with contact, address, code) |
| Circuit management | No analog | Circuits are a debate-specific organizational concept |
| Region/district | No analog | NSDA geographic structure |

**Recommendation: Extend**

The managing-org and school services provide solid foundations for institutional identity. They need extension for debate-specific organizational concepts (circuits, regions, districts) and the school-at-tournament registration entity. The core CRUD, search, and affiliation patterns are directly reusable.

---

### 5. Registration / Enrollment Service

**ScholarComp Service:** `registration-service` (port 8007, schema `registration_service`), `participant-service` (port 8004)

**Current Capability Summary:**
- registration-service: Competition registration workflows, enrollment management
- participant-service: Participant tracking, participant-to-competition relationships
- Both use schema-per-service isolation and standard CRUD patterns

**Tournament Domain Needs:**
- School-at-tournament registration (school entity with contact, address, code)
- Entry management (add/edit/drop entries with 15+ code styles)
- Student assignment to entries (individual or team)
- Entry status lifecycle (active, waitlisted, dropped, TBA, maverick, hybrid)
- Waitlist management with priority ranking and cap enforcement
- Multi-deadline enforcement (registration, freeze, drop, supplemental)
- Judge registration (separate from entry registration)
- On-site registration workflows

**Fit/Gap Analysis:**

| Aspect | Fit | Gap |
|--------|-----|-----|
| Registration workflow (sign up for competition) | Partial fit | Tournament registration is multi-entity: school registers, then adds entries and judges |
| Participant tracking | Partial fit | Tournament participants are typed: entry (individual/team), judge, observer |
| Registration status | Partial fit | Tournament has richer lifecycle (waitlisted, dropped, maverick, TBA) |
| Entry code generation | No analog | 15+ code styles (school initials + number, sequential, etc.) |
| Waitlist management | No analog | Priority ranking, cap-based auto-admission |
| Deadline enforcement | No analog | Multiple deadline types with fee implications |
| Judge registration | No analog | Judges register separately with pools, availability, obligations |
| Entry-student assignment | No analog | Team composition (partner assignment) is domain-specific |

**Recommendation: Wrap**

The registration and participant services provide useful infrastructure (registration state machines, participant-competition linkage, enrollment patterns) but tournament registration is fundamentally more complex. A tournament-registration domain layer must sit on top that handles: (1) the multi-entity registration model (school -> entries + judges), (2) waitlist management, (3) deadline enforcement, (4) entry code generation, and (5) judge-specific registration. The underlying services provide patterns and some reusable components, but the domain logic is largely new.

---

### 6. Competition / Event Service

**ScholarComp Service:** `competition-service` (port 8001), `event-service` (port 8002)

**Current Capability Summary:**
- competition-service: Competition CRUD, watchlists, competition lifecycle
- event-service: Event scheduling, event metadata
- Both publish CloudEvents and have event-driven integration
- Standard CRUD with Alembic migrations

**Tournament Domain Needs:**
- Tournament creation and cloning
- Tournament configuration (193 EAV settings -> strongly typed)
- Event creation with 7 event types (policy, LD, PF, speech events, congress, etc.)
- Category creation (Debate, Speech, Congress) with format polymorphism
- Protocol/rules configuration (scoring rules, tiebreaker chains, ballot format)
- Round management and lifecycle
- Schedule/timeslot management
- Double-entry conflict detection

**Fit/Gap Analysis:**

| Aspect | Fit | Gap |
|--------|-----|-----|
| Competition CRUD (as tournament shell) | Good fit | Need tournament-specific fields (circuit, type, timezone) |
| Event CRUD (as event shell) | Good fit | Need event-type polymorphism (debate vs. speech vs. congress) |
| Competition lifecycle | Partial fit | Tournament lifecycle is: setup -> registration -> active -> completed |
| Watchlists | Minor fit | Tournament following uses similar pattern |
| Tournament cloning | No analog | Deep-copy of tournament with all settings, events, categories |
| 193+ settings configuration | No analog | EAV tag system needs migration to strongly typed config |
| Category/format system | No analog | Format polymorphism (debate/speech/congress) is the key architectural driver |
| Round management | No analog | Round lifecycle (created -> paired -> started -> completed) is domain-specific |
| Protocol/rules configuration | No analog | Scoring rules, tiebreaker chains, ballot formats are entirely tournament-specific |

**Recommendation: Wrap**

The competition-service and event-service provide useful structural patterns (entity CRUD, lifecycle management, CloudEvents publishing) and could serve as the outer shell for tournament and event entities. However, the vast majority of tournament domain complexity (format polymorphism, settings management, round lifecycle, protocol configuration) is new. A tournament-service domain layer must be built that uses the competition/event infrastructure for basic entity management while adding the full tournament configuration and lifecycle system on top.

---

### 7. Scheduling Service

**ScholarComp Service:** `calendar-service` (port 8023, schema `calendar_service`)

**Current Capability Summary:**
- Calendar scheduling for competitions and events
- Date/time management
- Calendar CRUD operations

**Tournament Domain Needs:**
- Timeslot creation and management (day, start time, end time)
- Round-to-timeslot assignment
- Pattern templates (preset schedules for common tournament formats)
- Multi-weekend support
- Schedule conflict detection (double-entered students can't be in two rooms at once)
- Flight management within timeslots

**Fit/Gap Analysis:**

| Aspect | Fit | Gap |
|--------|-----|-----|
| Basic date/time scheduling | Partial fit | Calendar service is calendar-oriented, not timeslot-oriented |
| Timeslot management | Weak fit | Tournament timeslots are operational (when does round X happen?) not calendar-display |
| Round scheduling | No analog | Round-to-timeslot assignment with dependency ordering |
| Conflict detection | No analog | Cross-event conflict detection for double-entered competitors |
| Pattern templates | No analog | Pre-built schedule patterns |
| Multi-weekend | No analog | Multi-day/weekend tournament scheduling |

**Recommendation: Replace**

The calendar-service is designed for calendar display and scheduling in the "when is this event?" sense. Tournament scheduling is operationally different: it manages timeslots as containers for rounds, handles dependency ordering (round 3 must follow round 2), detects conflicts across events, and manages flights within timeslots. Building a tournament-scheduling service from scratch will be faster and cleaner than adapting the calendar-service.

---

### 8. Notification Service (Email, Push)

**ScholarComp Service:** `notification-service` (port 8011)

**Current Capability Summary:**
- Email delivery via Azure Communication Services (ACS)
- SMS support
- Push notification infrastructure
- Notification worker with scaling limits (max 2 replicas)
- Notification consumer (max 1 replica)
- Custom Strapi email provider integration via ACS

**Tournament Domain Needs:**
- Email blasts to selected recipients (all coaches, all judges, specific groups)
- Template composition with tournament-specific merge fields
- Push notifications for round postings and results
- Delivery tracking
- Tournament following notifications (R2)
- Blast notification triggers tied to schematic publication

**Fit/Gap Analysis:**

| Aspect | Fit | Gap |
|--------|-----|-----|
| Email delivery infrastructure (ACS) | Full fit | None |
| Push notification infrastructure | Full fit | None |
| SMS infrastructure | Full fit | None (bonus capability) |
| Recipient resolution | No analog | "All judges in JPool X for round 3" is tournament-specific |
| Tournament-specific templates | No analog | Tournament merge fields (round, room, opponent, side) |
| Blast triggers | No analog | "Publish schematic -> blast all affected" is domain logic |
| Delivery tracking | Likely present | May need extension for tournament reporting |

**Recommendation: Extend**

The notification-service provides excellent infrastructure for message delivery across channels (email via ACS, push, SMS). The delivery pipeline, worker architecture, and channel integrations are all reusable. What's needed is a tournament-notification layer that handles recipient resolution (tournament-specific group queries), template management with tournament merge fields, and blast trigger integration. This layer calls the notification-service for actual delivery.

---

### 9. Payment / Billing Service

**ScholarComp Service:** Not a single named service in the inventory, but payment/billing capabilities are implied across the platform (subscriber-service port 8040 handles subscriber management).

**Current Capability Summary:**
- Subscriber management (subscriber-service)
- Platform likely handles payments through integration with payment processors
- Financial tracking capabilities implied in PMC management domains (Finance section)

**Tournament Domain Needs:**
- Fee configuration (entry fees, judge fees, late fees, drop fees, per-person fees)
- Automatic invoice generation per school per tournament
- Fine management (judge obligation fines, forfeit fines, with multipliers)
- Fine forgiveness workflows
- Payment processing via Stripe (replacing legacy AuthorizeNet + PayPal)
- NSDA TMoney integration (deferred)
- Manual payment recording
- Discount application
- Concessions/store (R2)

**Fit/Gap Analysis:**

| Aspect | Fit | Gap |
|--------|-----|-----|
| Payment processor integration | Unknown (need to verify if Stripe is already integrated) | Tournament needs Stripe specifically |
| Subscriber/billing management | Partial fit | Subscriber billing differs from tournament per-event fee billing |
| Invoice generation | No analog | Tournament invoices aggregate fees, fines, payments per school |
| Fee configuration | No analog | Tournament fee models (entry fees, judge fees, late penalties) are domain-specific |
| Fine management | No analog | Judge obligation fines, forfeit fines with multipliers |
| Concessions | No analog | Mini e-commerce for tournament concessions |

**Recommendation: Wrap**

If ScholarComp has Stripe integration infrastructure, that is directly reusable for payment processing. The subscriber-service patterns may inform billing workflows. However, tournament financial operations are domain-specific: fee configuration tied to events, invoice generation aggregating multiple fee types per school, fine management with tournament-specific rules, and deadline-based fee escalation. A tournament-finance service must be built that uses any existing payment infrastructure for actual payment processing while implementing tournament-specific fee, invoice, and fine logic.

---

### 10. Search / Discovery Service (Elasticsearch)

**ScholarComp Service:** `search-service` (port 8028), `search-indexer-service`

**Current Capability Summary:**
- Elasticsearch 8.11 with dedicated VM in production
- search-service: Universal search endpoint with Azure OpenAI embeddings for semantic search
- search-indexer-service: Event-driven indexing via Redis Streams (consumes CloudEvents)
- Indexes: competitions, scholarships, programs, events, community content
- Supports limit/offset pagination with optional cursor support
- Embedding model via Azure OpenAI for vector search

**Tournament Domain Needs:**
- Tournament search by name, date, location, circuit
- Tournament listing with filters (date range, location, circuit, format)
- Judge paradigm search (R2)
- Historical results search (R2)
- Entry/school lookup within a tournament (operational search)

**Fit/Gap Analysis:**

| Aspect | Fit | Gap |
|--------|-----|-----|
| Elasticsearch infrastructure | Full fit | None |
| Search service API patterns | Full fit | None |
| Event-driven indexing pipeline | Full fit | Add tournament entity types to existing pipeline |
| Semantic/embedding search | Full fit (bonus) | Could enable "find tournaments like this one" |
| Tournament-specific indexes | No analog | Need tournament, entry, judge, result indexes |
| Operational search (within tournament) | No analog | Quick lookup during tournament operations (find entry by code, find judge by name) |

**Recommendation: Extend**

The search infrastructure (Elasticsearch, indexing pipeline, search API, embedding support) is directly reusable. New tournament-specific index definitions and entity types need to be added to the indexing pipeline. The search-service API patterns work well for tournament discovery. Operational within-tournament search may be better served by direct database queries rather than Elasticsearch, but the infrastructure is ready for discovery use cases.

---

### 11. File / Document Service (S3/Azure Blob)

**ScholarComp Service:** Azure Blob Storage (`scholarcom2ef321c361.blob.core.windows.net`), CDN Profile

**Current Capability Summary:**
- Azure Blob Storage for uploads and media
- CDN profile for static asset delivery
- Image optimization in Next.js (Sharp, WebP/AVIF, remote patterns for Azure Blob)
- Used for user avatars, content images, organization logos

**Tournament Domain Needs:**
- PDF generation and storage (ballots, results, certificates)
- Print-optimized report generation
- Tournament document uploads (invitation letters, rules documents)
- LaTeX-based PDF generation (or alternative)
- Exported CSV/report file storage

**Fit/Gap Analysis:**

| Aspect | Fit | Gap |
|--------|-----|-----|
| Blob storage infrastructure | Full fit | None |
| CDN delivery | Full fit | None |
| Image handling | Full fit | None |
| PDF generation | No analog | LaTeX or browser-based PDF generation is new |
| Report generation pipeline | No analog | 190+ report surfaces need a generation strategy |
| Document management | Partial fit | Storage is there; document metadata/lifecycle is new |

**Recommendation: Extend**

The storage infrastructure (Azure Blob, CDN) is directly reusable for all file storage needs. What's new is the PDF/report generation pipeline: either LaTeX-based (heavyweight, current Tabroom approach) or a browser-based alternative. The generation service is new; the storage and delivery layer is existing.

---

### 12. Audit / Logging Service

**ScholarComp Service:** `audit-service` (port 8020)

**Current Capability Summary:**
- Audit logging service for compliance and tracking
- Auth event logging (sign-in, sign-out, failures) in PMC
- Privacy compliance service (port 8021) for GDPR/DSR
- Log Analytics workspace for centralized log aggregation
- Correlation ID propagation across services

**Tournament Domain Needs:**
- Ballot entry audit trail (entered_by, audited_by, timestamps)
- Score correction logging with change history
- Tournament permission changes
- Registration change tracking (entry adds, drops, edits)
- Financial transaction audit (payments, refunds, fine adjustments)
- Double-entry ballot verification audit

**Fit/Gap Analysis:**

| Aspect | Fit | Gap |
|--------|-----|-----|
| Audit event recording | Full fit | Pattern is reusable |
| Correlation ID propagation | Full fit | None |
| Centralized logging | Full fit | None |
| Auth event auditing | Full fit | None |
| Ballot-specific audit | No analog | Ballot audit has domain-specific semantics (entered_by vs. audited_by, score diffs) |
| Score change history | No analog | Need before/after with reason |
| Privacy compliance (GDPR) | Full fit | None |

**Recommendation: Extend**

The audit-service infrastructure (event recording, storage, correlation tracking) is directly reusable. Tournament-specific audit events need to be defined (ballot changes, score corrections, permission changes), but these follow the existing audit event pattern. The privacy compliance service is a bonus for any future regulatory needs.

---

### 13. Feature Flag Service

**ScholarComp Service:** Environment variable-based feature flags (`NEXT_PUBLIC_ENABLE_*`, `ENABLE_*`)

**Current Capability Summary:**
- 20+ feature flags in main app via `NEXT_PUBLIC_ENABLE_*` environment variables
- 50+ feature flags in PMC via `NEXT_PUBLIC_PLATFORM_MANAGEMENT_ENABLE_*`
- Backend feature flags via `ENABLE_*` (e.g., `ENABLE_EVENTS=true`)
- No dedicated feature flag service (flags are environment variables)
- Feature flags gate all major features (architectural principle)

**Tournament Domain Needs:**
- Feature-gated rollout of tournament capabilities
- Per-tournament feature toggles (e.g., enable online mode for specific tournaments)
- Gradual rollout of new pairing algorithms
- A/B testing of UI changes (optional, R2+)

**Fit/Gap Analysis:**

| Aspect | Fit | Gap |
|--------|-----|-----|
| Environment-based feature flags | Full fit | Same pattern works |
| Flag convention and patterns | Full fit | None |
| Per-tournament feature toggles | No analog | Runtime, resource-scoped flags need a flag service or database-backed flags |
| Gradual rollout / percentage-based | No analog | Env vars are all-or-nothing |

**Recommendation: Reuse as-is**

The environment variable-based feature flag pattern is sufficient for Release 1. The existing convention (`NEXT_PUBLIC_ENABLE_*` for frontend, `ENABLE_*` for backend) works well. Per-tournament feature toggles and gradual rollout can be addressed in R2 with a dedicated flag service if needed, but for R1, the existing pattern is adequate.

---

### 14. API Gateway (Azure APIM)

**ScholarComp Service:** Azure API Management (`scv4prodapim` Standard SKU), Nginx for local dev

**Current Capability Summary:**
- Production: Azure APIM with 100+ routing rules
- Local development: Nginx gateway on port 8000
- CORS handling, request ID tracking, authorization header forwarding
- 50MB max body size
- Route pattern: `/v1/{resource}` -> `{service}:8000`
- APIM route deployment via GitHub Actions workflow

**Tournament Domain Needs:**
- Route new tournament-domain services through the gateway
- Maintain BFF pattern (Next.js API routes proxy to microservices)
- Rate limiting for public API endpoints (tournament search, results)
- Potentially higher concurrency during live tournament operations

**Fit/Gap Analysis:**

| Aspect | Fit | Gap |
|--------|-----|-----|
| API gateway infrastructure | Full fit | None |
| Routing patterns | Full fit | Add new routes for tournament services |
| Local dev gateway (Nginx) | Full fit | Add new upstream blocks |
| BFF pattern | Full fit | None |
| CORS handling | Full fit | None |
| Rate limiting | Partial (exists in user-service) | May need gateway-level rate limiting for public endpoints |
| Concurrency during tournaments | Unknown | May need scaling review for burst traffic patterns |

**Recommendation: Reuse as-is**

The API gateway infrastructure is directly reusable. New tournament services simply need new routing rules added to APIM (production) and Nginx (local). The existing patterns, deployment workflows, and architectural constraints all apply cleanly.

---

### 15. Shared Infrastructure (Redis, PostgreSQL, Docker, CI/CD)

**ScholarComp Service:** Redis Enterprise, PostgreSQL 17 Flexible Server, Docker Compose, GitHub Actions, Azure Container Apps, ACR

**Current Capability Summary:**
- **PostgreSQL 17:** Schema-per-service isolation, Alembic migrations, connection pooling (5-10 per service)
- **Redis 7:** Distributed caching, Redis Streams (CloudEvents 1.0), session storage, rate limiting
- **Docker:** Full local orchestration via docker-compose.yml (859 lines), multi-stage builds for production
- **CI/CD:** 16+ GitHub Actions workflows, Azure OIDC auth, matrix service deployments, ACR image registry
- **Container Apps:** Auto-scaling 1-10 replicas, health checks, managed environment
- **Shared Python module:** `/api/shared/` with database config, auth middleware, pagination, event publishing, caching
- **Key Vault:** Secret management (`scv4prodkv`)

**Tournament Domain Needs:**
- Database for all tournament domain entities (tournaments, entries, judges, rounds, ballots, results)
- Caching for live tournament operations (ballot status, round progress)
- Event streaming for real-time updates (schematic published, ballot completed)
- Local development environment for new services
- CI/CD pipelines for new services
- Container deployment for new services

**Fit/Gap Analysis:**

| Aspect | Fit | Gap |
|--------|-----|-----|
| PostgreSQL (schema-per-service) | Full fit | Add new schemas for tournament services |
| Alembic migration pattern | Full fit | None |
| Redis caching | Full fit | None |
| Redis Streams (events) | Full fit | Add new event types/streams for tournament domain |
| Docker Compose local dev | Full fit | Add new service entries |
| CI/CD workflows | Full fit | Add new services to deployment matrix |
| Container Apps hosting | Full fit | None |
| Shared Python module | Full fit | Extend with tournament-specific shared utilities |
| ULID ID generation | Full fit | None |
| Health check patterns | Full fit | None |
| Key Vault secrets | Full fit | None |

**Recommendation: Reuse as-is**

The shared infrastructure is the strongest reuse candidate. Every component (database, caching, events, containers, CI/CD, shared modules) applies directly to tournament services. New tournament services follow the exact same patterns: create a schema, write Alembic migrations, use the shared module, add to docker-compose, add to deployment workflows. This is the primary value of building on ScholarComp.

---

## Summary Table

| # | Service/Capability | Recommendation | Confidence | R1 Impact |
|---|-------------------|---------------|------------|-----------|
| 1 | Authentication / Identity | **Extend** | High | Core — saves 2-3 weeks |
| 2 | User / Profile | **Extend** | High | Core — saves 1-2 weeks |
| 3 | RBAC / Authorization | **Wrap** | High | Core — saves 1 week (infra), needs 3-4 weeks (domain layer) |
| 4 | Organization / Institution | **Extend** | Medium | Core — saves 1-2 weeks |
| 5 | Registration / Enrollment | **Wrap** | Medium | Core — saves 1 week (patterns), needs 4-6 weeks (domain) |
| 6 | Competition / Event | **Wrap** | Medium | Core — saves 1-2 weeks (shell), needs 6-8 weeks (domain) |
| 7 | Scheduling | **Replace** | High | Minimal savings — build new |
| 8 | Notification | **Extend** | High | Core — saves 2-3 weeks (delivery infra) |
| 9 | Payment / Billing | **Wrap** | Medium | Saves 1-2 weeks if Stripe exists, needs 3-4 weeks (domain) |
| 10 | Search / Discovery | **Extend** | High | Saves 2-3 weeks (full pipeline reuse) |
| 11 | File / Document | **Extend** | High | Saves 1 week (storage), needs 3-4 weeks (PDF generation) |
| 12 | Audit / Logging | **Extend** | High | Saves 1-2 weeks |
| 13 | Feature Flags | **Reuse as-is** | High | No additional work needed |
| 14 | API Gateway | **Reuse as-is** | High | Minimal config changes only |
| 15 | Shared Infrastructure | **Reuse as-is** | High | Largest single value — saves 4-6 weeks of platform setup |

---

## Overall Assessment

### How much of the Tabroom rebuild can leverage existing ScholarComp services?

**Infrastructure layer: ~90% reuse.** PostgreSQL, Redis, Docker, CI/CD, API Gateway, shared Python modules, feature flags, container hosting, and observability are all directly reusable. This eliminates the need to build or choose a platform stack.

**Identity and access layer: ~60% reuse.** Authentication, session management, OAuth, and user profiles are mostly reusable. The tournament-scoped permission model requires a new domain layer on top of the existing JWT/RBAC infrastructure.

**Domain services layer: ~15% reuse.** Registration, competition, event, notification, and search services provide patterns and partial functionality, but the tournament-specific business logic (pairing algorithms, ballot management, tiebreaker engines, judge preference systems, results computation) is almost entirely new. This aligns with the parity matrix finding that ~76% of capabilities require new domain-specific services.

**Estimated overall reuse: 25-30% of total effort**, concentrated in infrastructure and identity. This translates to roughly 15-20 weeks of saved effort on a Release 1 timeline, primarily by not having to build a platform from scratch.

### New Domain-Specific Services Needed for Release 1

These services have no ScholarComp analog and must be built from scratch:

| New Service | Domain | Complexity | R1 Priority |
|------------|--------|------------|-------------|
| **tournament-service** | Tournament lifecycle, configuration, cloning | High | R1 |
| **tournament-auth-service** | Tournament-scoped RBAC, permission cascade | High | R1 |
| **entry-service** | Entry management, waitlists, codes, status | High | R1 |
| **judge-service** | Judge registration, pools, obligations, availability | High | R1 |
| **judge-prefs-service** | Ordinal/tiered/percentage prefs, MJP calculation | Very High | R1 |
| **strikes-conflicts-service** | Judge strikes, person conflicts, auto-propagation | High | R1 |
| **pairing-service** | Powermatching, snake paneling, congress chambering | Very High | R1 |
| **judge-assignment-service** | Constraint-satisfaction judge placement | Very High | R1 |
| **ballot-service** | Ballot entry, validation, audit, status tracking | High | R1 |
| **results-service** | Tiebreaker engine, results computation, breaks | Very High | R1 |
| **tournament-finance-service** | Fees, invoices, fines, payment orchestration | Medium | R1 |
| **round-service** | Round lifecycle, room assignment, schematic publication | Medium | R1 |
| **report-service** | Report generation, PDF pipeline | Medium | R1 |

### Recommended Service Architecture for Release 1

```
REUSE AS-IS (no changes needed)
├── PostgreSQL 17 (add new schemas)
├── Redis 7 (add new streams/cache keys)
├── API Gateway — APIM + Nginx (add new routes)
├── Docker / Container Apps (add new services)
├── CI/CD — GitHub Actions (add to deployment matrix)
├── Shared Python Module /api/shared/ (extend)
├── Feature Flags (env var pattern)
└── Key Vault, ACR, Log Analytics

EXTEND (add features to existing services)
├── user-service (auth) — Add NSDA linking placeholder, SU capability (R2)
├── persona-profile-service — Add tournament-specific profile fields
├── notification-service — Add tournament recipient resolution, templates
├── search-service — Add tournament indexes
├── search-indexer-service — Add tournament entity types
├── audit-service — Add tournament-specific audit events
└── Azure Blob Storage — Add report/PDF storage

WRAP (existing infra + new domain layer)
├── tournament-auth-service (wraps RBAC for tournament-scoped permissions)
├── tournament-registration (wraps registration-service patterns)
└── tournament-finance-service (wraps payment processing if Stripe exists)

BUILD NEW (entirely domain-specific)
├── tournament-service
├── entry-service
├── judge-service
├── judge-prefs-service
├── strikes-conflicts-service
├── pairing-service
├── judge-assignment-service
├── ballot-service
├── results-service
├── round-service
└── report-service
```

### Key Takeaway

ScholarComp's greatest value to the Tabroom rebuild is not in direct service reuse but in **platform acceleration**: the entire infrastructure stack, development patterns, deployment pipelines, and operational tooling are production-proven and immediately available. This lets the Tabroom rebuild team focus 100% of their effort on the tournament domain logic — which is where the real complexity lives — rather than spending months building a platform from scratch. The 11 new domain-specific services represent the core intellectual property of the Tabroom rebuild and cannot be shortcut; ScholarComp ensures everything around them is already solved.
