# Source-Backed Recovery Frequency and Severity Ranking

## Purpose

This document provides a first-pass ranking of live recovery/correction workflows by likely operational frequency and likely operational severity, based on source prominence and workflow integration.

This is explicitly an inference from source evidence. It is not a log-based incident analysis.

## Source Basis

Inputs considered:

- number of dedicated recovery endpoints
- whether the workflow sits in core live-operation areas such as paneling/tabbing
- whether the workflow is referenced by validation/disaster checks
- whether the workflow appears to be a routine corrective tool versus a niche admin action
- prior codex recovery/pairing/results docs

## Ranking Dimensions

`Frequency`

- `High`
- `Medium`
- `Low`

`Severity if missing`

- `Critical`
- `High`
- `Moderate`

## Ranking Register

| Recovery Workflow Family | Frequency Hypothesis | Severity if Missing | Basis |
|---|---|---|---|
| Judge add/remove/swap | High | Critical | Many dedicated paths and obvious live-tournament need |
| Room reassignment | High | High | Last-minute room problems are common and directly disruptive |
| Entry/panel movement | High | Critical | Multiple movement actions suggest routine correction need |
| Disaster check / validation-driven correction | High | Critical | Dedicated validation surface implies central operational role |
| Seating / side / speaker-order corrections | Medium-High | High | Frequent enough to have dedicated tools; fairness-sensitive |
| Ballot / score correction | Medium-High | Critical | Direct trust/integrity impact once results exist |
| Round reset / dump / rebuild | Medium | Critical | Probably less frequent, but severe when needed |
| Publication rollback / republish | Medium | High | Becomes essential once wrong outputs are visible |
| Backup import / restore-like actions | Low-Medium | Critical | Likely less common, but a major safety net |

## Interpretation

The source suggests two broad classes:

## 1. Routine Live Corrections

Likely includes:

- judge swaps
- room moves
- entry/panel moves
- side/order fixes

These appear to be normal operator tools used to keep a tournament moving.

## 2. High-Severity Rescue Actions

Likely includes:

- round dump/reset
- import/restore paths
- publication rollback after visible errors

These may happen less often, but the platform clearly expects them to exist.

## Why This Matters

A rebuild can accidentally preserve only the happy path and lose the actual operational survivability of the product.

This ranking suggests that, from source evidence alone, the highest-risk omissions would be:

- judge/room correction tools
- panel/entry movement tools
- score correction tools
- disaster-check style validation

## Confidence Notes

Higher confidence:

- judge swaps
- room moves
- panel/entry movement
- disaster checks

Medium confidence:

- publication rollback frequency
- backup/import frequency

Reason:

- the source proves these flows exist and matter, but not exactly how often operators invoke them

## Best Follow-Up

Validate this ranking with:

- active tab room operators
- tournament directors who work large live events
- any available observations of real correction-heavy tournaments
