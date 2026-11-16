---
name: postgres-efcore
description: PostgreSQL with EF Core and Npgsql in Turnos App. Use when designing tables, writing entity configurations or migrations, adding indexes or constraints (including the exclusion constraint that prevents double bookings), writing queries, handling concurrency, or diagnosing database errors and slow queries.
when_to_use: Use when the task mentions "base de datos", "migración", "tabla", "índice", "constraint", "consulta lenta", "PostgreSQL", "Postgres", "Npgsql", "EF Core", "DbContext", "tstzrange", "doble reserva", or touches Migrations, entity configurations or SQL files.
---

# PostgreSQL + EF Core (Npgsql)

General EF Core guidance is in the plugin skill `dotnet-claude-kit:ef-core`. This skill covers what is specific to PostgreSQL and to this project. Check provider details against the Npgsql EF Core documentation for the version pinned in `Directory.Packages.props`; mapping APIs change between major versions.

## Conventions

- Tables and columns are **snake_case** and plural for tables (`bookings`, `starts_at`). Apply it with one naming convention for the whole model, not name by name.
- One `IEntityTypeConfiguration<T>` per aggregate, in `Infrastructure`. No mapping attributes on domain types.
- Strongly typed ids and value objects are mapped with value converters or owned types.
- Every table has `created_at`, `updated_at` and `created_by`, filled automatically on save.
- Name constraints and indexes explicitly (`ix_bookings_professional_id_starts_at`, `ex_bookings_no_overlap`), so error handling can recognise them.

## Types

| Data | PostgreSQL type | Notes |
|---|---|---|
| Instant | `timestamptz` | Always UTC. Npgsql rejects a `DateTime` that is not `Kind=Utc` |
| Calendar date, time of day | `date`, `time` | `DateOnly`, `TimeOnly`. Weekly availability is a time of day in the schedule's zone, not an instant |
| Time zone | `text` | IANA id, for example `America/Argentina/Buenos_Aires` |
| Time range | `tstzrange` | Start inclusive, end exclusive: `'[)'` |
| Money | `numeric(12,2)` plus a currency column | Never `float` or `double` |
| Open-ended per-offering fields | `jsonb` | Only when the shape really varies per row. Index what you filter on |
| Enum | `text` or `smallint` with a converter | Keep the stored value stable when the C# name changes |

## Preventing double bookings

A check in code cannot prevent two concurrent requests from taking the same slot. The database does it with an exclusion constraint: for one professional, the time ranges of active bookings cannot overlap.

```sql
CREATE EXTENSION IF NOT EXISTS btree_gist;

ALTER TABLE bookings
  ADD CONSTRAINT ex_bookings_no_overlap
  EXCLUDE USING gist (
    professional_id WITH =,
    tstzrange(starts_at, ends_at, '[)') WITH &&
  )
  WHERE (status IN (/* the states that occupy the slot */));
```

- Add it with raw SQL in a migration; EF Core does not model exclusion constraints.
- The range includes the break after the session when the offering defines one.
- The list of states that occupy a slot comes from the booking state machine and from the pending-payment policy of the offering. Keep that rule in one place and test it.
- On save, a violation raises a `PostgresException` with `SqlState` `23P01` (exclusion violation). Catch it in the repository or the handler and return the business error "that time has just been taken". A unique violation is `23505`.
- Cover it with an integration test that runs two bookings for the same slot concurrently and asserts that exactly one succeeds.

## Migrations

- One migration per logical change, named after it (`AddBookingOverlapConstraint`). Read the generated code before committing: check column types, nullability, default values and that nothing unexpected is dropped.
- Never edit a migration that has been merged; add a new one.
- A migration must be safe on a database with data: add a column as nullable or with a default, backfill, then tighten.
- Deployment applies an idempotent script (`dotnet ef migrations script --idempotent`). The application does not migrate itself at start-up in production.
- Extensions (`btree_gist`) are created in a migration with `IF NOT EXISTS`. Confirm the hosting allows it (card E1-05).

## Queries

- Read queries use `AsNoTracking()` and project to a DTO with `Select`. Load an aggregate with tracking only to change it.
- Lazy loading is off. Use `Include` for what the use case needs, or a projection. Check the generated SQL of a new query with `ToQueryString()` or the logs.
- Page every list. Prefer keyset pagination for long, frequently updated lists.
- Push filters to the database: do not call `ToListAsync` and then filter in memory.
- Index the columns used in `WHERE`, `JOIN` and `ORDER BY` of the frequent queries. Use partial indexes for queries over a small subset (for example active bookings) and a GiST index for range overlap.
- Use `EXPLAIN (ANALYZE, BUFFERS)` on a slow query before changing it, and record the result.
- Raw SQL is parameterized (`FromSql` with interpolation, never string concatenation).

## Concurrency and transactions

- One use case is one `SaveChangesAsync`. Use an explicit transaction only for a use case that needs several.
- Use optimistic concurrency on aggregates that two users can edit (PostgreSQL's `xmin` system column or a version column) and turn `DbUpdateConcurrencyException` into a business error or a retry.
- Keep transactions short and never call an external provider inside one.
- Take locks in a consistent order. Prefer a constraint to an explicit lock.

## Connections

- The shared hosting may put a connection pooler in front of PostgreSQL. Validate the port and the Npgsql settings that poolers need (automatic statement preparation, connection reset) in the hosting spike (card E1-05), and record the result in the standards.
- The connection string comes from configuration, never from code or a tracked file.
- Retry transient failures with the provider's execution strategy; a use case that retries must be idempotent.

## Tests

- Integration tests run against PostgreSQL in Testcontainers with the real migrations applied, never against an in-memory provider or SQLite: constraints, ranges and `jsonb` only behave correctly on PostgreSQL.
- Each test starts from a known state and does not depend on another test.
- Test the constraint, not only the code path that respects it.

## Related

The optional plugin `postgres-best-practices@supabase-agent-skills` has general PostgreSQL performance guidance (see `docs/claude-code-setup.md`).
