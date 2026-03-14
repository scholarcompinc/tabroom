# Results Format Variation Matrix

## Purpose

This document captures the first-pass matrix of how results behavior varies across major competition formats in the current Tabroom platform.

Its purpose is to prevent later requirements work from flattening debate, speech, congress, and special variants into one generic results model.

This is a descriptive source-analysis artifact, not a target data model.

## Source Basis

Primary evidence used:

- `web/funclib/results_debate.mas`
- `web/funclib/results_speech.mas`
- `web/funclib/results_congress.mas`
- `web/funclib/results_wudc.mas`
- `web/tabbing/results/order_entries.mas`
- `web/tabbing/results/order_speakers.mas`
- `web/tabbing/publish/generate_results.mhtml`
- `web/index/tourn/results/round_results.mhtml`
- prior codex results and workflow docs

## First-Pass Conclusions

High-confidence observations:

- debate, speech, and congress use materially different result semantics
- result publication thresholds expose different score types by format
- standings and awards are computed through different protocols depending on format
- some special formats preserve debate-like shape but alter position and display logic

## Matrix

| Concern | Debate / Debate-like | Speech | Congress | WUDC / Special Debate Variants |
|---|---|---|---|---|
| Core competitive unit | Entry on side in a panel | Entry in a section with speaking order | Entry in a chamber | Entry/team in position-sensitive debate panel |
| Round-result display basis | Side-aware panel outcome | Section rank and points | Chamber rank, points, speech values, chair handling | Position/room-aware variant of debate-style results |
| Key raw scores | `winloss`, `point`, sometimes speaker-level detail | `rank`, `point`, `refute` | `rank`, `point`, `speech`, chair semantics | Debate-like score families with variant positional meaning |
| Primary publication effect | Outcome visibility | Rank visibility | Rank visibility | Debate-style outcome visibility |
| Secondary publication effect | Additional point/detail visibility | Point visibility | Point and speech visibility | Debate-style expanded detail visibility |
| Feedback/post-feedback role | Limited in standard debate display | Not primary signal in first-pass evidence | Explicit part of visibility gate | Variant-dependent |
| Standings engine | `order_entries.mas` with debate protocol/tiebreaks | `order_entries.mas` section/rank-aware | `order_entries.mas` chamber/tie aware | `order_entries.mas` plus format-specific handling |
| Speaker awards path | Dedicated speaker protocol where configured | May exist depending on event settings | Less central in first-pass evidence | Variant-dependent |
| Final places generation | Prelim seed plus elim/final progression | Often based on final / last prelim and section results | Chamber/final result-set generation logic | Variant-dependent |
| Bracket usage | Central for elim progression | Used for breakout/finals depending on event | Less bracket-centric in first-pass evidence | Variant-specific bracket support |
| Display-specific concerns | Side labels, byes, forfeits, motion, judge names | Speaker order, section ordering, judge-by-judge ranks | Chair first ordering, chamber grouping, speech-value strings | Position labels, special pairing/result semantics |

## Debate / Debate-Like Results

Observed characteristics:

- side-aware entries
- byes and forfeits are represented in round-result output
- motion visibility may be separately gated
- round-result output can include speaker positions and judge results
- final places generation includes elim/final progression and may substitute “Round” as a result key in elim/final contexts

Observed sub-variants:

- standard debate
- `wsdc`
- `mock_trial`
- `wudc`

Implication:

- “debate” is already a family, not one uniform subtype

## Speech Results

Observed characteristics:

- section-centric presentation
- rank is the first visible result layer
- points are an additional detail layer
- speaker order matters in both display and later breakout handling
- speaker awards are a major sibling result system

Implication:

- speech needs both entry-result and speaker-result concepts from the start

## Congress Results

Observed characteristics:

- chamber-centric result display
- chair/non-chair distinctions affect display ordering
- `speech` scores are explicitly surfaced
- tied chamber rounds can be grouped into “chamber result” sets
- publication gating includes feedback/post-feedback signals

Implication:

- congress is not just speech with different labels
- chamber aggregation and chair handling are part of the native results model

## Final Places Variation

Observed first-pass differences:

- debate-like events use elim/final progression plus prelim seeding to derive final places
- speech/congress lean more heavily on ranked placements and section/chamber outcomes
- special labels like `Prelim`, round labels, ordinal places, and `Co-Champion` are all visible

Implication:

- final places are format-shaped outputs, not one universal transform

## Speaker / Individual Awards Variation

Observed characteristics:

- speaker awards require dedicated protocol configuration
- they are generated as their own result-set family
- they carry their own tiebreak descriptions and ballot-like display strings
- they can be limited by breakout/student filters

Implication:

- speaker awards should not be treated as a cosmetic column on team standings

## Format-Specific Fields That Should Not Be Flattened Later

- side / aff / neg
- speaker order
- chamber / chair
- speech value strings
- motion visibility
- elim round progression labels
- ballot-string presentation conventions

## Open Questions For Later Validation

- exact boundaries between debate-like variants that deserve distinct requirement treatment
- whether mock trial should be normalized with debate or separated early
- how much congress-specific qualification logic diverges from ordinary congress standings
