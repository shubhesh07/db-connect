# Changelog

All notable changes to DB Connect will be documented in this file.

## [3.0.0] - 2026-08-23

### Added — Redis (new engine)
- Connect with just host:port — standalone vs. cluster is detected automatically and the remaining cluster nodes are discovered; Sentinel profiles supported. TLS, ACL user/password, DB index, SSH tunnel (standalone).
- Visual key browser: glob/prefix filter, type filter, keys grouped into a collapsible `:` tree with per-folder counts, `Results · Scanned X / Total` progress, Load more and Scan all across every cluster master, and a main-pane results table for prefix/glob lookups.
- Type-aware key editor — string (Format JSON), hash, list, set, sorted set, stream (read-only); rename (works across cluster slots via DUMP/RESTORE), TTL edit, delete with confirm; read-only connections block edits.
- Analyze pane: Overview (version, uptime, clients, memory, keys, ops/s, hit rate, per-primary-node table, memory/keys share per node), Database Analysis (sampled memory-by-TTL histogram with extrapolate / no-expiry toggles, top namespaces by memory or keys with data types), Slow Log across all masters.
- Raw command console with history recall; Redis MCP tools (`redis_scan_keys`, `redis_get_key`, `redis_server_info`, `redis_run_command` — read-only unless a write grant is active).

### Added — Redshift
- SSH tunnel and IAM authentication (provisioned `GetClusterCredentials` or serverless `GetCredentials` via the AWS profile/SSO/default chain).
- Dialect-aware autocomplete: Redshift/PostgreSQL get `DISTKEY`/`SORTKEY`/`ENCODE`, `UNLOAD`/`COPY`/`VACUUM`, `LISTAGG`, `APPROXIMATE COUNT(DISTINCT)`, `DATE_TRUNC`/`DATEADD`, `NVL`, `DECODE`, `CONVERT_TIMEZONE`, `JSON_EXTRACT_PATH_TEXT`, `REGEXP_*`; MySQL-only suggestions no longer appear.
- Structured EXPLAIN — rows, cost and distribution strategy parsed; `DS_BCAST_INNER`, `DS_DIST_BOTH`, nested loops and ≥1M-row sequential scans flagged as expensive.
- Queue vs execution time (WLM) in slow queries; Generate UNLOAD… / COPY… statement builders on any table; table-health badges (unsorted % / skew) in the tree; Spectrum external schemas browsable.

### Added — DynamoDB
- Per-query cost visibility: every Scan/Query/PartiQL shows `Items returned · Items scanned · Efficiency · RCUs consumed`, with a warning when DynamoDB stopped at its 1 MB page.
- Load more pagination, strongly-consistent read toggle, and IAM Role / default credential chain auth in the connection form.

### Added — App
- Connections grouped by database engine in the sidebar (MySQL / PostgreSQL / DynamoDB / Redshift / Redis), with a one-click switch back to custom groups.
- Design refresh: surface/semantic tokens, native macOS title strip, new Modal/ContextMenu/ConfirmDialog primitives, regrouped editor toolbar, running overlay with elapsed time and Cancel.
- Typing no longer re-renders the whole app; memoized grid rows; async history recording; concurrent schema loads; Monaco bundled locally (no CDN).

### Fixed
- **Data safety:** inline edit/delete in the results grid is blocked when no primary key is known — the previous all-columns fallback combined with MySQL's `LIMIT 1` could silently modify the wrong of two look-alike rows.
- Credential encryption at rest now covers PostgreSQL and Redis passwords and all SSH secrets; the table editor runs inside the open transaction and batch apply is all-or-nothing; the server-side production guard is enforced on every execute path including MCP.
- Redshift: schema selection pins `search_path` and the query to the same connection (previously they could run on different pooled connections and unqualified names silently resolved against `public`); Export-as-INSERT used MySQL syntax for Redshift/PostgreSQL; `TIMESTAMPTZ` lost its offset; `BEGIN/COMMIT/ROLLBACK` silently auto-committed — now reported.
- DynamoDB: Query and Get Item failed on every table with a numeric partition/sort key; removing an attribute in the item editor reported success but left it in place.
- Connecting to a slow or unreachable server no longer freezes queries and health checks on every other open connection.
- The health check skipped Redis, so auto-reconnect recycled the client mid-scan.
- MCP server: enabling it could show "Not running" forever with no explanation — bind/token failures (e.g. port 47823 already in use) are now surfaced; Redis connections appear in `list_connections`.

