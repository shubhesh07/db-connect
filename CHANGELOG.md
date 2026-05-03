# Changelog

All notable changes to DB Connect will be documented in this file.

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
