# Testing Standards

## Purpose

This document defines testing standards for all .NET projects in this repository.
Tests are a first-class citizen — every feature must be covered by meaningful,
maintainable tests.

---

## Guidelines

### Testing Stack (.NET)

Always use:

- **xUnit** — test runner
- **FluentAssertions** — readable, expressive assertions
- **AutoFixture** — automatic test data generation
- **AutoFixture.AutoNSubstitute** — automatic mock creation
- **NSubstitute** — mocking library

Install packages:

```bash
dotnet add package xunit
dotnet add package FluentAssertions
dotnet add package AutoFixture
dotnet add package AutoFixture.AutoNSubstitute
dotnet add package NSubstitute
dotnet add package Microsoft.NET.Test.Sdk
dotnet add package xunit.runner.visualstudio
```

---

## Test Structure

### Arrange / Act / Assert

Every test must follow the Arrange / Act / Assert (AAA) pattern:

```csharp
[Fact]
public async Task HandleAsync_WithValidCommand_CreatesOrder()
{
    // Arrange
    var command = _fixture.CreateOrderCommand();
    var expectedOrder = _fixture.Order();

    _fixture.OrderRepository
        .GetByCustomerIdAsync(command.CustomerId, Arg.Any<CancellationToken>())
        .Returns(Task.FromResult<IReadOnlyList<Order>>([]));

    // Act
    await _sut.HandleAsync(command, CancellationToken.None);

    // Assert
    await _fixture.OrderRepository
        .Received(1)
        .AddAsync(Arg.Is<Order>(o => o.CustomerId == command.CustomerId), Arg.Any<CancellationToken>());
}
```

### Test Naming

Name tests using the pattern: `MethodName_Condition_ExpectedResult`

```csharp
// Good
HandleAsync_WithValidCommand_CreatesOrder
HandleAsync_WithNullCustomerId_ThrowsArgumentException
HandleAsync_WhenRepositoryThrows_PropagatesException
GetByIdAsync_WithExistingId_ReturnsOrderDto
GetByIdAsync_WithUnknownId_ThrowsOrderNotFoundException
```

---

## Fixture Pattern

Every test class must have its own Fixture that inherits from `BaseFixture`.

### BaseFixture

```csharp
// Tests/Fixtures/BaseFixture.cs
public abstract class BaseFixture
{
    protected IFixture AutoFixture { get; }

    protected BaseFixture()
    {
        AutoFixture = new Fixture()
            .Customize(new AutoNSubstituteCustomization { ConfigureMembers = true });
    }

    protected T Create<T>() => AutoFixture.Create<T>();
    protected IReadOnlyList<T> CreateMany<T>(int count = 3) => AutoFixture.CreateMany<T>(count).ToList();
}
```

### Feature Fixture

```csharp
// Tests/Fixtures/CreateOrderCommandFixture.cs
public class CreateOrderCommandFixture : BaseFixture
{
    public IOrderRepository OrderRepository { get; }
    public IUnitOfWork UnitOfWork { get; }
    public CreateOrderCommandHandler Sut { get; }

    public CreateOrderCommandFixture()
    {
        OrderRepository = Substitute.For<IOrderRepository>();
        UnitOfWork = Substitute.For<IUnitOfWork>();
        Sut = new CreateOrderCommandHandler(OrderRepository, UnitOfWork);
    }

    public CreateOrderCommand ValidCommand() =>
        Create<CreateOrderCommand>();
}
```

### Test Class

```csharp
public class CreateOrderCommandHandlerTests : IClassFixture<CreateOrderCommandFixture>
{
    private readonly CreateOrderCommandFixture _fixture;

    public CreateOrderCommandHandlerTests(CreateOrderCommandFixture fixture)
    {
        _fixture = fixture;
    }

    [Fact]
    public async Task HandleAsync_WithValidCommand_CallsRepositoryAddAsync()
    {
        // Arrange
        var command = _fixture.ValidCommand();

        // Act
        await _fixture.Sut.HandleAsync(command, CancellationToken.None);

        // Assert
        await _fixture.OrderRepository
            .Received(1)
            .AddAsync(Arg.Any<Order>(), Arg.Any<CancellationToken>());
    }
}
```

---

## Test Folder Structure

```
MyProject.Tests/
├── Fixtures/
│   ├── BaseFixture.cs
│   ├── CreateOrderCommandFixture.cs
│   ├── GetOrderByIdQueryFixture.cs
│   └── OrderValidatorFixture.cs
├── Handlers/
│   ├── CreateOrderCommandHandlerTests.cs
│   └── GetOrderByIdQueryHandlerTests.cs
├── Validators/
│   └── CreateOrderCommandValidatorTests.cs
└── Domain/
    └── OrderTests.cs
```

---

## What to Test

Create tests for:

### Happy Path

- The expected outcome for valid, complete input.
- Verify the correct results are returned.
- Verify the correct dependencies are called.

### Validation

- Missing required fields.
- Out-of-range values.
- Invalid formats.
- Business rule violations.

### Exceptions

- Domain exceptions thrown for invalid state.
- Repository failures propagated correctly.
- Not-found scenarios returning the correct exception.

### Edge Cases

- Empty collections.
- Boundary values (minimum and maximum).
- Concurrent or race conditions where relevant.
- Null input where applicable.

---

## Mocking Guidelines

- Mock only **external dependencies** (repositories, services, external APIs).
- Do not mock entities, value objects, or domain logic — test those directly.
- Do not mock the system under test.
- Use NSubstitute for mock creation.
- Verify interactions (`Received()`) when the side effect matters.
- Do not over-verify — only verify what is important to the test.

---

## FluentAssertions Examples

```csharp
// Object equality
result.Should().BeEquivalentTo(expected);

// Null checks
result.Should().NotBeNull();

// Exception assertion
var act = async () => await _sut.HandleAsync(invalidCommand, CancellationToken.None);
await act.Should().ThrowAsync<ArgumentException>()
    .WithMessage("*Customer ID*");

// Collection assertions
results.Should().HaveCount(3);
results.Should().Contain(x => x.Status == OrderStatus.Pending);
results.Should().BeInAscendingOrder(x => x.CreatedAt);

// Numeric assertions
order.Total.Should().BeGreaterThan(0);
order.Total.Should().BeApproximately(99.99m, 0.01m);
```

---

## Best Practices

- Each test must verify one behaviour — not multiple unrelated things.
- Tests must be independent — no shared mutable state between tests.
- Tests must be deterministic — no reliance on random data, time, or order.
- Tests must be fast — avoid real database calls, file I/O, or network calls.
- Name tests so that failures are self-explanatory.
- Keep test setup minimal — only prepare what the test needs.

---

## Bad Practices

- Using `Thread.Sleep` or `Task.Delay` in tests.
- Asserting on unrelated behaviour.
- Using `try/catch` inside tests to handle expected exceptions — use FluentAssertions.
- Testing private methods directly.
- Writing tests that always pass regardless of the code under test.
- Sharing state between tests using static fields.
- Not cleaning up after integration tests.

---

## Common Mistakes

- Using `Arg.Any<CancellationToken>()` everywhere but forgetting to verify important arguments.
- Not calling `await` on async assertions in FluentAssertions.
- Using `IClassFixture` when a per-test fixture is needed (use constructor instead).
- Asserting `result != null` instead of `result.Should().NotBeNull()`.
- Not testing the validator — only testing the handler.
