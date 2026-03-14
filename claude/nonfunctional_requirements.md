# Non-Functional Requirements (NFR) Catalog -- Tabroom Rebuild

This document catalogs the non-functional requirements for the Tabroom platform
rebuild, derived from direct analysis of the existing Perl/Mason codebase,
database schema (`doc/sql/current-schema.sql`), and existing catalog documents
in `claude/`. The target stack is Python/FastAPI, PostgreSQL, Next.js 15, Azure
(Container Apps), Redis Streams, with 28+ microservices.

---

## 1. Real-Time Requirements

### 1.1 Operations Requiring Real-Time or Near-Real-Time Updates

| Operation | Current Users | Update Frequency | Current Mechanism |
|-----------|--------------|------------------|-------------------|
| Ballot status on schematic view | Tab staff | 15-second poll | AJAX polling to indexcards API (`/tab/{tourn}/round/{round}/status`), updates judge-result cells in-place |
| Dashboard ballot progress | Tab staff | 15-second poll | AJAX polling to indexcards API (`/tab/{tourn}/all/dashboard`), builds/updates round status cards |
| Ballot entry status log | Tab staff | 15-second poll | `setInterval(refreshLogs, 15000)` on `status.mhtml` |
| Awards pickup tracking | Tab staff | 15-second poll | `setInterval(refreshPickups, 15000)` on `awards_pickup.mhtml` |
| Server count (admin) | Site admin | 15-second poll | AJAX to `/glp/servers/count` |
| Blast delivery status | Tab staff | 15-second poll | `setInterval(checkBlastStatus, 15000)` on schematic view |

**Source files:**
- `/home/jeloni/tabroom/web/panel/schemat/show.mhtml` (lines 360-640)
- `/home/jeloni/tabroom/web/tabbing/status/dashboard.mhtml` (lines 150-380)
- `/home/jeloni/tabroom/web/tabbing/status/status.mhtml` (lines 456-802)
- `/home/jeloni/tabroom/web/tabbing/report/awards_pickup.mhtml`

### 1.2 Current Approach

The system uses a **uniform 15-second AJAX polling** pattern throughout. There
are no WebSocket connections, no Server-Sent Events (SSE), and no long-polling.
All real-time data flows through a separate Node.js API service referred to
internally as "indexcards" (`$Tab::indexcards_url`), which serves as a read
cache / API layer on top of the MariaDB database.

Page refreshes are the fallback: the schematic view (`show.mhtml`) does not
auto-refresh the page itself -- it updates individual DOM elements via jQuery
based on the AJAX response. Dashboard and status pages follow the same pattern.

Push notifications use OneSignal (`OneSignalSDK.page.js`) for browser push
notifications, integrated via `push_login.mas`. This is opt-in per user and
supplements (does not replace) the polling model.

### 1.3 Acceptable Latency

| Operation | Tolerance |
|-----------|-----------|
| Ballot status updates on schematic | 15-30 seconds (current: 15s poll) |
| Dashboard round progress | 15-30 seconds |
| Schematic publication visibility | Seconds (blast triggers push + email; page already has data) |
| Ballot save acknowledgment | < 2 seconds (synchronous POST) |
| Pairing generation | 5-60 seconds depending on field size (synchronous, blocks UI) |
| Results computation | 5-30 seconds (synchronous) |
| Blast email/push delivery | 1-5 minutes (queued via indexcards API) |

### 1.4 Rebuild Recommendations

- Replace 15-second polling with **WebSocket or SSE** connections for the
  schematic view and dashboard, fed by Redis Streams from the ballot-entry
  microservice.
- Retain push notifications (OneSignal or equivalent) for blast notifications.
- The 15-second interval is adequate for status dashboards; true sub-second
  latency is not required for any current operation.
- Pairing and results computation should be moved to **async worker tasks**
  with progress reporting via WebSocket/SSE, rather than blocking the HTTP
  request.

---

## 2. Performance Expectations

### 2.1 Database Scale (from AUTO_INCREMENT values in schema dump)

