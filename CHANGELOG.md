# Changelog

All notable changes to DB Connect will be documented in this file.

## [3.7.0] - 2026-10-18

v3.7.0 adds **Elasticsearch and OpenSearch** as the seventh engine, fixes a long list of bugs found in a full review of every engine (several of them safety gates), and adds the debugging tools and snippets engineers reach for on each database.

#### Elasticsearch and OpenSearch
✨ **Seventh engine.** Connect to Elasticsearch or OpenSearch (self-managed, Elastic Cloud, AWS OpenSearch Service and AWS Elasticsearch domains) with username and password or an API key, TLS with a custom CA, and SSH tunnels. The distribution and version are detected on connect; AWS domains get a port and TLS hint if misconfigured.
✨ **Indices in the explorer**: indices with aliases, doc counts and health, expanding to mapping fields (nested objects, multi-fields). Right-click for Query (SQL), Search (console), Mapping, Settings, Stats, Count, Refresh, Reindex, Add alias, New index, Delete index, New document and Bulk insert; cluster-level Health, Nodes, Shards and Aliases.
✨ **SQL and console in one editor.** Write `SELECT … FROM index` (Elastic `_sql`, OpenSearch `_plugins/_sql`, or the Open Distro endpoint on AWS Elasticsearch domains, chosen automatically) or Kibana-style requests such as `GET /index/_search` with a JSON body, several per run. Console text gets its own syntax colouring, endpoint and query-DSL autocomplete with your mapped field names, and Format.
✨ **Results you can work with**: hits become rows with `_index`, `_id`, `_score` and the source fields; aggregations arrive as a second result set; `_cat` output becomes a table. Edit cells, delete rows and insert documents straight from the grid (one `_update` per document), or open the full document editor. Load more pages through the SQL cursor or `from`/`size`, and falls back cleanly on clusters that refuse paging. Export streams through the cursor or scroll API.
✨ **Profile and Explain** buttons show per-shard query, collector and aggregation timings as a tree, or why each hit scored.
✨ **Performance pane**: cluster health, search and indexing rates and latencies per node and index, heap, CPU and rejections, plus Tasks (with cancel), Shards (with allocation explain), Hot threads, Pending tasks and circuit breakers.
🛡️ Read-only profiles and production confirmation apply to every write, including `DELETE`, `PUT`, `_update_by_query`, `_delete_by_query`, task cancel and document edits.
✨ **MCP**: `list_tables`, `describe_table`, `run_query` (SQL or console), `explain_query`, `active_sessions` and `cluster_health` work on Elasticsearch connections; writes still need a grant.

#### Debugging tools for every engine
✨ **Snippets per engine**: PostgreSQL (30), Redshift (27), MongoDB (30), Redis (25), DynamoDB PartiQL (19), Elasticsearch (31) and a consolidated MySQL set, covering locks, blocking, slow queries, index usage, bloat, replication lag, WLM queues, hot keys, cluster health, unassigned shards and more. Snippets are filtered by engine and insert at the cursor instead of replacing your tab.
✨ **Redis**: Clients (with kill), Latency and Commands tabs in the performance pane; `CONFIG GET` no longer prompts.
✨ **MongoDB**: slow query shapes from the profiler with "where fired from", index usage via `$indexStats` with an unused-index hint and a draft `dropIndex`; `.hint()`, `.maxTimeMS()` and aggregate options are honoured.
✨ **MCP for AI agents**: new read-only `explain_query`, `active_sessions`, `slow_queries` and `cluster_health` tools across engines, PartiQL through `run_query`, and accurate descriptions.
✨ **Cross-engine**: history search across all connections, an "Explain this error" button that opens the assistant with the failing statement, Copy as Markdown table, IN list, CSV with header or curl, Watch for MongoDB, Elasticsearch, Redis and DynamoDB reads, a PartiQL full-scan warning, ⌘D back to multi-cursor and ⌘⇧O to jump to any table or column, a production or staging edge strip on the editor, and engine icons on every connection row.

