# Turnos App

Responsive web platform where independent professionals (coaching, tarot, astrology, reiki and others) manage schedules, offerings and payments, and their clients book and pay online.

**Status: planning.** There is no code yet. Stages and tasks are in [TAREAS.md](TAREAS.md) (stages E0 to E12, one card per `### Exx-yy` heading). Update this file when a stage changes what is described here.

## Language

- **Talk to the user in Spanish.** Planning artifacts written for the user (plans, task lists, analyses, wireframes notes) are also in Spanish.
- **English for every technical artifact**: code, comments, test names, commit messages, technical documentation, configuration, database schema, log and exception messages.
- **Spanish (es-AR) for text an end user reads**: page copy, validation messages shown in the UI, emails and notifications. Keep it in resources or templates, never in domain code.
- `TAREAS.md` and `privado/` stay in Spanish.
- **Ubiquitous language.** The domain is discussed in Spanish and coded in English with one fixed name per concept: Reserva = `Booking`, Agenda = `Schedule`, Servicio/Sesión = `Offering`, Consultante = `Client`, Vacaciones = `TimeOff`. Never introduce a second English name for a concept; extend the glossary first (card E0-02).

## Stack

| Concern | Decision |
|---|---|
| Backend | .NET 10, C#, DDD in layers, modular monolith |
| Web | ASP.NET Core Razor Pages + htmx + Alpine.js, Bootstrap 5, mobile-first, no jQuery. One deployable |
| API | Minimal APIs only for webhooks (Mercado Pago, Telegram) and htmx fragments |
| Database | PostgreSQL with EF Core + Npgsql |
| Jobs | Hangfire with PostgreSQL storage, plus a scheduled "tick" endpoint |
| Payments | Mercado Pago: Checkout Pro with OAuth, Subscriptions, and bank transfer with receipt upload |
| Notifications | Email (Brevo, behind our own interface), Telegram bot, WhatsApp link |
| Hosting | SmarterASP.NET (shared hosting), GitHub Actions for CI/CD |
| Tests | Unit tests per layer, Testcontainers (PostgreSQL) for integration, Playwright for E2E |

## Architecture

```text
Turnos.Web             Razor Pages, htmx fragments, Identity, Minimal APIs (webhooks)
Turnos.Infrastructure  EF Core/Npgsql, repositories, Mercado Pago, email, Telegram, Hangfire, file storage
Turnos.Application     use cases (commands and queries) and ports (IClock, IFileStorage, INotificationChannel...)
Turnos.Domain          aggregates, value objects, domain events, business rules
```

- Dependencies point inwards: `Domain` references no other project, and `Application` references only `Domain`. An architecture test enforces it (card E1-02).
- Bounded contexts are modules inside each layer: Identity, Catalog, Scheduling, Bookings, Payments, Subscriptions, Notifications, Backoffice.
- Business rules live in `Domain`. Page models and endpoints only map requests, call a use case and render the result.

## Standards

@docs/base-standards.md

Read the matching document before you write or review anything in its area:

| Area | Document |
|---|---|
| C#, EF Core, tests, security, git | [docs/backend-standards.md](docs/backend-standards.md) |
| Any documentation, including these standards | [docs/documentation-standards.md](docs/documentation-standards.md) |
| Razor Pages, htmx, Alpine.js, CSS | skill `razor-htmx-alpine` |

## Skills and plugins

Project skills live in `.claude/skills/`. Load the one that matches the task before you start:

| Skill | Use it for |
|---|---|
| `software-architect` | Architecture and design decisions, bounded contexts, ADRs, design reviews |
| `functional-analyst` | Requirements, user stories, acceptance criteria, business rules, state machines |
| `qa-engineer` | Test strategy, test cases, exploratory and E2E testing, bug reports |
| `dotnet-developer` | Implementing and reviewing C# code in this solution |
| `razor-htmx-alpine` | Pages, partials, htmx interactions, Alpine.js components, styles |
| `postgres-efcore` | Schema, migrations, queries, constraints, concurrency |
| `mercadopago-integration` | OAuth, Checkout Pro, webhooks, refunds, subscriptions |
| `hangfire-jobs` | Background and scheduled jobs |

Two plugins are enabled in `.claude/settings.json`: `dotnet-claude-kit` (.NET 10 knowledge, Roslyn tools and specialist agents) and `superpowers` (TDD, systematic debugging, verification before completion). Installation steps are in [docs/claude-code-setup.md](docs/claude-code-setup.md).

**Precedence when sources disagree:** the code, then `docs/*-standards.md`, then the project skills, then the plugin skills. A plugin describes good general practice; this project's decisions win. For example, the plugins default to Minimal APIs and may suggest MediatR, while this project uses Razor Pages and its own handlers.

## Rules that are easy to break

- `privado/` is git-ignored and private. Never stage it, and never copy or quote its content into a tracked file.
- This repository is public. Never commit secrets, tokens, real customer data, or internal material from other organizations.
- Do not add a NuGet package before checking its license. MediatR, AutoMapper and MassTransit are commercial now: use our own handlers and manual mapping.
- Use only fictitious data and Mercado Pago test users and test cards in development, tests and demos.

## Commands

The solution does not exist yet. When card E1-02 creates it, replace this section with the real build, test, run and migration commands.
