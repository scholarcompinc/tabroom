# Lightweight Parity Matrix

## Purpose

This is the first-pass parity matrix for triage, not a final release plan.

Columns:

- Capability
- Evidence
- Frequency / Criticality
- Release 1 candidate
- Notes

## Matrix

| Capability | Evidence | Frequency / Criticality | Release 1 candidate | Notes |
|---|---|---:|---:|---|
| User accounts and login | `web/user/login`, `Session`, `Person`, manual account references | Every tournament | Yes | Commodity-like, but still required |
| Tournament creation and cloning | `setup/tourn`, manual Ch. 2-3 | Every tournament | Yes | Core starting workflow |
| Tournament settings/configuration | `setup/*`, many `*_Setting` models | Every tournament | Yes | Large surface; likely staged within Release 1 |
| Events/divisions/categories | `setup/events`, `Event`, `Category` | Every tournament | Yes | Core structural capability |
| Schedule / rounds / timeslots | `setup/schedule`, `Round`, `Timeslot` | Every tournament | Yes | Core operational prerequisite |
| Sites and rooms | `setup/rooms`, `panel/room`, `Room`, `Site` | Most tournaments | Yes | Includes ADA/quality considerations |
| School and roster registration | `register/school`, `register/entry`, `register/data` | Every tournament | Yes | Core coach/admin workflow |
| Judge registration and obligations | `register/judge`, `setup/judges` | Every tournament | Yes | Core tournament staffing workflow |
| Judge prefs / conflicts / strikes | `judgemath`, `Strike`, `Conflict`, pref helpers | Most tournaments | Yes | High-value domain logic |
| Judge pool management | `JPool*`, `setup/judges`, `panel/judge` | Most tournaments | Yes | May vary by tournament sophistication |
| Pairing / schematics / chambering | `panel/schemat`, `make_pairing_hash.mas` | Every tournament | Yes | One of the crown-jewel domains |
| Judge assignment | `clean_judges.mas`, `panel/round`, `judgemath` | Every tournament | Yes | Core operational algorithm |
| Room assignment | `panel/round/rooms.mhtml`, room quality/ADA logic | Every tournament | Yes | Must work with pairing/judging |
| Disaster check / validation | `panel/schemat/disaster_check.mhtml` | Most tournaments | Yes | Operationally critical recovery tool |
| Manual pairing/manipulation | `panel/manipulate/*` | Most tournaments | Yes | Needed for live corrections |
| Judge ballot submission | `user/judge`, ballot routes/models | Every tournament | Yes | Core live workflow |
| Tab room ballot entry/correction | `tabbing/entry`, ballot helpers | Every tournament | Yes | Needed for mixed/failed submission cases |
| Ballot auditing/status monitoring | `tabbing/status`, `tabbing/report` | Every tournament | Yes | Core live operations visibility |
| Results computation | `results_*`, `tabbing/results` | Every tournament | Yes | Core product function |
| Tiebreak configuration and execution | `tiebreak_types.mas`, rules surfaces | Most tournaments | Yes | High-risk rules area |
| Breaks / advancement | `tabbing/break` | Most tournaments | Yes | Core competitive progression |
| Result publication | `tabbing/publish`, `index/results` | Every tournament | Yes | Required for public trust/value |
| Public tournament pages | `index/tourn`, public docs site | Most tournaments | Likely | May be thinner in first cut |
| Paradigms / judge philosophy | `index/paradigm.mhtml`, `judge_paradigms.mas` | Most tournaments | Likely | Important ecosystem feature |
| Reports and print outputs | `register/reports`, `panel/report`, `tabbing/report` | Every tournament | Yes | Some reports likely mandatory, not all |
| Financials / invoices / fees | `setup/money`, `Invoice`, `Fine` | Most tournaments | Likely | Need scoping within Release 1 |
| Sweepstakes and awards | `funclib/sweeps`, `Sweep*` models | Most tournaments | Likely | May split into base vs advanced |
| District / qualification workflows | `register/district`, `funclib/district_*`, `user/nsda` | Some tournaments | Later / TBD | Important niche, likely not base Release 1 |
| Nationals / NSDA specialized workflows | `funclib/nsda`, many `nsda_*` screens | Some tournaments | Later / TBD | Specialized and affiliation-heavy |
| Online / hybrid tournament support | `online_room.mas`, hybrid references | Some tournaments | TBD | Need explicit product decision |
| Concessions | `Concession*`, `setup/money/concessions.mhtml` | Rare / some tournaments | Later | Non-core for first operational release |
| Practice rounds / practice features | `Practice*` models | Rare / some tournaments | Later | Valuable but not core MVP |
| API / automation utilities | `web/api`, utility scripts | Some tournaments / maintainers | TBD | Separate product/API decision needed |

## Immediate Takeaways

- The operating core is clearly identifiable.
- Configuration, pairing, judging, ballots, and results should dominate Release 1 planning.
- District/NSDA, concessions, practice, and some online/hybrid features look like likely later-scope candidates unless strategic goals say otherwise.
- Reports/print outputs should not be underestimated; they appear operationally central.
