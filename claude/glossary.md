# Glossary and Terminology Map

## Core Tournament Terms

| Term | Definition | Notes |
|------|-----------|-------|
| **Tournament** | A competitive event hosted on specific dates at a specific location, containing one or more events | Called "tourn" in code |
| **Event** | A competitive division within a tournament (e.g., "Lincoln-Douglas Debate", "Original Oratory") | Sometimes "division"; events belong to categories |
| **Category** | A grouping of related events, typically by format (Debate, Speech, Congress) | Controls judge pools, prefs, and format-specific behavior |
| **Round** | A single round of competition within an event (e.g., "Round 1", "Quarterfinals") | Can be preliminary or elimination |
| **Prelim** | A preliminary round before elimination rounds | All entries compete; results determine seeding/breaks |
| **Elim** | An elimination round where losers are eliminated | Also called "out-round" or "break round" |
| **Panel** | A competition unit within a round — a group of entries competing together | Called "section" in speech; "room" colloquially in debate; "chamber" in congress |
| **Section** | Speech-specific term for a panel — a group of speakers competing in one room | Synonymous with "panel" in the data model |
| **Chamber** | Congress-specific term for a panel — a group of legislators in one session | Synonymous with "panel" in the data model |
| **Flight** | A subdivision of a round, used when not all panels compete simultaneously | Common in large tournaments; Flight 1 starts, then Flight 2 |
| **Timeslot** | A time block in the tournament schedule | Rounds are assigned to timeslots |
| **Pattern** | A scheduling template defining round order and timing | Used for consistent scheduling across events |

## Participant Terms

| Term | Definition | Notes |
|------|-----------|-------|
| **Entry** | A competitive unit in an event — one individual or one team | Can contain 1+ students via `entry_student` |
| **Student** | A competitor affiliated with a chapter | Person → Student → EntryStudent → Entry chain |
| **Chapter** | A school or program that competes (persistent across tournaments) | Parent of "school" which is tournament-specific |
| **School** | A chapter's registration at a specific tournament | Tournament-scoped projection of a chapter |
| **Coach** | A chapter administrator who manages entries and judges | May also be a judge |
| **TBA** | "To Be Announced" — an entry placeholder without assigned competitors | Used when entries are registered before students are named |
| **Maverick** | An incomplete team (e.g., one partner in a two-person debate event) | Allowed by some tournaments via `mavericks` setting |
| **Hybrid** | An entry that competes both in-person and online | Or an entry from a school that doesn't normally attend |
| **Independent** | An entry not affiliated with a school | Supported in some formats |

## Role Terms

| Term | Definition | Notes |
|------|-----------|-------|
| **Tournament Director (TD)** | Person who creates and administers a tournament | Permission tag: `owner` |
| **Tabber** | Person who runs tabulation operations during a tournament | Permission tag: `tabber`; functionally equivalent to owner |
| **Tab Room** | The physical/operational space where tabulation happens | Also used metaphorically for the tabbing team |
| **Checker** | Limited-access role for check-in operations | Permission tag: `checker` |
| **Site Admin** | System-level superuser | `person.site_admin` flag |
| **Circuit Admin** | Administrator of a circuit (league/conference) | Permission tag: `circuit` |
| **District Chair** | Administrator of an NSDA district | Permission tag: `chair` |

## Judging Terms