#### Safety fixes
🛡️ The read-only and production gates could be bypassed on PostgreSQL and Redshift by a backslash or `#` inside a string, or a `$$` block (`WITH … DELETE` hidden in a CTE). The classifier is now dialect-aware.
🛡️ MySQL `SET` tricks (`@@GLOBAL` after a comma, `PASSWORD`, `DEFAULT ROLE`), `PREPARE`/`EXECUTE`, `KILL`, `SHUTDOWN`, `PURGE` and friends now require confirmation on production.
🛡️ Redis console on production now confirms any command that is not a read, not only a short deny-list.
🛡️ MCP: `CONFIG GET` and `ACL GETUSER` removed from the no-grant list (they expose passwords); ungranted queries run inside a read-only transaction; DynamoDB numbers are decoded exactly so a delete by a large numeric key cannot hit a neighbouring item.

#### Bug fixes
🐛 Grid edits, SQL and CSV imports on a PostgreSQL tab bound to another database went to the connection's default database.
🐛 Postgres and Redshift Explain always failed with an argument-count error; the quiet pre-flight EXPLAIN failed silently.
🐛 MongoDB `UPDATE … LIMIT n` and `DELETE … LIMIT n` affected every match; `ORDER BY` was ignored. `COUNT(*)` on empty input returned no row; `WHERE 1 = 1` would not parse.
🐛 DynamoDB: a sidebar double-click scan wrote into whatever tab was active, even another connection's; Load more paged with the builder's current inputs; export then import turned numbers into strings; PartiQL export ignored parameters.
🐛 Grid edits could target the wrong table when the query had a subquery or CTE; the server filter dropped your WHERE and JOINs; Apply ignored the tab's open transaction; a MySQL edit that changed nothing failed the whole batch.
🐛 Closing a running tab never cancelled the query. Switching a tab's data source stranded its transaction. `` USE `my-db` `` truncated the tab database. Auto-LIMIT and parameters misread quotes and comments. Format corrupted `DELIMITER` scripts. Shortcuts fired inside dialogs. SQL-file import of a mysqldump lost `FOREIGN_KEY_CHECKS` and `LOCK TABLES` across pooled connections.
🐛 Redis: cluster SCAN could run until the timeout, replies were uncapped, stale scan pages mixed with new ones, keys with non-UTF-8 names could not be opened or deleted, TTL -2 showed as infinite.
🐛 Backend: Disconnect racing a slow Connect, settings file read-modify-write races, SSH handshake without a deadline, semver pre-release compare, CSV header formula injection, SQLite busy timeouts, diagnostics report leaking hostnames, a missing keychain entry silently minting a new key over existing ciphertext.
🐛 UI: Mongo collection names with reserved words failed to open; double-click opened three tabs; designer quoted function defaults on Postgres; AA contrast on dark primary text; nested buttons in two section headers; the dead "result memory cap" setting removed; health-ping and autosave intervals now apply.

## [3.6.0] - 2026-10-17

v3.6.0 is a UI and stability release. Nothing was removed; the app looks more structured, the sidebar is one tree, and connections stop dropping on you after a laptop sleep or a VPN blip.

#### Connections sidebar
✨ **One tree.** Every connection now expands in place: click the chevron and its databases, tables, views and columns appear right under it, indented under the engine group. Open several at once; the set is remembered between launches. The separate Explorer section is gone.
✨ **Structured rail**: header toolbar (add, import, switch, group), engine groups with colour dots, tinted PROD / STG / LOCAL chips, hover actions (disconnect, edit, more), a compact icon footer for history, saved queries, snippets, local history, notebooks, performance, backup, MCP, settings, about and theme.
✨ **Tree polish**: indent guides per level, folder icons for Tables / Views / Indexes / Foreign keys, "n of m" counts while filtering, Collapse all, per-connection filter and refresh.

