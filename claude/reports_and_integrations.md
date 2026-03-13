# Reports, Exports, and Integrations Inventory

## Part 1: Reports and Exports Inventory

### 1.1 Registration Reports (`web/register/reports/`)

These reports are used by tournament directors and registration staff during tournament setup and onsite registration.

#### Entry Statistics & Lists

| Report Name | File | Output Format | Purpose | User Role |
|---|---|---|---|---|
| Total Headcounts | `stats.mhtml` | HTML | Summary counts of entries, judges, schools | Tab Director |
| School Headcounts | `school_headcount.mhtml` | HTML | Entry counts broken down by school | Tab Director |
| School Headcount Print | `school_headcount_print.mhtml` | Print | Printable version of school headcounts | Tab Director |
| School List | `school_list.mhtml` | HTML | List of registered schools | Tab Director |
| School List CSV | `school_list_csv.mhtml` | CSV | Exportable school listing | Tab Director |
| Contact List | `contact_list.mhtml` | HTML | Coach/contact information for registered schools | Tab Director |
| Contact Sheets | `contact_sheets.mhtml` | Print | Printable contact info sheets for onsite use | Registration Staff |
| All Entries CSV | `entries_csv.mhtml` | CSV | Export of all entries with details | Tab Director |
| All Judges CSV | `judges_csv.mhtml` | CSV | Export of all judges with details | Tab Director |
| All Individuals CSV | `bodies_csv.mhtml` | CSV | Export of every person associated with the tournament | Tab Director |
| Multiple Entries Totals | `multiple_totals.mhtml` | HTML | Count of double-entered competitors | Tab Director |
| Multiple Entries List | `multiple_entries.mhtml` | HTML | Detailed list of double-entered competitors | Tab Director |
| ADA Room Needs | `ada.mhtml` | HTML | Accessibility accommodation requirements | Tab Director |
| Dietary Info | `diets.mhtml` | HTML | Dietary restriction information | Tab Director |
| Site Attendance | `site_attendance.mhtml` | HTML | Attendance data per tournament site | Tab Director |
| School Events Matrix | `school_events.mhtml` | HTML | Matrix of which events each school is in | Tab Director |
| Judge Obligations | `check_burdens.mhtml` | HTML | Judge obligation compliance by school | Tab Director |
| Judge Shenanigans | `shenanigans.mhtml` | HTML | Judge irregularities and issues | Tab Director |
| Prefs Totals | `prefs.mhtml` | HTML | Summary of preference data completion | Tab Director |
| Strike Totals | `strikes.mhtml` | HTML | Summary of strike usage | Tab Director |
| Unlinked Students | `unlinked_students.mhtml` | HTML | Students not linked to NSDA accounts | Tab Director |
| Student Confirmations | `student_status.mhtml` | HTML | Status of student form confirmations | Tab Director |
| Student Contacts | `student_contacts.mhtml` | CSV | Contact info for students | Tab Director |
| Video Links | `video_links.mhtml` | HTML | Links to async video submissions | Tab Director |
| Timeslot IDs | `timeslots.mhtml` | CSV | Timeslot reference data | Tab Director |
| Paradigms | `questionnaire.mhtml` | HTML | Judge paradigm responses | Tab Director |
| Bulk Questionnaire Data | `bulk_questionnaire.mhtml` | HTML | Bulk export of questionnaire responses | Tab Director |

#### Financial Reports

| Report Name | File | Output Format | Purpose | User Role |
|---|---|---|---|---|
| Entry Fees & Fines | `finance_report.mhtml` | HTML | Comprehensive financial report of fees and fines | Tab Director |
| Finance CSV | `finance_csv.mhtml` | CSV | Exportable financial data | Tab Director |
| School Balances | `school_balances.mhtml` | HTML | Individual school financial balances | Tab Director |
| Fines List | `fines.mhtml` | HTML | Detailed listing of fees and fines | Tab Director |
| Payments List | `payments.mhtml` | HTML | Listing of all payments received | Tab Director |
| Hotel Counts | `hotel_counts.mhtml` | HTML | Hotel room reservation counts | Tab Director |
| Concession Orders | `concessions.mhtml` | HTML | Concession stand orders by school | Tab Director |
| Concessions Print | `concessions_print.mhtml` | Print | Printable concession orders | Registration Staff |
| Concessions Totals | `concessions_totals.mhtml` | HTML | Aggregate concession order totals | Tab Director |
| Invoice All | `invoice_all.mhtml` | Print | Print all school invoices at once | Tab Director |
| Refund Report | `refund_report.mhtml` | HTML | Refund mailing information | Tab Director |
| Refund Report CSV | `refund_report_csv.mhtml` | CSV | Exportable refund data | Tab Director |
| Diocese Financial Balances | `diocese_finance.mhtml` | HTML | NCFL diocese-level financial data | Tab Director |

#### Onsite Registration Packets

| Report Name | File | Output Format | Purpose | User Role |
|---|---|---|---|---|
| Onsite Status | `onsite_status.mhtml` | HTML | Registration check-in status by school | Registration Staff |
| Onsite Print (Reg List) | `onsite_print.mhtml` | Print | Printable registration list | Registration Staff |
| School Labels | `school_labels.mhtml` | Print/Labels | Mailing/packet labels for schools | Registration Staff |
| Student Dance Cards | `student_cards.mhtml` / `student_card_picker.mhtml` | Print | Individual student info cards | Registration Staff |
| Judge Dance Cards | `judge_cards.mhtml` / `judge_card_picker.mhtml` | Print | Individual judge info cards | Registration Staff |
| Packet Registrations | `packet_registrations.mhtml` | Print | Full registration packet with invoice | Registration Staff |
| Packet Assignments | `packet_assignments.mhtml` | Print | Registration packet with room assignments | Registration Staff |
| Packet Invoices | `packet_invoices.mhtml` | Print | Fee/concession invoices for packets | Registration Staff |

#### NSDA Nationals-Specific Reports

