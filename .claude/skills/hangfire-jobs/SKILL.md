---
name: hangfire-jobs
description: Background and scheduled jobs in Turnos App with Hangfire on PostgreSQL storage, including the "tick" endpoint needed on shared hosting. Use when adding, changing, testing or reviewing a job - booking expiry, payment reconciliation, notifications, reminders, token refresh - or the Hangfire configuration and dashboard.
when_to_use: Use when the task mentions "job", "tarea programada", "segundo plano", "background", "Hangfire", "cron", "tick", "recordatorio", "vencimiento automático", "reintento", or touches job classes or the Hangfire setup.
---

# Hangfire Jobs

Check configuration details against the documentation of the Hangfire and Hangfire.PostgreSql versions pinned in the project.

## Why the tick exists

The application runs on shared hosting. The app pool stops when there is no traffic, and a stopped process runs no jobs. So:

- An endpoint, `/jobs/tick`, is called on a schedule by the hosting's scheduled task or by an external cron service. The request wakes the application and lets Hangfire process what is due.
- The endpoint is protected by a secret key sent in a header, compared in constant time, and rate-limited. It returns no information about the jobs.
- The schedule is coarse (every 15 minutes at best). Design every job to tolerate running late.

Never assume a job runs at the exact time it was scheduled for. A job computes what is due from the current time (`IClock`) and the data, not from the time it was supposed to run.

## Rules for every job

- **Thin wrapper.** A job class resolves a use case and calls it. The rule lives in `Application` and `Domain`, where it is tested without Hangfire.
- **Ids as arguments.** Pass the booking id, never the entity or a large object. Arguments are serialized and stored; the job loads fresh data when it runs.
- **Idempotent.** Hangfire delivers at least once, and the tick can overlap a retry. Running a job twice must have the effect of running it once. The use case checks the current state before acting ("expire only if still pending and past the deadline").
- **Safe under concurrency.** Two workers may pick up work for the same booking. Rely on the aggregate's state check plus a concurrency token or a database constraint, not on timing.
- **Short transactions, no provider calls inside them.** A job that calls an external provider commits its own state first, then calls, then records the result.
- **Bounded work.** A recurring job processes a page of items per run and continues in the next run, so one run cannot exhaust the memory or the time the hosting allows.
- **UTC.** Schedules and comparisons are in UTC; time zones apply only when a rule is about the user's local time (a reminder 24 hours before a session).

## Kinds of job

| Kind | Use it for | Example |
|---|---|---|
| Fire-and-forget | Work triggered by a request or a domain event that should not block the response | Process a webhook, send a notification |
| Delayed | One action at a later time for one item | Remind one client 24 hours before |
| Recurring | A sweep that finds what is due | Expire unpaid bookings, reconcile pending payments, refresh tokens |

Prefer a recurring sweep to many delayed jobs when the due time can change (a rescheduled booking would leave a stale delayed job behind). A delayed job must re-check that it still applies.

## Retries and failures

- Let Hangfire retry transient failures. Set the number of attempts per job type instead of relying on the default.
- A failure that will never succeed (the booking no longer exists) is not an error: the job ends without retrying.
- When retries are exhausted, the job stays visible in the dashboard and is logged with the ids needed to investigate.
- A notification job records each attempt and its result: to whom, what, when and the outcome.

## Configuration

- Storage is PostgreSQL, in its own schema, using the application's connection settings.
- The dashboard is visible only to the Admin role. It is never public and never open by IP alone.
- Queues and worker count are small: the hosting has limited memory.
- Recurring jobs are registered in one place at start-up, with stable ids.
- The tick key is a secret from configuration, never in the repository.

## Tests

- Test the use case, with a fake clock: before the deadline nothing happens, after it the transition occurs, and a second run changes nothing.
- Test the job wrapper only for what it adds: resolving the use case and passing the id.
- Test the tick endpoint: wrong or missing key is rejected, a valid key is accepted.
- An integration test covers one job end to end against PostgreSQL in Testcontainers when the job depends on a query or a constraint.

## Review checklist

- [ ] The job only calls a use case and passes ids.
- [ ] Running it twice, late, or concurrently is safe, and a test proves it.
- [ ] Time comes from `IClock`.
- [ ] Each run processes a bounded number of items.
- [ ] No external call inside a database transaction.
- [ ] No secret or personal data in job arguments or logs.
- [ ] The dashboard and the tick endpoint are protected.