| Table | AUTO_INCREMENT | Interpretation |
|-------|---------------|----------------|
| `score` | 68,145,123 | ~68M score rows (largest table) |
| `ballot` | 33,620,535 | ~33.6M ballot rows |
| `score_value` (via `student_ballot`) | 37,345,562 | ~37M |
| `panel` | 29,313,586 | ~29M panel rows |
| `entry_setting` | 28,329,947 | ~28M EAV setting rows |
| `change_log` | 12,548,301 | ~12.5M audit log entries |
| `strike` | 7,062,894 | ~7M strike/conflict entries |
| `session` | 6,078,583 | ~6M historical sessions |
| `entry` | 6,264,704 | ~6.3M entry records |
| `entry_student` | 5,649,089 | ~5.6M |
| `round_setting` | 5,883,300 | ~5.9M |
| `campus_log` | 4,976,780 | ~5M campus interaction logs |
| `person` | 514,212 | ~514K user accounts |
| `student` | 1,269,584 | ~1.3M student records |
| `chapter` | 111,501 | ~111K schools/chapters |
| `tourn` | 82,038 | ~82K tournaments |
| `category` | 67,281 | ~67K categories |
| `event` | 51,486 | ~51K events |
| `autoqueue` | 1,133,792 | ~1.1M queued operations |

**Key observations:**
- The `score` table at 68M rows is the dominant table. Each ballot has
  multiple scores (points, ranks, winloss, comments, RFDs). A large
  tournament can generate hundreds of thousands of score rows.
- The `ballot` table at 33.6M is the second-largest transactional table.
- EAV tables (`*_setting`) collectively hold 40M+ rows. The rebuild should
  consider promoting frequently-queried settings to typed columns.
- `person` at 514K and `student` at 1.3M represent the full user population.

### 2.2 Concurrent Users During a Large Tournament

A large invitational tournament has 500-1500 entries across events, with:
- 200-500 judges entering ballots simultaneously in a round
- 20-50 tab staff operating on schematics, ballot entry, and results
- 500-2000 coaches, students, and spectators viewing postings/results

NSDA Nationals represents the extreme: 6,000+ entries, 3,000+ judges,
thousands of spectators. Concurrent users during a round can reach 5,000-10,000.

NSDA Districts represent the concurrency challenge: 100+ district tournaments
run simultaneously on the same weekend, each with 50-200 entries. System-wide
concurrent users can reach 10,000-20,000 during peak District weekends.

### 2.3 Simultaneous Tournament Volume

- Typical weekend: 20-80 tournaments running concurrently
- Peak weekends (Districts, bid tournaments): 100-200+ tournaments
- All tournaments share a single MariaDB instance in the current architecture

### 2.4 Heaviest Operations

| Operation | Computational Cost | Current Duration | I/O Pattern |
|-----------|--------------------|------------------|-------------|
| Powermatching (debate pairing) | O(n^2) penalty matrix + O(n^3) swap improvement, 100 iterations | 5-30s for 64-entry field | CPU-bound, DB reads at start |
| Judge assignment | O(judges x panels) score matrix, 30 random-start iterations with O(n^2) swap | 10-60s for large categories | CPU-bound, heavy DB reads |
| Speech paneling | O(n^2) section scoring + O(n^3) swap improvement, 7 passes | 3-15s | CPU-bound |
| Tiebreak/results computation | O(entries x rounds x tiebreak_types) | 5-30s for 500+ entry event | DB-read-heavy (all scores) |
| Blast email/push | Send to 500-3000 recipients | Minutes (queued) | I/O-bound (email API) |
| PDF generation (LaTeX) | pdflatex invocation x 2 (two passes for page numbers) | 5-30s per document | Disk I/O, CPU for LaTeX |
| Tournament data export/backup | JSON serialization of all tournament data | 10-60s | DB-read-heavy |

**Source:** `algorithmically_complex_areas.md`, `printout.mas`, `download_data.mhtml`

### 2.5 Rebuild Requirements

- **Database:** PostgreSQL must handle 68M+ row tables efficiently. Partition
  `score`, `ballot`, and `panel` tables by tournament or date range.
- **Compute-intensive operations** (pairing, judge assignment, tiebreaks):
  Run in dedicated worker containers with autoscaling. Do not share CPU with
  web-serving containers.
- **Target response times:** Page loads < 500ms, API calls < 200ms,
  compute operations < 60s with progress feedback.
- **Connection pooling:** Current system creates a new DB connection per
  request (Perl/DBI). PostgreSQL connection pooling (PgBouncer or equivalent)
  is essential.

---

## 3. Publish/Read Spike Behavior

### 3.1 Schematic Publication Spike

When a tabber publishes a round and sends a blast:

1. **Publish action** (`publish_switch.mhtml`): Sets `round.published = 1|2|3`
   in the database. Also triggers `docshare_rooms.mas` (room sharing),
   `online_usage.mas` (usage tracking), and `publish_flips.mas` (side flip
   notifications).

2. **Blast action** (`blast_pairing.mas`): Sends an AJAX POST to the indexcards
   API (`/tab/{tourn}/round/{round}/blast` or `/tab/{tourn}/timeslot/{ts}/blast`).
   The indexcards service then:
   - Sends email notifications to all affected participants (entries, judges,
     coaches)
   - Sends push notifications via OneSignal
   - Sends SMS/text notifications (carrier-based via email-to-SMS)