| Report Name | File | Output Format | Purpose | User Role |
|---|---|---|---|---|
| Nametags CSV | `nats_nametags.mhtml` | CSV | Name tag data for badge printing | Nats Staff |
| Coach Contacts | `nats_coachlist.mhtml` | HTML | Coach contact information | Nats Staff |
| Problem Sheets | `problem_children.mhtml` | HTML | Schools with registration problems | Nats Staff |
| District Chair Printout | `nats_district_print.mhtml` | Print | District chair reference sheets | Nats Staff |
| Bondless Schools | `nats_bond_check.mhtml` | HTML | Schools without required bonds | Nats Staff |
| Bond Revocations | `bond_revocations.mhtml` | HTML | Bond and refund tracking | Nats Staff |
| Shipping Report | `shipping_report.mhtml` | HTML | Order shipping status | Nats Staff |
| Purchase Orders | `nats_po.mhtml` | HTML | Purchase order data | Nats Staff |
| Congress Bills | `nats_congress.mhtml` | HTML | Legislation/bill submissions | Nats Staff |
| Worlds Sheets | `usa_debate.mhtml` | HTML | WSDC-format data sheets | Nats Staff |
| Four-Year Qualifiers | `four_year_qualifiers.mhtml` | HTML | Students qualifying four or more years | Nats Staff |
| BQ Sheets | `category_card.mhtml` | Print | Category breakdown sheets | Nats Staff |
| Years Attended | `nats_attended.mhtml` | HTML | Student attendance history | Nats Staff |
| Script List | `script_list.mhtml` | CSV | Interp script titles and sources | Nats Staff |
| Single Entry Intents | `single_entry_intents.mhtml` | PDF/CSV | District single-entry intent forms | District Chair |
| Release/Eligibility Forms | `release_forms.mhtml` | HTML | Entry release form tracking | Nats Staff |
| Nats Book Data | `nats_book.mhtml` | HTML | Program book data | Nats Staff |

#### NCFL-Specific Reports

| Report Name | File | Output Format | Purpose | User Role |
|---|---|---|---|---|
| Diocese List | `ncfl_contact.mhtml` | HTML | NCFL diocese contact list | NCFL Staff |
| Judge & Entry Reports | `ncfl_reports.mhtml` | HTML | Combined NCFL reports | NCFL Staff |
| NCFL Entry Reports | `ncfl_entry_reports.mhtml` | HTML | NCFL entry breakdowns | NCFL Staff |
| NCFL Entry Cards | `ncfl_entry_cards.mhtml` | Print | Printable entry cards | NCFL Staff |
| NCFL Judge Cards | `ncfl_judge_cards.mhtml` | Print | Printable judge cards | NCFL Staff |
| NCFL Cards (Combined) | `ncfl_cards.mhtml` | Print | Combined entry and judge cards | NCFL Staff |
| NCFL Book Data | `ncfl_book_data.mhtml` | CSV | Program book data export | NCFL Staff |
| NCFL Codes | `ncfl_codes.mhtml` | HTML | NCFL code assignments | NCFL Staff |
| Diocese Fines | `ncfl_fines.mhtml` | HTML | Fine tracking by diocese | NCFL Staff |
| Diocese Fines CSV | `ncfl_fines_csv.mhtml` | CSV | Exportable fines data | NCFL Staff |
| Diocese Fines Print | `ncfl_fines_print.mhtml` | Print | Printable fines report | NCFL Staff |

#### Vaccine/Health Reports

| Report Name | File | Output Format | Purpose | User Role |
|---|---|---|---|---|
| VaccineCheck Report | `vaccine_check.mhtml` | HTML | Vaccination status overview | Tab Director (owner) |
| VaccineCheck CSV | `vaccine_csv.mhtml` | CSV | Exportable vaccination data | Tab Director (owner) |
| Vaccine Import | `vaccine_import.mhtml` | HTML (upload) | Import vaccination records | Tab Director (owner) |
| Vaccine Status by School | `vaccine_schools.mhtml` | HTML | School-level vaccine compliance | Tab Director (owner) |

---

### 1.2 Panel/Schematic Reports (`web/panel/report/`)

These reports are used by tab staff during active tournament rounds for pairing, ballot, and room management.

#### Schematics & Postings

| Report Name | File | Output Format | Purpose | User Role |
|---|---|---|---|---|
| Postings | `postings.mhtml` | Print | Standard round postings (who debates where) | Tab Staff |
| Big Postings | `bigass_posting.mhtml` | Print | Large-format postings for wall display | Tab Staff |
| Giant Postings | `giant_postings.mhtml` | Print | Extra-large postings | Tab Staff |
| Half Postings | `half_postings.mhtml` | Print | Half-page postings | Tab Staff |
| List Postings | `list_postings.mhtml` | Print | List-format postings | Tab Staff |
| Schematic | `schematic.mhtml` | HTML/Print | Full round schematic view | Tab Staff |
| Round Print (Debate) | `schemat/round_print_debate.mas` | Print | Printable debate schematic | Tab Staff |
| Round Print (Speech Horiz) | `schemat/round_print_speech_horizontal.mas` | Print | Horizontal speech schematic | Tab Staff |
| Round Print (Speech Vert) | `schemat/round_print_speech_vertical.mas` | Print | Vertical speech schematic | Tab Staff |
| Round Print (Congress Horiz) | `schemat/round_print_congress_horizontal.mas` | Print | Horizontal congress schematic | Tab Staff |
| Round Print (Congress Vert) | `schemat/round_print_congress_vertical.mas` | Print | Vertical congress schematic | Tab Staff |
| Round Print (WUDC) | `schemat/round_print_wudc.mas` | Print | WUDC-format schematic | Tab Staff |
| Round Print (Entry List) | `schemat/round_print_entrylist.mas` | Print | Entry list for round | Tab Staff |
| Round Print (Judge List) | `schemat/round_print_judgelist.mas` | Print | Judge list for round | Tab Staff |
| Round CSV (Debate) | `schemat/round_csv_debate.mhtml` | CSV | Exportable debate round data | Tab Staff |
| Slideshow | `slideshow.mhtml` | HTML | Slideshow-format postings for projection | Tab Staff |

#### Ballots

