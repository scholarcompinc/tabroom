# Parity Matrix — Lightweight First Pass

## Purpose

This matrix maps every major Tabroom capability against Release 1 candidacy, complexity, and ScholarComp reuse potential. It is intentionally lightweight — detailed scoping belongs in Phase 3 (Release 1 Triage).

## Legend

**Release 1 Priority:**
- **R1** = Required for Release 1 (every-tournament capability)
- **R1?** = Strong candidate, needs triage
- **R2** = Release 2 candidate
- **R3+** = Release 3 or later
- **Defer** = Deliberately deferred, niche or NSDA-specific

**Complexity:** Low / Medium / High / Very High

**ScholarComp Reuse:** Which existing ScholarComp services could accelerate this capability
- **Full** = Existing service covers this well
- **Partial** = Existing service needs extension
- **New** = New domain-specific service required
- **N/A** = Not applicable

---

## Tournament Administration

| Capability | Frequency | Release | Complexity | ScholarComp Reuse | Notes |
|-----------|-----------|---------|------------|-------------------|-------|
| Create tournament | Every | R1 | Medium | Partial (Competition Service) | Core entity creation |
| Clone tournament | Most | R1 | Medium | New | Template/clone system needed |
| Tournament settings & config | Every | R1 | High | New | 193 EAV tags → strongly typed |
| Tournament access/permissions | Every | R1 | Medium | Partial (RBAC Service) | 13 permission tags |
| Tournament dates/deadlines | Every | R1 | Low | Partial (Competition Service) | |
| Tournament web pages/CMS | Most | R1? | Low | Partial (CMS) | Simple content management |
| Tournament backup/restore | Some | R2 | Medium | New | State snapshot system |
| Tournament merge | Rare | R3+ | High | New | |

## Events, Categories, and Scheduling

| Capability | Frequency | Release | Complexity | ScholarComp Reuse | Notes |
|-----------|-----------|---------|------------|-------------------|-------|
| Create/edit events | Every | R1 | Medium | Partial (Competition Service) | 7 event types |
| Create/edit categories | Every | R1 | Medium | New | Format polymorphism driver |
| Configure protocol/rules | Every | R1 | High | New | Scoring, tiebreakers, ballot format |
| Schedule/timeslots | Every | R1 | Medium | Partial (Scheduling) | |
| Round management | Every | R1 | Medium | New | Round lifecycle is domain-specific |
| Pattern templates | Some | R2 | Low | New | Scheduling convenience |
| Double-entry configuration | Most | R1 | Medium | New | Cross-event conflict detection |

## Sites and Rooms

| Capability | Frequency | Release | Complexity | ScholarComp Reuse | Notes |
|-----------|-----------|---------|------------|-------------------|-------|
| Sites and rooms CRUD | Every | R1 | Low | Partial (Venue Service) | |
| Room pools | Every | R1 | Medium | New | Parallel to judge pools |
| Room assignment algorithm | Every | R1 | Medium | New | ADA/quality matching |
| Room strikes | Some | R2 | Low | New | |

## Registration

| Capability | Frequency | Release | Complexity | ScholarComp Reuse | Notes |
|-----------|-----------|---------|------------|-------------------|-------|
| School registration at tournament | Every | R1 | Medium | Partial (Registration Service) | |
| Entry management (add/edit/drop) | Every | R1 | Medium | Partial (Registration Service) | |
| Student roster management | Every | R1 | Medium | Partial (Participant Service) | |
| Waitlist management | Most | R1 | Medium | New | Priority rules needed |
| Entry caps and limits | Most | R1 | Low | New | |
| Contact management | Every | R1 | Low | Partial (Contact Service) | |
| Registration deadlines enforcement | Every | R1 | Low | New | |
| On-site registration | Most | R1? | Medium | New | Day-of workflow |
| Entry code generation | Every | R1 | Low | New | 15+ code styles |
| TBA entry management | Some | R2 | Low | New | |
| Hybrid/independent entries | Some | R2 | Medium | New | |

## Judge Management

| Capability | Frequency | Release | Complexity | ScholarComp Reuse | Notes |
|-----------|-----------|---------|------------|-------------------|-------|
| Judge registration | Every | R1 | Medium | New | Chapter→Judge lifecycle |
| Judge pools (JPool) | Every | R1 | Medium | New | Core judge organization |
| Judge obligation calculation | Every | R1 | High | New | 10+ adjustment factors |
| Judge shifts/availability | Most | R1 | Medium | New | |
| Judge hiring marketplace | Some | R2 | High | New | 4 hiring models |
| Judge bonds | Some | R2 | Medium | New | Financial guarantee system |
| Judge conflicts (auto-propagation) | Every | R1 | High | New | Person→Strike propagation |
| Judge strikes | Every | R1 | High | New | 7+ overloaded types; decompose |
| Judge prefs (ordinal/tiered/%) | Most | R1 | Very High | New | MJP is core to debate |
| Tab ratings | Most | R1? | Medium | New | Director quality ratings |

## Pairing and Paneling

