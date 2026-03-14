# Glossary First Pass

## Purpose

This is the first-pass working glossary for the discovery phase.

It is not a final product-language decision document. Its purpose is to:

- keep humans and agents speaking consistently,
- flag ambiguous terms,
- preserve legacy meaning before later normalization.

## Core Terms

| Term | First-Pass Meaning | Notes |
|---|---|---|
| Tabroom | The existing tournament platform being analyzed | Product/platform name |
| Tournament / Tourn | The top-level competitive event container | Legacy code commonly uses `tourn` |
| Category | A grouping construct used around events/judging/divisions | Needs more exact extraction |
| Division | User-facing competitive grouping term | May overlap with category in practice |
| Event | A competition type such as debate, speech, or congress format | Central domain term |
| Entry | The competing unit entered into an event | Could be person, pair, or team depending on format |
| School | Tournament-specific school presence/record | Distinct from persistent chapter in some cases |
| Chapter | Persistent school/program organizational record | Needs careful mapping |
| Circuit | Regional/league-like grouping of schools/tournaments | Administrative scope term |
| Region | Administrative grouping term | May differ from circuit and district |
| District | NSDA/district qualification-related grouping | Specialized but important |
| Judge | Adjudicator | Core role and entity |
| Paradigm | Judge philosophy/profile content | Public-facing ecosystem concept |
| Round | A competitive round in the tournament schedule | Central operational unit |
| Timeslot | Scheduled time block for rounds/events | Scheduling term |
| Panel | Assignment unit within a round | Meaning likely differs by format |
| Section | Another assignment/grouping term within a round | May overlap with panel |
| Chamber | Congress-specific grouping | Format-specific |
| Schemat / Schematic | The displayed pairing/assignment structure for a round | Mostly an operator-facing term |
| Pairing | Assignment of entries/opponents to competitive units | Especially central in debate |
| Powermatch / Powermatching | Debate pairing based on competitive results/brackets | High-complexity term |
| JPool | Judge pool | Legacy shorthand; likely safe to preserve in analysis |
| RPool | Room pool | Legacy shorthand |
| Conflict | A disqualifying or restrictive relationship affecting assignment | Broad term |
| Strike | A preclusion or disallow rule affecting judge/room/entry assignment | Related to but not identical with conflict |
| Preference / Pref | Mutual preference judging or rating input | High-value judging concept |
| Burden | Judge-obligation or assignment load concept | Needs exact rules extraction |
| Ballot | Judge scoring/decision artifact | Core domain object |
| Score | Numeric/rank data associated with a ballot | Depends heavily on format |
| Protocol | Scoring/ranking rules configuration | Likely a format/rules construct |
| Tiebreak | Rule chain for resolving ties | Results-critical term |
| Break | Advancement/cut from prelims to elimination rounds | User-facing and operator-facing |
| Elim / Elimination Round | Post-prelim bracketed round | Standard competition term |
| Sweepstakes / Sweeps | Team or school aggregate award scoring | Legacy and user-facing term |
| Fine | Charge/penalty in billing workflows | Financial term |
| Invoice | Tournament billing document | Financial term |
| Waitlist | Not-yet-accepted registration state | Registration term |
| Drop | Removal of entry/judge/school from active participation | Registration/operations term |
| Late Add | Midstream registration addition after ordinary registration | Recovery/change-management term |
| ADA | Accessibility/accommodation requirement, especially room assignment | Directly visible in rooming and reports |
| Hybrid | Partly online / partly in-person mode | Modern operational term |
| Online Room | Virtual competition room/session | Online/hybrid support term |

## Format-Specific Terms Needing Care

| Term | Why It Needs Care |
|---|---|
| Entry | Means different things in debate, speech, and congress |
| Panel | May mean debate pairing panel, speech section, or other assignment unit |
| Ballot | Structure differs sharply by event type |
| Result | Computation differs sharply by event type |
| Speaker Order | Especially important in speech/congress contexts |
| Side Lock | Debate-specific balancing concept |
| Pullup / Pulldown | Debate pairing adjustment concept |
| Chamber | Congress-specific, should not be flattened into generic sections |

## Role Terms

| Term | First-Pass Meaning | Notes |
|---|---|---|
| Coach | School-affiliated user managing entries/judges and tournament participation | Major self-service role |
| Tournament Director | User responsible for tournament setup and operations | High-authority operational role |
| Tabber / Tab Room Operator | User actively running live rounds/results | Likely distinct from coach workflows |
| School Admin / Chapter Admin | Org-level management role | Needs exact extraction |
| Circuit Admin | Circuit/league management role | Broader than a single tournament |
| District Admin | Qualification/district management role | Specialized |
| Site Admin | Platform-level administrator | System role |
| Student | Competitor-facing user role | Self-service capabilities vary by setup |
| Judge | Adjudicator role and person type | Has self-service plus operational implications |

## Terms Likely To Stay User-Facing

These are legacy/community terms that may be worth preserving even if internal architecture normalizes them differently.

- judge paradigm
- break
- sweeps / sweepstakes
- tab room
- pairings
- rounds
- ballots
- district tournament

## Ambiguous Terms To Resolve Later

- category vs division
- school vs chapter
- panel vs section
- tournament vs competition
- judge conflict vs strike vs preclusion
- owner vs admin vs tabber permissions

## Notes

- Preserve legacy meaning during discovery.
- Do not normalize terms too early.
- Add source references and more precise definitions as workflow and rules artifacts mature.
