# Naming Conventions

## Purpose

This document defines consistent naming conventions for all layers, files, classes,
methods, and variables across all technologies used in this repository. Consistent
naming reduces cognitive load and makes codebases easier to navigate.

---

## Guidelines

### General Rules

- Use descriptive, meaningful names that express intent.
- Avoid abbreviations unless they are universally understood (e.g., `id`, `dto`, `url`).
- Avoid single-letter variables except for loop indexes (`i`, `j`) and LINQ lambdas.
- Do not use type suffixes in names unless they add semantic value.
- Names must be in English.

---

## C# Naming Conventions

| Construct | Convention | Example |
|-----------|-----------|---------|
| Class | `PascalCase` | `OrderService`, `UserRepository` |
| Interface | `IPascalCase` | `IOrderService`, `IUserRepository` |
| Method | `PascalCase` | `GetOrderByIdAsync`, `CreateOrder` |
| Property | `PascalCase` | `OrderId`, `CustomerName` |
| Private field | `_camelCase` | `_orderRepository`, `_logger` |
| Local variable | `camelCase` | `orderDto`, `cancellationToken` |
| Constant | `PascalCase` | `MaxRetryCount`, `DefaultPageSize` |
| Enum type | `PascalCase` | `OrderStatus`, `PaymentMethod` |
| Enum value | `PascalCase` | `OrderStatus.Pending`, `PaymentMethod.Card` |
| Generic type parameter | `TPascalCase` | `TEntity`, `TResult`, `TQuery` |
| Extension method class | `PascalCaseExtensions` | `ServiceCollectionExtensions` |
| Record | `PascalCase` | `CreateOrderCommand`, `OrderDto` |
| Exception class | `PascalCaseException` | `OrderNotFoundException`, `InvalidOrderException` |

### Async Methods

Always suffix async methods with `Async`:

```csharp
// Good
public async Task<OrderDto> GetOrderByIdAsync(Guid id, CancellationToken cancellationToken)

// Bad
public async Task<OrderDto> GetOrderById(Guid id, CancellationToken cancellationToken)
```

---

## Layer-Specific Naming

### Commands and Queries

Commands and queries are records describing intent:

```csharp
// Commands — imperative, describe an action
public record CreateOrderCommand(Guid CustomerId, IReadOnlyList<OrderLineItem> Items);
public record CancelOrderCommand(Guid OrderId, string Reason);
public record UpdateCustomerEmailCommand(Guid CustomerId, string NewEmail);

// Queries — describe what is being requested
public record GetOrderByIdQuery(Guid OrderId);
public record GetOrdersByCustomerQuery(Guid CustomerId, int Page, int PageSize);
public record GetActiveProductsQuery(string? Category);
```

### Handlers

```csharp
// Command handlers
public class CreateOrderCommandHandler : IAsyncCommandHandler<CreateOrderCommand>
public class CancelOrderCommandHandler : IAsyncCommandHandler<CancelOrderCommand>

// Query handlers
public class GetOrderByIdQueryHandler : IAsyncQueryHandler<GetOrderByIdQuery, OrderDto>
public class GetOrdersByCustomerQueryHandler : IAsyncQueryHandler<GetOrdersByCustomerQuery, IReadOnlyList<OrderSummaryDto>>
```

### DTOs

Suffix all data transfer objects with `Dto`.

Prefer immutable classes for DTOs whenever possible.

```csharp
public class OrderDto
{
    public Guid Id { get; init; }

    public Guid CustomerId { get; init; }

    public OrderStatus Status { get; init; }

    public decimal Total { get; init; }
}
```

Use `init` properties to make DTOs immutable where appropriate.

Do not use records for DTOs by default.

Records generate additional functionality such as value equality, which is unnecessary for most DTOs and increases complexity.

Use records only when value-based equality or other record-specific features are explicitly required.

### Repositories

Define interfaces in Domain, implementations in Infrastructure:

```csharp
// DomainLayer/DbRepositories/IOrderRepository.cs
public interface IOrderRepository
{
    Task<Order?> GetByIdAsync(Guid id, CancellationToken cancellationToken);
    Task AddAsync(Order order, CancellationToken cancellationToken);
}

// InfrastructureLayer/DbRepositories/OrderRepository.cs
public class OrderRepository(ApplicationDbContext dbContext)
    : IOrderRepository
```

### Entity Configurations

```csharp
// InfrastructureLayer/EntityConfigurations/OrderConfiguration.cs
public class OrderConfiguration : IEntityTypeConfiguration<Order>
```