3. **Read spike**: Within 30-60 seconds of the blast, hundreds to thousands of
   users simultaneously load the schematic page. Each page load:
   - Executes the full `show.mhtml` template (server-side rendering)
   - Runs 5+ SQL queries to build the schematic view
   - Initiates AJAX polling (2 additional AJAX requests per 15-second cycle)

4. **Scheduled blasts** (`blast_schedule.mhtml`): Blasts can be scheduled for
   future delivery. An `autoqueue` record is created with `active_at` set to
   the desired time. A background worker processes these queued operations.

### 3.2 Results Publication Spike

When results are published:

1. **Results posting** (`publish_switch.mhtml`): Sets `round.post_primary`,
   `round.post_secondary`, `round.post_feedback` to visibility levels
   (0=hidden, 1=coaches, 2=coaches+competitors, 3=public).

2. **Per-panel result blasts** (`blast_results.mas`): When
   `judge_publish_results` is enabled, individual debate results are blasted
   to entry followers and school followers after each ballot is confirmed.
   This creates a rolling spike rather than a single burst.

3. **Mass publish** (`publish_everything.mhtml`): Publishes all rounds across
   multiple events with a single bulk UPDATE. Can affect dozens of rounds.

### 3.3 Current Handling

The current system has **no explicit spike handling**. The single MariaDB
instance and Apache/mod_perl processes handle all load. The indexcards Node.js
service provides some read offloading but is primarily an API layer, not a
cache.

### 3.4 Rebuild Requirements

- **CDN/edge caching** for public schematic and results pages. Published
  schematics are read-heavy, write-once. Cache invalidation on publish.
- **Queue-based blast processing**: Blasts must be processed asynchronously
  via Redis Streams. Do not block the publish action on blast delivery.
- **Read replicas**: Published data (schematics, results) should be served
  from read replicas or a Redis cache, not the primary database.
- **Rate limiting** on schematic page loads during spike windows.
- **Pre-render** schematic and results pages at publish time and serve from
  cache.

**Source files:**
- `/home/jeloni/tabroom/web/panel/publish/publish_switch.mhtml`
- `/home/jeloni/tabroom/web/panel/publish/publish_everything.mhtml`
- `/home/jeloni/tabroom/web/panel/schemat/blast_pairing.mas`
- `/home/jeloni/tabroom/web/funclib/blast_results.mas`
- `/home/jeloni/tabroom/web/funclib/blast_flips.mas`

---

## 4. Degraded-Network Considerations

### 4.1 Current State: No Offline Support

The current system has **no offline capabilities**:

- No service worker registration for Tabroom application logic (the only
  service worker is OneSignal's push notification worker:
  `OneSignalSDK.sw.js`).
- No client-side data caching or local storage of ballot state.
- No retry logic for failed form submissions.
- A `manifest.json` exists (`/index/manifest.mhtml`) indicating basic PWA
  metadata, but no offline cache strategy.

### 4.2 What Happens on Poor Connectivity

- **Ballot entry**: If a judge fills out a ballot form and submits while
  disconnected, the browser shows a connection error. The form data is lost
  unless the browser preserves it via back-button (browser-dependent). There
  is no auto-save, no draft state, and no local persistence.

- **Schematic viewing**: Pages fail to load. No cached version is available.

- **AJAX polling**: Silent failures. The jQuery AJAX calls have error handlers
  that log to console but do not alert the user or retry. Status displays
  freeze at the last successful poll.

- **Tab operations**: Form submissions (moving entries, changing rooms,
  entering scores) are standard HTML form POSTs that redirect on success.
  A failed POST results in a browser error page; partial data is not saved.

### 4.3 Venue WiFi Reality

Tournament venues (high school gymnasiums, college campuses, convention
centers) frequently have:
- Overloaded WiFi (500+ devices competing for bandwidth)
- Intermittent connectivity
- NAT/firewall issues blocking WebSocket connections
- Captive portals that interfere with authentication cookies

### 4.4 Rebuild Requirements

- **Offline ballot entry**: Implement a Progressive Web App (PWA) with
  service worker caching of ballot forms. Store ballot data in IndexedDB
  and sync when connectivity returns. This is the single highest-impact
  offline feature.
- **Auto-save ballot drafts**: Save ballot state to localStorage/IndexedDB
  on every field change. Restore on page reload.
- **Retry with exponential backoff**: All API calls should retry on network
  failure with exponential backoff and jitter.
- **Optimistic UI updates**: Show saved state locally before server
  confirmation, with conflict resolution on sync.