| Capability | Frequency | Release | Complexity | ScholarComp Reuse | Notes |
|-----------|-----------|---------|------------|-------------------|-------|
| Debate powermatching | Every (debate) | R1 | Very High | New | 900+ line algorithm |
| Speech section paneling (snake) | Every (speech) | R1 | Very High | New | |
| Congress chamber assignment | Every (congress) | R1 | Very High | New | Recency tracking |
| Preset pairing | Most | R1 | High | New | Early rounds |
| Bracket/elimination pairing | Most | R1 | High | New | Seeded brackets |
| WUDC 4-team pairing | Rare | R3+ | Very High | New | Niche format |
| Judge assignment optimization | Every | R1 | Very High | New | 2,120 lines, 15+ dimensions |
| Room assignment | Every | R1 | Medium | New | |
| Manual panel adjustments (swap/move) | Every | R1 | Medium | New | |
| Speaker order assignment | Every (speech) | R1 | Medium | New | |
| Side assignment (debate) | Every (debate) | R1 | High | New | Serpentine, flip, lock |
| Coin flip management | Most (debate) | R1 | Low | New | |
| Flight management | Some | R2 | Medium | New | |
| Schematic publication | Every | R1 | Medium | New | |
| Blast notifications | Every | R1 | Medium | Partial (Notification Service) | |

## Ballots and Scoring

| Capability | Frequency | Release | Complexity | ScholarComp Reuse | Notes |
|-----------|-----------|---------|------------|-------------------|-------|
| Online ballot entry (judge) | Every | R1 | High | New | Format-specific ballot UI |
| Tab room ballot entry | Every | R1 | High | New | Staff entry interface |
| Ballot validation | Every | R1 | High | New | Low-point wins, rank consistency |
| Double-entry audit | Most | R1? | Medium | New | Two-pass verification |
| Audit trail | Every | R1 | Medium | Partial (Audit Service) | entered_by, audited_by |
| Score correction | Every | R1 | Medium | New | |
| Ballot status dashboard | Every | R1 | Medium | New | Real-time monitoring |
| RFD/comments collection | Most | R1? | Low | New | |

## Results, Breaks, and Tiebreakers

| Capability | Frequency | Release | Complexity | ScholarComp Reuse | Notes |
|-----------|-----------|---------|------------|-------------------|-------|
| Results computation | Every | R1 | Very High | New | 4,069-line tiebreak engine |
| Tiebreaker configuration | Every | R1 | Very High | New | 30+ tiebreaker types |
| Break/advancement (debate) | Most | R1 | High | New | Single/double elim |
| Break/advancement (speech) | Most | R1 | High | New | Section-based |
| Break/advancement (congress) | Some | R1? | High | New | Chamber-based |
| Results publication | Every | R1 | Medium | New | 3-tier visibility |
| Speaker awards | Most | R1 | Medium | New | |
| Public results pages | Every | R1 | Medium | Partial (Public API) | |
| CSV/export of results | Most | R1 | Low | New | |

## Sweepstakes and Awards

| Capability | Frequency | Release | Complexity | ScholarComp Reuse | Notes |
|-----------|-----------|---------|------------|-------------------|-------|
| Sweepstakes configuration | Most | R1? | High | New | 15+ rule types |
| Sweepstakes calculation | Most | R1? | High | New | Recursive sweep sets |
| Awards display/printing | Most | R1? | Medium | New | |
| Circuit-level season awards | Some | R2 | High | New | Cross-tournament |

## Financials

| Capability | Frequency | Release | Complexity | ScholarComp Reuse | Notes |
|-----------|-----------|---------|------------|-------------------|-------|
| Fee configuration | Most | R1 | Medium | Partial (Billing Service) | |
| Invoice generation | Most | R1 | Medium | Partial (Billing Service) | |
| Fine management | Most | R1? | Medium | New | |
| Payment processing (Stripe) | Most | R1 | Medium | Full (Payment Service) | Replace AuthorizeNet+PayPal with Stripe |
| Concessions/store | Some | R2 | Medium | Partial (E-commerce) | Mini e-commerce |

## User Accounts and Identity

| Capability | Frequency | Release | Complexity | ScholarComp Reuse | Notes |
|-----------|-----------|---------|------------|-------------------|-------|
| Account creation/login | Every | R1 | Medium | Full (Auth Service) | |
| Profile management | Every | R1 | Low | Full (User Service) | |
| Password management | Every | R1 | Low | Full (Auth Service) | |
| Session management | Every | R1 | Low | Full (Auth Service) | |
| Permission/RBAC | Every | R1 | Medium | Partial (RBAC Service) | Tournament-specific model |
| Admin switch-user | Rare | R2 | Low | Partial (Auth Service) | |

## Public Pages and Discovery

| Capability | Frequency | Release | Complexity | ScholarComp Reuse | Notes |
|-----------|-----------|---------|------------|-------------------|-------|
| Tournament search/listing | Every | R1 | Medium | Partial (Search Service) | |
| Tournament detail pages | Every | R1 | Medium | Partial (Public API) | |
| Circuit directory | Some | R2 | Low | New | |
| Paradigm display | Most | R1? | Low | New | |
| Historical results archive | Some | R2 | Medium | New | |

