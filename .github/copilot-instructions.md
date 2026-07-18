# GitHub Copilot Instructions

These instructions apply globally to all AI-assisted development in this repository.
They are automatically loaded by GitHub Copilot and define the engineering standards
that must be followed across all technologies and layers.

---

## Global Rules

Always:

- Generate production-ready code.
- Use the latest stable language and framework versions.
- Follow SOLID principles and Clean Code practices.
- Prefer readability over clever implementations.
- Use Dependency Injection.
- Keep classes focused on a single responsibility.
- Keep methods short and purposeful.
- Avoid duplicated code (DRY principle).
- Consider security, performance, and maintainability in every change.
- Never generate TODO placeholders — implement or leave out.
- Generate compilable, runnable code.
- Explain architectural decisions inline when they are non-obvious.

---

## Instruction Files

This repository contains detailed instruction files for each technology domain.
Reference the appropriate file when working on each area:

| File | Purpose |
|------|---------|
| [architecture.md](instructions/architecture.md) | Clean Architecture, DDD, CQRS |
| [coding-style.md](instructions/coding-style.md) | General coding conventions |
| [naming.md](instructions/naming.md) | Naming conventions for all layers |
| [security.md](instructions/security.md) | Security standards and practices |
| [code-review.md](instructions/code-review.md) | Code review criteria |
| [testing.md](instructions/testing.md) | Unit and integration testing |
| [dotnet.md](instructions/dotnet.md) | .NET / ASP.NET Core / EF Core |
| [maui.md](instructions/maui.md) | .NET MAUI mobile development |
| [angular.md](instructions/angular.md) | Angular frontend development |
| [react.md](instructions/react.md) | React frontend development |
| [azure.md](instructions/azure.md) | Azure cloud services |
| [github.md](instructions/github.md) | GitHub workflows and conventions |

---

## Technology-Specific Highlights

### .NET

- Target the latest stable .NET.
- Prefer ASP.NET Core Minimal APIs, EF Core, async/await, CancellationToken.
- Always use `Kmaraszkiewicz86.SimpleCqrs` for CQRS — never MediatR.
- Follow DDD architecture with ConfigurationLayer as Composition Root.
- Never register services directly in `Program.cs`.
- Use `IEntityTypeConfiguration<TEntity>` for all EF Core entity configuration.

### .NET MAUI

- Always use MVVM with CommunityToolkit.Mvvm.
- Use `Kmaraszkiewicz86.Maui.Behaviors` instead of custom Behaviors when possible.
- Never place business logic in Views or code-behind.
- Resolve ViewModels through Dependency Injection.
- Do not use `x:DataType` or Compiled Bindings — set BindingContext in code-behind.
- Use explicit `SetProperty()` properties — not the `[ObservableProperty]` source generator.

### Angular

- Use the latest stable Angular with Standalone Components and Signals.
- Feature-based folder structure with lazy loading.
- Keep business logic out of components — use services.

### React

- Use the latest stable React with TypeScript, Functional Components, Hooks.
- Use React Query for server state, Axios for HTTP.
- Keep API logic in the `api/` or `services/` layer.

### Testing (.NET)

- Always use xUnit, FluentAssertions, AutoFixture, AutoFixture.AutoNSubstitute, NSubstitute.
- Follow Arrange / Act / Assert.
- Every test class must have its own Fixture inheriting from `BaseFixture`.

---

## Code Quality Checklist

Before submitting any code:

- [ ] Follows SOLID and Clean Code
- [ ] No business logic in Controllers, Views, or code-behind
- [ ] Dependency Injection used throughout
- [ ] No hardcoded secrets or connection strings
- [ ] async/await used correctly (no `.Result` or `.Wait()`)
- [ ] Proper null handling
- [ ] Unit tests cover happy path, validation, and edge cases
- [ ] No TODO placeholders
- [ ] No duplicated code
- [ ] Security considerations applied