| Report Name | File | Output Format | Purpose | User Role |
|---|---|---|---|---|
| Print Ballots | `print_ballots.mhtml` | Print | Generate printable ballots | Tab Staff |
| Debate Ballots | `ballot/debate.mas` | Print | Debate-format ballot template | Tab Staff |
| Speech Ballots | `ballot/speech.mas` | Print | Speech-format ballot template | Tab Staff |
| Congress Student Ballots | `ballot/congress_student.mhtml` | Print | Congress individual scoring sheet | Tab Staff |
| WSDC Ballots | `ballot/wsdc.mas` | Print | WSDC-format ballot template | Tab Staff |
| WUDC Ballots | `ballot/wudc.mas` | Print | WUDC-format ballot template | Tab Staff |
| Combined Ballots | `ballot/combined.mas` | Print | Multi-format combined ballot | Tab Staff |
| Ballot Labels | `ballot_labels.mhtml` | Print/Labels | Ballot envelope labels | Tab Staff |
| Ballot Table | `ballot_table.mhtml` | HTML | Ballot tracking table | Tab Staff |
| Adjust Labels | `adjust_labels.mhtml` | Print/Labels | Custom label adjustment | Tab Staff |

#### Tab Cards & Sheets

| Report Name | File | Output Format | Purpose | User Role |
|---|---|---|---|---|
| Debate Tab Cards | `cards/debate.mhtml` | Print | Debate cumulative record cards | Tab Staff |
| Speech Tab Cards | `cards/speech.mhtml` | Print | Speech cumulative record cards | Tab Staff |
| Debate Judge Cards | `cards/debate_judges.mhtml` | Print | Judge assignment record cards | Tab Staff |
| Congress Tab Sheets | `cards/congress_tabsheets.mhtml` | Print | Congress tabulation sheets | Tab Staff |
| Speech Tab Sheets | `cards/speech_tabsheets.mhtml` | Print | Speech tabulation sheets | Tab Staff |
| NCFL Cleared | `ncfl/cleared.mhtml` | HTML | NCFL cleared entries list | NCFL Staff |
| NCFL Tab Cards | `ncfl/tab_cards.mhtml` | Print | NCFL-specific tab cards | NCFL Staff |

#### Other Panel Reports

| Report Name | File | Output Format | Purpose | User Role |
|---|---|---|---|---|
| Chamber Report | `chamber_report.mhtml` | HTML | Congress chamber composition | Tab Staff |
| Chamber Roster | `chamber_roster.mhtml` | Print | Printable chamber rosters | Tab Staff |
| Congress Scoresheet | `congress_scoresheet.mhtml` | Print | Congress scoring sheets | Tab Staff |
| Seating Chart | `seating.mhtml` | Print | Congress seating arrangements | Tab Staff |
| Placards | `placards.mhtml` | Print | Table/podium placards | Tab Staff |
| Strike Cards | `strike_cards.mhtml` | Print | Judge strike information cards | Tab Staff |
| Double Entry Report | `double_entry.mhtml` | HTML | Double-entry conflict detection | Tab Staff |
| Disasters | `disasters.mhtml` | HTML | Round problem/conflict detection | Tab Staff |
| Judge Chart | `judge_chart.mhtml` | HTML | Judge assignment overview | Tab Staff |
| Judge Points | `judge_points.mhtml` | HTML | Judge point averages | Tab Staff |
| Pref Experience | `pref_experience.mhtml` | HTML | Judge preference fulfillment analysis | Tab Staff |
| Preset Draw | `preset_draw.mhtml` | HTML | Preset pairing draw results | Tab Staff |
| Rooms Master | `rooms_master.mhtml` | HTML | Master room assignment list | Tab Staff |
| Round Report | `round_report.mhtml` | HTML | Detailed round-by-round report | Tab Staff |
| Nats Elim Bios | `nats_elim_bios.mhtml` | HTML | Elimination round judge bios | Nats Staff |
| Tab Sheets | `tabs.mhtml` | Print | Tab sheet printouts | Tab Staff |

---

### 1.3 Tabbing Reports (`web/tabbing/report/`)

These reports are used by tab directors for results, auditing, and post-tournament reporting.

#### Results & Awards

| Report Name | File | Output Format | Purpose | User Role |
|---|---|---|---|---|
| Awards Ceremony | `awards_ceremony.mhtml` | HTML | Awards ceremony script/display | Tab Director |
| Awards Script | `awards_script.mhtml` | HTML | Presenter script for awards | Tab Director |
| Awards Pickup | `awards_pickup.mhtml` | HTML | Award distribution tracking | Tab Director |
| Awards by School | `awards_school.mhtml` | HTML | Awards organized by school | Tab Director |
| Sweepstakes by School | `sweep_schools.mhtml` | HTML | Sweepstakes standings by school | Tab Director |
| Sweepstakes Print | `sweep_schools_print.mhtml` | Print | Printable sweepstakes standings | Tab Director |
| Sweepstakes by Entry | `sweep_entries.mhtml` | HTML | Sweepstakes by individual entry | Tab Director |
| Sweepstakes Students | `sweep_students.mhtml` | HTML | Sweepstakes by student | Tab Director |
| Sweepstakes Students Print | `sweep_students_print.mhtml` | Print | Printable student sweepstakes | Tab Director |
| Post Sweeps to NSDA | `sweep_post.mhtml` | HTML (action) | Post sweepstakes to NSDA database | Tab Director |
| School Results | `school.mhtml` | HTML | Results organized by school | Tab Director |
| School Results Print | `school_results_print.mhtml` | Print | Printable school results | Tab Director |
| Round Robin Script | `round_robin_script.mhtml` | HTML | Round robin results script | Tab Director |
| Codebreaker | `codebreaker.mhtml` | HTML | Reveal entry codes to names | Tab Director |
| Code List | `code_list.mhtml` | HTML | Master code-to-entry mapping | Tab Director |
| Code Print | `code_print.mhtml` | Print | Printable code list | Tab Director |
| Event Speakers | `event_speakers.mhtml` | HTML | Speaker award results by event | Tab Director |
| Event Speakers CSV | `event_speakers_csv.mhtml` | CSV | Exportable speaker results | Tab Director |

#### Audit & Verification