| Term | Definition | Notes |
|------|-----------|-------|
| **Judge** | A person who adjudicates competition rounds | Exists at both chapter level (`chapter_judge`) and tournament level (`judge`) |
| **Chair** | The presiding judge in a multi-judge panel | Has tie-breaking authority in some formats |
| **Obligation** | The number of rounds a school must provide judges for | Calculated from entry count and tournament rules |
| **Burden** | The total judging workload assigned to a judge | Used for equitable assignment |
| **Judge Pool (JPool)** | A group of judges available for specific rounds | Constrains judge assignment |
| **Hire** | A judge made available by one school and used by another | `judge_hire` entity; marketplace-style exchange |
| **Bond** | A financial guarantee that a judge will fulfill obligation | Auto-assessed; refunded on fulfillment |
| **Strike** | A constraint preventing a judge from judging a specific entry/school | Can be entered by coaches or administrators |
| **Conflict** | A personal relationship that prevents fair judging | Person-level (vs strike which is entry/school-level) |
| **Prefs** | Preference ratings coaches assign to judges | Three systems: ordinal, tiered, percentage |
| **MJP** | Mutual Judge Preference — matching pref system using both sides' ratings | Common in debate |
| **Rating** | A quality assessment of a judge | Can be coach rating or tab rating |
| **Paradigm** | A judge's self-reported philosophical approach to adjudication | Public profile content |

## Pairing and Scheduling Terms

| Term | Definition | Notes |
|------|-----------|-------|
| **Pairing** | The assignment of entries to panels (who debates/speaks against whom) | Debate-specific usage; more general is "paneling" |
| **Paneling** | The assignment of entries to sections in speech | Same operation as pairing, different context |
| **Powermatching** | Pairing entries based on their win-loss record (wins hit wins) | Primary debate pairing algorithm for prelims |
| **Preset** | A pre-determined pairing (not based on results) | Used for early rounds before records differentiate |
| **Bracket** | Elimination bracket structure | Standard single/double elimination tournament format |
| **Snake** | An assignment algorithm that distributes entries in serpentine order | Used for speech paneling and side assignment |
| **Pullup** | When an entry is paired against a higher-bracket opponent due to odd numbers | Can happen when bracket sizes are uneven |
| **Bye** | When an entry doesn't have an opponent and receives a default result | Occurs with odd numbers of entries |
| **Forfeit** | When an entry fails to appear for a round | Different from bye; usually carries penalties |
| **Side** | The assigned position in a debate (Affirmative/Negative, Government/Opposition) | Labels configurable via `aff_label`/`neg_label` |
| **Side lock** | Requiring a specific side assignment in elimination rounds | Based on seeding |
| **Flip** | A coin flip to determine sides in a debate round | Can be manual or system-managed |
| **Schematic** | The published pairing/paneling for a round | Also called "postings" |

## Results and Scoring Terms

| Term | Definition | Notes |
|------|-----------|-------|
| **Ballot** | The official scoring record linking a judge to entries in a panel | Core data entity |
| **Audit** | Verification that ballot data was entered correctly | `ballot.audit` flag; `audited_by` tracking |
| **Double Entry** | A verification method where ballots are entered twice and compared | `audit_method` setting |
| **Tiebreaker** | A metric used to order entries with identical primary results | Configurable order via `tiebreak` entity |
| **Break** | Advancement from preliminary rounds to elimination rounds | "Making the break" = qualifying for elims |
| **Seed** | An entry's ranking/position going into elimination rounds | Determines bracket placement |
| **Speaker Points** | Individual performance scores in debate | Abbreviated "speaks" or "spkr pts" |
| **Ranks** | Ordinal placement within a speech section (1st, 2nd, etc.) | Primary speech scoring method |
| **Reciprocal** | 1/rank — used in speech tiebreaking | Lower is worse (1/1 > 1/6) |
| **Win** | A ballot decision in favor of an entry in debate | Binary per judge; majority of judges determines round winner |
| **Low-Point Win** | Winning a debate despite having lower speaker points | Usually disallowed; `allow_lowpoints` setting |
| **RFD** | "Reason for Decision" — judge's written explanation | Requested via `rfd_plz` setting |
| **Outstanding Speaker** | Award for best individual speaker performance | Separate from team results |
| **Top Novice** | Award for best first-year competitor | `top_novice` setting |
| **Sweepstakes** | Aggregate school-level awards based on all entries' performance | Complex point calculation system |
| **Qualification** | Earning the right to compete at a higher-level tournament | NSDA districts → nationals pipeline |

## Format-Specific Terms

