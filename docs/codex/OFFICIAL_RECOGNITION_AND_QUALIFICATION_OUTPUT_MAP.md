# Official Recognition and Qualification Output Map

## Purpose

This document captures the first-pass map of result outputs that function as official recognition, advancement, or ecosystem-reporting artifacts in the current Tabroom platform.

Its purpose is to separate:

- ordinary competitive visibility outputs,
- official placement and award outputs,
- qualification outputs with downstream obligations,
- governing-body reporting outputs.

This is a descriptive source-analysis artifact, not a future product scope decision.

## Source Basis

Primary evidence used:

- `web/tabbing/publish/index.mhtml`
- `web/tabbing/publish/generate_results.mhtml`
- `web/tabbing/publish/generate_sweeps.mhtml`
- `web/tabbing/results/nsda_qualifiers.mhtml`
- `web/funclib/district_qualifiers.mas`
- `web/funclib/nsda/qualifier_count.mas`
- `web/tabbing/report/toc/post_bids.mhtml`
- `web/funclib/nsda/post_points.mas`
- prior codex results/publication/sweeps docs

## First-Pass Conclusions

High-confidence observations:

- “results” in the current platform includes official recognition and compliance-style outputs
- some outputs are internal-to-tournament awards
- some outputs are external-facing obligations to NSDA, TOC, or similar ecosystems
- several official outputs depend on previously generated result artifacts such as `Final Places`

## Output Families

## 1. Final Competitive Placement Outputs

Observed examples:

- `Final Places`
- brackets
- `Prelim Seeds`
- `Prelims Table`

Operational role:

- establish official placement and competitive ordering
- feed later qualification and reporting workflows

Why it matters:

- these are not just display views; other flows depend on them as upstream inputs

## 2. Award Outputs

Observed examples:

- `Speaker Awards`
- sweeps outputs by school, student, and entry
- top novice / honorable-mention related outputs where configured

Operational role:

- support awards ceremonies and recognition
- generate official named ranked outputs beyond core event placement

## 3. Qualification Outputs

Observed examples:

- `District Qualifiers`
- NSDA-related qualifier flows
- promotion / vacancy logic in district helper functions

Operational role:

- determine qualifying entries, alternates, and ineligibility/vacancy handling
- support downstream district/nationals administration

Observed dependency:

- publish UI explicitly indicates that district qualifier posting requires a generated `Final Places` result sheet first

## 4. Governing-Body / Circuit Reporting Outputs

Observed examples:

- TOC qualifying bid report / posted bid result set
- NSDA points posting
- NSDA sweeps-related outputs

Operational role:

- transmit or formalize outcomes outside the tournament’s own local display context
- satisfy circuit or governing-body obligations

## Relationship Map

## Core Tournament Competition Layer

Primary outputs:

- round results
- standings / prelim seeds
- brackets
- final places

These establish the tournament’s competitive truth.

## Tournament Recognition Layer

Primary outputs:

- speaker awards
- sweeps awards
- honorable mentions / novice variants

These transform competitive truth into awards and recognition artifacts.

## External Qualification / Compliance Layer

Primary outputs:

- district qualifiers
- TOC bids
- NSDA points

These translate competitive truth into external ecosystem consequences.

## Dependency Patterns Visible In Source

### Final Places as upstream dependency

Observed signals:

- publish UI checks for `Final Places`
- district qualifier flows reference `Final Places`
- NSDA point posting references `Final Places`

Implication:

- `Final Places` acts as a canonical upstream artifact for several official outputs

### Sweep set as separate rules-driven dependency

Observed signals:

- sweeps generation is driven by dedicated `sweep_set` rules
- generated outputs are grouped by scope such as school / student / entry

Implication:

- sweeps is not merely “another result set”; it is a configured award engine producing result sets

### Qualification count and eligibility dependency

Observed signals:

- qualifier count is computed from event size, event code, district level, overrides, and special-case rules
- qualifiers helper evaluates vacates and student eligibility

Implication:

- qualification outputs are partly downstream of results and partly downstream of policy logic

## Output Matrix

| Output Family | Typical Audience | Upstream Dependencies | Notes |
|---|---|---|---|
| Final Places | Public / coach / operator / external stakeholders | standings + elim/final outcomes | Feeds later official outputs |
| Speaker Awards | Public / coach / operator | speaker protocol + score data | Recognition-specific ranking path |
| Sweepstakes | Operator / ceremony / public in some cases | sweep rules + standings/results data | Separate configured award engine |
| District Qualifiers | Operator / district admin / coaches | `Final Places` + qualifier-count rules + eligibility | Qualification pipeline, not generic standings |
| TOC Bids | Operator / circuit / TOC stakeholders | bid round configuration + event ordering | Circuit-specific recognition/reporting |
| NSDA Points | Operator / NSDA ecosystem | event category mapping + posted/finalized results | External posting obligation |

## Why These Outputs Need Separate Treatment

The source corpus suggests these outputs differ from ordinary result displays because they:

- may carry external consequences
- may depend on policy/eligibility rules beyond pure placement
- may be generated on their own schedules
- may require dedicated UI/reporting flows rather than a generic standings page

## Likely Release-1 Criticality From Source Perspective

High source-side criticality:

- final places
- speaker awards
- basic sweep outputs where awards are core
- district qualifier outputs if targeting NSDA district workflows

Context-dependent criticality:

- TOC bid outputs
- NSDA point posting
- niche governing-body or award-report integrations

## Open Questions For Later Validation

- which recognition outputs are mandatory in the target customer slice versus long-tail ecosystem features
- whether some tournaments treat sweeps as optional ceremony material or as operationally essential
- whether districts consider qualifier output generation part of normal tabbing or a specialized admin workflow