| Report Name | File | Output Format | Purpose | User Role |
|---|---|---|---|---|
| Print Audit | `print_audit.mhtml` | Print | Printable ballot audit trail | Tab Director |
| Audit CSV | `audit_csv.mas` | CSV | Exportable audit data | Tab Director |
| Audit Print | `audit_print.mas` | Print | Formatted audit printout | Tab Director |
| Audit Table | `audit_table.mas` | HTML | Interactive audit table | Tab Director |
| Section Audit | `section_audit.mhtml` | HTML | Section/panel composition audit | Tab Director |
| Score Report | `score_report.mhtml` | HTML | Detailed scoring analysis | Tab Director |
| Raw Ballots | `raw_ballots.mhtml` | HTML | Unprocessed ballot data view | Tab Director |
| Print Pending | `print_pending.mhtml` | Print | Ballots still pending entry | Tab Staff |
| Forfeits | `forfeits.mhtml` | HTML | Forfeit and no-show listing | Tab Director |
| Congress Scores | `congress_scores.mhtml` | HTML | Congress scoring details | Tab Director |
| PO Report | `po_report.mhtml` | HTML | Presiding officer listing | Tab Director |
| Room Cleaning | `room_cleaning.mhtml` | HTML | Room usage/cleaning schedule | Tab Director |
| Actual Schedule | `actual_schedule.mhtml` | HTML | Actual (vs planned) schedule by event | Tab Director |

#### CSV Exports

| Report Name | File | Output Format | Purpose | User Role |
|---|---|---|---|---|
| Last Elim Participated CSV | `last_round_csv.mhtml` | CSV | Last elimination round per entry | Tab Director |
| Rounds Judged CSV | `judge_work.mhtml` | CSV | Judge work/round count export | Tab Director |
| NDCA Points CSV | `ndca/points_csv.mhtml` | CSV | NDCA ranking points export | Tab Director |

#### Specialized / Organization Reports

| Report Name | File | Output Format | Purpose | User Role |
|---|---|---|---|---|
| NAUDL Student Export | `naudl_student_export.mhtml` | JSON | BDL/NAUDL student data export | Admin |
| NAUDL Tournament Export | `naudl_tourn_export.mhtml` | JSON | BDL/NAUDL tournament data export | Admin |
| NSDA Autoqualifiers | `nsda/autoqualifiers.mhtml` | HTML | Next-year automatic qualifiers | Nats Staff |
| NSDA Awards Report | `nsda/awards_report.mhtml` | CSV | Nationals contact results CSV | Nats Staff |
| NSDA Final Cumes | `nsda/final_cumes.mhtml` | CSV | Final cumulative scores | Nats Staff |
| NSDA Full Packet | `nsda/full_packet.mhtml` | HTML | Complete NSDA results packet | Nats Staff |
| NSDA Repeat Finalists | `nsda/repeat_finalists.mhtml` | HTML | Multi-year finalist tracking | Nats Staff |
| NSDA School Awards | `nsda/school_awards.mhtml` | HTML | School-level NSDA awards | Nats Staff |
| NCFL Diocesan Sweeps | `ncfl/show_diocesan_sweeps.mhtml` | HTML | NCFL diocese sweepstakes | NCFL Staff |
| NCFL Print Diocesan Sweeps | `ncfl/print_diocesan_sweeps.mhtml` | Print | Printable diocese sweepstakes | NCFL Staff |
| NCFL Cooke Points Save | `ncfl/save_cooke_points.mhtml` | HTML (action) | Save Cooke Points to running total | NCFL Owner |
| NDCA Points | `ndca/points.mhtml` | HTML | NDCA ranking points display | Tab Director |
| Legion Report | `legion_report.mhtml` | Print | American Legion results | Tab Director |
| Send Legion | `send_legion.mhtml` | HTML (action) | Email Legion results | Tab Director |
| Reading Assignments | `reading.mhtml` | HTML | Interp reading assignments | Tab Staff |
| Reading Judges | `readingjudges.mhtml` | HTML | Judge reading assignments | Tab Staff |
| Pickup | `pickup.mhtml` | HTML | Ballot pickup tracking | Tab Staff |
| Roles | `roles.mhtml` | HTML | Tournament role assignments | Tab Director |
| Stats | `stats.mhtml` | HTML | Tabbing statistics overview | Tab Director |
| Packet | `packet.mhtml` | HTML | Results packet generation | Tab Director |
| Prelims Order | `prelims_order.mhtml` | HTML | Preliminary round ordering | Tab Director |

---

### 1.4 Tabbing Publish / Results (`web/tabbing/publish/`)

| Report Name | File | Output Format | Purpose | User Role |
|---|---|---|---|---|
| Generate Results | `generate_results.mhtml` | HTML (action) | Generate result sets for publication | Tab Director |
| Generate Sweeps | `generate_sweeps.mhtml` | HTML (action) | Generate sweepstakes calculations | Tab Director |
| Generate Bracket | `generate_bracket.mas` | HTML | Generate elimination bracket | Tab Director |
| Bracket Display | `bracket.mhtml` | HTML | Display elimination bracket | Public |
| Bracket WUDC | `bracket_wudc.mhtml` | HTML | WUDC-format bracket | Public |
| Publish All | `publish_all.mhtml` | HTML (action) | Publish all results at once | Tab Director |
| Publish All Rounds | `publish_all_rounds.mhtml` | HTML (action) | Publish all round results | Tab Director |
| Upload Results | `upload_results.mhtml` | HTML (action) | Upload results to external systems | Tab Director |
| Nationals Ranks | `nationals_ranks.mhtml` | HTML | NSDA nationals ranking display | Tab Director |
| Register Nationals | `register_nationals.mhtml` | HTML (action) | Auto-register qualifiers for nationals | Tab Director |
| NSDA Sweepstakes District | `swdistrict.mhtml` | HTML | District-level sweepstakes | Tab Director |
| Result Set Management | `result_set_add.mhtml`, `result_set_adjust.mhtml`, `result_set_switch.mhtml` | HTML | Create and manage result sets | Tab Director |

### 1.5 Panel Publish (`web/panel/publish/`)

| Report Name | File | Output Format | Purpose | User Role |
|---|---|---|---|---|
| Publish Everything | `publish_everything.mhtml` | HTML (action) | Publish all schematics/pairings | Tab Staff |
| Publish Switch | `publish_switch.mhtml` | HTML (action) | Toggle publication of individual rounds | Tab Staff |

---

### 1.6 Public Results (`web/index/results/`)

These are public-facing results pages accessible without tournament staff login.

