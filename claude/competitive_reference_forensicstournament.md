# Competitive Reference: ForensicsTournament.net

## Purpose

This document captures observations from ForensicsTournament.net's public help documentation as a competitive reference for the Tabroom rebuild. It identifies features and UX patterns worth considering, and documents where ForensicsTournament differs from Tabroom.

**Source:** https://forensicstournament.helpscoutdocs.com/ (20 help articles total)
**Date analyzed:** 2026-03-13

---

## Platform Overview

ForensicsTournament.net is a simpler, lighter-weight competitor to Tabroom for debate and forensics tournament management. It covers the core tournament operating loop (registration → pairing → balloting → results) with significantly less configuration surface and fewer advanced features.

---

## Features Worth Incorporating in Rebuild

### 1. Portable Competitor Waivers
- TDs create waivers (liability, video recording consent, etc.)
- Signed waivers follow the student to subsequent tournaments using the same waiver document
- Coaches can opt schools out of specific waivers
- Students/parents sign electronically with date tracking
- Students can **revoke consent** at any time
- **Tabroom gap:** No built-in waiver system. This is a genuine value-add, especially for liability and video consent.

### 2. Ballot Auto-Save with Link Resume
- eBallots auto-save every ~10 seconds
- If the browser closes, the judge returns via the same unique link and picks up where they left off
- **Tabroom gap:** Online ballots have no auto-save. Judges can lose work if the browser crashes.

### 3. Structured Ballot Escalation Timeline
- 3 minutes after push → automatic reminder sent to judge
- 5 minutes after push → TD alerted if ballot not yet accepted
- **Tabroom gap:** Tabroom has a manual "missing ballots" dashboard but no structured escalation with automatic reminders.

### 4. Five-Phase Registration State Machine
- **Accepting Entries** → open registration
- **Modifications for Existing Schools Only** → roster changes, no new schools (~2 days before)
- **Drops Only** → only reductions allowed (night before)
- **Registration System Shutdown** → coaches locked out, TD can still modify
- **Tournament Completed** → archived to results page
- **Tabroom gap:** Tabroom infers registration state from dates and settings (freeze_deadline, drop_deadline, etc.) rather than explicit phases. The explicit state machine is cleaner and easier for TDs to understand.

### 5. Text/SMS as First-Class Communication Channel
- Postings and ballot notifications sent via text message, not just email
- Preference set per person (text vs email vs paper)
- Unique links in text messages for ballot access
- **Tabroom gap:** Tabroom is email/web-centric. Push notifications exist via OneSignal but SMS is not a primary channel.

### 6. Immutable Random Tiebreaker
- Random number assigned at tournament start
- Cannot be modified by TDs — prevents manipulation concerns
- **Worth considering** for the rebuild's tiebreaker engine as a fairness guarantee.

### 7. Judge Dance Cards
- Printable scheduling documents showing all rounds assigned to a judge
- Configuration: include judge types, page breaks, custom message field
- Simple but useful print artifact for day-of operations.

---

## Tiebreaker Comparison

ForensicsTournament has ~28 tiebreaker types across categories vs Tabroom's 30+. No documented mechanism for TDs to configure tiebreaker priority ordering.

### IE Prelim Tiebreakers (7)
| Tiebreaker | Description |
|-----------|-------------|
| Ranks | Sum of judge ranks (lower better) |
| Points | Sum of judge rates/points (higher better) |
| Firsts | Count of first-place ranks |
| Decimal | Sum of 1/rank per round (reciprocal) |
| Drop High Rank | Sum minus highest rank |
| Drop Low Rate | Sum minus lowest points |
| Random | Pre-assigned immutable random number |

### Debate Tiebreakers (10)
| Tiebreaker | Description |
|-----------|-------------|
| Wins | Total wins |
| High-Low Points | Points minus highest and lowest |
| Total Points | Sum of all speaker points |
| Opp Wins | Opponent wins (strength of schedule) |
| Two High-Low Points | Points minus two highest and two lowest |
| ZScore | Statistical normalization |
| Ranks | Judge ranks |
| Ballots | Individual ballot wins |
| Real Wins | Wins excluding byes |
| Random | Immutable random |

**Note:** Bye rounds receive the team's average point total.

### IE Elim Tiebreakers (11)
Rank In Elim, Judge Pref In Elim (2-way ties only), Total Rank In Event, Total Points in Event, Total Rank Drop High Prelim Rank, Number Of Firsts in Event, Majority of Firsts in Elim, Decimal in Elim, Points in Elim, Points in Prelim, Random.

### Congress Prelim Tiebreakers
Documented as "In progress" — no content available.

---

## Ballot/Scoring Workflow

### Judge Side
1. Coach sets judge's ballot preference (text or email) on roster
2. TD pushes eBallot notifications from Pair/Section > Ballots page
3. Judge receives text/email with unique link
4. Judge clicks link to "accept" (confirms receipt)
5. Ballot displays event-specific fields (ranking, rating, win/loss, RFD for debate)
6. Auto-save every ~10 seconds
7. Confirm → review → final submit

### TD Side
1. Enable eBallots in tournament settings
2. Verify judge contact info and ballot preferences
3. Push notifications (sent within 1 minute)
4. Escalation: 3 min reminder, 5 min TD alert
5. Monitor via Missing Ballots page
6. Can reassign ballots to replacement judges
7. Mixed paper/eBallot environment supported

---

## Registration Flow

- **Account creation:** Name, email, password, CAPTCHA. Email verification required for tournament creation but not registration.
- **School creation:** Basic fields, international support. No verification/approval.
- **Student roster:** Individual add only (no bulk import). First name, last name, optional email/phone.
- **Judge roster:** Individual add with ballot preference (text/email/paper).
- **Tournament registration:** Five-phase lockdown model (see above).

---

## Notable Limitations vs Tabroom

| Area | ForensicsTournament | Tabroom |
|------|-------------------|---------|
| Judge prefs/paradigms | None | Full MJP, ordinal, tiered, paradigm authoring |
| Pairing algorithms | Not documented | Powermatching, snake, round robin, preset |
| Payment processing | Fee estimation only, no payment | Stripe, AuthorizeNet, PayPal |
| Sweepstakes | Not documented | Multiple rule types, recursive sets |
| Entry caps/waitlists | Not documented | Full cap/waitlist system |
| Scale | Smaller tournaments | Handles nationals (10K+ entries) |
| Hired judges | Workaround via fake school | Dedicated hiring marketplace (4 models) |
| Reports | Dance cards, fee sheets | 190+ report surfaces |
| Tiebreaker config | No priority ordering documented | Full priority ordering, 30+ types |
| Settings surface | Minimal | 570+ configurable settings |

---

## Summary

ForensicsTournament.net is a lightweight competitor that handles basics well with some smart UX choices. It's not a threat to Tabroom on features or scale, but its approaches to **waivers, ballot auto-save, escalation timers, registration phases, and SMS-first communication** are worth adopting in the rebuild. The platform appears to target smaller, less complex tournaments where Tabroom's configuration surface is overwhelming.