- **Connectivity indicator**: Display connection status to users with
  clear feedback when offline.
- **Graceful degradation for AJAX polling**: If WebSocket/SSE connections
  fail, fall back to polling. If polling fails, display stale-data warning
  rather than blank state.
- **SSE/WebSocket fallback**: Support HTTP long-polling as a fallback for
  environments that block WebSocket connections.

---

## 5. Ballot-Entry Reliability

### 5.1 Ballot Save Flow

The ballot save process (`ballot_save.mhtml`, ~1,800+ lines) is a single
synchronous HTTP POST that:

1. Validates the judge is authorized for the panel
2. Retrieves all unaudited ballots for this judge/panel
3. Processes scores from form parameters (points, ranks, winloss, comments,
   RFDs)
4. Creates `Score` records for each ballot/entry/score-type combination
5. Validates scores (low-point wins check, point range validation, rank
   uniqueness for speech)
6. On validation failure: deletes all scores just created and re-renders
   the ballot form with error messages
7. On success: redirects to `ballot_confirm.mhtml`

**There is no transaction wrapping**. Score records are created individually
via `Tab::Score->create()`, and on validation failure they are deleted via
individual `$score->delete()` calls. A crash between creation and deletion
would leave orphaned score records.

### 5.2 Ballot Confirmation (Double-Entry) Flow

`ballot_confirm.mhtml` handles ballot confirmation:

1. Sets `ballot.audit = 1` for all ballots in the judge/panel combination
2. Records `ballot.entered_by = person.id`
3. Calls `round_done.mas` to check if all ballots for the round are complete

**There is no explicit double-entry system** in the online ballot flow. The
"double-entry" concept exists only in the tab-room ballot entry flow
(`tabbing/entry/panel.mhtml` and `tabbing/entry/panel_save.mhtml`), where
tab staff manually enter paper ballots. In that flow:

- Tab staff enter scores on a per-panel screen
- Scores are saved directly to the `score` table
- The `audit` flag is set when scores are confirmed
- There is no comparison of independently entered copies; it is
  single-entry with manual review

### 5.3 Session Expiration During Ballot Entry

Session handling (`authenticate.mas`) checks the `TabroomToken` cookie
against the `session` table. If the session is invalid:

- The user is redirected to the login page with an error message
- **All unsaved ballot data is lost** -- there is no draft persistence
- Session cookies expire after 1024 hours (~42 days) per `login_save.mhtml`
  (`-expires => '+1024h'`)
- Sessions are limited: 3 per user (6 for admins, 10 for specific hardcoded
  user IDs). Creating a new session deletes the oldest.

The `updateLastAccess()` function (in `autohandler`) pings the indexcards
API to refresh the session every 2 hours, preventing server-side timeout
during active use.

### 5.4 Rebuild Requirements

- **Transaction wrapping**: All ballot save operations must be atomic.
  Use database transactions; commit only after all validations pass.
- **Draft persistence**: Auto-save ballot state to a draft table or
  Redis on every field change. Allow recovery after session loss.
- **Idempotent submission**: Assign a client-side submission ID to each
  ballot save to prevent double-submission.
- **True double-entry** (optional enhancement): For paper ballot entry
  by tab staff, implement independent dual entry with automated comparison
  and discrepancy flagging.
- **Session resilience**: Do not discard ballot data on session expiration.
  Allow re-authentication and resume.
- **Conflict detection**: If a ballot is modified by both judge and tab
  staff simultaneously, detect and resolve the conflict.

**Source files:**
- `/home/jeloni/tabroom/web/user/judge/ballot_save.mhtml` (~1,800 lines)
- `/home/jeloni/tabroom/web/user/judge/ballot_confirm.mhtml`
- `/home/jeloni/tabroom/web/user/judge/ballot.mhtml`
- `/home/jeloni/tabroom/web/user/login/authenticate.mas`
- `/home/jeloni/tabroom/web/user/login/login_save.mhtml`
- `/home/jeloni/tabroom/web/tabbing/entry/panel_save.mhtml`

---

## 6. Print/PDF Requirements

### 6.1 Current Approach: Server-Side LaTeX

The system generates PDFs via pdflatex (`printout.mas`):

1. A Mason component builds a `.tex` file by writing LaTeX commands to disk
2. pdflatex is invoked twice (for page numbers/cross-references)
3. The resulting PDF is served as a redirect to `/tmp/{filename}.pdf`
4. Temporary files are cleaned up on production

The LaTeX infrastructure includes custom fonts (Bera Sans/Mono, Electrum)
stored in `doc/tex/` with full TFM, VF, PFB, and map files.

### 6.2 What Generates PDFs