| Report Name | File | Output Format | Purpose | User Role |
|---|---|---|---|---|
| Circuit Stats | `circuit_stats.mhtml` | HTML | Circuit-wide statistical overview | Public |
| Circuit Chapter | `circuit_chapter.mhtml` | HTML | Chapter/school results within a circuit | Public |
| Circuit Tourney Portal | `circuit_tourney_portal.mhtml` | HTML | Tournament results portal for a circuit | Public |
| Debate Cumesheet | `debate_cumesheet.mhtml` | HTML | Cumulative debate records | Public |
| Debate Stats | `debate_stats2.mhtml` | HTML | Debate performance statistics | Public |
| Debate Stats ADA | `debate_stats_ada.mhtml` | HTML | ADA-specific debate statistics | Public |
| Speaker Rankings | `speaker_rankings_by_circuit.mhtml` | HTML | Speaker point rankings by circuit | Public |
| Speaker Detail | `speaker_detail.mhtml` | HTML | Individual speaker statistics | Public |
| Team Lifetime Record | `team_lifetime_record.mhtml` | HTML | Historical team record | Public |
| Team Results | `team_results.mhtml` | HTML | Team tournament results | Public |
| Results by Chapter | `results_by_chapter.mhtml` | HTML | Results organized by school/chapter | Public |
| NDCA Standings | `ndca_standings.mhtml` | HTML | NDCA national standings | Public |
| NDT/CEDA Points | `ndt_ceda_points.mhtml` | HTML | NDT/CEDA ranking points | Public |
| ADA Points | `ada_points.mhtml` | HTML | ADA ranking points | Public |
| TOC Bids | `toc_bids.mhtml` | HTML | Tournament of Champions bid tracking | Public |
| Qualifiers | `qualifiers.mhtml` | HTML | Qualification status display | Public |
| RPI Detail | `rpi_detail.mhtml` | HTML | RPI ranking methodology detail | Public |

---

### 1.7 Tabbing Results Exports (`web/tabbing/results/`)

| Report Name | File | Output Format | Purpose | User Role |
|---|---|---|---|---|
| Results CSV | `csv.mhtml` / `results_csv.mas` | CSV | General results CSV export | Tab Director |
| Speakers CSV | `speakers_csv.mhtml` | CSV | Speaker point results CSV | Tab Director |
| NSDA Points | `nsda_points.mhtml` | HTML (action) | Calculate/upload NSDA merit points | Tab Director |
| NSDA Qualifiers | `nsda_qualifiers.mhtml` | HTML/CSV | District qualifier tracking | Tab Director |
| NSDA Sweepstakes | `nsda_sweepstakes.mhtml` | HTML | NSDA sweepstakes calculation | Tab Director |
| Results Table | `results_table.mas` | HTML | Formatted results table component | Tab Director |
| Speakers View | `speakers.mhtml` | HTML | Speaker results display | Tab Director |
| See Order | `see_order.mhtml` | HTML | View results order sequence | Tab Director |
| Roles | `roles.mhtml` | HTML | Results role assignments | Tab Director |

---

### 1.8 User-Facing Exports (`web/user/` and `web/register/judge/`)

| Report Name | File | Output Format | Purpose | User Role |
|---|---|---|---|---|
| Entry CSV | `user/enter/entry_csv.mhtml` | CSV | Coach export of own entries | Coach |
| Entry by Person CSV | `user/enter/by_person_csv.mhtml` | CSV | Export entries organized by person | Coach |
| Entry Print | `user/enter/entry_print.mhtml` | Print | Printable entry list for coaches | Coach |
| By Person Print | `user/enter/by_person_print.mhtml` | Print | Printable person-organized entry list | Coach |
| Invoice Print | `user/enter/invoice_print.mhtml` | Print | Printable school invoice | Coach |
| Event Results CSV | `user/results/event_csv.mhtml` | CSV | Event results CSV export | Public |
| School Results Print | `user/results/school_results_print.mhtml` | Print | Printable school results | Coach |
| Online Ballots Print | `user/results/online_ballots_print.mhtml` | Print | Printable online ballot record | Coach/Judge |
| Report Print | `user/results/report_print.mhtml` | Print | General results printout | Public |
| Judge CSV | `register/judge/csv.mhtml` | CSV | Judge roster CSV export | Tab Director |
| Judge Pref Report | `register/judge/pref_report.mhtml` | HTML | Judge preference report | Tab Director |
| Hired Judge Report | `register/judge/hired_judge_report.mhtml` | HTML | Hired/obligated judge report | Tab Director |
| Chapter Students CSV | `user/chapter/student_csv.mhtml` | CSV | School student roster export | Coach |
| Chapter Judges CSV | `user/chapter/judges_csv.mhtml` | CSV | School judge roster export | Coach |
| Export Prefs | `user/enter/ratings/export_prefs.mhtml` | CSV | Export pref sheets | Coach |
| BDL Student Export | `user/circuit/bdl_student_export.mhtml` | CSV | BDL/NAUDL circuit student export | Circuit Admin |
| Diocese Tourn Print (Event) | `user/diocese/tourn_print_event.mhtml` | Print | Diocese tournament report by event | Diocese Admin |
| Diocese Tourn Print (School) | `user/diocese/tourn_print_school.mhtml` | Print | Diocese tournament report by school | Diocese Admin |
| Diocese Invoice Print | `user/diocese/invoice_print.mhtml` | Print | Diocese invoice printout | Diocese Admin |
| Diocese Invoice School Print | `user/diocese/invoice_school_print.mhtml` | Print | Diocese school-level invoice | Diocese Admin |
| NSDA WSDC Print | `user/nsda/wsdc_print.mhtml` | Print | WSDC team data printout | NSDA Admin |
| NSDA Online Ballots Print | `user/nsda/online_ballots_print.mhtml` | Print | District online ballot printout | District Chair |

---

## Part 2: Integrations Inventory

### 2.1 Payment Processing

#### PayPal
- **Purpose**: Online tournament fee payment
- **Required/Optional**: Optional (tournament-configurable)
- **User-Visible Impact**: Coaches can pay tournament fees online via PayPal checkout button. Includes 4% processing fee. Minimum payment $10.
- **Configuration**: Tournament sets `paypal_client_id` in tournament settings
- **Source References**:
  - `web/user/enter/paypal.mas` - PayPal SDK integration and order creation
  - `web/user/enter/fees.mhtml` - Fee display page that embeds PayPal
  - `web/setup/tourn/payment.mhtml` - Payment configuration
  - `web/setup/tourn/paypal_save.mhtml` - Save PayPal settings
