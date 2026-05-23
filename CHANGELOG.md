# Changelog

All notable changes to DB Connect will be documented in this file.

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