## [2.4.0] - 2026-08-22

### Added — Full PostgreSQL support
- PostgreSQL joins MySQL, Amazon Redshift, and DynamoDB as a fully-supported engine: transactions (BEGIN/COMMIT/ROLLBACK + auto-commit), an index manager, table relations (outgoing + incoming foreign keys), a full Alter Table designer, CSV/SQL file import, connection-string paste-to-parse, and EXPLAIN ANALYZE.
- PostgreSQL-native features with no MySQL equivalent: an extensions viewer/installer, custom ENUM types (creatable and selectable directly in the table designer), materialized views (create/refresh/drop), array column types (`TYPE[]`), sequences (list/create/restart/drop, with SERIAL-owned sequence detection), table bloat stats (dead-tuple % and last-vacuum time) with one-click VACUUM/ANALYZE, and read-only visibility into Row-Level Security policies and declarative table partitioning.

### Fixed
- The query editor's autocomplete only ever called MySQL's schema-loading functions regardless of a tab's actual engine, so PostgreSQL and Redshift tabs never fetched column/table suggestions through that path — and no engine reopened the suggestion list once a lazy load finished, so the first reference to any not-yet-loaded table showed an empty dropdown. Suggestions now load per-engine and reappear automatically.
- PostgreSQL and Redshift query plans were run through MySQL's severity heuristics and always flagged as a false "full table scan," regardless of the actual plan. Text-based plans now render correctly.

## [2.3.0] - 2026-08-18

### Security — MCP server hardening
- Per-install bearer token + Host/Origin request validation, closing a DNS-rebinding / malicious-webpage localhost-probing gap in the local MCP server.
- Connection allow-list replaces "every open connection is reachable via MCP" — MCP tools now only ever see connections you've explicitly allowed.
- Per-connection, time-bounded write grants replace the global write-enable toggle; grant/revoke are desktop-app-only, never exposed as MCP tools.
- Fixed a SQL-injection-style bypass in `run_query`'s Redshift schema parameter, a read-only classifier gap around `SELECT ... INTO` disguised writes, and MCP tool responses that returned a bare JSON array (rejected by strict MCP clients).

### Added — DynamoDB PartiQL query mode
- New PartiQL tab in the DynamoDB explorer — write real SQL-like statements (`SELECT * FROM "table" WHERE ...`) instead of building filters through the GUI.
- Context-aware autocomplete: keywords, table names (only right after FROM/INTO), and attribute names learned per-table from key schema and prior query results.
- `Cmd/Ctrl+Enter` runs the statement; destructive PartiQL on production connections goes through the same confirmation dialog as SQL writes; truncated results now show a warning instead of silently looking complete.

## [2.2.0] - 2026-08-14

### Fixed — DynamoDB
- Results grid no longer guesses which table produced a scan/query by scanning every table in the account — was slow and could lock onto the wrong table when key names collided. It now reads which table was actually queried directly from the backend response.
- "Get Item" mode now actually gets the item instead of silently running a scan/query
- Table-info load failures now show an error instead of failing silently

### Changed — Smarter SQL Autocomplete
- `WHERE col=val ` now suggests `AND`/`OR` + the next column instead of re-offering comparison operators on an already-complete condition

### Added — Explorer & Editor
- Tables/views grouped into collapsible Tables (N) / Views (N) folders with counts shown up front
- Drop Table.../Drop View... in the table context menu — still goes through the existing production/read-only safety confirmation on Run
- In-toolbar data-source switcher — rebind a query tab to a different connection without the sidebar

