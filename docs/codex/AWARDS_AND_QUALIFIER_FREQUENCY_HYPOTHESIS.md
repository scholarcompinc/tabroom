# Awards and Qualifier Frequency Hypothesis

## Purpose

This document provides a first-pass hypothesis about how frequently different award, qualifier, and external-posting workflows are likely to matter across the current Tabroom product surface.

This is explicitly an inference from source prominence, workflow integration, and configuration footprint. It is not direct usage telemetry.

## Source Basis

Inputs considered:

- number of dedicated routes and reports
- presence in publication menus
- dependence on affiliation-specific settings
- appearance in public-facing versus operator-only surfaces
- spread across tournament types and event types
- prior codex docs in the results/recognition cluster

## Frequency Scale

For this document:

- `Very Common`: likely relevant to a large share of ordinary tournaments
- `Common`: likely relevant to many tournaments, but not universal
- `Segment-Specific`: likely important within a meaningful subset only
- `Rare / Long-Tail`: likely specialized or infrequent

## Hypothesis Register

| Workflow / Output Family | Hypothesized Frequency | Basis |
|---|---|---|
| Final places / placers | Very Common | Broad menu presence, upstream dependency for many later flows |
| Speaker awards | Common | Dedicated generation/report paths and repeated settings support |
| Printed standings / prelim order | Common | Multiple report paths and print helpers suggest regular operator use |
| Awards ceremony scripts | Common | Dedicated ceremony flows imply repeated operational need |
| Awards pickup tracking | Segment-Specific | Dedicated workflow exists, but likely more important in larger / in-person events |
| Sweepstakes outputs | Common to Segment-Specific | Broad rule infrastructure, but not every event/tournament depends on it equally |
| Top novice / honorable mention variants | Segment-Specific | Repeated support exists, but clearly conditional on event configuration |
| District qualifiers | Segment-Specific | Deep support, but explicitly NSDA district mode dependent |
| NSDA points posting | Segment-Specific | Strong integration, but affiliation-specific |
| TOC bid reporting | Segment-Specific | Dedicated support, but clearly circuit/event-specific |
| School awards packets / pickup details | Segment-Specific | Suggests operational importance at larger ceremony-heavy events |
| Niche governing-body award packets | Rare / Long-Tail | Present but specialized and affiliation-scoped |

## Why These Hypotheses Matter

These hypotheses help answer:

- what likely belongs in a broad Release 1 core
- what should be gated by target market selection
- where SME validation is most needed before making parity commitments

## Confidence Notes

Higher confidence:

- final places
- speaker awards
- printed standings / prelim order

Medium confidence:

- sweeps
- ceremony scripts
- awards pickup tracking

Lower confidence:

- specific governing-body packets
- TOC / external posting workflows outside known affiliated segments

## Required Caution

This document is intentionally inferential. A workflow can be:

- low-frequency globally,
- but absolutely mandatory in a target segment

So this should be used as triage input, not as proof of market importance.

## Best Follow-Up

Validate these hypotheses with:

- tournament directors
- district operators
- awards-desk staff
- event segments you expect to target first
