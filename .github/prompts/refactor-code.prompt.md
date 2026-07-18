# Refactor Code Prompt

Refactor the code provided below to improve its quality while preserving its
existing behaviour, following the standards defined in this repository.

---

## Instructions

Refactor the code following all standards defined in:
- `.github/instructions/architecture.md`
- `.github/instructions/coding-style.md`
- `.github/instructions/naming.md`
- `.github/instructions/dotnet.md` (if .NET code)
- `.github/instructions/maui.md` (if MAUI code)
- `.github/instructions/angular.md` (if Angular code)
- `.github/instructions/react.md` (if React code)

---

## Refactoring Goals

Apply the following improvements as relevant to the code:

### Structure

- Apply Single Responsibility Principle — split classes with multiple concerns.
- Extract business logic to the correct layer.
- Move infrastructure concerns to InfrastructureLayer.
- Move application concerns to ApplicationLayer.
- Ensure ConfigurationLayer remains the Composition Root.

### Code Quality

- Replace magic numbers and strings with named constants or enumerations.
- Eliminate duplicated code — extract shared logic into methods or services.
- Replace deep nesting with guard clauses and early returns.
- Break large methods into smaller, focused methods.
- Replace loops with LINQ where it improves readability.

### C# Modernisation

- Replace traditional constructors with primary constructors for DI.
- Replace block-scoped namespaces with file-scoped namespaces.
- Replace `List<T>` with `IReadOnlyList<T>` for public read-only collections.
- Replace manual null checks with null-conditional operators where appropriate.
- Replace `Task.Result` / `Task.Wait()` with proper `async/await`.
- Add `CancellationToken` to all async methods that lack it.

### MAUI Specific

- Replace `[ObservableProperty]` with explicit `SetProperty()` properties.
- Replace `[RelayCommand]` with explicit `IAsyncRelayCommand` properties.
- Move `BindingContext` assignment to code-behind if set in XAML.
- Remove `x:DataType` and Compiled Bindings.
- Replace business logic in Views/code-behind with ViewModel methods.

### Angular Specific

- Convert NgModule-based components to Standalone Components.
- Replace `BehaviorSubject` state with Signals.
- Add `ChangeDetectionStrategy.OnPush` where missing.
- Move HTTP calls from components to services.
- Replace constructor DI with `inject()`.

### React Specific

- Replace `useEffect` data fetching with a dedicated `loadData()` function called inside `useEffect`.
- Move API calls from components to service files.
- Add missing TypeScript types — remove `any`.
- Replace default exports with named exports.

---

## Constraints

- **Do not change behaviour** — only improve structure and quality.
- **Do not add new features** — stay within the scope of the existing code.
- **Preserve all tests** — ensure existing tests still pass.
- **Explain each significant change** — document why the change was made.

---

## Output Format

For each significant change:

1. **Original code** — show the before.
2. **Refactored code** — show the after.
3. **Reason** — explain why the change improves the code.

At the end, provide:

- A summary of all changes made.
- Any further improvements that could be made in a follow-up.

---

## Code to Refactor

[Paste the code to refactor here, or describe the files to refactor]
