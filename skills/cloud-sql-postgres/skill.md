---
name: cloud-sql-postgres
description: Use this skill to inspect, monitor, administer, and run SQL against
  a Cloud SQL for PostgreSQL instance via the MCP Toolbox CLI. Covers schema
  discovery, query execution, performance and replication diagnostics, and
  instance/database/user lifecycle management.
---

## Usage

All operations run through the local `toolbox` CLI against the bundled
`cloud-sql-postgres` prebuilt config — no server, no separate setup.

### Discover before you call

This skill intentionally does not enumerate every parameter — the CLI is
self-describing. Use it:

    # See every tool available with a one-line description
    toolbox --prebuilt cloud-sql-postgres list-tools

    # See the exact parameter signature (name, type, required, description)
    toolbox --prebuilt cloud-sql-postgres describe-tool <tool_name>

Run `describe-tool` before invoking anything you haven't called in this session.
Tool signatures are the source of truth — do not assume parameters from the
names below.

### Invoke

Typed flags (preferred for simple scalars):

    toolbox --prebuilt cloud-sql-postgres invoke <tool_name> \
      --<param_name> <param_value>

JSON blob (preferred when a parameter contains SQL, quotes, or nested JSON):

    toolbox --prebuilt cloud-sql-postgres invoke <tool_name> \
      '{"<param_name>": "<param_value>"}'

PowerShell — escape the inner quotes:

    toolbox --prebuilt cloud-sql-postgres invoke <tool_name> `
      '{\"<param_name>\": \"<param_value>\"}'

All `invoke` output is JSON on stdout. Errors go to stderr with a non-zero exit.

## Tools by purpose

Pick the closest-matching tool, then run `describe-tool` for its signature.

**Schema discovery** — `list_schemas`, `list_tables`, `list_views`,
`list_indexes`, `list_sequences`, `list_triggers`, `list_stored_procedure`,
`list_databases`, `list_roles`, `list_tablespaces`, `list_publication_tables`,
`database_overview`.

**Ad-hoc SQL** — `execute_sql` (single statement),
`get_query_plan` (EXPLAIN without executing).

**Performance & stats** — `list_query_stats`, `get_query_metrics`,
`list_table_stats`, `list_database_stats`, `get_column_cardinality`,
`list_top_bloated_tables`, `list_invalid_indexes`,
`list_autovacuum_configurations`, `list_memory_configurations`,
`list_pg_settings`, `get_system_metrics`.

**Live activity & locking** — `list_active_queries`, `long_running_transactions`,
`list_locks`.

**Replication** — `replication_stats`, `list_replication_slots`.

**Extensions** — `list_available_extensions`, `list_installed_extensions`.

**Instance lifecycle (Cloud SQL Admin API)** — `list_instances`, `get_instance`,
`create_instance`, `clone_instance`, `create_database`, `create_user`,
`create_backup`, `restore_backup`, `postgres_upgrade_precheck`,
`wait_for_operation`.

## Conventions

- **Always prefer a dedicated tool over `execute_sql`** when one exists. The
  dedicated tools return structured metadata and are safer against injection.
- **`project` and instance details are pre-configured** — do not pass them
  unless the user explicitly overrides.
- **Long-running admin operations** (`create_instance`, `clone_instance`,
  `restore_backup`, `create_backup`) return an Operation object. Follow up with
  `wait_for_operation` before doing anything that depends on the result.
  Use a polling multiplier of 4 for `clone_instance`.
- **Tool not found** → re-run `list-tools` to confirm the name; do not guess.
- **Connection error** → surface the missing env var rather than retrying.
