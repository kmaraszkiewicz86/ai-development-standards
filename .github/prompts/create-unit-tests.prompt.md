# Create Unit Tests Prompt

Create complete, production-ready unit tests for the class or feature described
below, following the testing standards defined in this repository.

---

## Instructions

Generate comprehensive unit tests following the standards in:
- `.github/instructions/testing.md`
- `.github/instructions/naming.md`

---

## What to generate

### BaseFixture (if not already present)

```csharp
// Tests/Fixtures/BaseFixture.cs
namespace MyProject.Tests.Fixtures;

public abstract class BaseFixture
{
    protected IFixture AutoFixture { get; }

    protected BaseFixture()
    {
        AutoFixture = new Fixture()
            .Customize(new AutoNSubstituteCustomization { ConfigureMembers = true });
    }

    protected T Create<T>() => AutoFixture.Create<T>();
    protected IReadOnlyList<T> CreateMany<T>(int count = 3) =>
        AutoFixture.CreateMany<T>(count).ToList().AsReadOnly();
}
```

### Feature Fixture

- One fixture per test class, inheriting `BaseFixture`
- Contains mocks for all external dependencies
- Constructs the system under test (SUT)
- Provides factory methods for test data

```csharp
// Tests/Fixtures/CreateOrderCommandFixture.cs
namespace MyProject.Tests.Fixtures;

public sealed class CreateOrderCommandFixture : BaseFixture
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

    public CreateOrderCommand ValidCommand() => Create<CreateOrderCommand>();

    public CreateOrderCommand CommandWithEmptyItems() =>
        new CreateOrderCommand(Guid.NewGuid(), []);
}
```

### Test Class

Cover all of the following:

#### Happy Path Tests

- Expected result returned for valid input
- Correct dependencies called with expected arguments
- Correct return value / side effects

#### Validation Tests

- Missing required fields
- Out-of-range values
- Business rule violations

#### Exception Tests

- Correct exception type thrown for invalid state
- Repository failure propagated correctly
- Not-found scenarios return the correct exception

#### Edge Case Tests

- Empty collections
- Boundary values
- Concurrent scenarios where applicable

---

## Test Naming Convention

Use the pattern: `MethodName_Condition_ExpectedResult`

```csharp
// Examples
HandleAsync_WithValidCommand_CallsRepositoryAddAsync
HandleAsync_WithEmptyItems_ThrowsValidationException
HandleAsync_WhenRepositoryThrows_PropagatesException
GetByIdAsync_WithExistingId_ReturnsOrderDto
GetByIdAsync_WithUnknownId_ThrowsOrderNotFoundException
```

---

## Requirements

- Use xUnit, FluentAssertions, AutoFixture, AutoNSubstitute, NSubstitute.
- Follow Arrange / Act / Assert.
- Each test verifies one behaviour.
- Use `await act.Should().ThrowAsync<TException>()` for exception assertions.
- Verify interactions with `Received(n)` — only when the interaction matters.
- No shared mutable state between tests.
- No `Thread.Sleep` or `Task.Delay`.
- No `.Result` or `.Wait()`.

---

## Class or Feature to Test

[Describe the class or feature to test here — e.g., "CreateOrderCommandHandler — a CQRS command handler that creates a new order and saves it via the repository."]