- **API**: PayPal JavaScript SDK (`https://www.paypal.com/sdk/js`)
- **Flow**: Client-side order creation -> PayPal approval -> POST to indexcards API `/user/enter/paypal`

#### Authorize.net
- **Purpose**: Credit card and ACH/E-Check payment processing
- **Required/Optional**: Optional (tournament-configurable, alternative to PayPal)
- **User-Visible Impact**: Coaches can pay via credit card or ACH. Configurable processing fee percentages for CC vs ACH. Minimum $10.
- **Configuration**: `authorizenet_api_login`, `authorizenet_client_key`, `authorizenet_cc_fee`, `authorizenet_ach_fee`, `authorizenet_ach_enable`
- **Source References**:
  - `web/user/enter/authorizenet.mas` - Authorize.net Accept.js integration
  - `web/setup/tourn/payment.mhtml` - Payment configuration
- **API**: Authorize.net Accept.js (`https://js.authorize.net/v3/AcceptUI.js`)
- **Flow**: Client-side tokenization -> POST to indexcards API `/user/enter/authorize`

#### TMoney (Tabroom Money)
- **Purpose**: Internal NSDA payment/money management system
- **Required/Optional**: Optional (enabled via `tmoney_enable` or `tmoney_staging` tournament setting)
- **User-Visible Impact**: Alternative payment pathway for tournaments; visible in invoices and fee pages
- **Configuration**: `$Tab::tmoney_url`, `$Tab::tmoney_staging_url` (global config); `tmoney_enable`, `tmoney_staging` (tournament settings)
- **Source References**:
  - `web/autohandler` (lines 113-117) - URL resolution
  - `web/index/tourn/tournament_money.mhtml` - TMoney portal
  - `web/register/school/invoice.mhtml` - Invoice integration
  - `web/setup/tourn/settings.mhtml` - Settings toggle

---

### 2.2 NSDA API (speechanddebate.org)

- **Purpose**: Core integration with the National Speech and Debate Association database for member verification, points tracking, roster sync, and results posting
- **Required/Optional**: Required for NSDA-affiliated tournaments; optional for independent tournaments
- **User-Visible Impact**: Member ID verification, automatic roster import, merit point calculation and upload, qualifier tracking, district management
- **Configuration**: `$Tab::nsda_api_user`, `$Tab::nsda_api_key`, `$Tab::nsda_api_endpoint` (`https://api.speechanddebate.org`), `$Tab::nsda_api_version` (`/v2`)
- **Authentication**: HTTP Basic Auth (base64-encoded user:key)
- **Source References**:
  - `web/funclib/nsda/api_client.mas` - REST client wrapper for all NSDA API calls
  - `web/funclib/nsda/create_invoice.mas` - Create NSDA invoices
  - `web/funclib/nsda/nats_appearances.mas` - Track nationals appearances
  - `web/user/nsda/import_chapter.mhtml` - Import chapter from NSDA
  - `web/user/nsda/import_nsda_roster.mhtml` - Sync student roster from NSDA
  - `web/user/nsda/sync_roster.mhtml` - Roster synchronization
  - `web/user/nsda/link.mhtml` / `link_confirm.mhtml` - Link Tabroom account to NSDA
  - `web/user/nsda/student_link.mhtml` - Link student to NSDA ID
  - `web/user/login/import_account.mhtml` - Import account from NSDA
  - `web/tabbing/results/nsda_points.mhtml` - Upload merit points
  - `web/tabbing/results/nsda_qualifiers.mhtml` - Manage qualifiers
  - `web/tabbing/report/sweep_post.mhtml` - Post sweepstakes to NSDA
  - `web/tabbing/report/nsda/awards_report.mhtml` - NSDA awards CSV
  - `web/tabbing/publish/sw_nsda_save.mhtml` / `sw_nsda_students.mas` - Save NSDA sweepstakes data

---

### 2.3 NSDA Store API

- **Purpose**: Integration with the NSDA online store for purchasing tournament-related products (Tabroom usage, Campus licenses, observer access)
- **Required/Optional**: Optional
- **User-Visible Impact**: Redirects users to NSDA store with pre-filled cart for purchasing tournament services
- **Configuration**: `$Tab::nsda_store_api` (`https://www.speechanddebate.org/store/wp-json/cocart/v2/cart/add-item`), `$Tab::nsda_store_redirect`, `$Tab::nsda_product_codes`
- **Source References**:
  - `doc/conf/General.pm` (lines 57-64) - Configuration

---

### 2.4 Push Notifications (OneSignal)

- **Purpose**: Browser push notifications for tournament blasts (pairing releases, announcements, schedule changes)
- **Required/Optional**: Optional (user opt-in)
- **User-Visible Impact**: Bell icon in header. Users can subscribe/unsubscribe to receive push notifications on desktop and mobile browsers. Replaced SMS notifications (carriers blocked free texting).
- **Configuration**: `$Tab::onesignal_app_id`, `$Tab::onesignal_safari_id`
- **Source References**:
  - `web/funclib/push_login.mas` - Push notification subscription UI and OneSignal initialization (login flow)
  - `web/funclib/push_notifications.mas` - Push notification management (subscribe/unsubscribe/logout)
  - `web/lib/javascript/onesignal/OneSignalSDK.page.js` - Bundled OneSignal SDK
  - `web/autohandler` (line 855) - Push login component inclusion
  - `web/user/login/profile.mhtml` - Profile-level push settings
- **API Calls**: OneSignal JavaScript SDK for client-side subscription; server-side via indexcards API (`/user/push/sync`, `/user/push/show`, `/user/push/enable`)

---

### 2.5 Email (SMTP)

