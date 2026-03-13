# Release 1 Triage

## Purpose

This document converts the parity matrix's 103 capabilities into concrete Release 1 scope decisions, backed by evidence from the Phase 1/2 discovery artifacts. It defines what "Release 1 done" means and what is deliberately deferred.

---

## Triage Criteria

A capability is **Release 1** if it meets ALL of:
1. **Every tournament needs it** (or nearly every — >80% usage)
2. **Cannot be worked around** with manual processes
3. **Competitive legitimacy depends on it** (coaches/judges would reject a platform without it)

A capability is **Release 1?** (needs decision) if:
- Most tournaments use it, but workarounds exist
- It's high-value but high-complexity

A capability is **Release 2** if:
- Some tournaments use it
- Manual workaround is acceptable for initial release
- Complexity is disproportionate to user base

---

## Triage Decisions

### Decision 1: Congress Format — **R2**

**Rationale:** Congress is the smallest of the three format families (~15% of tournament events). It requires unique chambering algorithm, recency tracking, legislation management, PO scoring, seating charts, and student voting — all entirely separate code paths. The ~15 congress-specific settings add configuration surface without benefiting debate or speech users.

**Workaround:** Congress events can be managed outside the platform for Release 1 (as they were before Tabroom existed).

**Implication:** Removes ~8 capabilities from R1 scope, including congress chambering, recency, legislation, PO protocol, and congress-specific ballot/results paths.

### Decision 2: Sweepstakes — **R1 (simplified)**

**Rationale:** Most tournaments with 3+ events compute sweepstakes. The recursive sweep set composition and 15+ rule types are complex, but the core calculation (points for placement, aggregated by school) is not.

**R1 scope:** Support 3-5 most common rule types (place-based points, entry cap, event cap). Defer recursive sweep includes, circuit-level season awards, and rare rule types to R2.

### Decision 3: Push Notifications — **R2**

**Rationale:** Email blasts are the primary communication channel and are sufficient for R1. Push notifications (OneSignal) add value but are not required for tournament operation. The legacy system ran for years with email-only.

**R1 scope:** Email blasts only. Add push in R2.

### Decision 4: PDF Generation — **R1 (browser-based)**

**Rationale:** Printed ballots, schematics, and results are critical for day-of operations. However, the legacy LaTeX approach (141 files, full TeX Live required) is fragile and heavyweight.

**R1 approach:** Use browser-based PDF generation (Puppeteer/Playwright or a PDF service) with print-optimized CSS. This aligns with the Next.js frontend and eliminates LaTeX infrastructure. Start with the 10 most critical print outputs:
1. Debate ballot
2. Speech ballot
3. Round schematic/postings
4. Results standings
5. Speaker awards
6. Entry list
7. Judge list
8. Invoice
9. School registration summary
10. Award certificates

### Decision 5: Payment Processing — **R1 (Stripe only)**

**Rationale:** Legacy supports AuthorizeNet, PayPal, and TMoney. Consolidating to Stripe reduces integration surface by 2/3 while providing a modern, well-documented payment platform. TMoney is NSDA-specific and deferred with other NSDA integrations.

### Decision 6: Double-Entry Ballot Audit — **R1 (single entry + audit)**

**Rationale:** True double-entry (enter ballot twice, compare) is used by some tournaments but not most. Single entry with an audit confirmation step and full audit trail is sufficient for R1.

**R1 scope:** Single ballot entry → audit confirmation → audited flag. Defer double-entry comparison to R2.

### Decision 7: Judge Hiring Marketplace — **R2**

**Rationale:** 4 hiring models (entry-based, judge-based, round-based, exchange) is substantial complexity. Most tournaments manage judge registration without the marketplace. Defer to R2.

### Decision 8: Paradigm Display — **R2**

**Rationale:** Paradigms are valuable but not operationally critical. Judges can share paradigms via external means for R1. Defer paradigm authoring and search to R2.

### Decision 9: WUDC Format — **R3+**

**Rationale:** World Universities format (4 teams per round) is a niche format used at <1% of tournaments. Extremely specialized pairing algorithm. Defer indefinitely.

### Decision 10: Online/Hybrid Tournament Support — **R2**

**Rationale:** Post-COVID usage has stabilized. Most tournaments are in-person. The Campus/video infrastructure is NSDA-specific. Defer online mode to R2.

### Decision 11: NSDA/District Integration — **Defer**

**Rationale:** NSDA integration (46 funclib files, member verification, points reporting, qualification tracking, TMoney) is deeply NSDA-specific. The rebuild targets independent operation with monetization potential. NSDA integration can be added as a premium feature in R2+ if there is demand.

### Decision 12: Circuit-Level Features — **R2**

**Rationale:** Circuit administration, season awards, circuit-wide results aggregation are cross-tournament features used by organized leagues. Important but not required for individual tournament operation.

---

## Release 1 Final Scope

### Core Operating Loop (Must Work End-to-End)

```
Create Tournament → Configure Events → Open Registration →
  Register Schools → Add Entries → Add Judges → Set Prefs →
  Build Schedule → Pair Rounds → Assign Judges → Assign Rooms →
  Publish Schematics → Enter Ballots → Audit → Compute Results →
  Run Breaks → Pair Elims → Final Results → Publish → Sweepstakes
```

### Release 1 Capability Count