#### Fewer false "connection lost"
🐛 Most drops were one 3-second ping failing right after sleep or a VPN reconnect. The app now pings up to three times over ~15 s with an 8 s budget, and when a drop is real it rebuilds the pool or SSH tunnel silently and toasts "Reconnected". You are only asked when the rebuild fails or a tab has an open transaction that a reconnect would roll back.

#### Look and feel
✨ Layered surfaces and hairline borders in all four themes (dark, light, dim, solarized dark); the editor's Monaco theme is generated from the app theme so gutters, selection and current line match.
✨ Connection-coloured accent on editor tabs and a PROD / STG strip so you always know which environment a tab is on.
✨ Grouped toolbars with a prominent Run; labels collapse to icons before the toolbar overflows on narrow windows.
✨ Zebra-striped grid rows and a selection tint; elevated popovers and menus with short motion (honours reduce-motion).
✨ Shortcut cheat-sheet in empty states; a single keyboard-shortcut registry so the editor, menus and tooltips all show the same keys.

#### Stability and accessibility
🛡️ Every panel and overlay has its own error boundary that resets when you switch tabs, so one bad render no longer takes the shell down.
🛡️ Closing a tab now frees its editor model, grid state and per-tab stores (memory no longer grows with every closed tab).
🛡️ Shell no longer re-renders on every store update; snippets, notebook and performance panel open state persists; splitters work with the keyboard.
♿ Tree, tabs and dialogs expose proper roles, labels and arrow-key navigation; Escape closes the innermost dialog or menu first; AA contrast on text and status colours in every theme.

## [3.5.0] - 2026-10-10

v3.5.0 is about writing SQL faster and running it with fewer surprises: a rebuilt autocomplete, dry-run writes, an AI assistant that knows your schema, a workload index advisor, and a notebook mode.

#### Autocomplete and editor
✨ **Alias-aware completions**: `o.` lists the columns of the table aliased `o` anywhere in the statement, including CTEs, derived tables, INSERT targets and UPDATE … SET. Context is detected from the whole statement, not the current line.
✨ **JOIN … ON suggestions from foreign keys**: after `ON`, the editor proposes the matching key pairs; column items show type, nullability and key badges.
✨ **Signature help and hover** for MySQL and Postgres functions (`IFNULL(`, `DATE_FORMAT(`, `COALESCE(` …) and for tables and columns.
✨ Completion index keyed per database, bulk-loaded in one query and invalidated on DDL; identifiers that need quoting are quoted automatically.
✨ **Pre-flight lint** while you type: `DELETE`/`UPDATE` without WHERE, `SELECT *` into production tables, missing LIMIT, implicit cross joins. A quiet background EXPLAIN flags full scans on SELECTs before you run them (optional on production, Settings → Safety).

#### Safer writes
🛡️ **Dry-run preview** for a single UPDATE or DELETE: see the affected rows, before/after values and the row count inside a transaction that is rolled back; Apply then runs the real statement under the usual production confirmation.
🛡️ **Where is this used?** on any table or column: references from views, routines, triggers, foreign keys and saved queries before you drop or rename.
🔐 **PII masking** toggle in the results grid (emails, phones, cards, national IDs) that also applies to copies and exports.

#### Performance
✨ **Workload index advisor** (Performance → Index Health → Workload): mines slow-log / `pg_stat_statements` digests, proposes composite indexes with the expected gain, and trials them with MySQL invisible indexes or Postgres hypopg before you create anything.

#### Assistant
✨ **Ask** (⌘⇧A): a Claude-powered side panel that drafts, explains and fixes SQL using your live schema (tables, columns, keys, sampled types). Rows are never sent by default and never on production connections. Bring your own Anthropic API key (Settings → AI assistant, stored encrypted).