### Fixed — Performance
- The sidebar's expanded table tree was fully re-rendering on every keystroke while typing a query — this was the main cause of reported typing lag
- Dropping/creating/altering a table or view now refreshes the sidebar automatically; disconnecting a connection clears its cached schema so reconnecting shows current state

### Added — Local MCP Server
- Now works with Redshift connections too (previously MySQL-only)
- New DynamoDB tools: list tables, describe table, scan, query, get item, plus write tools when "Allow writes" is enabled

## [2.1.0] - 2026-08-14

### Added — MySQL Productivity
- Stored procedure/function/trigger/event visibility (list + view `SHOW CREATE` definitions)
- Real database backup/restore via `mysqldump`/`mysql` CLI (distinct from app-config backup)
- Full MySQL user management: create/alter/drop user, grant/revoke privileges
- `SHOW VARIABLES` viewer in the Performance pane
- Connection-string paste-to-parse (`mysql://user:pass@host:port/db`)
- Auto-update check against GitHub releases
- Table Relations view (outgoing + incoming foreign keys)
- Saved/reusable result-grid filters
- CSV import field-mapper (remap/skip columns, insert-vs-upsert)

### Added — Local MCP Server
- Lets local MCP clients (Claude Code, Cursor, etc.) query your open connections through DB Connect
- Off by default, localhost-only, read-only by default — toggle under Sidebar → "MCP Server"

### Changed — Connection UI
- Permanent connection rail replaces the connection dropdown; `⌘P` fuzzy switcher
- Brand-stripe engine picker and redesigned three-section connection form

### Fixed
- Query-timeout setting no longer gets silently overridden by a hardcoded socket timeout, which previously surfaced as "invalid connection" well before the configured timeout
- Several error toasts that showed "undefined" now show the real error message
- Smoother results-grid scrolling and paste handling for heavy queries
- Closed an identifier-injection gap in the Create Table flow

## [2.0.0] - 2026-05-23

### Added — Performance Monitoring
- **MySQL Performance pane** with four tabs: Slow Queries, Live Activity, Locks (Blocking Tree / Locks / Wait Time), Index Health (Unused / Redundant / Table-Scan Hot Spots)
- **One-click Kill Query** for live sessions (uses `KILL QUERY` via `CONNECTION_ADMIN`)
- **"Where this query is fired from"** drill-down — per slow-query digest, expand to see top `user@host` aggregations and recent raw samples with full `SQL_TEXT`
- **Suggestion-to-draft** — copy `DROP INDEX` statements straight from Index Health into a new tab
- **Redshift Performance pane** — slow queries, live activity, lock waits via `STV_*` / `STL_*` system views
- **DynamoDB Performance pane (native)** — read/write capacity, throttle counts, hot-partition signals, GSI status. No CloudWatch dependency.

### Added — Smarter SQL Autocomplete
- Step-aware suggestions for statement-start, after `SELECT`, after `FROM`/`JOIN`, in `WHERE`, after a column reference (operators), and in `GROUP BY`/`ORDER BY`
- FROM-clause scoping — `WHERE order_` now suggests columns of `order_details` (the table in scope) instead of unrelated tables containing "order_"
- Cross-database completions — `db.` lists tables; `db.table.` lists columns
- MySQL keyword and function catalogues with prefix-match boost (`SELECT`, `UPDATE`, `COUNT(...)`, `NOW()`, etc.)
- After-column operator snippets — `WHERE col ` suggests `= != > < LIKE NOT LIKE IN BETWEEN IS NULL ...`
- Same-server cross-database querying — pick a target DB per tab without losing autocomplete

### Added — Production Safety
- Tiered destructive-op confirmation modal (`DROP`/`TRUNCATE` require typing the connection name; `DELETE`/`UPDATE`/`ALTER` show a one-click confirm with statement preview) replacing the easy-to-dismiss `confirm()` prompt
- Server-side query cancel — Stop now issues `KILL QUERY <thread_id>` on MySQL and `PG_CANCEL_BACKEND($1)` on Redshift
- Local audit trail (SQLite-backed) for destructive actions and kill events
- Configurable persistent query timeout with clear error messages