| Category | R1 Final | Deferred | Notes |
|----------|----------|----------|-------|
| Identity & Access | 5 | 1 | SU deferred |
| Tournament Lifecycle | 18 | 3 | Backup/restore, merge, web CMS deferred |
| Registration | 9 | 3 | TBA, hybrid, entry upload deferred |
| Judging | 7 | 3 | Hiring marketplace, bonds, tab ratings deferred |
| Pairing & Paneling | 10 | 3 | WUDC, flights, congress deferred |
| Ballots & Scoring | 6 | 1 | Double-entry comparison deferred |
| Results & Publication | 8 | 1 | Congress breaks deferred |
| Sweepstakes | 2 | 2 | Simplified; recursive/circuit deferred |
| Financials | 3 | 2 | Concessions, multi-gateway deferred |
| Communication | 1 | 3 | Email only; push, inbox, following deferred |
| Public & Discovery | 3 | 3 | Paradigms, circuit dir, archive deferred |
| Reports & Exports | 5 | 1 | Core reports; PDF via browser |
| Org Admin | 1 | 4 | Chapter only; circuit, district, region deferred |
| Online/Hybrid | 0 | 4 | All deferred |
| **Total** | **78** | **34** | **70% in R1** |

### Release 1 Format Support

| Format | Pairing | Scoring | Results | Breaks |
|--------|---------|---------|---------|--------|
| **Debate** | Powermatching, preset, bracket, round robin | Win/loss + speaker points | Full tiebreaker engine | Single elimination |
| **Speech** | Snake paneling | Ranks | Full tiebreaker engine | Section-based breaks |
| **Congress** | Deferred to R2 | Deferred | Deferred | Deferred |

### Release 1 Algorithms (Must Implement)

| Algorithm | Complexity | Lines (Legacy) | Expert Review |
|-----------|-----------|----------------|---------------|
| Debate powermatching | Very High | 2,725 | Required |
| Speech snake paneling | Very High | ~800 | Required |
| Judge assignment optimization | Very High | 2,120 | Required |
| Tiebreaker engine | Very High | 4,069 | Required |
| Side assignment (serpentine) | High | ~300 | Recommended |
| Break/bracket construction | High | ~500 | Recommended |
| Speaker order optimization | Medium | ~200 | Optional |
| Room assignment | Medium | ~150 | Optional |
| Sweepstakes calculation (basic) | Medium | ~300 | Optional |

### Release 1 Settings Count

From the Configuration and Settings Spec, ~80-90 settings are R1-critical:
- Registration: ~15
- Scoring/Ballot: ~15
- Pairing: ~15
- Judge/Prefs: ~10
- Results/Break: ~8
- Display/Labels: ~6
- Financial: ~8
- Ballot Display: ~4

### Release 1 Reports (Top 10)

1. Round schematic/postings (print)
2. Printed ballots (debate + speech)
3. Entry list by event
4. Judge list with assignments
5. Results standings
6. Speaker awards
7. Invoice/financial summary
8. School registration summary
9. Pref completion report
10. Sweepstakes results

---

## Release 2 Scope (Preview)

| Category | Capabilities |
|----------|-------------|
| Congress format | Chambering, recency, legislation, PO, seating |
| Judge marketplace | 4 hiring models, bonds, exchange |
| Push notifications | OneSignal or equivalent |
| Online/hybrid | Video integration, hybrid entries, online monitoring |
| Circuit features | Circuit admin, season awards, aggregation |
| Advanced reports | Full 190+ report catalog, LaTeX parity |
| Double-entry audit | Two-pass ballot verification |
| Paradigms | Authoring, search, display |
| Tournament following | Follow/unfollow, notification preferences |
| Internal messaging | Inbox system |
| TBA entries | Placeholder entry management |
| Room strikes | Room constraint system |
| Flights | Sub-round scheduling |
| Advanced sweepstakes | Recursive sets, 15+ rule types |

## Release 3+ Scope

| Category | Capabilities |
|----------|-------------|
| WUDC format | 4-team pairing |
| NSDA integration | Points, qualifications, TMoney, member verification |
| District operations | District tournaments, qualification pipeline |
| Multi-region deployment | Geographic distribution |
| Advanced offline | Service worker, ballot draft persistence |

---

## Risk Assessment

### Highest-Risk R1 Items

| Risk | Impact | Mitigation |
|------|--------|-----------|
| Powermatching algorithm fidelity | Coaches would immediately detect wrong pairings | Extract exact algorithm from code; domain expert review; extensive test suite |
| Tiebreaker accuracy | Wrong results = loss of trust | Port all 30+ tiebreaker types; validate against legacy output |
| Judge assignment quality | Poor judge placement visible to coaches | Replicate scoring dimensions; A/B test against legacy |
| Settings migration completeness | Missing settings = broken tournaments | Enumerate all R1 settings; test each with real tournament data |
| Print/PDF quality | Day-of operations depend on printable outputs | Invest in print CSS; test with physical printers |

### Deliberate Scope Risks

| Decision | Risk | Acceptance Rationale |
|----------|------|---------------------|
| No Congress in R1 | ~15% of market excluded initially | Congress community is smaller; can be added in R2 |
| No NSDA integration | Cannot be used for NSDA-sanctioned events | Target is independent market; NSDA is a premium add-on |
| Stripe only | Schools with existing AuthorizeNet/PayPal setup must switch | Stripe is widely accepted; migration is one-time |
| No online/hybrid | Cannot serve fully-online tournaments | In-person is >80% of current usage |
