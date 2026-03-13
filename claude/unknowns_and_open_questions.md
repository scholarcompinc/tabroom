# Unknowns and Open Questions

## Purpose

This document records questions discovered during cataloging that need resolution before or during the requirements build phase. These represent gaps in our understanding that could affect architectural decisions, scope, or correctness.

---

## Behavioral Unknowns (Code vs Documentation Gaps)

### 1. Setting Interactions and Precedence
- **Question:** When the same conceptual setting exists at multiple scopes (tournament, category, event, round), which takes precedence?
- **Example:** `deadline` appears on both `tourn_setting` and `event_setting` and `category_setting`. Override rules are implicit in code.
- **Risk:** Incorrect precedence in rebuild would change registration behavior silently.
- **Resolution path:** Trace specific multi-scope settings through code to document precedence rules.

### 2. Powermatching Algorithm Details
- **Question:** What is the exact powermatching algorithm? How are ties within brackets handled? What is the pullup selection criterion?
- **Sources:** `web/panel/round/pair_powered.mas`, `web/panel/round/pair_debate.mas`
- **Risk:** Core competitive fairness depends on exact algorithm reproduction.
- **Resolution path:** Code-level extraction + domain expert review.

### 3. Speech Paneling Snake Algorithm
- **Question:** How exactly does the speech snake work? How are repeat competitor/judge constraints balanced against section size?
- **Sources:** `web/panel/round/snake_speech.mas`, `web/panel/round/pair_speech.mas`
- **Risk:** Incorrect paneling would be immediately visible to experienced coaches.
- **Resolution path:** Code-level extraction.

### 4. Judge Assignment Optimization
- **Question:** Is the judge assignment a greedy algorithm, local search, or full optimization? How are competing constraints (prefs, conflicts, burden, diversity) weighted?
- **Sources:** `web/panel/round/debate_judge_assign.mhtml`, `web/panel/round/judges.mhtml`
- **Risk:** Judge assignment quality directly affects user satisfaction and competitive fairness.
- **Resolution path:** Code-level extraction + domain expert review.

### 5. Tiebreaker Calculation Precision
- **Question:** What are all available tiebreaker metrics? How are edge cases handled (forfeits, byes, drops mid-tournament)?
- **Sources:** `web/tabbing/results/`, `Tab::Tiebreak` model, tournament-manual.pdf
- **Risk:** Tiebreaker errors directly affect competitive outcomes.
- **Resolution path:** Enumerate all tiebreaker types from code + manual comparison.

### 6. Congress Recency Tracking
- **Question:** How does the recency system work? How does it affect chamber reassignment across sessions?
- **Sources:** `web/panel/round/congress_recency.mhtml`
- **Risk:** Congress format has unique requirements not shared with debate/speech.
- **Resolution path:** Code-level extraction.

### 7. Ballot Validation Rules
- **Question:** What validates a ballot as complete? What checks prevent invalid score combinations?
- **Example:** Low-point win detection, rank consistency in speech, score range enforcement
- **Risk:** Missing validation could corrupt results data.
- **Resolution path:** Trace ballot submission and audit code paths.

---

## Data Model Unknowns

### 8. Diocese Entity Purpose
- **Question:** What is the `diocese` table for? Is it NCFL-specific? Is it actively used?
- **Evidence:** Table exists in schema; `web/user/diocese/` route family exists
- **Risk:** Low — may be legacy or niche.
- **Resolution path:** Check for non-zero rows in production or usage in recent code changes.

### 9. Student Ballot / Student Vote Usage
- **Question:** Are `student_ballot` and `student_vote` tables used for Congress-specific scoring? Or are they separate features?
- **Evidence:** Tables exist; `student_ballot.mhtml` exists in `web/setup/events/`
- **Risk:** Missing these in rebuild could lose a Congress feature.
- **Resolution path:** Trace code usage.

### 10. Circuit Membership vs Chapter Circuit
- **Question:** What is `circuit_membership` vs `chapter_circuit`? Are both active?
- **Evidence:** Both tables exist in schema
- **Risk:** Could affect org hierarchy modeling.
- **Resolution path:** Check ORM models and usage.

### 11. Weekend Entity Semantics
- **Question:** Does `weekend` support multi-weekend tournaments or multi-day scheduling within one weekend?
- **Evidence:** `weekend` table exists; `web/setup/tourn/district_weekend_events.mhtml` suggests district use
- **Risk:** Scheduling model needs to account for this.
- **Resolution path:** Check usage in setup and pairing code.

### 12. Housing System Status
- **Question:** Is the `hotel`/`housing`/`housing_slots` system actively used? How widely?
- **Evidence:** Tables exist; `require_hotel_confirmation` setting exists
- **Risk:** Low — but scope decision needed.
- **Resolution path:** Check for recent usage or removal.

### 13. Practice / Practice Student
- **Question:** Are `practice` and `practice_student` tables actively used?
- **Evidence:** Tables and ORM models exist
- **Risk:** Low — may be legacy.
- **Resolution path:** Check recent code changes.

---

## Terminology Ambiguities

### 14. Category vs Division
- **Question:** Tabroom uses "category" in code but some users say "division." Are they the same?
- **Risk:** Terminology mismatch between code and user-facing language could cause confusion.
- **Resolution path:** Confirm with domain expert.

