# .NET Standards

## Purpose

This document defines engineering standards for all .NET projects in this
repository, including ASP.NET Core APIs, background services, and class libraries.

---

## Guidelines

### Target Framework

Always target the latest stable .NET version.

### Preferred Technologies

| Category | Technology |
|----------|-----------|
| API framework | ASP.NET Core Minimal APIs |
| ORM | Entity Framework Core (latest stable) |
| CQRS | `Kmaraszkiewicz86.SimpleCqrs` |
| Validation | FluentValidation |
| Mapping | Manual mapping or Mapster |
| Logging | `Microsoft.Extensions.Logging` with structured output |
| Configuration | `IConfiguration` + `IOptions<T>` pattern |
| Authentication | ASP.NET Core Identity + JWT / Azure AD |
| HTTP client | `IHttpClientFactory` with typed clients |

### Install SimpleCqrs

```bash
dotnet add package Kmaraszkiewicz86.SimpleCqrs --version 0.1.4
```

Reference: https://github.com/kmaraszkiewicz86/SimpleCqrs

Always use:

- `IAsyncQueryHandler<TQuery, TResult>` for queries
- `IAsyncCommandHandler<TCommand>` for commands

Never use MediatR.

---

## Project Structure

```
src/
├── MyProject.Api
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
│   ├── DbRepositories/
│   ├── Specifications/
│   ├── Enums/
│   └── Exceptions/
├── MyProject.InfrastructureLayer
│   ├── DbContext/
│   ├── DbRepositories/
│   ├── DbQueries/
│   ├── EntityConfigurations/
│   ├── Migrations/
│   ├── Seed/
│   ├── Services/
│   ├── Integrations/
│   ├── Storage/
│   └── Identity/
├── MyProject.ConfigurationLayer
│   ├── ServiceCollectionExtensions.cs
│   ├── SimpleCqrsConfiguration.cs
│   ├── RepositoryConfiguration.cs
│   ├── InfrastructureConfiguration.cs
│   ├── PersistenceConfiguration.cs
│   └── ValidationConfiguration.cs
└── MyProject.Tests
```

---

## Program.cs

Must remain minimal. Only call:

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddProjectDependencies(builder.Configuration);

var app = builder.Build();

app.UseProjectMiddleware();
app.MapProjectEndpoints();

app.Run();
```

Never register services directly inside `Program.cs`.

---

## ConfigurationLayer

The ConfigurationLayer is the Composition Root. It:

- Registers all dependencies.
- References ApplicationLayer, DomainLayer, and InfrastructureLayer.
- Exposes `AddProjectDependencies()` as the single entry point.

```csharp
// ConfigurationLayer/ServiceCollectionExtensions.cs
namespace MyProject.ConfigurationLayer;

public static class ServiceCollectionExtensions
{
    public static IServiceCollection AddProjectDependencies(
        this IServiceCollection services,
        IConfiguration configuration)
    {
        services
            .AddPersistence(configuration)
            .AddRepositories()
            .AddSimpleCqrs()
            .AddValidators()
            .AddInfrastructureServices(configuration);

        return services;
    }
}
```

---

## Required Extension Classes

Always create these in the `Api/Extensions` folder:

```csharp
// ServiceCollectionExtensions.cs — registers Swagger, auth, CORS, etc.
// WebApplicationExtensions.cs    — configures middleware pipeline
// EndpointRouteBuilderExtensions.cs — maps all endpoint groups
// SwaggerExtensions.cs           — Swagger/OpenAPI configuration
```

---

## Minimal API Endpoints

Group endpoints by feature using `MapGroup`:

```csharp
// Api/Endpoints/OrdersEndpoints.cs
namespace MyProject.Api.Endpoints;

public static class OrdersEndpoints
{
    public static IEndpointRouteBuilder MapOrdersEndpoints(this IEndpointRouteBuilder routes)
    {
        var group = routes.MapGroup("/api/orders")
            .WithTags("Orders")
            .RequireAuthorization();

        group.MapGet("/{id:guid}", GetOrderByIdAsync)
            .WithName("GetOrderById")
            .Produces<OrderDto>()
            .ProducesProblem(StatusCodes.Status404NotFound);

        group.MapPost("/", CreateOrderAsync)
            .WithName("CreateOrder")
            .Accepts<CreateOrderRequestDto>("application/json")
            .Produces<OrderDto>(StatusCodes.Status201Created)
            .ProducesValidationProblem();

        return routes;
    }

    private static async Task<IResult> GetOrderByIdAsync(
        Guid id,
        IAsyncQueryDispatcher dispatcher,
        CancellationToken cancellationToken)
    {
        var result = await dispatcher.DispatchAsync(new GetOrderByIdQuery(id), cancellationToken);
        return Results.Ok(result);
    }