#### Compare, watch and notebooks
✨ **Compare on…** runs the same query on two connections and diffs the results row by row (added / removed / changed cells).
✨ **Watch…** re-runs a query on an interval with a threshold alert and a desktop notification when it fires.
✨ **Notebooks**: SQL and markdown cells on one connection, run all, inline charts, export to a self-contained HTML report.

#### Fixes
🐛 **Grid columns no longer shift while scrolling.** Every column now gets a pinned width from the loaded rows, so scrolling right and then down keeps the header and cells aligned (with or without Mask PII). Longer values further down truncate with a hover tooltip; drag the header edge to widen.
🐛 ⌘⇧Enter (Run All) did nothing while the editor had focus; it inserted a blank line instead.
🐛 Shift-click range selection could select only the two end rows, so the selection stats were wrong.
🐛 EXPLAIN summary showed "Worst Access: N/A" when every step was efficient.

## [3.4.0] - 2026-10-10

v3.4.0 is the largest release so far: ten commits, ~760 tests, every engine touched. Highlights by area.

#### Results grid and editor
✨ **Find in results** (⌘F in the grid): live match count, Enter/Shift+Enter cycling, highlighted hits.
✨ **Column resize, reorder and hide**, a Columns… popover, and a **row detail side panel** that follows the focused cell. Arrow keys move between cells.
✨ **Copy rows as INSERT or JSON**, **selection stats** (count / sum / avg / min / max) in the footer, and **charts** from any result (bar, line, area, pie, scatter) with per-tab config.
✨ **⌘S saves the query** (in place when linked), **dirty-tab confirm** on close, **Local History** with autosave every 2 s plus run/close snapshots, diff and restore.
✨ **Query parameters**: `{{name}}` / `:name` prompt at run time with safe literals. **Error squiggles** at the server-reported position. **Routine templates** with DELIMITER, **Run… with params** for procedures, event scheduler tooling.
✨ **Visual EXPLAIN tree** (MySQL, Postgres, Mongo), **FK click-through** (open the referenced row, or rows referencing this one), **Run All** button and ⌘⇧Enter, History / Saved Queries / Snippets / Local History in the sidebar.
🐛 Session restore could be overwritten by an early save and lose your tabs; the panels now follow the tab's connection; many more.

#### Explorer, schema and transfer
✨ **Schema diff** between two connections with a migration script (A→B or B→A, DROPs opt-in).
✨ **Virtualised explorer** (5,000 tables: 35k DOM nodes → 266), per-node Refresh, inline "Failed to load — Retry".
✨ **Truncate / Duplicate table**, **Copy DDL**, **Import connections** from DBeaver (credentials decrypted), DataGrip and `~/.my.cnf`.
✨ **Streaming export to file** (CSV / JSON / XLSX) with Stop, formula protection and a warning on capped exports.

#### Transactions and safety
🛡️ **Transactions are scoped to a tab** on MySQL, Postgres and Redshift; Run All runs on one pinned connection (`SET @x`, temp tables carry across); typed COMMIT/ROLLBACK stay in sync; rollback on tab close.
🛡️ **Production confirmation on every mutating binding** (designers, table editor, imports, restore, kill, Redis keys, Dynamo items, Postgres indexes, views). Side-effect calls like `pg_terminate_backend`, `SET ROLE` and `COPY` to files count as destructive.
🛡️ SSH keepalive + ssh-agent auth, single-instance lock, SQLite busy timeout, AWS keys redacted from history, Mongo tunnel secrets encrypted.

#### Stability
🧯 **Error boundaries** around every pane with Reload panel and **Copy diagnostics**; a global error funnel; a clear message when frontend and backend are out of sync.
🧯 Pollers pause when the window is hidden and back off on failures. **Lost connections show a Reconnect banner** that rebuilds the pool and SSH tunnel in place.
🧯 **Result retention cap** (released results show Re-run), 20,000-line scripts no longer lag, **Settings dialog** (⌘,), layout persists across restarts, grid accessibility (roles, `aria-sort`, keyboard row selection).

