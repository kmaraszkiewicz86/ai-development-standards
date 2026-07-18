# Create API Prompt

Create a complete, production-ready ASP.NET Core API project following the DDD
and Clean Architecture standards defined in this repository.

---

## Instructions

Generate a fully functional ASP.NET Core API for the feature described below.

Follow all standards defined in:
- `.github/instructions/architecture.md`
- `.github/instructions/dotnet.md`
- `.github/instructions/security.md`
- `.github/instructions/testing.md`
- `.github/instructions/naming.md`

---

## What to generate

Generate all layers for the given feature:

### Api Layer

- Endpoint class using Minimal API with `MapGroup`
- `EndpointRouteBuilderExtensions` mapping
- OpenAPI / Swagger metadata (`.Produces<T>()`, `.ProducesProblem()`, etc.)
- Middleware registration if required

### ApplicationLayer

- Command record(s) for write operations
- Query record(s) for read operations
- `IAsyncCommandHandler<TCommand>` implementation(s)
- `IAsyncQueryHandler<TQuery, TResult>` implementation(s)
- DTO record(s)
- FluentValidation validator(s)
- Mapping logic (manual or Mapster)

### DomainLayer

- Entity/Aggregate with domain behaviour
- Value Objects if applicable
- Repository interface(s) in `DbRepositories/`
- Domain exception(s)
- Domain events if applicable

### InfrastructureLayer

- `IEntityTypeConfiguration<TEntity>` for every entity
- Repository implementation(s)
- Query implementation(s) if applicable
- EF Core migration instructions

### ConfigurationLayer

- Register all new dependencies in the appropriate configuration file

### Tests

- `BaseFixture` (if not already present)
- Feature-specific Fixture
- Unit tests for:
  - Command handler(s) — happy path, validation, exception cases
  - Query handler(s) — happy path, not found, exception cases
  - Validator(s) — valid input, each invalid rule

---

## Requirements

- Use `Kmaraszkiewicz86.SimpleCqrs` for CQRS — never MediatR.
- `Program.cs` must remain minimal.
- All secrets from configuration — never hardcoded.
- All inputs validated with FluentValidation.
- All async methods accept and propagate `CancellationToken`.
- Use primary constructors.
- Use file-scoped namespaces.
- Use `IEntityTypeConfiguration<TEntity>` — never Data Annotations.
- Domain entities must be persistence-agnostic.

---

## Feature Description

[Describe the feature here — e.g., "An Order Management API supporting create, get by ID, list by customer, and cancel operations."]