### 15. Panel vs Section vs Room vs Chamber
- **Question:** These all map to the same `panel` table but are used in different format contexts. Should the rebuild use format-neutral terminology?
- **Risk:** Using wrong terminology for a format could confuse users.
- **Resolution path:** Design decision for rebuild.

### 16. Invoice vs Bill
- **Question:** Code references both "invoice" and "bills" — are these the same thing?
- **Evidence:** `web/setup/tourn/bills_upload.mhtml`, `web/register/school/invoice.mhtml`
- **Resolution path:** Trace financial workflow.

---

## Settings Unknowns

### 17. Deprecated / Unused Settings
- **Question:** How many of the 570+ identified setting tags are actually used in current tournaments?
- **Risk:** Porting deprecated settings wastes effort; missing active ones breaks functionality.
- **Resolution path:** Would require production data analysis or exhaustive code-path tracing.

### 18. Setting Name Typos
- **Question:** Are `invoice_wailtist` and `invoice_waitlist` both used? Is `instruction_url` vs `instructions_url` intentional?
- **Evidence:** Both variants appear in grep results
- **Risk:** Rebuild needs to handle both for data migration, or pick the correct one.
- **Resolution path:** Check which spelling is actually used in setting creation vs reading.

### 19. Compressed JSON Settings
- **Question:** Which settings store compressed JSON? What is the schema of the JSON content?
- **Evidence:** `value = "json"` → compressed JSON in `value_text` (seen in Person.pm)
- **Risk:** Loss of structured data if not properly extracted.
- **Resolution path:** Enumerate JSON-type settings and document their schemas.

---

## Integration Unknowns

### 20. NSDA API Surface
- **Question:** What is the full NSDA API integration surface? What data flows to/from NSDA?
- **Evidence:** `nsda_*` settings, NSDA points reporting, member verification, billing
- **Risk:** NSDA integration may be required for legitimacy even in independent rebuild.
- **Resolution path:** Document all NSDA touchpoints; determine which are required vs optional.

### 21. TMoney Payment System
- **Question:** What is TMoney exactly? Is it NSDA's proprietary payment system?
- **Evidence:** `tmoney_enable`, `tmoney_url`, `tmoney_staging` settings
- **Risk:** Payment integration is critical for tournament operations.
- **Resolution path:** Research TMoney; determine if ScholarComp can offer alternative.

### 22. Indexcards API Dependencies
- **Question:** Which Tabroom features now depend on Indexcards endpoints vs legacy code?
- **Evidence:** `$Tab::indexcards_url` referenced in autohandler for session updates and server monitoring
- **Risk:** Some behavior may have migrated to Indexcards without legacy code removal.
- **Resolution path:** Audit all `indexcards_url` references.

### 23. Campus / Online Tournament Infrastructure
- **Question:** How does the "Campus" online tournament system work? Is it a separate service?
- **Evidence:** `campus_log` table (4.9M+ rows), `web/user/campus/`, `campus_zone` setting
- **Risk:** Online tournament support is expected in a modern rebuild.
- **Resolution path:** Investigate campus infrastructure and video platform integration.

---

## Business Rule Unknowns

### 24. Judge Obligation Calculation
- **Question:** How exactly is judge obligation calculated from entry count? What are the formula and special cases?
- **Risk:** Getting this wrong affects tournament finances and judge availability.
- **Resolution path:** Code-level extraction from registration and financial code.

### 25. Waitlist Priority Rules
- **Question:** When entries are admitted from a waitlist, what determines priority?
- **Evidence:** `waitlist_rank` setting exists
- **Risk:** Perceived unfairness if priority rules differ from expectations.
- **Resolution path:** Trace waitlist admission code.

### 26. NSDA Qualification Rules
- **Question:** What are the exact NSDA qualification criteria? Are they published or embedded in code?
- **Risk:** Critical for district tournament functionality.
- **Resolution path:** Check NSDA published rules + code comparison.

### 27. Double Entry Conflict Detection
- **Question:** How does the system prevent scheduling conflicts when entries compete in multiple events?
- **Evidence:** `double_entry`, `double_max`, `no_waitlist_double_entry` settings
- **Risk:** Scheduling algorithm must respect double-entry constraints.
- **Resolution path:** Trace double-entry handling in pairing code.

### 28. Forfeit Handling
- **Question:** What happens to standings/results when an entry forfeits? How does it affect opponents' records?
- **Evidence:** `ballot.forfeit` flag, `forfeit_judge_fine` settings
- **Risk:** Inconsistent forfeit handling affects competitive fairness.
- **Resolution path:** Trace forfeit paths in ballot and results code.

---

## Prioritization

**Must resolve before Release 1 architecture:**
- #1 (Setting precedence), #2 (Powermatching), #4 (Judge assignment), #5 (Tiebreakers), #20 (NSDA API)

**Must resolve before Release 1 implementation:**
- #3 (Speech paneling), #6 (Congress recency), #7 (Ballot validation), #17 (Deprecated settings), #24 (Judge obligation), #27 (Double entry conflicts), #28 (Forfeit handling)

**Can defer to Release 2+:**
- #8 (Diocese), #11 (Weekend), #12 (Housing), #13 (Practice), #21 (TMoney), #23 (Campus)

**Terminology decisions (ongoing):**
- #14, #15, #16