- **Purpose**: Transactional and bulk email delivery for tournament communications, account management, pairing blasts, and notifications
- **Required/Optional**: Required (core platform feature)
- **User-Visible Impact**: Account verification, password reset, tournament registration confirmations, pairing notifications, custom coach-to-coach emails, circuit emails
- **Configuration**: `$Tab::smtp_server` (localhost), `$Tab::admin_smtp_server` (localhost), `$Tab::admin_email` (help@tabroom.com)
- **Source References**:
  - `web/funclib/send_notify.mas` - Central email sending component (uses REST::Client to post to indexcards API)
  - `web/lib/Tab/Email.pm` - Email ORM model (stores sent emails in database)
  - `web/user/enter/send_email.mhtml` - Coach email sending
  - `web/user/enter/emails.mhtml` - Email history view
  - `web/user/circuit/email_send.mhtml` - Circuit-wide email
  - `web/user/login/forgot_send.mhtml` - Password reset email
  - `web/panel/schemat/blast_pairing.mas` - Pairing blast email
  - `web/panel/schemat/blast_message.mas` - Custom blast message
  - `web/register/emails/send.mhtml` - Registration emails
  - `web/tabbing/report/send_legion.mhtml` - Legion results email

---

### 2.6 Pairing/Schedule Blasts (Indexcards API)

- **Purpose**: Real-time notification system for releasing pairings, schedule changes, and tournament announcements to competitors, judges, and coaches
- **Required/Optional**: Required (core tournament operation feature)
- **User-Visible Impact**: Competitors and judges receive immediate notifications when pairings are released or announcements are made
- **Configuration**: `$Tab::indexcards_url` (Node.js API backend)
- **Source References**:
  - `web/panel/schemat/blast_pairing.mas` - Blast pairing release (calls `/tab/{tourn}/round/{round}/blast` or `/tab/{tourn}/timeslot/{ts}/blast`)
  - `web/panel/schemat/blast_message.mas` - Custom message blast
  - `web/tabbing/status/blast_missing.mhtml` - Blast to missing judges/entries

---

### 2.7 NSDA Campus (Jitsi Video Conferencing)

- **Purpose**: Virtual competition rooms for online tournaments using NSDA's Jitsi-based video platform
- **Required/Optional**: Optional (for online/hybrid tournaments)
- **User-Visible Impact**: Competitors and judges get links to virtual rooms; rooms auto-created per round; observer access management
- **Configuration**: `$Tab::jitsi_key`, `$Tab::jitsi_uri` (`https://campus.speechanddebate.org`)
- **Source References**:
  - `web/setup/rooms/nsda_campus.mhtml` - Campus room configuration
  - `web/setup/events/online.mhtml` - Online event mode settings
  - `web/user/enter/campus_observers.mhtml` - Observer management
  - `web/user/campus/room_log.mhtml` - Room activity logging
  - `web/panel/schemat/show_debate.mas`, `show_speech.mas`, `show_congress.mas` - Campus links in schematics
  - `web/panel/room/utility.mhtml` / `utility_save.mhtml` - Room utility management
  - `web/user/admin/campus_report.mhtml` - Campus usage report

---

### 2.8 Document Sharing (DocShare)

- **Purpose**: Real-time document sharing for debate rounds (evidence exchange)
- **Required/Optional**: Optional (event-configurable via `auto_docshare` setting)
- **User-Visible Impact**: Debaters can share documents/evidence within their assigned round rooms
- **Configuration**: `$Tab::share_api_endpoint` (`/v1/share`), `$Tab::docshare_key`, `$Tab::indexcards_url`
- **Source References**:
  - `web/funclib/docshare_rooms.mas` - Creates document sharing rooms via REST API call to `/ext/share/makeShareRooms`

---

### 2.9 Google Calendar API

- **Purpose**: Calendar event creation with Google Hangout links for virtual meetings/rounds
- **Required/Optional**: Optional
- **User-Visible Impact**: Automated calendar events with video meeting links
- **Configuration**: `$Tab::google_client_email`, `$Tab::google_client_key`, `$Tab::google_user_email`, `$Tab::google_calendar_id`
- **Authentication**: OAuth 2.0 JWT (RS256) service account
- **Source References**:
  - `web/lib/Tab/GoogleCalendar.pm` - Full Google Calendar API client (auth, createEvent, deleteEvent)
- **API**: Google Calendar API v3 (`https://www.googleapis.com/calendar/v3/calendars/`)

---

### 2.10 Amazon S3 (File Storage)

- **Purpose**: Cloud storage for tournament files (uploads, documents, legislation, release forms, evidence)
- **Required/Optional**: Required (core file storage infrastructure)
- **User-Visible Impact**: File uploads for legislation, release forms, school forms, evidence, and other tournament documents
- **Configuration**: `$Tab::s3_config`, `$Tab::s3_cmd` (s3cmd CLI), `$Tab::s3_bucket` (`s3://tabroom-files`), `$Tab::s3_base` (`https://s3.amazonaws.com/tabroom-files`), `$Tab::s3_url`
- **Source References**:
  - `doc/conf/General.pm` (lines 79-84) - S3 configuration
  - `web/user/enter/release_upload.mhtml` - Release form upload
  - `web/user/enter/legislation_upload.mhtml` / `legislation.mhtml` - Legislation upload
  - `web/user/nsda/upload_legislation.mhtml` - NSDA legislation upload
  - `web/lib/Tab/File.pm` - File ORM model

---

### 2.11 NAUDL / Salesforce API

- **Purpose**: Integration with National Association for Urban Debate Leagues' Salesforce CRM for student and tournament data synchronization
- **Required/Optional**: Optional (NAUDL-affiliated tournaments only)
- **User-Visible Impact**: Automatic posting of student participation and tournament results to NAUDL's Salesforce system
- **Configuration**: `$Tab::naudl_username`, `$Tab::naudl_password`, `$Tab::naudl_token`, `$Tab::naudl_host` (`https://cs45.salesforce.com`), `$Tab::naudl_client_id`, `$Tab::naudl_client_secret`, `$Tab::naudl_tourn_endpoint`, `$Tab::naudl_student_endpoint`, `$Tab::naudl_sta_endpoint`
- **Source References**:
  - `web/user/admin/naudl/salesforce_autopost.mhtml` - Automated Salesforce data posting
  - `web/user/admin/naudl/sections.mhtml` - NAUDL section management
  - `web/user/admin/naudl/sta_pairs.mhtml` - STA pairing data
  - `web/user/admin/naudl/sta.mhtml` - STA management
  - `web/user/admin/naudl/students.mhtml` - NAUDL student data
  - `web/user/admin/naudl/tournaments.mhtml` - NAUDL tournament data
  - `web/tabbing/report/naudl_student_export.mhtml` - Student export for NAUDL
  - `web/tabbing/report/naudl_tourn_export.mhtml` - Tournament export for NAUDL