## Communication

| Capability | Frequency | Release | Complexity | ScholarComp Reuse | Notes |
|-----------|-----------|---------|------------|-------------------|-------|
| Email blast system | Every | R1 | Medium | Partial (Notification Service) | |
| Push notifications | Most | R1? | Medium | Partial (Notification Service) | |
| Tournament following | Some | R2 | Low | New | |
| Internal messaging/inbox | Some | R2 | Medium | New | |

## Organization Administration

| Capability | Frequency | Release | Complexity | ScholarComp Reuse | Notes |
|-----------|-----------|---------|------------|-------------------|-------|
| Chapter/school management | Every | R1 | Medium | Partial (Org Service) | |
| Circuit management | Some | R2 | Medium | New | |
| Region management | Some | R2 | Low | New | |
| District management | Some (NSDA) | Defer | High | New | NSDA-specific |
| NSDA integration | Some (NSDA) | Defer | Very High | New | 46 funclib files |

## Reports and Exports

| Capability | Frequency | Release | Complexity | ScholarComp Reuse | Notes |
|-----------|-----------|---------|------------|-------------------|-------|
| Registration reports | Every | R1 | Medium | New | Core operational reports |
| Schematic/pairing printouts | Every | R1 | Medium | New | Print-optimized views |
| Ballot printing | Every | R1 | Medium | New | Format-specific layouts |
| Results reports | Every | R1 | Medium | New | |
| Financial reports | Most | R1 | Medium | New | |
| CSV exports | Most | R1 | Low | New | |
| PDF generation (LaTeX) | Most | R1? | High | New | Evaluate alternatives to LaTeX |

## Online/Hybrid Tournament Support

| Capability | Frequency | Release | Complexity | ScholarComp Reuse | Notes |
|-----------|-----------|---------|------------|-------------------|-------|
| Online tournament mode | Some | R2 | High | New | Video platform integration |
| Hybrid mode (per-entry) | Some | R2 | High | New | |
| Online room monitoring | Some | R2 | Medium | New | |
| Async/video events | Rare | R3+ | Medium | New | |

---

## Summary Statistics

| Category | R1 | R1? | R2 | R3+ | Defer | Total |
|----------|-----|------|-----|------|-------|-------|
| Tournament Admin | 5 | 1 | 1 | 1 | 0 | 8 |
| Events/Categories/Schedule | 5 | 0 | 1 | 0 | 0 | 6 |
| Sites/Rooms | 3 | 0 | 1 | 0 | 0 | 4 |
| Registration | 8 | 1 | 2 | 0 | 0 | 11 |
| Judge Management | 6 | 1 | 2 | 0 | 0 | 9 |
| Pairing/Paneling | 11 | 0 | 1 | 1 | 0 | 13 |
| Ballots/Scoring | 5 | 2 | 0 | 0 | 0 | 7 |
| Results/Breaks | 7 | 1 | 0 | 0 | 0 | 8 |
| Sweepstakes | 0 | 3 | 1 | 0 | 0 | 4 |
| Financials | 2 | 1 | 1 | 0 | 0 | 4 |
| User Accounts | 4 | 0 | 1 | 0 | 0 | 5 |
| Public Pages | 2 | 1 | 2 | 0 | 0 | 5 |
| Communication | 1 | 1 | 2 | 0 | 0 | 4 |
| Org Admin | 1 | 0 | 2 | 0 | 2 | 5 |
| Reports/Exports | 5 | 1 | 0 | 0 | 0 | 6 |
| Online/Hybrid | 0 | 0 | 3 | 1 | 0 | 4 |
| **Total** | **65** | **13** | **20** | **3** | **2** | **103** |

**Release 1 core: 65 capabilities (63%)**
**Release 1 candidates needing triage: 13 (13%)**
**Release 2: 20 (19%)**
**Release 3+ / Defer: 5 (5%)**

## ScholarComp Reuse Summary

| Reuse Level | Count | Examples |
|------------|-------|---------|
| Full reuse | 5 | Auth, User, Payment, Session |
| Partial reuse | ~20 | Registration, Notification, Billing, RBAC, Venue, Search, Audit |
| New (domain-specific) | ~78 | Pairing, ballots, tiebreakers, prefs, judge pools, results |

**~76% of capabilities require new domain-specific services.** ScholarComp provides foundational infrastructure (auth, payments, notifications, RBAC) but the tournament-specific business logic is almost entirely new work.

## Open Questions for Release 1 Triage

1. Should sweepstakes be R1 or R2? It's "most tournaments" but complex.
2. Should push notifications be R1 or R2? Email may suffice initially.
3. Should double-entry audit be R1? Single-entry with audit trail may suffice.
4. Should paradigm display be R1? Valuable but not operationally critical.
5. PDF generation strategy — LaTeX is heavyweight; evaluate browser-based alternatives.
6. Should congress be R1 or R2? Smaller user base but distinct format.
7. Payment processor: consolidate to Stripe only, or support multiple?
