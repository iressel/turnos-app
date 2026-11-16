# Backend Standards

Standards for the C# code of Turnos App: .NET 10, ASP.NET Core, EF Core on PostgreSQL. They refine [base-standards.md](./base-standards.md). Where the code does not exist yet, they describe how it must be written; the card that builds each part is named in brackets.

## Contents

- [Architecture](#architecture)
- [Project Settings](#project-settings)
- [Naming](#naming)
- [File Layout and Syntax](#file-layout-and-syntax)
- [Domain Layer](#domain-layer)
- [Application Layer](#application-layer)
- [Web Layer](#web-layer)
- [Error Handling](#error-handling)
- [Validation](#validation)
- [Logging](#logging)
- [Data Access](#data-access)
- [Transactions and Side Effects](#transactions-and-side-effects)
- [Dates and Time Zones](#dates-and-time-zones)
- [Background Jobs](#background-jobs)
- [Testing](#testing)
- [Security](#security)
- [Performance](#performance)
- [Dependencies](#dependencies)
- [Git Workflow](#git-workflow)

## Architecture

```text
Turnos.Web             Razor Pages, htmx fragments, Identity, Minimal APIs (webhooks)
Turnos.Infrastructure  EF Core/Npgsql, repositories, payment, email and messaging adapters, Hangfire, files
Turnos.Application     use cases (commands and queries), DTOs, ports
Turnos.Domain          aggregates, value objects, domain events, business rules
```

- `Domain` references no other project and no infrastructure package. `Application` references only `Domain`. `Infrastructure` and `Web` reference inwards. An architecture test fails the build when this is broken (E1-02).
- Each bounded context (Identity, Catalog, Scheduling, Bookings, Payments, Subscriptions, Notifications, Backoffice) is a folder inside each layer. A context reaches another through its public use cases or its domain events, never through its tables or internal types.
- An external system (Mercado Pago, the email provider, Telegram, the file system, the clock) is reached through a port declared in `Application` and implemented in `Infrastructure`.

## Project Settings

- `Directory.Build.props` enables nullable reference types and implicit usings and treats warnings as errors. Do not disable a warning for a project; fix it, or suppress one occurrence with a comment that says why.
- `Directory.Packages.props` holds every package version (central package management). Pin exact versions: no floating or range versions.
- The NuGet audit stays on: a vulnerable package fails the build.
- `.editorconfig` is the source of formatting rules. Run `dotnet format` on the files you change.

## Naming

- Standard .NET casing: PascalCase for types, methods and properties; camelCase for parameters and locals; `_camelCase` for private fields; `I` prefix for interfaces.
- Async methods end in `Async` and take a `CancellationToken` as their last parameter.
- Use cases are named after the action: `ConfirmBookingCommand`, `ConfirmBookingHandler`, `GetAvailableSlotsQuery`.
- Domain events are named in the past tense: `BookingRequested`, `PaymentConfirmed`.
- Tests: one `<Subject>Tests` class per subject; test names are sentences joined by underscores (`Booking_cannot_be_confirmed_after_it_expired`).
- Use the glossary name of each domain concept ([base-standards.md](./base-standards.md), section 2).

## File Layout and Syntax

These rules apply to every new file and to every line a change writes. In an existing file, the lines the change does not touch keep their layout.

- **One type per file**, named after the type.
  - Do not declare a class, record, struct, interface or enum inside another type, or a second top-level type in the same file.
  - This includes private helpers: a record that only one class uses becomes an `internal` type in its own file.
- **File-scoped namespaces**: write `namespace Turnos.Domain.Bookings;`, never `namespace X { ... }`. The namespace matches the folder.
- **Collection expressions** wherever there is a target type. Write `[a, b]`, `[]` and `[.. items]` instead of `new[] { ... }`, `new List<T> { ... }`, `Array.Empty<T>()`, `Enumerable.Empty<T>()`, `ToArray()` or `ToList()` on an in-memory sequence.
  - A database query is the exception: `[.. query]` runs it synchronously. Materialize queries with `await query.ToListAsync(cancellationToken)`.
  - Replace `AsReadOnly()` only when you want a copy. It returns a view that follows later changes to the list, while a collection expression copies.
  - Where C# has no target type, keep the explicit form: an argument of type `object` or an element of `params object[]`, such as the arguments of a parameterized test attribute.
- Prefer an early return to an empty fallback that only feeds a loop (`items ?? []` followed by `foreach`).
- Prefer `sealed` classes, primary constructors for simple dependency capture, `required` members for DTOs, and records for value objects and messages.

## Domain Layer

- Aggregates protect their invariants. State changes only through methods of the aggregate root; setters are private.
- A value object is immutable and compared by value. Use it instead of a primitive whenever a rule is attached to the value (money, time range, slug, phone number).
- Identifiers are strongly typed (`BookingId`, not `Guid`).
- State machines live in the aggregate: each transition is a method that checks the current state and returns a failure for an invalid transition.
- The aggregate records domain events; it does not send emails, call providers or read the clock directly.
- No dependency on EF Core, ASP.NET Core or any provider SDK.

## Application Layer

- One handler per use case. A command changes state and returns a `Result`; a query returns a DTO and changes nothing.
- A handler loads the aggregates, calls the domain, saves, and returns. Rules stay in the domain.
- Handlers are our own or generated by a source-generator mediator with an MIT licence. MediatR is not used (commercial licence).
- Mapping between domain objects and DTOs is written by hand. AutoMapper is not used.
- DTOs returned to the web layer are a contract: changing a field's name, type or nullability is a breaking change and must be called out.

## Web Layer

- A page model or endpoint binds the request, checks authorization, calls one use case and renders the result. It contains no business rule and no query against the `DbContext`.
- Minimal APIs are only for webhooks and htmx fragments. Every other screen is a Razor Page.
- Every webhook endpoint authenticates its caller (signature, secret or allow-list) before doing anything else.
- State-changing requests carry the antiforgery token, including the ones htmx sends.
- Authorization is by policy (resource owner, role, active subscription), checked on the server for every handler, never only by hiding a button.

## Error Handling

- An expected business failure is a `Result` with a typed `Error` (stable code plus message), not an exception. Examples: the slot was taken, the payment deadline passed, the transition is not allowed.
- Exceptions are for the unexpected: bugs, broken infrastructure, violated preconditions. Do not use them for control flow.
- A global handler logs unhandled exceptions once and shows a friendly error page. Do not catch an exception only to log and rethrow it.
- No empty `catch`. Catch the narrowest type you can handle, and translate known infrastructure errors (a unique or exclusion violation) into the matching business error.
- Error messages shown to the user are in Spanish and come from resources; error codes and log messages are in English.

## Validation

- Validate the shape of the input at the edge (required, length, format) and the business rules in the domain. A rule is never enforced only in the browser.
- Validate before any write. A handler must not leave a partial change when a later check fails.
- Options classes are validated at start-up (`ValidateOnStart`), so a missing setting stops the application instead of failing at the first request.

## Logging

- Use `ILogger<T>` with message templates and named properties (`"Booking {BookingId} expired"`), never string interpolation.
- Log an exception with its stack trace, once, where it is handled.
- **Never log secrets or personal data**: tokens, API keys, passwords, payment data, identity documents, full request bodies of webhooks or login forms.
- Log every state change of a booking, a payment and a subscription with the ids needed to trace it.

## Data Access

- EF Core with Npgsql. Tables and columns are snake_case. One `IEntityTypeConfiguration` per aggregate; no data annotations on domain types.
- Schema changes go through EF Core migrations, reviewed like code. Deployment applies an idempotent script. Never edit a migration that has been merged; add a new one.
- Data access is asynchronous end to end. No `.Result`, `.Wait()` or `GetAwaiter().GetResult()`.
- Lazy loading is off. Load what a use case needs with `Include` or a projection, and avoid N+1 queries.
- Read-only queries use `AsNoTracking()` and project to DTOs. Lists are paged.
- An invariant that must hold under concurrency is enforced by the database as well (unique index, exclusion constraint, concurrency token), not only by a check in code.
- Seed data for development and staging is fictitious and never runs in production.

## Transactions and Side Effects

- One use case is one unit of work: one `SaveChangesAsync`. Open an explicit transaction only when a use case really needs more than one.
- Keep transactions short. Never call an external provider (payment, email, messaging) while one is open.
- Work that must happen only after the commit (notifications, calls to providers, cache invalidation) goes through the outbox: the domain event is stored in the same transaction and dispatched afterwards (E1-04).
- Build a response that needs to read the database after the transaction block has ended, not after completing it inside the block.
- A handler that an external system may call twice (webhooks, retried jobs) is idempotent.

## Dates and Time Zones

- Store every instant in UTC. Convert to a time zone only at the edges, using the IANA zone of the schedule or the user.
- Never call `DateTime.Now`, `DateTime.UtcNow` or `DateTimeOffset.Now` directly: inject `IClock`, so tests are deterministic (E1-12).
- A time range has an inclusive start and an exclusive end.

## Background Jobs

- A job is a thin wrapper that calls a use case. It takes ids as arguments, never entities.
- Every job is idempotent: running it twice has the effect of running it once.
- Details and the "tick" endpoint are in the `hangfire-jobs` skill.

## Testing

- **TDD**: write the failing test first, then the code that makes it pass, then refactor.
- Put decision logic in units that need no database or network (aggregates, value objects, domain services, pure helpers) and keep handlers thin. Most tests are fast unit tests of the domain.
- Use hand-written fakes for ports (`FakeClock`, an in-memory file storage). Reserve mocking libraries for the rare case a fake cannot cover.
- Integration tests run against a real PostgreSQL in Testcontainers, never against a shared or production database. They cover what only the database can prove: constraints, migrations, queries, concurrency.
- Authorization has its own tests for every sensitive endpoint.
- E2E smoke tests use Playwright on the critical flows: book, pay, confirm, cancel.
- **Coverage is measured on the lines a change adds or modifies**: aim for 90%. Collect it with coverlet (`--collect:"XPlat Code Coverage"`).
- A test that changes shared state restores it. Tests do not depend on each other or on execution order.
- Run the tests yourself and read the output before you say a change works. The test framework is chosen in E1-02; until then these rules are framework-independent.

## Security

- **Secrets never enter the repository.** Local values come from user secrets or environment variables; deployed values from the hosting configuration and GitHub secrets. Provider tokens stored in the database are encrypted.
- Check ownership on every read and write of a resource: a user reaches only their own bookings, receipts and clients.
- Uploaded files: validate the real type by content and the size, rename them, and store them outside `wwwroot`. Serve them only through an authorized endpoint.
- Rate-limit login, registration, contact and webhook endpoints. Send the security headers and keep antiforgery on.
- Do not widen CORS or weaken a security check to make a test pass.

## Performance

- Page every list and index the columns used to filter and sort.
- Keep the app light: the hosting is shared, with limited memory and an app pool that stops without traffic.
- Measure before optimizing, and record what was measured.

## Dependencies

- Check the licence before adding a package. MediatR, AutoMapper and MassTransit have commercial licences and are not used.
- Wrap a new external library behind a port ([base-standards.md](./base-standards.md), dependency agnosticism).
- Prefer what the framework already provides to a new dependency.

## Git Workflow

- `main` is always deployable. Work on `feature/<card>-<short-name>` branches (for example `feature/E4-05-slot-engine`) and merge through a pull request with CI green.
- Commits: `<card> <imperative sentence>` in English, one logical change per commit (for example `E6-02 Add the exclusion constraint that prevents double bookings`).
- Never commit secrets, build output, local settings or anything under `privado/`.
- Before a commit, review which documentation the change affects ([documentation-standards.md](./documentation-standards.md)).