There are **141 files** that call `printout.mas`, generating PDFs for:

| Category | Examples | Approximate Count |
|----------|---------|-------------------|
| Schematics/Postings | Round postings, half-page postings, giant postings, bigass postings, list postings | 8+ |
| Ballots/Score sheets | Debate ballots, speech ballots, congress scoresheets, tab sheets | 10+ |
| Results/Awards | Sweep schools, sweep students, awards scripts, awards ceremony, qualifier lists | 12+ |
| Registration/Admin | Entry cards, judge cards, school registrations, contact sheets, head counts | 25+ |
| Invoices/Financial | Invoice prints (multiple variants: school, diocese, NSDA) | 8+ |
| Judge Reports | Judge pool prints, conflict sheets, judge charts, availability | 8+ |
| NSDA/NCFL Specific | District reports, NCFL entry/judge cards, diocesan sweeps, ribbons | 15+ |
| School Reports | Dance cards, full registration, assignment sheets | 6+ |
| Audit/Status | Audit prints, pending prints, code prints | 5+ |

### 6.3 Volume

A large tournament generates:
- 200-500 ballots per round (one per judge per panel)
- 3-8 rounds per event, 5-30 events per tournament
- Total: 3,000-120,000 ballot PDFs per tournament (usually printed in batch)
- Schematic postings: 10-50 per tournament
- Invoices: 50-200 per tournament

### 6.4 LaTeX Capabilities Used

From `printout.mas`:
- Paper size: letter (default) or A4
- Orientation: portrait or landscape, with "wider" variant
- Margins: configurable horizontal (0.45-0.75in) and vertical (0.45-0.6in)
- Font sizes: `\small` through `\scriptsize`
- Features: multirow tables, color/shading (`xcolor`, `colortbl`), strikethrough
  (`ulem`), watermarks (`draftwatermark`), truncation, page numbers, headers/footers
- Packages: fullpage, minibox, setspace, multirow, graphicx, fancyhdr, changepage,
  selinput, T1 fontenc

### 6.5 Rebuild Requirements

- **Replace LaTeX** with a browser-based or server-side PDF engine (e.g.,
  Puppeteer/Playwright rendering HTML to PDF, or a library like WeasyPrint).
  LaTeX is a heavy dependency that requires a full TeX distribution on the
  server.
- **Batch PDF generation**: Support generating hundreds of ballots in a single
  request as a merged PDF. Current system does this via LaTeX `\newpage`.
- **Print stylesheets**: For simple outputs (schematics, results tables),
  HTML with CSS `@media print` may be sufficient, avoiding PDF generation
  entirely.
- **Async generation**: Large PDF jobs (full tournament ballot sets) should
  generate asynchronously with download link notification.
- **Template system**: Create a PDF template DSL or component system that
  replaces the 141 individual LaTeX-generating files.
- **A4 support**: International users need A4 paper size support (currently
  configurable per tournament via `papersize` setting).

**Source files:**
- `/home/jeloni/tabroom/web/funclib/printout.mas` (274 lines -- LaTeX generation engine)
- `/home/jeloni/tabroom/web/funclib/print_format.mas`
- `/home/jeloni/tabroom/doc/tex/` (font files: Bera, Electrum)

---

## 7. Auditability and Recoverability

### 7.1 Change Log

The `change_log` table (12.5M+ rows) records operational changes:

```
change_log:
  id, tag, description, person, event, category, tourn, circuit,
  entry, judge, fine, new_panel, old_panel, school, round,
  timestamp, deleted, created_at
```

The `log.mas` component creates `ChangeLog` entries on:
- Tabbing operations (pairing, publishing, blast scheduling)
- Entry/judge moves between panels
- Score changes
- Publication status changes
- Access control violations

Each log entry captures who (`person`), what (`tag` + `description`),
and context (`tourn`, `event`, `round`, `panel`, `entry`, `judge`).

### 7.2 Campus Log