#### MySQL
🐛 Index Manager broke on tables with a functional index; `WITH`/`CALL` showed 0 rows when a database was selected; BLOB/BIT rendering; hyphenated database names; ALTER designer dropped UNSIGNED/charset/ON UPDATE/GENERATED; SSH + SSL never connected; EXPLAIN on MariaDB; table editor could not write NULL; `KILL` did nothing for idle-in-transaction sessions.
✨ MySQL 8 **roles**, **partition view**, foreign keys in the designer, mysqldump flags + "Backup Schema Only", custom CA / client cert.

#### PostgreSQL
✨ **Performance pane**: activity with cancel/terminate, blocking tree, `pg_stat_statements`, unused/redundant/invalid index advisor, vacuum and bloat, overview tiles.
✨ **Multiple databases per server**, functions/procedures/triggers viewer and editor, sequence and enum editors, default privileges and role membership, `pg_dump`/`pg_restore`, COPY-based CSV import, NOTICE messages and the "transaction is aborted" hint, LISTEN/NOTIFY panel, RDS IAM auth, PgBouncer pooler mode, SSL CA / client cert.
🐛 Tree actions overwrote the active tab, `SELECT … INTO` was auto-limited, `search_path` dropped `public`, single-row edits ran without a transaction, PK found only by `_pkey` name, `pg_%` filter hid `pgboss`-style schemas.

#### Redshift
✨ **Serverless support** (`SYS_*`), load-error viewer with fix hints, WLM queues + QMR actions, per-step query report, table advisor, **COPY/UNLOAD dialog**, designer with DISTSTYLE / SORTKEY / ENCODE, users, groups and grants, matviews/procedures/datashares in the tree, editor transactions.
🐛 IAM credentials expired after 15 minutes; `prefer_simple_protocol` was silently ignored by pgx v5.

#### DynamoDB
🐛 Numbers above 2^53 were rounded (wrong item deleted or a phantom created), sets saved as lists, every save rewrote untouched attributes, "Add Item" overwrote silently, GSI queries used base-table keys, BETWEEN never worked.
✨ Typed items end to end, conditional minimal writes, Stop, **batch delete / delete by filter** (dry run first), **JSON import and export** with a pre-scan cost gate, **table admin** (create/delete, capacity, GSI, TTL, PITR, backups, streams), PartiQL paging + parameters, assume-role with MFA, free-text region.

#### MongoDB
🐛 HAVING and DISTINCT were silently dropped, COUNT(col) counted everything, aliases became nested paths, the shell rewrote text inside strings, `NumberLong` became Int32, `upsert` was ignored.
✨ **Document editor** (typed extended JSON, optimistic save, clone), **index manager**, **explain tree**, collection stats, sampled schema analyzer, **performance pane** with kill op, **aggregation pipeline builder** with per-stage preview, JSON/CSV import and export, validation rules, users and roles, change-stream watch, auth mechanisms, read preference, TLS certs, SSH tunnel.

#### Redis
🐛 A sorted set with `+inf` exited the app; `SELECT`/`MULTI`/`AUTH` poisoned pooled connections; the read-only allow-list blocked common reads; "Scan all" looped on error.
✨ Timeouts follow settings, binary-safe values with a hex view, delete by pattern with dry run, multi-line console, cluster seeds and sentinel password, paging inside big values, stream groups, console history survives navigation.

#### Known limits
- Redshift catalog columns for serverless views and some Dynamo/Mongo flows were verified with unit tests, not a live server. Try them on a non-production target first.
- Postgres COPY import falls back to INSERT for enum/custom-type columns (untick COPY).
- Dynamo lists and maps are edited as typed DynamoDB JSON.

## [3.3.0] - 2026-10-09

