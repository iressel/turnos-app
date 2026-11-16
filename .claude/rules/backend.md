---
paths:
  - "**/*.cs"
  - "**/*.csproj"
  - "**/*.props"
  - "**/*.slnx"
  - "**/appsettings*.json"
---

# Backend files

Before you edit or review these files, read `docs/backend-standards.md` and follow it. Load the `dotnet-developer` skill for implementation work.

Load the matching skill as well when the change touches its area:

- EF Core configurations, migrations or queries: `postgres-efcore`.
- Mercado Pago code: `mercadopago-integration`.
- Hangfire jobs or the tick endpoint: `hangfire-jobs`.

The rules most often missed: one type per file, file-scoped namespaces, collection expressions on the lines you write, no reformatting of lines the change does not need, and a failing test before the code.