### Validators

```csharp
// ApplicationLayer/Validators/CreateOrderCommandValidator.cs
public class CreateOrderCommandValidator : AbstractValidator<CreateOrderCommand>
```

### Exceptions

```csharp
// DomainLayer/Exceptions/OrderNotFoundException.cs
public class OrderNotFoundException : DomainException

// ApplicationLayer/Exceptions/DuplicateOrderException.cs
public class DuplicateOrderException : ApplicationException
```

---

## Angular Naming Conventions

| Construct | Convention | Example |
|-----------|-----------|---------|
| Component class | `PascalCaseComponent` | `OrderListComponent` |
| Component file | `kebab-case.component.ts` | `order-list.component.ts` |
| Service class | `PascalCaseService` | `OrderService` |
| Service file | `kebab-case.service.ts` | `order.service.ts` |
| Pipe class | `PascalCasePipe` | `FormatCurrencyPipe` |
| Guard class | `PascalCaseGuard` | `AuthGuard` |
| Interceptor class | `PascalCaseInterceptor` | `AuthInterceptor` |
| Model/Interface | `PascalCase` | `Order`, `OrderSummary` |
| Module file | `kebab-case.module.ts` | `orders.module.ts` |
| Route const | `SCREAMING_SNAKE_CASE` | `ORDERS_ROUTES` |

---

## React Naming Conventions

| Construct | Convention | Example |
|-----------|-----------|---------|
| Component | `PascalCase` | `OrderList`, `ProductCard` |
| Component file | `PascalCase.tsx` | `OrderList.tsx`, `ProductCard.tsx` |
| Hook | `useCamelCase` | `useOrders`, `useProductDetails` |
| Service file | `camelCase.service.ts` | `orderService.ts` |
| Model/Type | `PascalCase` | `Order`, `ProductSummary` |
| Context | `PascalCaseContext` | `AuthContext`, `CartContext` |
| Enum | `PascalCase` | `OrderStatus` |
| Constant | `SCREAMING_SNAKE_CASE` | `MAX_PAGE_SIZE`, `API_BASE_URL` |

---

## File Naming Conventions

| Technology | Convention | Example |
|-----------|-----------|---------|
| C# class | `PascalCase.cs` | `OrderService.cs` |
| C# test class | `PascalCaseTests.cs` | `OrderServiceTests.cs` |
| C# fixture | `PascalCaseFixture.cs` | `OrderServiceFixture.cs` |
| Angular component | `kebab-case.component.ts` | `order-list.component.ts` |
| Angular template | `kebab-case.component.html` | `order-list.component.html` |
| React component | `PascalCase.tsx` | `OrderList.tsx` |
| React hook | `useCamelCase.ts` | `useOrders.ts` |
| MAUI page | `PascalCase.xaml` | `OrderListPage.xaml` |
| MAUI view model | `PascalCaseViewModel.cs` | `OrderListPageViewModel.cs` |

---

## Database Naming

| Construct | Convention | Example |
|-----------|-----------|---------|
| Table name | `PascalCase` (plural) | `Orders`, `Customers` |
| Column name | `PascalCase` | `OrderId`, `CreatedAt` |
| Primary key | `Id` | `Id` |
| Foreign key | `{Entity}Id` | `CustomerId`, `ProductId` |
| Index | `IX_{Table}_{Column}` | `IX_Orders_CustomerId` |
| Unique constraint | `UQ_{Table}_{Column}` | `UQ_Users_Email` |

---

## Best Practices

- Use domain language (ubiquitous language) in names — align with business terminology.
- Name booleans as questions: `isActive`, `hasPermission`, `canEdit`.
- Name collections in the plural: `orders`, `customers`, `items`.
- Prefer noun phrases for properties and fields, verb phrases for methods.
- Avoid generic names like `Manager`, `Helper`, `Util`, `Service` without context.

---

## Bad Practices

- Using abbreviations like `usr`, `ord`, `mgr`.
- Naming variables `data`, `info`, `obj`, `temp`.
- Mixing naming conventions in the same codebase.
- Using numeric suffixes like `Order2`, `ServiceNew`.
- Hungarian notation like `strName`, `intCount`.

---

## Common Mistakes

- Naming query handlers with `Command` suffix or vice versa.
- Forgetting the `Async` suffix on async methods.
- Using `List` instead of `IReadOnlyList` in public APIs.
- Naming DTOs without the `Dto` suffix when they are data transfer objects.
- Inconsistent use of `Id` vs `ID` (always use `Id` in C#).