The `campus_log` table (5M+ rows) tracks interactions in the "campus"
system (NSDA's online tournament platform):
- UUID-based event tracking
- Person, panel, school, judge, entry, student references
- Tag-based categorization

### 7.3 Tournament Backup and Restore

**Backup mechanism** (`auto_backups.mas` + `download_data.mhtml`):

- When the last ballot in a round is entered, the system automatically
  triggers a backup of that round's data
- Backup data is serialized as JSON via `download_data.mhtml`
- The JSON file is emailed as an attachment to configured "backup followers"
  (per-event and per-tournament settings)
- Subject line format: `TRBKP: {event} {round} Tabroom Backup`
- Tab staff can also manually export tournament data (whole tournament,
  single event, single round, or single school)

**Restore mechanism** (`upload_data.mhtml`, 4,543+ lines):

- Tab staff upload a previously exported JSON file
- The system recreates panels, ballots, scores, and entries from the backup
- Only users with tournament-wide tabber access can restore
- Restore is available per-round via the schematic settings UI

### 7.4 What Is NOT Audited

- Individual score value changes (the change_log records "ballot changed"
  but does not store before/after values)
- Setting changes (EAV table modifications)
- Registration changes (entry adds, drops, waitlist moves)
- User profile changes
- Session creation/destruction (beyond login logging)

### 7.5 Rebuild Requirements

- **Structured audit log**: Every state-changing operation should create an
  audit record with before/after values, not just descriptions.
- **Event sourcing** (recommended for ballot data): Store all score changes
  as immutable events, enabling full reconstruction of ballot state at any
  point in time.
- **Automated backups**: Replace email-based backup delivery with cloud
  storage (Azure Blob Storage). Retain point-in-time backups per round.
- **Backup verification**: Add integrity checks (checksums) on backup data.
- **Self-service restore**: Allow tournament directors to restore from backup
  without site admin intervention.
- **Retention policy**: Define retention periods for audit logs and backups
  (current: indefinite, growing at 12M+ change_log rows).

**Source files:**
- `/home/jeloni/tabroom/web/funclib/log.mas` (47 lines)
- `/home/jeloni/tabroom/web/funclib/auto_backups.mas` (130 lines)
- `/home/jeloni/tabroom/web/api/download_data.mhtml`
- `/home/jeloni/tabroom/web/api/upload_data.mhtml`
- `/home/jeloni/tabroom/doc/sql/current-schema.sql` (change_log table, lines 204-236)

---

## 8. Scalability

### 8.1 Current Architecture Limitations

| Component | Limitation |
|-----------|-----------|
| **MariaDB (single instance)** | All 82K+ tournaments, 514K users, 68M scores on one DB server. No read replicas, no sharding. Every page load hits the primary. |
| **Perl/mod_perl on Apache** | Process-per-request model. Each Apache worker holds a persistent DB connection and loaded Perl modules (~50MB per process). Horizontal scaling requires identical server images. |
| **No caching layer** | No Redis/Memcached for frequently-read data. Tournament settings, event configurations, and user permissions are queried from DB on every request (via `all_settings()` calls). |
| **LaTeX on every server** | PDF generation requires a full TeX Live installation on each application server. |
| **Indexcards (Node.js)** | Single Node.js service handles all API requests. Not horizontally scaled in the current architecture (though it could be). |
| **File system coupling** | PDF generation writes to local disk (`$Tab::file_root/tmp/`). Multi-server deployments need shared storage or a different PDF strategy. |
| **Monolithic codebase** | All functionality in one Apache/Mason application. Cannot scale ballot entry independently from pairing generation or registration. |
| **EAV pattern overhead** | 19 `*_setting` tables require extra JOINs or separate queries for every entity. Aggregate queries across settings are expensive. |

### 8.2 Expected Growth

- User base growing: 514K accounts, likely doubling over 5 years with
  increased NSDA adoption
- International expansion (WSDC, WUDC) adds timezone and localization needs
- Online/hybrid tournaments (post-COVID) remain a significant use case
- Tournament count growing ~10% annually based on `tourn` AUTO_INCREMENT

### 8.3 Multi-Tournament Concurrency

The rebuild must handle 100-200+ concurrent tournaments without:
- Cross-tournament performance interference (noisy neighbor problem)
- Shared connection pool exhaustion
- Compute-intensive operations in one tournament (pairing, tiebreaks)
  degrading ballot entry in another

### 8.4 Rebuild Requirements

- **Database per-tournament or schema isolation** is not practical given
  cross-tournament features (circuits, person accounts, NSDA integration).
  Instead, use **row-level tenant isolation** with tournament_id indexing
  and connection pooling.
- **Horizontal scaling**: Stateless API containers that autoscale based on
  load. Azure Container Apps with KEDA scaling.
- **Read replicas**: PostgreSQL streaming replicas for read-heavy queries
  (schematic views, results, public postings).
- **Compute isolation**: Pairing and tiebreak workers in separate container
  pools from web-serving containers.
- **Caching**: Redis for tournament settings, permissions, session data.
  Cache invalidation on write.
- **Object storage**: Azure Blob Storage for PDFs, backups, and uploaded
  files (currently local filesystem).

---

## 9. Availability

### 9.1 Current Uptime Expectations

Tabroom is a **mission-critical platform during tournament weekends**.
Tournament operations cannot be paused or rescheduled. When the platform is
down during a tournament, all competitive activity stops.

- **Implicit SLA**: 99.9%+ availability during tournament hours (Friday
  evening through Sunday evening, US time zones)
- **Maintenance windows**: No formal maintenance window policy visible in
  code. The staging/dev hostnames (`staging.tabroom.com`,
  `mason.staging.tabroom.com`, `mason.dev.tabroom.com`, `local.tabroom.com`)
  suggest a staging environment exists.
- **Admin site**: `admin.tabroom.com` exists as a separate admin-only
  interface.

### 9.2 Backup Site

A backup site exists at `backup.tabroom.com`:

```
# From web/autohandler, line 1009:
if ($r->hostname eq "backup.tabroom.com") {
    <div class="backupwarning">
        This is a backup copy of Tabroom. Please use it only to access
        data and information. Any registration or tournament changes
        entered here will not be copied back to the master Tabroom site.
    </div>
}
```

This is a **read-only failover** -- changes made on the backup site do not
sync back. The backup site serves the same codebase but presumably points
to a database replica or snapshot.

### 9.3 Failure Modes Visible in Code

- **Database connection failure**: `$dbh->disconnect()` is called at the
  end of every request (`autohandler` line 1130). No connection retry logic.
- **Session store failure**: Session lookup failure results in redirect to
  login page with error message. No graceful degradation.
- **Indexcards API failure**: AJAX calls to indexcards have error handlers
  that log to console. Schematic status display freezes at last known state.

### 9.4 Rebuild Requirements

- **Target SLA**: 99.95% availability (< 22 minutes downtime per month),
  with 99.99% during peak tournament weekends.
- **Zero-downtime deployments**: Blue-green or canary deployments via Azure
  Container Apps revision management.
- **Health checks**: Liveness and readiness probes for each microservice.
- **Circuit breakers**: Between microservices (e.g., if email blast service
  is down, do not fail ballot entry).
- **Disaster recovery**: Automated failover to a secondary Azure region.
  RPO (Recovery Point Objective) < 1 minute for transactional data.
  RTO (Recovery Time Objective) < 15 minutes.
- **Graceful degradation**: Core operations (ballot entry, schematic viewing)
  must function even if non-critical services (email, PDF generation,
  analytics) are unavailable.

---

## 10. Security

### 10.1 Current Authentication

**Cookie-based session authentication** (`authenticate.mas`):

1. Login (`login_save.mhtml`):
   - Username (email) and password submitted via HTML form
   - Password verified via `crypt()` comparison against stored SHA-512 hash
   - Session created in `session` table with IP address
   - Session key: `crypt(session_id + $Tab::string, '$6$' + random_salt)`
   - Cookie set: `TabroomToken` with 1024-hour expiration, httponly, secure
     (when HTTPS), domain-scoped

2. Authentication check (`authenticate.mas`):
   - Read `TabroomToken` cookie
   - Look up session by `userkey`
   - Verify: `crypt(session.id + $Tab::string, userkey) == userkey`
   - Return person + session objects

3. CSRF protection (`login_save.mhtml`):
   - Login form includes a SHA-512 hash of `IP + day_of_year + hour + $Tab::string`
   - Verified on submission. If mismatch, tries previous hour (clock skew tolerance)
   - Failed CSRF check causes 5-second sleep (basic rate limiting)

4. Session limits:
   - 3 concurrent sessions per user (6 for admins, 10 for specific users)
   - Oldest sessions deleted when limit exceeded
   - No session revocation on password change

### 10.2 Authorization

**Permission-based access control** (`autohandler`, `tourn_checks.mas`):

- `person.site_admin`: Global admin flag (boolean on person table)
- `permission` table: Per-tournament roles (owner, tabber, chapter access)
  with optional event/category scoping
- Checked on every request to `/setup/`, `/register/`, `/panel/`, `/tabbing/`
- Session SU (switch-user): Admins can impersonate other users via
  `session.su` field

### 10.3 Data Sensitivity

| Data Type | Table(s) | Sensitivity |
|-----------|----------|------------|
| Student PII | `person` (email, name, address, phone, gender), `student` (name, gender, grad_year) | **HIGH** -- minors' data, COPPA/FERPA considerations |
| Password hashes | `person.password` (SHA-512 with salt) | **CRITICAL** |
| Financial data | Payment processing via PayPal (`paypal.mas`) and Authorize.net (`authorizenet.mas`) | **HIGH** -- PCI DSS scope |
| Session tokens | `session.userkey` | **HIGH** |
| Login history | `person_setting` (last_login_ip, last_attempt_ip, last_attempt_agent) | **MEDIUM** |
| Competitive results | `score`, `ballot` | **MEDIUM** -- competitive integrity |
| School addresses | `chapter` (street, city, state, zip) | **LOW** |

### 10.4 Security Concerns Visible in Code

1. **XSS handling** (`autohandler` lines 488-495): Input sanitization for
   display messages strips non-alphanumeric characters. Comment in code:
   "People apparently care about XSS now from Tabroom which is baffling but
   maybe a sign I've arrived in the Big Time?" This suggests XSS protection
   was added reactively, not systematically.

2. **SQL injection exposure**: Many Mason components build SQL queries via
   string interpolation of user input. Example: `blast_flips.mas` uses
   `$limit = " and panel.id = $panel_id "` after `int()` sanitization. The
   `int()` conversion provides some protection, but the pattern is fragile.

3. **Rate limiting**: Only mechanism is `sleep 5` on failed login attempts.
   No IP-based rate limiting, no account lockout, no CAPTCHA.

4. **Hardcoded user IDs**: Multiple places reference specific user IDs
   (e.g., `person.id == 7270` for Easter egg, `person.id == 1` for debug
   logging, specific IDs for elevated session limits). These would be
   vulnerabilities if IDs are predictable.

5. **Session key derivation**: Uses `crypt()` with the server-side secret
   `$Tab::string`. If this string is compromised, all sessions can be forged.

6. **No HTTPS enforcement in code**: HTTPS is determined by `$Tab::url_prefix`
   configuration, not enforced at the application level.

7. **Cookie scope**: Cookie is set on `$Tab::cookie_domain` which allows
   sharing across subdomains (needed for backup.tabroom.com integration but
   expands attack surface).

### 10.5 Rebuild Requirements

- **Authentication**: Replace custom session management with industry-standard
  auth (OAuth 2.0 / OpenID Connect). Consider Azure AD B2C or Auth0.
- **Password handling**: Use bcrypt or Argon2 instead of SHA-512 crypt.
  Implement password complexity requirements and breach detection.
- **CSRF**: Use framework-provided CSRF tokens (FastAPI middleware) on all
  state-changing endpoints.
- **Input validation**: Parameterized queries everywhere (SQLAlchemy ORM).
  No string interpolation of user input into SQL.
- **Rate limiting**: Per-IP and per-account rate limiting on login, ballot
  submission, and API endpoints.
- **Data encryption**: Encrypt PII at rest (PostgreSQL column-level
  encryption or Azure Database encryption). Encrypt all data in transit
  (TLS 1.3).
- **COPPA/FERPA compliance**: Implement parental consent workflows for
  students under 13. Data minimization for student records.
- **PCI DSS**: Payment processing should use tokenized payment providers
  (Stripe Elements, PayPal hosted fields). Never store card numbers.
- **Audit logging**: Log all authentication events, permission changes,
  and data access to student PII.
- **Session management**: Implement session revocation on password change,
  configurable session timeout, and maximum concurrent session limits.
- **Secrets management**: Use Azure Key Vault for all secrets. No hardcoded
  secrets in code or configuration files.

**Source files:**
- `/home/jeloni/tabroom/web/user/login/authenticate.mas` (117 lines)
- `/home/jeloni/tabroom/web/user/login/login_save.mhtml` (209 lines)
- `/home/jeloni/tabroom/web/autohandler` (1,130 lines)
- `/home/jeloni/tabroom/web/funclib/session_agent.mas`
- `/home/jeloni/tabroom/web/funclib/session_location.mas`

---

## Summary: Priority Matrix

| NFR Area | Current State | Risk Level | Rebuild Priority |
|----------|---------------|------------|------------------|
| Security (auth, PII, injection) | Basic/fragile | CRITICAL | P0 -- must be addressed before launch |
| Ballot-entry reliability | No transactions, no drafts, no offline | CRITICAL | P0 -- data integrity is paramount |
| Availability | Single point of failure, no formal SLA | HIGH | P0 -- platform must not go down during tournaments |
| Performance (DB scale, compute) | Single DB, synchronous compute | HIGH | P1 -- design for from day one |
| Publish/read spike handling | No caching, no spike handling | HIGH | P1 -- affects user experience at critical moments |
| Degraded-network support | None | HIGH | P1 -- affects every tournament venue |
| Real-time requirements | 15s polling works | MEDIUM | P1 -- improve with WebSocket/SSE but polling is acceptable fallback |
| Scalability | Monolith, single DB | MEDIUM | P1 -- microservice architecture addresses this by design |
| Print/PDF generation | LaTeX, heavy but functional | MEDIUM | P2 -- functional alternative needed but not blocking |
| Auditability | Basic change_log, email backups | MEDIUM | P2 -- improve incrementally |
