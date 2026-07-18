# Architecture Standards

## Purpose

This document defines the architectural standards used across all projects in this
repository. Every project must follow these principles to ensure consistency,
maintainability, and scalability across teams and technologies.

---

## Guidelines

### Clean Architecture

Organise every project into concentric layers where outer layers depend on inner
layers — never the reverse:

```
Presentation / API Layer
    ↓
Application Layer
    ↓
Domain Layer
    ↓
Infrastructure Layer  (implements Domain abstractions)
```

- **Domain Layer** — entities, value objects, domain events, domain services, repository interfaces. No external dependencies.
- **Application Layer** — use cases (commands/queries), DTOs, application services, validators, mapping. Depends only on Domain.
- **Infrastructure Layer** — EF Core implementations, external service integrations, repositories. Depends on Domain and Application.
- **Presentation Layer** — controllers/endpoints, API models, middleware. Depends on Application and Configuration.
- **ConfigurationLayer** — Composition Root. Wires all dependencies. References Application, Domain, and Infrastructure.

### Dependency Inversion Principle

- Abstractions (interfaces) are defined in the Domain or Application layer.
- Concrete implementations live in the Infrastructure layer.
- The Composition Root (ConfigurationLayer) wires abstractions to implementations.

### Modern C# Style

Always use the latest stable C# language features.

Prefer:

- Primary constructors
- Extension blocks
- Collection expressions
- Target-typed new
- File-scoped namespaces
- Required members where appropriate

Do not generate legacy C# syntax unless explicitly requested.

### Primary Constructors

Always use primary constructors for services, handlers and ViewModels.

```csharp
public class CreateOrderCommandHandler(
    IOrderRepository orderRepository,
    IUnitOfWork unitOfWork)
    : IAsyncCommandHandler<CreateOrderCommand>
{
    public async Task HandleAsync(CreateOrderCommand command, CancellationToken cancellationToken)
    {
        var order = Order.Create(command.CustomerId, command.Items);
        await orderRepository.AddAsync(order, cancellationToken);
        await unitOfWork.SaveChangesAsync(cancellationToken);
    }
}
```

### CQRS

- Separate read models (queries) from write models (commands).
- Use `IAsyncQueryHandler<TQuery, TResult>` for queries.
- Use `IAsyncCommandHandler<TCommand>` for commands.
- Always use `Kmaraszkiewicz86.SimpleCqrs` — never MediatR.

Documentation:
https://github.com/kmaraszkiewicz86/SimpleCqrs

Follow the documentation and examples from the repository.

### DDD (Domain-Driven Design)

Apply DDD where the domain is complex enough to justify it:

- **Entities** — objects with identity that persists over time.
- **Value Objects** — immutable objects defined by their attributes.
- **Aggregates** — consistency boundaries. All changes go through the Aggregate Root.
- **Domain Events** — capture facts that have occurred in the domain.
- **Domain Services** — logic that does not belong to a single entity.
- **Specifications** — encapsulate query/validation criteria as reusable objects.

---

## .NET Solution Structure

```
src/
├── MyProject.Api                    ← Presentation Layer
│   ├── Endpoints/
│   ├── Extensions/
│   ├── Middlewares/
│   ├── OpenApi/
│   └── Program.cs
├── MyProject.ApplicationLayer
│   ├── Commands/
│   ├── Queries/
│   ├── Handlers/
│   ├── DTOs/
│   ├── Interfaces/
│   ├── Validators/
│   ├── Mappings/
│   ├── Behaviors/
│   └── Exceptions/
├── MyProject.DomainLayer
│   ├── Entities/
│   ├── Aggregates/
│   ├── ValueObjects/
│   ├── DomainEvents/
│   ├── DomainServices/
│   ├── DbRepositories/              ← Repository interfaces
│   ├── DbQueries/
│   ├── Specifications/
│   ├── Enums/
│   └── Exceptions/
├── MyProject.InfrastructureLayer
│   ├── DbContext/
│   ├── DbRepositories/              ← Repository implementations
│   ├── DbQueries/
│   ├── EntityConfigurations/
│   ├── Migrations/
│   ├── Seed/
│   ├── Services/
│   ├── Integrations/
│   ├── Storage/
│   └── Identity/
├── MyProject.ConfigurationLayer
│   └── ServiceCollectionExtensions.cs
└── MyProject.Tests
```

### ConfigurationLayer Rules

- Acts as the Composition Root between the client application and the DDD architecture.
- Responsible only for wiring dependencies together.
- May reference ApplicationLayer, DomainLayer, and InfrastructureLayer.
- Exposes a single extension method: `AddProjectDependencies()`.
- Client applications reference ConfigurationLayer instead of ApplicationLayer or InfrastructureLayer directly.

### Program.cs Rules

- Must remain minimal.
- Never register services directly inside `Program.cs`.
- Call only:

```csharp
builder.Services.AddProjectDependencies(builder.Configuration);
```

---

## Best Practices

- Keep each layer independently testable.
- Avoid circular dependencies between layers.
- Domain entities must not reference infrastructure concerns.
- Use interfaces to decouple implementation details.
- Prefer composition over inheritance.
- Keep aggregates small and focused.
- Raise domain events for side effects that cross aggregate boundaries.
- Use the Specification pattern for reusable query/validation logic.

---

## Bad Practices

- Placing business logic in controllers, endpoints, views, or code-behind.
- Referencing Infrastructure from Domain or Application.
- Fat controllers or fat services.
- Anemic domain models (entities with only properties and no behaviour).
- Using MediatR — always use `Kmaraszkiewicz86.SimpleCqrs`.
- Registering dependencies directly in `Program.cs`.
- Bypassing the ConfigurationLayer from client applications.

---

## Examples

### Command Handler

```csharp
// ApplicationLayer/Commands/CreateOrderCommand.cs
public record CreateOrderCommand(Guid CustomerId, IReadOnlyList<OrderItem> Items);

// ApplicationLayer/Handlers/CreateOrderCommandHandler.cs
public class CreateOrderCommandHandler(
    IOrderRepository orderRepository,
    IUnitOfWork unitOfWork)
    : IAsyncCommandHandler<CreateOrderCommand>
{
    public async Task HandleAsync(CreateOrderCommand command, CancellationToken cancellationToken)
    {
        var order = Order.Create(command.CustomerId, command.Items);
        await orderRepository.AddAsync(order, cancellationToken);
        await unitOfWork.SaveChangesAsync(cancellationToken);
    }
}
```

### Query Handler

```csharp
// ApplicationLayer/Queries/GetOrderByIdQuery.cs
public record GetOrderByIdQuery(Guid OrderId);

// ApplicationLayer/Handlers/GetOrderByIdQueryHandler.cs
public class GetOrderByIdQueryHandler(IOrderQueryRepository queryRepository)
    : IAsyncQueryHandler<GetOrderByIdQuery, OrderDto>
{
    public async Task<OrderDto> HandleAsync(GetOrderByIdQuery query, CancellationToken cancellationToken)
        => await queryRepository.GetByIdAsync(query.OrderId, cancellationToken)
           ?? throw new OrderNotFoundException(query.OrderId);
}
```

---

## Common Mistakes

- Putting queries and commands in the same handler class.
- Injecting `DbContext` directly into Application layer (use repository interfaces).
- Creating God classes that handle multiple unrelated concerns.
- Returning domain entities directly from query handlers instead of DTOs.
- Leaking EF Core types (like `IQueryable`) across layer boundaries.