    private static async Task<IResult> CreateOrderAsync(
        CreateOrderRequestDto request,
        IAsyncCommandDispatcher dispatcher,
        CancellationToken cancellationToken)
    {
        await dispatcher.DispatchAsync(new CreateOrderCommand(request.CustomerId, request.Items), cancellationToken);
        return Results.Created();
    }
}
```

---

## Entity Framework Core

Always use the latest stable EF Core.

### Entity Configuration

Every entity must have its own `IEntityTypeConfiguration<TEntity>`:

```csharp
// InfrastructureLayer/EntityConfigurations/OrderConfiguration.cs
namespace MyProject.InfrastructureLayer.EntityConfigurations;

public class OrderConfiguration : IEntityTypeConfiguration<Order>
{
    public void Configure(EntityTypeBuilder<Order> builder)
    {
        builder.ToTable("Orders");

        builder.HasKey(o => o.Id);

        builder.Property(o => o.CustomerId)
            .IsRequired();

        builder.Property(o => o.Status)
            .IsRequired()
            .HasConversion<string>();

        builder.Property(o => o.Total)
            .IsRequired()
            .HasPrecision(18, 2);

        builder.HasMany(o => o.Items)
            .WithOne()
            .HasForeignKey("OrderId")
            .OnDelete(DeleteBehavior.Cascade);

        builder.HasIndex(o => o.CustomerId)
            .HasDatabaseName("IX_Orders_CustomerId");
    }
}
```

### DbContext

Keep it minimal — only DbSets and configuration:

```csharp
// InfrastructureLayer/DbContext/ApplicationDbContext.cs
namespace MyProject.InfrastructureLayer.DbContext;

public class ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
    : DbContext(options)
{
    public DbSet<Order> Orders => Set<Order>();
    public DbSet<Customer> Customers => Set<Customer>();

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.ApplyConfigurationsFromAssembly(typeof(ApplicationDbContext).Assembly);
    }
}
```

### Rules

- Never place entity configuration inside `DbContext`.
- Always register configurations using `ApplyConfigurationsFromAssembly`.
- Prefer Fluent API over Data Annotations.
- Domain entities must remain persistence-agnostic.
- Use `IEntityTypeConfiguration<TEntity>` for every entity.

---

## Modern C# Style

Always prefer:

- **Primary constructors** for dependency injection
- **File-scoped namespaces**
- **Collection expressions** (`[]`)
- **Target-typed new** (`new()`)
- **Records** for commands, queries, and DTOs
- **Required members** where appropriate

```csharp
// Good — modern C#
namespace MyProject.ApplicationLayer.Handlers;

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

---

## Best Practices

- Always pass `CancellationToken` through all async operations.
- Enable Nullable Reference Types in every project.
- Use `IOptions<T>` for strongly typed configuration.
- Use `IHttpClientFactory` for all outbound HTTP calls.
- Use `ILogger<T>` for structured logging.
- Use global usings for common namespaces.
- Apply the Repository pattern — never use `DbContext` directly from Application layer.
- Use FluentValidation for all command/request validation.

---

## Bad Practices

- Using MediatR — always use `Kmaraszkiewicz86.SimpleCqrs`.
- Registering services in `Program.cs`.
- Accessing `DbContext` directly from the Application layer.
- Using Data Annotations for EF Core entity configuration.
- Using `Task.Result` or `Task.Wait()`.
- Returning domain entities from Application layer — always map to DTOs.
- Using static classes for services that have dependencies.

---

## Common Mistakes

- Forgetting to register `SimpleCqrs` handlers in ConfigurationLayer.
- Not calling `ApplyConfigurationsFromAssembly` in `DbContext`.
- Placing entity relationships in domain entities instead of `EntityConfiguration`.
- Not propagating `CancellationToken` from endpoints to repositories.
- Using `async Task` in endpoints instead of the minimal API delegate pattern.

---

## Checklist

- [ ] `Kmaraszkiewicz86.SimpleCqrs` installed and configured
- [ ] `Program.cs` is minimal — only calls `AddProjectDependencies`
- [ ] ConfigurationLayer is the Composition Root
- [ ] Every entity has an `IEntityTypeConfiguration<TEntity>`
- [ ] `ApplyConfigurationsFromAssembly` used in `DbContext`
- [ ] Nullable Reference Types enabled
- [ ] `CancellationToken` propagated through all async calls
- [ ] FluentValidation used for all validation
- [ ] Domain entities are persistence-agnostic
- [ ] Application layer returns DTOs — not domain entities
