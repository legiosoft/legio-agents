# Legio .NET Host Instructions

This folder is the active .NET backend host. Keep backend implementation,
project files, migrations, and backend configuration in this folder unless a
root-level repo setting must change.

## Reference Material

These instructions already encode the intended backend structure. Do not depend
on local reference paths from another machine.

## Top-Level Layout

Follow the Legio-style host layout:

- `Database/`: EF Core persistence layer.
- `Endpoints/`: Minimal API endpoint slices. This replaces MVC controllers.
- `Hosting/`: application startup composition and ASP.NET hosting concerns.
- `Infrastructure/`: reusable runtime implementation and external integrations.
- `Shared/`: cross-cutting contracts, constants, helpers, options, and result
  types.

Do not add a `Controllers/` folder. This backend is organized as Minimal API
endpoint slices; each API operation belongs under `Endpoints/`.

## Endpoint Organization

Organize API code as:

```text
Endpoints/{Surface}/{Feature}/{Operation}/
```

Examples:

```text
Endpoints/CustomerPortal/Billing/CreateBillingProfile/
Endpoints/CloudConsole/Customer/GetCustomers/
Endpoints/Website/Products/CreateFreeTrialProduct/
```

Each operation folder should own the files for one API operation:

- `{Operation}.cs`: endpoint mapper and small HTTP handler.
- `{Operation}Request.cs`: request DTO when the operation has input.
- `{Operation}Response.cs`: response DTO when the operation returns a body.
- `{Operation}RequestValidator.cs`: FluentValidation validator for request DTOs.
- `{Operation}Repository.cs`: operation-specific persistence when needed.
- `{Operation}Handler.cs`: operation-specific orchestration when the endpoint
  needs more than a small handler method.

Keep request and response DTOs separate from database entities. DTOs belong with
the endpoint operation that owns the API contract. Move DTOs to `Shared/` only
when they are genuinely reused across endpoint areas.

## IEndpoint And IFeature

`IEndpoint` is the operation-level contract. An endpoint class should implement
the local `IEndpoint` interface and expose:

```csharp
public static void MapEndpoint(IEndpointRouteBuilder app)
```

Use `MapEndpoint` to map one route and attach route-specific concerns:
authorization, validation filters, tags, summaries, accepted request/response
types, and endpoint metadata. Keep the handler small; delegate persistence to a
repository and larger workflows to a handler or service.

`IFeature` is the feature-level grouping contract when the project defines one.
Use it to group related endpoints under a shared route prefix, tags, filters,
authorization policy, or versioned route group. A feature should not contain the
business logic for individual operations. It should compose endpoint mappings
for the feature and leave each operation in its own `IEndpoint` slice.

Expected relationship:

- `IFeature`: groups a feature such as Billing, Customer, Products, or Webhooks.
- `IEndpoint`: maps a single operation such as CreateBillingProfile or
  GetCustomers.
- Endpoint operation folders stay independent even when a feature groups their
  routes.

When using source-generated or scanning-based registration, keep marker
interfaces such as `IEndpoint`, `IFeature`, `IRepository`, and endpoint handler
interfaces consistent so new slices are discovered without manual registration.

## Database Folder

Organize persistence code inside `Database/` by responsibility:

- `Base/`: base entity classes and shared persistence abstractions, such as
  simple entity base types or audit base types.
- `Configurations/`: EF Core `IEntityTypeConfiguration<T>` classes. Mirror the
  entity domain folders where useful, for example `Configurations/Customers/`.
- `Entities/`: database POCO entity classes only. Group by domain, for example
  `Entities/Customers/`, `Entities/Locations/`, `Entities/Payments/`, and
  `Entities/Storage/`.
- `Enums/`: persistence/domain enums used by entities. Group by domain instead
  of creating one large enum folder.
- `Extensions/`: EF model builder extensions and persistence-only helper
  extensions.
- `Migrations/`: EF Core generated migrations and model snapshot.
- `Seed/`: seed orchestration and seed data lists.
- `BaseDbContext.cs` or the project-specific DbContext file: DbSet properties,
  model configuration entry point, and database context setup.

Do not put API DTOs, endpoint validators, HTTP handlers, or integration clients
inside `Database/`.

Raw SQL should be rare. When SQL is necessary, do not hard-code table or column
names as string literals. Build names from the exact database entity POCO and
properties using `nameof(EntityClass)` and `nameof(EntityClass.Property)` so the
query points back to the entity model.

## Migrations And Database Updates

Never create, remove, or run EF migrations automatically.

Only run `dotnet ef migrations add` when the current user message explicitly
contains the text `Add Migration`. Before doing so, explain the model changes
that will be captured and wait for the user's confirmation/manual review.

Only run `dotnet ef database update` when the current user message explicitly
contains the text `DB Update`. Never run database update as part of a normal
build, refactor, or verification step.