### Fixed
- **Results grid hung and jumped while scrolling** — rows are measured (not estimated at the wrong height) and the per-cell copy button is gone; wide 100-row results scroll at full frame rate.
- **Query history showed nothing** — the panel now follows the tab's connection, refreshes after every run, and the favourite star updates.
- **`WITH` / `CALL` / `TABLE` / `VALUES` returned 0 rows** when a database was selected in the tab. `CALL` now shows every result set.
- **Index Manager broke on tables with a functional index** (which also broke tree expand and the ALTER designer). `SHOW INDEX` is read by column name (MySQL + MariaDB); prefix (`name(20)`) and DESC index parts are supported; the list refreshes after create/drop.
- **ALTER designer dropped column attributes** (UNSIGNED, charset/collation, ON UPDATE, GENERATED, INVISIBLE, SRID, comment) and move up/down marked every column modified.
- **Statement splitting**: `DELIMITER` (procedures/triggers), `"…"`, backticks, `\'`, `#` comments and `$$` bodies are handled by one scanner for Run, Run All and the safety classifier. Auto-LIMIT handles CTEs, `FOR UPDATE`, trailing comments and subquery LIMITs.
- **Session state lost between statements** — Run All executes on one pinned connection, so `SET @x`, temp tables and `SET SESSION` carry across.
- **MySQL types**: BLOB/VARBINARY/GEOMETRY rendered as garbage (now `0x…` hex), `BIT(n)` showed `[0 5]` (now an integer), BIT(1) edits failed in strict mode. Database names with `-` or space could not be queried.
- **SSH tunnel + SSL never connected** (TLS verified against 127.0.0.1). EXPLAIN failed on MariaDB/MySQL 5.6 and ignored the tab database. Locks tab enabled on MariaDB.
- **Table editor could not write NULL** (stored the string `'NULL'`). CSV import turned empty cells into `''` and ran without a transaction.
- **Kill** did nothing for idle-in-transaction sessions — "Kill connection" is offered for them (the only thing that releases their locks).
- **Redis**: opening a sorted set with a `+inf`/`-inf` score exited the app; `SELECT`/`MULTI`/`AUTH`/`HELLO` ran on pooled connections; the read-only allow-list blocked common reads (set/bit/geo/stream ops, `CONFIG GET`, `CLIENT LIST`, RedisJSON/RediSearch/TimeSeries); "Scan all" looped forever on an error; Add Key overwrote existing keys.
- A stale run could overwrite a newer result or steal its cancel handle; per-tab state leaked on close; dumps/exports were world-readable; session/settings writes were not atomic.

### Added
- Production confirmation on CreateTable, every ALTER, index create/drop, table-editor writes, grid Apply, CSV import, Restore and Kill; a `requireWritable` gate enforces read-only on 40 mutating bindings. Typed `BEGIN` is refused on MySQL/Postgres (use the toolbar transaction controls).
- Result size cap (32 MB per result, 256 KB per cell) with a warning and a "limited to N rows · Load more" badge.
- Foreign key add/drop and PK/UNIQUE changes in the ALTER designer.
- Custom CA / client certificate fields for MySQL; mysqldump `--no-tablespaces --hex-blob --set-gtid-purged=OFF --column-statistics=0` + TLS flags; "Backup Schema Only".
- "Set to NULL" in the table editor; "Treat empty cells as NULL" in CSV import; BOM stripping and streaming for large files.
- Logs at `~/.querypilot/logs/querypilot.log` (Wails panics captured, credentials redacted).

### Performance
- App is ~9 MB smaller: Monaco ships only the SQL and JSON languages. Undo history survives pane switches.

## [3.2.3] - 2026-09-16

