# Improve Performance Prompt

Analyse and improve the performance of the code provided below, following the
standards defined in this repository.

---

## Instructions

Identify performance issues and provide optimised solutions for the code below.

Consider standards from:
- `.github/instructions/dotnet.md`
- `.github/instructions/architecture.md`
- `.github/instructions/coding-style.md`

---

## Analysis Areas

Evaluate performance across all of the following:

### Database / EF Core

- **N+1 queries** — are related entities loaded with `Include()` instead of queried in a loop?
- **Unbounded queries** — are queries filtered, sorted, and paginated before executing?
- **Select projection** — does the query select only needed columns using `.Select()`?
- **Change tracking** — use `.AsNoTracking()` for read-only queries.
- **Raw SQL** — are raw queries used where LINQ produces inefficient SQL?
- **Indexes** — are queries filtering on indexed columns?

```csharp
// Bad — N+1 query
foreach (var order in orders)
{
    var customer = await dbContext.Customers.FindAsync(order.CustomerId);
}

// Good — eager loading
var orders = await dbContext.Orders
    .Include(o => o.Customer)
    .AsNoTracking()
    .ToListAsync(cancellationToken);
```

### Memory / Allocation

- **Large object allocations** — are large collections created unnecessarily?
- **String concatenation** — use `StringBuilder` or string interpolation, not `+=` in loops.
- **LINQ materialisation** — avoid calling `.ToList()` prematurely in query chains.
- **Unnecessary boxing** — avoid boxing value types in hot paths.
- **Pooling** — use `ArrayPool<T>` or `MemoryPool<T>` for large, short-lived buffers.

### Async / Concurrency

- **Sequential async** — are independent async operations run sequentially when they could be parallelised with `Task.WhenAll()`?
- **Unnecessary awaits** — are methods awaiting a single `Task.FromResult()` that could return directly?
- **ConfigureAwait** — use `ConfigureAwait(false)` in library code.
- **Blocking async** — eliminate any `.Result` or `.Wait()`.

```csharp
// Bad — sequential independent calls
var orders = await orderService.GetOrdersAsync(cancellationToken);
var customers = await customerService.GetCustomersAsync(cancellationToken);

// Good — parallel independent calls
var (orders, customers) = await (
    orderService.GetOrdersAsync(cancellationToken),
    customerService.GetCustomersAsync(cancellationToken)
).WhenBoth();
```

### Caching

- Are frequently accessed, rarely changing results cached?
- Is `IMemoryCache` or `IDistributedCache` used appropriately?
- Are cache expiry policies configured?

### HTTP / API

- Are response payloads large? Consider pagination or partial responses.
- Are compression middleware (`UseResponseCompression`) applied?
- Are HTTP clients reused via `IHttpClientFactory`?

### Angular / React

- Is `ChangeDetectionStrategy.OnPush` used in Angular?
- Are React lists using stable keys?
- Are expensive computations memoised (`useMemo`, `computed`)?
- Are large lists virtualised?

---

## Output Format

For each performance issue found:

1. **Issue** — describe the problem and its impact.
2. **Location** — file and line number or method name.
3. **Root cause** — why it causes a performance problem.
4. **Fix** — the improved code with explanation.
5. **Expected improvement** — estimated gain (e.g., reduces N+1 to 1 query, reduces allocations, enables parallelism).

At the end, provide:

- A prioritised list of issues by impact.
- Any profiling or measurement recommendations.

---

## Code to Analyse

[Paste the code to analyse here, or describe the files to review]
