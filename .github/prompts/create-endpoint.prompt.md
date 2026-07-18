# Create Endpoint Prompt

Create a complete, production-ready ASP.NET Core Minimal API endpoint following
the standards defined in this repository.

---

## Instructions

Generate a fully functional Minimal API endpoint for the operation described below.

Follow all standards defined in:
- `.github/instructions/dotnet.md`
- `.github/instructions/security.md`
- `.github/instructions/naming.md`

---

## What to generate

### Endpoint Class

- Create a static endpoint class in `Api/Endpoints/`
- Use `MapGroup` to group related endpoints
- Apply appropriate `WithTags`, `WithName`, and response metadata
- Use `RequireAuthorization()` unless the endpoint is explicitly public

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

        group.MapGet("/{id:guid}", GetByIdAsync)
            .WithName("GetOrderById")
            .Produces<OrderDto>()
            .ProducesProblem(StatusCodes.Status404NotFound);

        group.MapPost("/", CreateAsync)
            .WithName("CreateOrder")
            .Accepts<CreateOrderRequestDto>("application/json")
            .Produces<OrderDto>(StatusCodes.Status201Created)
            .ProducesValidationProblem();

        return routes;
    }

    private static async Task<IResult> GetByIdAsync(
        Guid id,
        IAsyncQueryDispatcher dispatcher,
        CancellationToken cancellationToken)
    {
        var result = await dispatcher.DispatchAsync(new GetOrderByIdQuery(id), cancellationToken);
        return Results.Ok(result);
    }

    private static async Task<IResult> CreateAsync(
        CreateOrderRequestDto request,
        IAsyncCommandDispatcher dispatcher,
        CancellationToken cancellationToken)
    {
        await dispatcher.DispatchAsync(
            new CreateOrderCommand(request.CustomerId, request.Items),
            cancellationToken);
        return Results.Created();
    }
}
```

### Registration

- Register the endpoint group in `Api/Extensions/EndpointRouteBuilderExtensions.cs`

### Request / Response DTOs

- Generate request and response DTOs as records in `ApplicationLayer/DTOs/`

### OpenAPI Metadata

- Add XML summary comments for Swagger documentation on each endpoint handler
- Configure `.WithOpenApi()` if needed

---

## Requirements

- Always use `CancellationToken` in endpoint handlers.
- Dispatch through `IAsyncCommandDispatcher` or `IAsyncQueryDispatcher`.
- Return `IResult` using `Results.*` methods.
- Use `RequireAuthorization()` unless explicitly public.
- Add correct `.Produces<T>()` and `.ProducesProblem()` metadata.
- Use `MapGroup` — do not map individual routes in `Program.cs`.

---

## Endpoint Description

[Describe the endpoint here — e.g., "A GET endpoint to retrieve an order by ID, returning 200 with the order DTO or 404 if not found. Requires authentication."]