### Changed
- **Run executes the selection or the statement under the cursor** (previously the whole tab, displaying the last result — which looked like a query returning another query's columns). **Run All Statements** is in the ⋯ menu; the statement that will run is highlighted.

### Fixed
- Duplicate column names (`SELECT a, a`, JOINs sharing `id`/`created_at`) corrupted the results grid — stale header/cell DOM from a previous result could appear in the next one rendered in that tab.
- JOINs silently lost column values: with duplicate names only the last column's data was kept. Duplicates are now renamed (`id`, `id_2`, …) so all values survive, in the grid and exports.
- Sticky headers smeared while scrolling (WebKit `position:sticky` + `border-collapse` bug) — rows bled through the header band. Grids use a WebKit-safe layout with opaque sticky rows.
- CSV import mapping with duplicate CSV headers.

## [3.2.2] - 2026-09-15

### Fixed
- **A tab could execute different SQL than it displayed** — after switching tabs, a tab's stored query could silently go stale, so Run kept executing an older statement (old columns in results) while the editor showed the new text. The executed SQL is now reconciled from the visible editor on every tab switch.
- Statement splitter no longer splits on `;` inside double-quoted strings.
- Sort and column filters survive re-running a query (per tab); row selection still resets with fresh results.
- Editing the Apply-changes SQL and having it fail no longer discards pending edits.
- MongoDB `INSERT` maps a bare `id` column to `_id`, matching WHERE/SET, so inserted rows stay queryable.

### Added
- JSON cell values (DynamoDB maps/lists, MongoDB nested documents, MySQL JSON columns) expand into a collapsible JSON tree with per-node copy.

### Performance
- JSON detection no longer parses every visible cell per render; the editor no longer re-reads the document per keystroke.

## [3.2.1] - 2026-09-03

### Fixed
- DynamoDB: selecting a table in one tab changed it in every open tab — the query builder is now per-tab (table, Scan/Query/PartiQL mode and statement text independent and remembered per tab).
- Results grid: sort, column filters, filter-row visibility and row selection leaked across tabs; a filter from one tab could silently hide rows in another. All are now per-tab, restored on switch-back, reset on a new result.
- Results grid: the sticky row-number column was translucent, letting horizontally-scrolled cell text bleed through the numbers. Now opaque in both themes.

## [3.2.0] - 2026-08-31

### Added — MongoDB engine
- **MongoDB support** — sixth engine. Connect via host/port or a full `mongodb://` / `mongodb+srv://` URI (credentials kept in separate encrypted fields), browse databases → collections → sampled fields, and query with **SQL** (`SELECT`/`WHERE`/`ORDER BY`/`LIMIT`/`OFFSET`, `GROUP BY` with COUNT/SUM/AVG/MIN/MAX translated to an aggregation pipeline, `INSERT`/`UPDATE`/`DELETE`, dotted paths for nested fields, bare `id` meaning `_id`, 24-hex strings promoted to ObjectID, `LIKE` as anchored case-insensitive regex) or **mongosh syntax** (`db.coll.find({...}).sort({...}).limit(n)`, `aggregate`, `countDocuments`, `insertOne/Many`, `updateOne/Many`, `deleteOne/Many`, `ObjectId()`, `ISODate()`, bare keys and single quotes).
- Read-only connections refuse writes; production-tagged connections route writes (SQL or shell) through the same tiered confirmation dialog as the SQL engines. Reads without a LIMIT are capped at 500 documents.
- **MCP**: `list_databases`, `list_tables`, `describe_table` and `run_query` now dispatch to MongoDB connections — reads always available, writes require a per-connection grant, results capped at 500 rows.

### Fixed
- Results grid jumped back to the top after switching tabs and returning — scroll position is now kept per tab, restored through both the DOM and the virtualizer. Pending inline edits also survive a tab switch instead of being discarded.
- Undo (⌘Z) stopped working after switching tabs — the editor now keeps one Monaco model per tab, preserving undo history and cursor position.
- The Apply-changes review dialog is now a wide, **editable** SQL script (MySQL Workbench style); edited SQL still runs through the read-only/production gates.
- Query history now records MongoDB statements, Redis console commands and DynamoDB PartiQL alongside SQL.

## [3.1.0] - 2026-08-25

### Added — MySQL server monitoring
- **Performance > Overview** — live rates derived from consecutive `SHOW GLOBAL STATUS` samples rather than raw counters: queries/s split into reads, writes, prepared statements (`Com_stmt_execute`, which never touches `Com_select`) and other; threads running against `max_connections`; InnoDB buffer-pool hit rate and dirty pages; slow queries/s; full table scans and unindexed joins/s; temp-table disk spill; row lock waits; network; new vs aborted connections; uptime; replication lag. Sparklines cover the last ~5 minutes, and a health strip states the verdict with the tab to check next. Server restarts, zero intervals and idle servers are handled explicitly instead of rendering nonsense rates.
- **Performance > Connections** — `PROCESSLIST` aggregated by user@host / user / host / database (total, active, sleeping, in-transaction, longest), plus sessions past a chosen age. The threshold also applies to open-transaction age, so a session asleep for seconds while holding a ten-minute transaction is caught and flagged **idle in transaction**. Kill routes through the existing production-gated dialog.
- **Metric drill-downs** — the full-table-scan tile opens the statement digests that ran without a usable index, each expandable into who runs it (user@host), a sample statement, copy, and EXPLAIN-in-editor. Other tiles jump to Slow Queries, Locks, Index Health or Connections.

### Added — schema and query tools
- **ER diagrams generated from foreign keys** for MySQL databases and PostgreSQL schemas — one whole-schema constraint query, layered so referenced tables sit left of the tables referencing them, with pan/zoom, neighbour spotlighting and click-to-open. PostgreSQL reads `pg_constraint` so composite foreign keys pair correctly.
- **EXPLAIN before/after comparison** — capture a baseline plan, change an index, re-run, and compare access type, chosen key and rows examined with an improved/regressed verdict per table. A better access path outranks small row-estimate movement in both directions.
- **`EXPLAIN ANALYZE` for MySQL 8.0.18+** — measured per-operator time, actual rows and loops. Version-gated (MariaDB and older MySQL excluded) instead of failing at run time.
- **ANALYZE / CHECK / OPTIMIZE TABLE** from the table context menu, surfacing MySQL's own `Msg_type: Msg_text`. OPTIMIZE confirms first; ANALYZE and OPTIMIZE are blocked on read-only connections.

### Fixed
- Right-click context menus did nothing in every v3.0.x build. The menu subscribed to document-level close events on mount, and React flushes a discrete event's update and effects inside the dispatch — so the same right-click that opened the menu closed it again.
- Performance > Variables could not scroll: the panel's height chain never resolved, so the table was clipped rather than scrollable.

## [3.0.2] - 2026-08-25

### Fixed
- Enabling the MCP server did nothing on a fresh install — the toggle saved but the server stayed "Not running". Starting it deadlocked while minting the per-install auth token on machines that had none yet.
- MySQL autocomplete had no tables or columns until a database was expanded in the tree. Connecting now loads the active database's tables (matching PostgreSQL/Redshift), and switching a tab's database — toolbar picker, tree, or an in-query `USE db` / `SET search_path` — loads that database too.
- PostgreSQL, Redshift and DynamoDB showed a stale schema tree after disconnect → reconnect; the engine-specific caches were never cleared, and the sidebar only refetches when a connection's cache is missing.

### Performance
- Schema loads on one connection no longer invalidate every other connection's completion index, which made each keystroke re-walk every cached schema.
- Searching the schema tree runs 4 `information_schema` queries at a time instead of one per database simultaneously.

## [3.0.1] - 2026-08-25

### Fixed
- MCP Server settings dialog failed to open on a fresh install (`Failed to load MCP server settings: null is not an object`) — the allowed-connections list was serialized as `null` instead of an empty list. It now opens on first run and on upgrades from older settings files.

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