---

### 2.12 Matomo Analytics

- **Purpose**: Web analytics and usage tracking (self-hosted, hosted at `analytics.speechanddebate.org`)
- **Required/Optional**: Optional (currently disabled in production via hostname check - set to `no-www.tabroom.com`)
- **User-Visible Impact**: No direct user impact; provides site administrators with usage analytics
- **Configuration**: Hardcoded in autohandler - tracker URL `https://analytics.speechanddebate.org/`, site ID `4`
- **Source References**:
  - `web/autohandler` (lines 548-570) - Matomo JavaScript tracker embed (conditionally loaded)

---

### 2.13 Discourse SSO (Single Sign-On)

- **Purpose**: Single sign-on integration with Discourse-based support forum (`support.tabroom.com`)
- **Required/Optional**: Optional (support site access)
- **User-Visible Impact**: Users can log into the Tabroom support forum using their Tabroom credentials
- **Configuration**: `$Tab::discourse_secret`
- **Authentication**: HMAC-SHA256 payload validation with nonce exchange
- **Source References**:
  - `web/user/login/sso.mhtml` - SSO endpoint that validates Discourse payload and returns user info
  - `doc/conf/General.pm` (line 87) - Secret configuration

---

### 2.14 GeoIP (MaxMind)

- **Purpose**: Geographic location lookup from IP addresses for session tracking and security
- **Required/Optional**: Optional (enrichment feature)
- **User-Visible Impact**: Session location display; geographic-based security checks
- **Configuration**: `$Tab::geoip` (`/var/lib/geoip/GeoLite2-City.mmdb`), `$Tab::geoisp` (`/var/lib/geoip/GeoIP2-ISP.mmdb`)
- **Source References**:
  - `doc/conf/General.pm` (lines 46-47) - GeoIP database paths
  - `web/funclib/session_location.mas` - IP-to-location resolution
  - `web/funclib/session_agent.mas` - Session agent/location tracking
  - `web/lib/Tab/Session.pm` - Session model with geoip field

---

### 2.15 LaTeX / PDF Generation

- **Purpose**: Server-side PDF generation for printable ballots, reports, and formatted documents
- **Required/Optional**: Required (core printing infrastructure)
- **User-Visible Impact**: All printed ballots, tab cards, labels, and formatted PDF reports
- **Configuration**: `$Tab::latex_path_prefix` (`/usr/bin`), `$Tab::pdflatex_path`, `$Tab::dvipdfm_path`, `$Tab::latex2rtf_path`, `$Tab::gs_path` (ghostscript)
- **Source References**:
  - `doc/conf/General.pm` (lines 6, 112-120) - Binary paths
  - `doc/conf/General.pm` `texify()` function - LaTeX string sanitization
  - Multiple ballot and print templates across `web/panel/report/ballot/` directory

---

### 2.16 Indexcards API (Node.js Backend)

- **Purpose**: Modern Node.js API backend that handles real-time operations, payment processing callbacks, push notification management, blast delivery, and various async operations
- **Required/Optional**: Required (core backend service)
- **User-Visible Impact**: Powers all real-time features including blasts, payment processing, push notifications, and session management
- **Configuration**: `$Tab::indexcards_url`, `$Tab::indexcards_user`, `$Tab::indexcards_key`
- **Source References**:
  - Referenced throughout the codebase via `$Tab::indexcards_url` for AJAX calls
  - Payment callbacks: `/user/enter/paypal`, `/user/enter/authorize`
  - Push notifications: `/user/push/sync`, `/user/push/show`, `/user/push/enable`
  - Blasts: `/tab/{tourn}/round/{round}/blast`, `/tab/{tourn}/timeslot/{ts}/blast`
  - Session: `/user/updateLastAccess`
  - Server monitoring: `/glp/servers/count`
  - Email delivery: Notfications routed through indexcards

---

### 2.17 Follower / Notification Subscriptions

- **Purpose**: Internal system allowing users to follow entries, judges, schools, and students to receive notifications about their activities
- **Required/Optional**: Optional (user opt-in)
- **User-Visible Impact**: Users receive notifications when followed entities have pairings, results, etc.
- **Source References**:
  - `web/lib/Tab/Follower.pm` - Follower ORM model (tracks person, tourn, judge, entry, school, student)
  - `web/register/school/followers.mhtml` - School follower management
  - `web/user/chapter/follow.mhtml` - Chapter follow toggle
  - `web/user/student/follower_switch.mhtml` - Student follower toggle

---

## Summary Statistics

### Reports & Exports
- **Registration Reports**: ~70+ distinct reports/exports
- **Panel/Schematic Reports**: ~35+ print outputs and reports
- **Tabbing Reports**: ~45+ reports, audits, and exports
- **Public Results Pages**: ~17 public-facing results views
- **User-Facing Exports**: ~22 exports available to coaches and public users
- **Total**: approximately **190+ distinct report/export surfaces**

### Output Format Distribution
- **HTML (interactive)**: ~100 reports
- **Print (formatted for printing)**: ~55 reports
- **CSV (data export)**: ~25 exports
- **PDF/LaTeX**: ~10 formatted documents
- **JSON**: ~3 data exports (NAUDL)
- **Action (triggers processing)**: ~10 action endpoints

### Integrations
- **Payment**: 3 systems (PayPal, Authorize.net, TMoney)
- **NSDA Ecosystem**: 3 integrations (NSDA API, NSDA Store, NSDA Campus/Jitsi)
- **Notifications**: 2 systems (Email/SMTP, OneSignal Push)
- **Cloud Services**: 2 systems (Amazon S3, GeoIP/MaxMind)
- **External Orgs**: 1 system (NAUDL/Salesforce)
- **Video/Collab**: 2 systems (Jitsi/Campus, Google Calendar/Hangouts)
- **Content**: 2 systems (DocShare, LaTeX/PDF)
- **Analytics/Auth**: 2 systems (Matomo, Discourse SSO)
- **Internal Backend**: 1 system (Indexcards Node.js API)
- **Total**: **18 distinct integration points**