Create and run EF Core migrations only from the active backend project. Do not
reuse or execute archived database scripts directly.

## Hosting Folder

Put startup and ASP.NET composition under `Hosting/`:

- service-registration extension methods
- middleware and web application extension methods
- endpoint scanning/registration helpers
- filters, validation filters, and error response filters
- exception handlers
- configuration mappers and hosting-specific configuration setup

Keep `Program.cs` thin. It should compose configuration, register services, set
up middleware, map endpoint groups, and run the app.

## Infrastructure Folder

Put reusable runtime implementation under `Infrastructure/`:

- background queues and workers
- repositories shared by multiple endpoint slices
- scheduler jobs
- external service clients and integration services
- email, identity, storage, payment, realtime, or similar services
- validators that are not tied to a single endpoint request DTO

Endpoint-specific repositories or handlers may stay inside the endpoint
operation folder. Move them to `Infrastructure/` only when more than one
endpoint slice uses them.

## Shared Folder

Put cross-cutting code under `Shared/`:

- options/configuration classes
- constants
- result and error types
- helper classes
- shared interfaces
- general extension methods
- shared models that are not API DTOs and not EF entities

Do not use `Shared/` as a dumping ground. Prefer endpoint-local code until
there is real reuse.

## General Backend Rules

Follow normal .NET backend conventions: dependency injection, options-based
configuration, typed services, async I/O, and clear API contracts.

## Date, Time, And Time Zones

Use NodaTime for date, time, and time zone handling. Do not introduce
`DateTime`, `DateTimeOffset`, or `TimeSpan` in new domain logic, API contracts,
persistence models, scheduling code, or business rules when a NodaTime type can
represent the value.

Prefer these NodaTime types:

- `Instant`: an exact UTC point in time, such as audit timestamps, event times,
  token expirations, and background job markers.
- `LocalDate`: a calendar date without a time or zone, such as birthdays,
  service dates, and reporting dates.
- `LocalTime`: a wall-clock time without a date or zone, such as daily schedule
  times.
- `LocalDateTime`: a local calendar date and time only when the time zone is
  stored or supplied separately.
- `ZonedDateTime`: a local date and time combined with a real time zone for
  scheduling, conversion, and user-facing calculations.
- `Duration` or `Period`: elapsed time or calendar-based amounts, depending on
  whether the value is exact duration or date/calendar arithmetic.
- `DateTimeZone`: a time zone. Store IANA time zone IDs, not raw offsets or
  local machine time zone assumptions.

Use `IClock` for current time. Do not call `DateTime.Now`, `DateTime.UtcNow`,
`DateTime.Today`, or equivalent local-system clock APIs in application code.

Configure serialization and persistence for NodaTime instead of converting
through `DateTime`:

- For `System.Text.Json`, use `NodaTime.Serialization.SystemTextJson`.
- For PostgreSQL with EF Core/Npgsql, enable the NodaTime mappings such as
  `UseNodaTime()`.
- Keep provider-specific conversions at the infrastructure boundary when a
  database provider or external SDK has no native NodaTime support.

Only use `DateTime`, `DateTimeOffset`, or `TimeSpan` for unavoidable
third-party, framework, or BCL interop. Keep those conversions close to the
boundary, convert immediately to NodaTime types, and do not let BCL temporal
types leak into domain or endpoint contracts.

Prefer simple, readable C# over compact or clever code. Use `if`/`else` when it
is easier to read than a ternary expression. Avoid nested ternaries.

Do not split code into many methods, classes, constants, endpoints, or helper
abstractions just to make it look layered. Add a new method or class only when
it reduces real duplication, makes the operation easier to read, or will be
reused in multiple places.

Do not put many unrelated classes, constants, DTOs, validators, or helpers in
one file. Keep files small and place separate types in the appropriate folder.

Validate inputs at the API boundary with endpoint-local FluentValidation
validators and consistent error response filters.

Keep data access behind explicit services, repositories, or endpoint-local
repositories instead of mixing EF Core queries directly into endpoint handler
bodies.

For simple queries and commands, use EF Core in the repository. Examples:
getting profiles, finding one row, filtering a small list, checking existence,
or updating a profile status.

For complex reporting, multi-join, aggregate-heavy, or performance-sensitive
queries, use SQL through Dapper from the repository. Keep SQL readable and tied
to entity/property names with `nameof(...)` as described in the database rules.

Do not use AutoMapper or mapping layers. Prefer projecting directly from EF Core
or SQL into the response/model shape needed by the endpoint.

Do not commit real connection strings, API keys, patient data, local env files,
or other secrets. Use safe example settings only.

If project files exist, verify backend changes with the local `dotnet build`.
Do not add or depend on a test project unless the user explicitly asks for one.
