# Create Service Prompt

Create a complete, production-ready service class following the standards defined
in this repository.

---

## Instructions

Generate a fully functional service for the feature described below.

Follow all standards defined in:
- `.github/instructions/architecture.md`
- `.github/instructions/dotnet.md`
- `.github/instructions/coding-style.md`
- `.github/instructions/testing.md`
- `.github/instructions/naming.md`

---

## What to generate

### Service Interface

- Define the interface in `ApplicationLayer/Interfaces/`
- Use `Async` suffix on all async methods
- Pass `CancellationToken` on all async methods
- Return DTOs — not domain entities

```csharp
// ApplicationLayer/Interfaces/IOrderService.cs
public interface IOrderService
{
    Task<OrderDto> GetByIdAsync(Guid id, CancellationToken cancellationToken);
    Task<IReadOnlyList<OrderSummaryDto>> GetByCustomerAsync(Guid customerId, CancellationToken cancellationToken);
    Task<OrderDto> CreateAsync(CreateOrderCommand command, CancellationToken cancellationToken);
    Task CancelAsync(Guid id, CancellationToken cancellationToken);
}
```

### Service Implementation

- Implement in `ApplicationLayer/` or `InfrastructureLayer/Services/` as appropriate
- Use primary constructors
- Use file-scoped namespaces
- Inject dependencies through the constructor
- Throw domain-specific exceptions for error conditions
- Never access `DbContext` directly — use repository interfaces

### DI Registration

- Register the service in the appropriate `ConfigurationLayer` file

### Unit Tests

- Feature-specific Fixture inheriting `BaseFixture`
- Tests for:
  - Happy path
  - Not found / empty results
  - Exception propagation
  - Validation errors

---

## Requirements

- All async methods accept and propagate `CancellationToken`.
- Return DTOs — never domain entities.
- Throw specific exceptions — not generic `Exception`.
- No business logic duplication across services.
- Use Dependency Injection for all dependencies.
- Primary constructors for DI.

---

## Service Description

[Describe the service here — e.g., "An OrderService that retrieves, creates, and cancels orders, coordinating between the repository and domain logic."]
