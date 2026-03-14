# Recovery and Correction Workflow Inventory

## Purpose

This document captures the first-pass inventory of live operational recovery and correction workflows visible in the current Tabroom platform.

Its purpose is to make explicit:

- which operator interventions the system supports,
- which corrections happen before versus after publication,
- which flows appear critical to live tournament trust.

This is a descriptive inventory, not a future PMC design.

## Source Basis

Primary evidence used:

- `panel/schemat/*`
- `panel/manipulate/*`
- `tabbing/entry/*`
- `disaster_check.mhtml`
- `move_panel.mhtml`
- `move_speech.mhtml`
- `flight_judge_swap.mhtml`
- `judge_add.mhtml`
- `judge_rm.mhtml`
- `panel_room_save.mhtml`
- `round_dump*`
- `import_round.mhtml`
- `upload_backup.mhtml`
- `score_remove.mhtml`
- `round_log.mhtml`
- prior workflow and business-rules docs

## First-Pass Conclusions

High-confidence observations:

- the current platform assumes operators will correct live data
- correction flows exist at pairing, assignment, ballot, score, and publication layers
- some recovery actions are clearly normal operational tools, not rare emergency-only paths

## Recovery Workflow Families

## 1. Round-Level Reset and Rebuild

Observed examples:

- `round_dump.mas`
- `round_dump_entries.mhtml`
- `round_dump_judges.mhtml`
- `round_dump_rooms.mhtml`
- `round_rm.mhtml`
- `import_round.mhtml`
- `upload_backup.mhtml`

Observed purpose:

- clear existing round contents
- partially reset a round
- reconstruct from imported or backed-up material

Why it matters:

- this is the highest-level recovery mechanism when a round is too broken to patch incrementally

## 2. Entry/Panel Movement

Observed examples:

- `move_panel.mhtml`
- `move_speech.mhtml`
- `move_confirm.mhtml`
- `rm_panel.mhtml`
- debate side/order swap helpers

Observed purpose:

- move a section between rounds
- move an entry between speech sections or positions
- swap placement/order semantics without rebuilding the whole round

Why it matters:

- repairing a bad automatic paneled round appears to be routine

## 3. Judge Reassignment and Swaps

Observed examples:

- `judge_add.mhtml`
- `judge_rm.mhtml`
- `judge_remove.mhtml`
- `judge_push.mhtml`
- `flight_judge_swap.mhtml`
- `panel_judges.mhtml`

Observed purpose:

- add or remove a judge
- steal a judge from another panel
- swap judges between flighted panels
- repair insufficient or bad judge assignments

Why it matters:

- live judge changes are clearly expected operational behavior

## 4. Room Reassignment

Observed examples:

- `panel_room_save.mhtml`
- `room_save.mhtml`
- `seating_rooms_save.mhtml`

Observed purpose:

- move a panel into another room
- fix room conflicts or suitability issues
- propagate room changes across linked congress sections when needed

Why it matters:

- room mistakes or last-minute room issues are part of live operations

## 5. Seating / Position Corrections

Observed examples:

- `speaker_order_save.mhtml`
- `debate_order_swap.mhtml`
- `debate_side_save.mhtml`
- `debate_sides_swap.mhtml`
- `wudc_side_save.mhtml`
- `seating_assign.mhtml`
- `seating_move.mhtml`

Observed purpose:

- correct speaking order
- correct side assignments
- rebalance positions
- fix room seating layout

Why it matters:

- fairness and correctness include positional details, not just who is in which panel

## 6. Ballot / Score Corrections

Observed examples:

- `screen_audit_save.mhtml`
- `score_remove.mhtml`
- `screen_audit_*`
- tab-room ballot entry and edit flows

Observed purpose:

- remove or correct invalid scores
- audit ballots
- repair inconsistent ballot entry

Why it matters:

- results trust depends on operator ability to repair score-level issues

## 7. Publication Corrections

Observed examples:

- `schemat_switch.mhtml`
- publish/unpublish result-set and round controls
- post-primary/post-secondary/post-feedback toggles

Observed purpose:

- undo or stage visibility changes
- publish corrected artifacts after repair

Why it matters:

- recovery sometimes includes visibility rollback, not just data change

## 8. Validation-Driven Correction

Observed examples:

- `disaster_check.mhtml`
- `disasters.mhtml`
- panel review and usage checks

Observed purpose:

- detect double-bookings and structural problems
- guide manual fixes before or after publication

Why it matters:

- recovery begins with validation and diagnosis

## Recovery Timing Categories

The source corpus suggests correction happens in at least three timing windows:

## 1. Before publication

Typical activities:

- repair pairings
- fix judges and rooms
- adjust speech order
- rerun disaster checks

## 2. After pairing publication but before results

Typical activities:

- move entries
- reassign judges or rooms
- handle no-shows and late changes

## 3. After ballots/results begin to exist

Typical activities:

- remove bad scores
- audit ballots
- unpublish/republish results
- regenerate result artifacts

## Highest-Confidence Must-Have Recovery Flows

Based on the source corpus, the most likely must-have recovery workflows are:

- disaster check / validation
- judge add/remove/swap
- room reassign
- entry move between panels/sections
- score removal / ballot correction
- publish rollback and republish
- round reset/import in extreme cases

## Recovery Artifacts and Logging

Observed signals:

- operator actions often write explicit log entries
- round logs and change-like artifacts exist
- audit and attribution are part of the correction story

Implication:

- recovery is not just a UI convenience; it participates in the product’s trust and audit model

## Suggested Next Deeper Pass

If more source analysis is done here, the next version should enumerate:

- workflow name
- source file
- entity scope touched
- preconditions
- whether it is used before or after publication
- whether it affects audit or visibility state

That would make this directly useful for future implementation briefs.
