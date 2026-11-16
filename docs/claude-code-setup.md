# Claude Code Setup

Everyone on the team uses the same Claude Code configuration. It is committed in this repository, so cloning it gives you the instructions, the rules and the project skills. The two plugins need one installation step per machine.

## What the repository provides

| Path | What it is |
|---|---|
| `CLAUDE.md` | Project context loaded in every session. It imports `docs/base-standards.md` |
| `docs/base-standards.md` | Principles for every change |
| `docs/backend-standards.md` | C#, EF Core, testing, security and git standards |
| `docs/documentation-standards.md` | How documentation and these AI specs are written and updated |
| `.claude/rules/` | Short rules loaded when Claude works on matching files (C#, Razor, documentation) |
| `.claude/skills/` | Project skills: four roles and four technology guides |
| `.claude/settings.json` | Shared settings: the plugins to enable and the files Claude must not read |

Personal preferences go in `.claude/settings.local.json`, which is not committed.

## One-time installation

1. Install [Claude Code](https://code.claude.com/docs) and the .NET 10 SDK.
2. Clone the repository, start `claude` in its root and accept the trust prompt. The shared settings apply only after you trust the folder.
3. Install the two plugins the project enables:

   ```bash
   claude plugin marketplace add codewithmukesh/dotnet-claude-kit
   claude plugin install dotnet-claude-kit@dotnet-claude-kit --scope project
   claude plugin install superpowers@claude-plugins-official --scope project
   ```

4. Install the Roslyn tool that `dotnet-claude-kit` uses for code navigation:

   ```bash
   dotnet tool install -g CWM.RoslynNavigator
   ```

5. Restart Claude Code. Run `/plugin` to confirm both plugins are installed with no errors, and type `/` to see the project skills (`software-architect`, `qa-engineer` and the rest).

## Project skills

| Skill | Role or technology |
|---|---|
| `software-architect` | Senior software architect: design decisions, bounded contexts, ADRs, design reviews |
| `functional-analyst` | Senior functional analyst: user stories, acceptance criteria, business rules |
| `qa-engineer` | Senior QA engineer: test strategy, test design, exploratory and E2E testing, bug reports |
| `dotnet-developer` | Senior .NET developer: implementation method, definition of done, code review |
| `razor-htmx-alpine` | Razor Pages, htmx, Alpine.js, Bootstrap 5, design tokens, accessibility |
| `postgres-efcore` | PostgreSQL with EF Core and Npgsql: schema, migrations, constraints, queries |
| `mercadopago-integration` | OAuth, Checkout Pro, webhooks, reconciliation, refunds, subscriptions |
| `hangfire-jobs` | Background and scheduled jobs, and the tick endpoint for shared hosting |

Claude loads a skill when the task matches its description. You can also call one directly, for example `/functional-analyst E4-03`.

## Enabled plugins

| Plugin | Why | Source |
|---|---|---|
| `dotnet-claude-kit` | Built for .NET 10 and C# 14: skills for DDD, clean architecture, EF Core, minimal APIs, authentication, error handling, testing, Serilog and CI/CD; specialist agents (architect, EF Core, test, security, code review); Roslyn navigation tools. MIT | [codewithmukesh/dotnet-claude-kit](https://github.com/codewithmukesh/dotnet-claude-kit) |
| `superpowers` | Engineering discipline independent of the stack: test-driven development, systematic debugging, verification before completion, git worktrees, parallel agents. MIT | [obra/superpowers](https://github.com/obra/superpowers), official Claude plugin marketplace |

The project's standards prevail over a plugin's defaults (`CLAUDE.md`, "Precedence"). Two examples: the plugins default to Minimal APIs, and this project uses Razor Pages; they may suggest MediatR, which this project does not use.

## Evaluated and not enabled

These were reviewed in November 2026. They are not enabled, to keep one source of guidance per topic and a small skill list. Install one for yourself (`--scope user`) if you want to try it, and propose it to the team if it earns its place.

| Candidate | What it offers | Why it is not enabled |
|---|---|---|
| [dotnet/skills](https://github.com/dotnet/skills) (`dotnet-data`, `dotnet-test`, `dotnet-aspnetcore`, `dotnet-diag` at `dotnet-agent-skills`) | The .NET team's own skills: data access, testing, ASP.NET Core, diagnostics, upgrades | Overlaps `dotnet-claude-kit`. `dotnet-diag` is the first one to add when performance work starts (stage E12) |
| [Aaronontheweb/dotnet-skills](https://github.com/Aaronontheweb/dotnet-skills) | EF Core patterns, Testcontainers, snapshot testing, concurrency | Good on testing, but much of it targets Akka.NET, Aspire and Blazor, which this project does not use |
| [supabase/agent-skills](https://github.com/supabase/agent-skills) (`postgres-best-practices@supabase-agent-skills`) | PostgreSQL performance guidance | Useful for query tuning; part of it is specific to Supabase. `postgres-efcore` covers this project's needs |
| [alirezarezvani/claude-skills](https://github.com/alirezarezvani/claude-skills) (`senior-architect`, engineering and product bundles) | Generic role skills | Large generic bundles. The four role skills in this repository are written for this domain and stack |
| Community htmx skills (for example [damusix/skills](https://github.com/damusix/skills)) | htmx reference | None covers Razor Pages, and none was found for Alpine.js. `razor-htmx-alpine` covers the combination |

## Optional: Playwright MCP

The QA skill can drive a browser when the Playwright MCP server is available. It is not part of the shared configuration. To add it for yourself:

```bash
claude mcp add playwright -- npx -y @playwright/mcp@latest
```

If `npx` does not start on native Windows, wrap it: `claude mcp add playwright -- cmd /c npx -y @playwright/mcp@latest`.

## Changing this configuration

A change to a standard, a rule, a skill or the enabled plugins affects the whole team. Propose it in a pull request, following `docs/documentation-standards.md`, and update this document and the skills table of `CLAUDE.md` in the same change.
