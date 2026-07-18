# Coding Style Standards

## Purpose

This document defines general coding style conventions that apply across all
languages and frameworks in this repository. Technology-specific styles are
described in their respective instruction files.

---

## Guidelines

### General Principles

- Prefer readability over cleverness.
- Write code for the next developer, not for the compiler.
- One class per file, one responsibility per class.
- Keep methods short — aim for fewer than 20 lines per method.
- Limit cyclomatic complexity — prefer early returns and guard clauses.
- Avoid deep nesting; flatten logic using guard clauses or extraction.
- Remove dead code — do not comment out code, delete it.
- Avoid magic numbers and magic strings — use named constants or enumerations.

### C# Coding Style

Use the latest stable C# language features:

- **File-scoped namespaces** — preferred over block-scoped namespaces.
- **Primary constructors** — preferred for dependency injection.
- **Collection expressions** — use `[]` syntax where appropriate.
- **Target-typed new** — use `new()` when the type is clear from context.
- **Required members** — use `required` keyword for mandatory initialisation.
- **Record types** — use for immutable DTOs, commands, queries, and value objects.

```csharp
// Good — file-scoped namespace, primary constructor
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

```csharp
// Bad — traditional constructor, block-scoped namespace
namespace MyProject.ApplicationLayer.Handlers
{
    public class CreateOrderCommandHandler
    {
        private readonly IOrderRepository _orderRepository;
        private readonly IUnitOfWork _unitOfWork;

        public CreateOrderCommandHandler(IOrderRepository orderRepository, IUnitOfWork unitOfWork)
        {
            _orderRepository = orderRepository;
            _unitOfWork = unitOfWork;
        }
    }
}
```

### Async / Await

- Always use `async/await` — never `.Result` or `.Wait()`.
- Always propagate `CancellationToken` through every async call chain.
- Name async methods with the `Async` suffix.
- Avoid `async void` — use `async Task` instead.

```csharp
// Good
public async Task<OrderDto> GetOrderAsync(Guid id, CancellationToken cancellationToken)
    => await _repository.GetByIdAsync(id, cancellationToken);

// Bad
public OrderDto GetOrder(Guid id)
    => _repository.GetByIdAsync(id).Result;
```

### Null Handling

- Enable Nullable Reference Types (`<Nullable>enable</Nullable>`).
- Use null-conditional operators (`?.`) and null-coalescing operators (`??`).
- Use guard clauses to fail fast on null inputs.
- Prefer returning `null` over throwing when absence is a valid state.
- Throw specific exceptions — not `NullReferenceException` or bare `Exception`.

```csharp
// Good
public async Task<OrderDto> GetOrderAsync(Guid id, CancellationToken cancellationToken)
{
    var order = await _repository.GetByIdAsync(id, cancellationToken);
    return order ?? throw new OrderNotFoundException(id);
}

// Bad
public async Task<OrderDto> GetOrderAsync(Guid id, CancellationToken cancellationToken)
{
    var order = await _repository.GetByIdAsync(id, cancellationToken);
    if (order == null)
        throw new Exception("Not found");
    return order;
}
```

### Error Handling

- Use domain-specific exceptions rather than generic ones.
- Do not swallow exceptions with empty `catch` blocks.
- Log exceptions at the appropriate level — not everywhere.
- Use middleware or global error handlers in APIs for consistent error responses.
- Never expose stack traces to external clients.

### Comments

- Write self-documenting code — names should explain intent.
- Only add comments when the *why* is not obvious from the code.
- Do not add comments that simply restate what the code does.
- Use XML documentation (`///`) for public APIs.
- Remove all TODO comments before committing — implement or remove.

---

## Best Practices

- Use `var` when the type is obvious from the right-hand side; avoid it when ambiguous.
- Use `readonly` for fields that are set only in the constructor.
- Prefer immutable types where possible.
- Avoid side effects in property getters.
- Use `using` declarations or `using` blocks for disposable resources.
- Follow the Single Level of Abstraction Principle (SLAP) in methods.

---

## Bad Practices

- Using magic strings or magic numbers directly in code.
- Returning `null` from methods that should always return a value.
- Catching `Exception` without specific intent.
- Writing methods longer than 50 lines.
- Nesting more than 3 levels deep without extraction.
- Using `#region` blocks to hide large amounts of code.
- Commenting out old code instead of deleting it.

---

## Examples

### Guard Clause Pattern

```csharp
// Good — guard clauses, early return
public void ProcessOrder(Order? order)
{
    if (order is null)
        throw new ArgumentNullException(nameof(order));

    if (!order.IsValid())
        throw new InvalidOrderException(order.Id);

    // Main logic here
    _orderProcessor.Process(order);
}

// Bad — nested conditions
public void ProcessOrder(Order? order)
{
    if (order != null)
    {
        if (order.IsValid())
        {
            _orderProcessor.Process(order);
        }
    }
}
```

### Constants

```csharp
// Good
public static class ApiRoutes
{
    public const string Orders = "/api/orders";
    public const string OrderById = "/api/orders/{id}";
}

// Bad
app.MapGet("/api/orders/{id}", ...);
```

---

## Common Mistakes

- Mixing abstraction levels inside a single method.
- Writing logic inside constructors.
- Using `static` classes for dependency-injected services.
- Returning `IEnumerable` when `IReadOnlyList` is more appropriate.
- Calling async methods from synchronous contexts.

---

## Checklist

- [ ] No magic numbers or strings
- [ ] All async methods pass `CancellationToken`
- [ ] Nullable Reference Types enabled
- [ ] No `.Result` or `.Wait()` calls
- [ ] Methods are short and focused
- [ ] Guard clauses used instead of deep nesting
- [ ] No commented-out code
- [ ] No TODO placeholders
- [ ] Self-documenting names used throughout
- [ ] Primary constructors used for dependency injection