### Debate
| Term | Definition |
|------|-----------|
| **LD (Lincoln-Douglas)** | One-on-one values debate format |
| **PF (Public Forum)** | Two-on-two debate format with coin flip |
| **Policy (CX)** | Two-on-two policy/research debate format |
| **Parli (Parliamentary)** | Various parliamentary debate formats |
| **WUDC** | World Universities Debating Championship format (4 teams per round) |
| **Round Robin** | Format where every entry debates every other entry |

### Speech
| Term | Definition |
|------|-----------|
| **IE (Individual Events)** | General term for speech competition events |
| **OI (Oral Interpretation)** | Performance events using published literature |
| **OO (Original Oratory)** | Original persuasive speech event |
| **Extemp** | Extemporaneous speaking — limited prep time on drawn topics |
| **DI (Dramatic Interpretation)** | Performance of dramatic literature |
| **HI (Humorous Interpretation)** | Performance of humorous literature |
| **Duo** | Two-person interpretation performance |

### Congress
| Term | Definition |
|------|-----------|
| **Legislation** | Bills and resolutions debated in Congress format |
| **PO (Presiding Officer)** | Student who chairs a Congress chamber |
| **Recency** | Tracking how recently a student spoke in a chamber |
| **Seating Chart** | Assigned seats in a Congress chamber |

## Organizational Terms

| Term | Definition | Notes |
|------|-----------|-------|
| **Circuit** | An organizational grouping of schools (state league, national circuit) | Cross-tournament structure |
| **District** | An NSDA geographic/organizational unit for qualification | Specific to NSDA qualification pipeline |
| **Region** | A subdivision within a circuit or district | Geographic or organizational |
| **Diocese** | Possibly a legacy term for a religious organizational unit | NCFL (National Catholic Forensic League) related |
| **NSDA** | National Speech and Debate Association | Governing body; operator of tabroom.com |
| **NCFL** | National Catholic Forensic League | Separate organization with specific tournament formats |
| **CEDA** | Cross Examination Debate Association | College debate organization |
| **TOC** | Tournament of Champions | Prestigious invitational tournament |
| **NDCA** | National Debate Coaches Association | Professional organization |

## Technical/System Terms

| Term | Definition | Notes |
|------|-----------|-------|
| **Mason** | HTML::Mason template engine | Legacy Perl web framework used by Tabroom |
| **Funclib** | Function library of reusable Mason components | `web/funclib/` — 417 files |
| **Autohandler** | Mason's middleware layer for access control and setup | `autohandler` files in each directory |
| **Campus** | Tabroom's online tournament infrastructure | Used for virtual competition rooms |
| **TMoney** | NSDA's payment processing system | Integration for financial operations |
| **Indexcards** | NSDA's new Node.js backend being built alongside legacy | Partial API replacement |
| **Schemats** | NSDA's new SvelteKit frontend being built | Partial UI replacement |

## Ambiguous Terms Requiring Clarification

| Term | Ambiguity | Context |
|------|-----------|---------|
| **Category** vs **Division** | Used interchangeably in some contexts; technically different in Tabroom | Category groups events; not the same as a single event/division |
| **Panel** vs **Section** vs **Room** | Same entity in DB, different terminology by format | Debate = "panel/room", Speech = "section", Congress = "chamber" |
| **Round** (overloaded) | Can mean a round of competition OR a physical room | Context-dependent |
| **Entry** vs **Competitor** vs **Student** | Different levels of abstraction | Entry is the competitive unit; student is the person; competitor is informal |
| **School** vs **Chapter** | Chapter is persistent; school is tournament-specific | Source of confusion for users |
| **Strike** vs **Conflict** | Both prevent judging but at different scopes | Strike = judge↔entry/school; Conflict = person↔person |
| **Prefs** vs **Ratings** vs **Rankings** | Different preference expression systems | Ordinal rankings, tiered ratings, percentage prefs |
| **Code** | Can mean entry code, school code, or judge code | Used for anonymous identification |