### Added — Cross-Schema Queries & Backup
- **Cross-Schema Queries** — JOIN and query across multiple schemas in a single connection. Autocomplete resolves tables across schemas automatically (MySQL and Redshift)
- **Backup &amp; Restore** — One-click logical backups of tables, schemas, or full databases, saved locally to your laptop. Restore into any connection without leaving the app (MySQL, Redshift, DynamoDB)

### Added — Editor & UI
- Monaco editor theme follows the app theme (light/dark/system) with runtime switching
- Sortable columns in every Performance pane
- Expandable cells for long SQL / lock data
- Row-count footer plus per-panel client-side filter
- SQL `VIEW` create and inspect for MySQL and Redshift

### Changed
- Results-grid filter now works correctly for queries with no derivable source table
- Performance-pane filter matches rendered cell values, not just raw row data
- Connection-layer security hardening for credential handling and DB clients

### Install
- **Homebrew Cask** (recommended): `brew install --cask shubhesh07/db-connect/db-connect` — Homebrew unblocks Gatekeeper automatically; upgrade with `brew upgrade --cask db-connect`. Tap: [shubhesh07/homebrew-db-connect](https://github.com/shubhesh07/homebrew-db-connect)
- macOS one-line installer: `curl -fsSL https://github.com/shubhesh07/db-connect/releases/latest/download/install.sh | bash` (no Gatekeeper prompt — curl downloads don't get quarantined)
- DMG still bundles `install-mac.sh` for users who prefer the GUI path

## [1.0.0] - 2026-05-03

### Added
- **MySQL Support** — Full SQL execution, transactions (BEGIN/COMMIT/ROLLBACK), auto-commit toggle, SSH tunnels, SSL modes
- **Amazon Redshift Support** — SQL queries via PostgreSQL wire protocol, schema browser, EXPLAIN plans (text format)
- **DynamoDB Support** — Visual Scan/Query/GetItem builder, filter expression builder, item create/edit/delete, GSI-aware querying, multiple auth modes (Access Key, SSO Profile, IAM Role)
- **Monaco SQL Editor** — Syntax highlighting, autocomplete (tables, columns, keywords), multi-cursor, statement selection (Cmd+D), SQL formatting
- **Multi-Tab / Multi-Connection** — Work across multiple databases simultaneously with connection badges per tab
- **Schema Browser** — MySQL (Databases > Tables > Columns/Indexes/FKs), Redshift (Schemas > Tables > Columns), DynamoDB (Tables > Items)
- **EXPLAIN Visualization** — Color-coded query plans with severity indicators (Efficient/Moderate/Slow), warning badges for full table scans and missing indexes
- **Query History** — Auto-saved with search, favorites, date grouping (Today/Yesterday)
- **Saved Queries** — Named queries with `{{param}}` parameter support and tags
- **Table Designer** — Visual CREATE TABLE (columns, types, PK, engine, charset), ALTER TABLE (change tracking, DDL preview), Index Manager (create/drop with type badges)
- **Inline Row Editing** — Batch Apply/Revert with PK-based WHERE clauses, Insert/Delete rows from results grid
- **Command Palette** — Cmd+K for quick access to all actions
- **Export** — CSV, JSON, Excel (XLSX) via native save dialog
- **Built-in SQL Snippets** — Admin (Processlist, Table Status, Variables), Info (Table Size, Index Usage, Database Size), Performance (Long Running Queries, Table Locks, InnoDB Status)
- **Security** — AES-256-GCM encrypted credentials, OS keychain integration (macOS Keychain / Windows Credential Manager), production safety warnings before destructive queries
- **Virtual Scrolling** — Handle large result sets without UI lag
- **Dark Theme** — Default dark UI
- **Keyboard Shortcuts** — Cmd+Enter (run), Cmd+D (select statement), Cmd+K (palette), Cmd+T (new tab), Cmd+W (close tab), Cmd+S (save), Cmd+Shift+F (format), Cmd+E (explain)
