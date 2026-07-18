# AI Development Standards

> Reusable AI instructions, engineering standards, and prompt templates for GitHub
> Copilot and other AI coding assistants.

---

## Overview

This repository is the single source of truth for AI-assisted development
standards across projects using .NET, ASP.NET Core, .NET MAUI, Angular, React,
and Azure.

### Purpose

Modern AI coding assistants — GitHub Copilot, ChatGPT, Claude, Gemini, and others
— generate code based on context. Without explicit standards, every assistant
generates code in its own style, leading to inconsistent architecture, naming
conventions, patterns, and security practices across projects and teams.

This repository solves that problem by providing:

- **Copilot Instructions** — automatically loaded by GitHub Copilot into every
  session, ensuring consistent code generation from the first keystroke.
- **Instruction Documents** — detailed engineering standards for every technology
  and domain, referenced when working on specific areas.
- **Prompt Templates** — ready-to-use prompts for common development tasks that
  encode best practices directly into the request.

### Goals

- Enforce consistent architecture (Clean Architecture, DDD, CQRS) across all projects.
- Eliminate inconsistent naming, patterns, and anti-patterns in AI-generated code.
- Reduce the cost of code review by generating production-ready code from the start.
- Accelerate onboarding by codifying team standards in machine-readable form.
- Work with multiple AI coding assistants without duplicating standards.

### Supported AI Coding Assistants

| Assistant | Integration | How to Use |
|-----------|-------------|-----------|
| **GitHub Copilot** | Native — `.github/copilot-instructions.md` | Automatically applied to all sessions in the repository |
| **ChatGPT** | Manual | Copy the relevant instruction files and paste them into the system prompt |
| **Claude** | Manual | Copy the relevant instruction files and paste them into the conversation |
| **Gemini** | Manual | Copy the relevant instruction files and paste them into the conversation |
| **Other compatible assistants** | Manual | Copy the relevant instruction files as context |

---

## Repository Structure

```
.github/
├── copilot-instructions.md      ← Global GitHub Copilot instructions (auto-loaded)
├── instructions/                ← Detailed engineering standards by domain
│   ├── architecture.md
│   ├── coding-style.md
│   ├── naming.md
│   ├── security.md
│   ├── code-review.md
│   ├── testing.md
│   ├── dotnet.md
│   ├── maui.md
│   ├── angular.md
│   ├── react.md
│   ├── azure.md
│   └── github.md
└── prompts/                     ← Reusable prompt templates
    ├── create-api.prompt.md
    ├── create-service.prompt.md
    ├── create-endpoint.prompt.md
    ├── create-unit-tests.prompt.md
    ├── create-maui-page.prompt.md
    ├── create-angular-feature.prompt.md
    ├── create-react-feature.prompt.md
    ├── review-code.prompt.md
    ├── refactor-code.prompt.md
    ├── improve-performance.prompt.md
    └── explain-code.prompt.md
README.md
LICENSE
CONTRIBUTING.md
```

### Folder Purposes

#### `.github/`

The `.github/` directory is a well-known location recognised by GitHub and GitHub
Copilot. Placing instruction and prompt files here ensures they are automatically
discovered by GitHub Copilot and are clearly separated from application source code.

#### `.github/instructions/`

Contains detailed engineering standards organised by technology and domain.
Each file defines the rules, patterns, conventions, examples, and anti-patterns
for a specific area of development.

Use these files when:
- Working on a specific technology and needing detailed guidance.
- Asking a non-Copilot AI assistant for help (paste the relevant file as context).
- Onboarding a new developer to understand team standards.
- Reviewing whether an existing codebase meets standards.

#### `.github/prompts/`

Contains reusable prompt templates for common development tasks. Each prompt
encodes best practices and references the relevant instruction files to ensure
generated code meets standards.

Use these files when:
- Asking any AI assistant to generate a new feature, service, endpoint, or page.
- Starting a new task from scratch and wanting a complete, standards-compliant scaffold.
- Requesting code reviews, refactoring, performance improvements, or explanations.

---

## Copilot Instructions

### What is `copilot-instructions.md`?

The `.github/copilot-instructions.md` file is a special file recognised by
GitHub Copilot. It contains natural language instructions that are automatically
injected into every Copilot chat session and inline suggestion context within
the repository.

This means that every time a developer uses GitHub Copilot in this repository —
whether through chat, inline completions, or agent sessions — these instructions
are automatically applied without any manual action.

### How GitHub Copilot Uses It

GitHub Copilot reads `.github/copilot-instructions.md` and treats its contents
as persistent context. This influences:

- **Inline code completions** — Copilot generates code following the defined patterns.
- **Copilot Chat** — responses apply the global rules and technology standards.
- **Copilot Agents** — autonomous coding sessions follow the defined architecture.

### When It Is Applied

The file is applied automatically whenever:

- A developer opens GitHub Copilot Chat in a repository that contains this file.
- Copilot generates inline completions in a project with this file.
- A Copilot coding agent is run on a repository containing this file.

No configuration is required beyond placing the file in `.github/`.

### How to Customise It

To customise the global instructions:

1. Edit `.github/copilot-instructions.md` directly.
2. Add or remove technology references in the instruction table.
3. Add or remove global rules in the "Global Rules" section.
4. Add technology-specific highlights for new frameworks.

Keep the file concise — Copilot works best with clear, specific instructions
rather than exhaustive detail. Detailed standards belong in the `instructions/`
files, which are referenced from `copilot-instructions.md`.

---

## Instructions Folder

Each file in `.github/instructions/` defines comprehensive engineering standards
for a specific technology domain. Every document follows the same structure:

- **Purpose** — why this document exists
- **Guidelines** — the core rules and conventions
- **Best Practices** — recommended approaches
- **Bad Practices** — patterns to avoid
- **Examples** — concrete code examples
- **Common Mistakes** — frequent pitfalls
- **Checklist** — verifiable list of requirements

### Document Reference

#### `architecture.md`

Defines Clean Architecture, DDD (Domain-Driven Design), and CQRS patterns. Covers
the layer structure for .NET solutions, how to use the ConfigurationLayer as the
Composition Root, when to use aggregates and domain events, and how to implement
commands and queries using `Kmaraszkiewicz86.SimpleCqrs`.

Use when: Designing a new solution, reviewing architectural decisions, or
ensuring existing code follows the correct layer structure.

#### `coding-style.md`

Defines general coding conventions that apply across all technologies. Covers
modern C# style (primary constructors, file-scoped namespaces, collection
expressions), async/await rules, null handling, error handling, and comment
standards.

Use when: Writing or reviewing code in any language to ensure consistent style.

#### `naming.md`

Defines naming conventions for all constructs: classes, interfaces, methods,
properties, fields, files, DTOs, commands, queries, repositories, and
database objects. Covers C#, Angular, React, and MAUI naming patterns.

Use when: Creating new files or classes, or reviewing existing names for consistency.

#### `security.md`

Defines security standards covering authentication, authorisation, secrets
management, input validation, SQL injection prevention, XSS, CSRF, transport
security, and Azure-specific security practices.

Use when: Implementing any feature that handles user data, authentication,
external integrations, or secrets.

#### `code-review.md`

Defines what must be reviewed in every pull request, severity levels for review
comments, and the complete review checklist. Covers readability, architecture,
naming, async correctness, nullability, performance, security, and testability.

Use when: Reviewing a pull request or asking an AI assistant to review code.

#### `testing.md`

Defines the testing stack (xUnit, FluentAssertions, AutoFixture, NSubstitute),
the Fixture pattern, test naming conventions, what to test (happy path, validation,
exceptions, edge cases), and FluentAssertions examples.

Use when: Writing unit tests for any .NET class or feature.

#### `dotnet.md`

Defines engineering standards for .NET, ASP.NET Core Minimal APIs, Entity
Framework Core, and the solution layer structure. Covers `Program.cs` rules,
ConfigurationLayer, endpoint organisation, EF Core entity configuration,
and modern C# style.

Use when: Creating or working on any .NET project.

#### `maui.md`

Defines MVVM standards for .NET MAUI applications. Covers ViewModel conventions
(explicit `SetProperty()`, no `[ObservableProperty]` generator), command patterns
(explicit `IAsyncRelayCommand`, no `[RelayCommand]` generator), view binding rules
(no `x:DataType`, `BindingContext` in code-behind), Shell navigation, and
localization requirements.

Use when: Building or reviewing any .NET MAUI application.

#### `angular.md`

Defines Angular standards including Standalone Components, Signals for state,
lazy loading, service patterns, HTTP interceptors, and `ChangeDetectionStrategy.OnPush`.

Use when: Building or reviewing any Angular application.

#### `react.md`

Defines React standards including TypeScript usage, functional components, React
Query for server state, Axios for HTTP, custom hooks, folder structure, and
form validation with React Hook Form and Zod.

Use when: Building or reviewing any React application.

#### `azure.md`

Defines Azure cloud service standards covering Managed Identity, Key Vault,
Azure Functions, App Service, Azure OpenAI, AI Search, Blob Storage, Service Bus,
and Application Insights.

Use when: Integrating with any Azure service.

#### `github.md`

Defines GitHub conventions including branch naming, Conventional Commits, pull
request standards, GitHub Actions workflow templates, and repository settings.

Use when: Creating branches, writing commit messages, configuring CI/CD, or
setting up a new repository.

---

## Prompt Library

### What Are Prompt Files?

Prompt files (`.prompt.md`) are reusable, structured prompts that can be sent to
any AI coding assistant to generate production-ready code. Each prompt:

- Describes the task clearly and completely.
- References the relevant instruction files.
- Specifies what to generate (files, folders, classes).
- Defines requirements and constraints.
- Provides a placeholder for the specific feature description.

### When to Use Them

Use prompt files when:

- Starting a new feature from scratch and wanting a complete, standards-compliant scaffold.
- Generating boilerplate code for a common task (creating an API, a service, a MAUI page).
- Requesting a code review with consistent criteria.
- Asking for refactoring or performance improvements with clear guidelines.

### Usage Examples

#### Example 1: Create a new API feature

1. Open `.github/prompts/create-api.prompt.md`.
2. Fill in the "Feature Description" section with your feature details.
3. Paste the entire file into GitHub Copilot Chat or another AI assistant.
4. The assistant generates the complete API following all standards.

```
[Feature Description]
An Order Management API supporting:
- POST /api/orders — create a new order
- GET /api/orders/{id} — get order by ID
- GET /api/orders?customerId={id} — list orders by customer
- POST /api/orders/{id}/cancel — cancel an order
```

#### Example 2: Generate unit tests

1. Open `.github/prompts/create-unit-tests.prompt.md`.
2. Fill in the "Class or Feature to Test" section.
3. Paste the prompt and the class to test into Copilot Chat.
4. Receive complete tests following the Fixture pattern with all test cases.

```
[Class or Feature to Test]
CreateOrderCommandHandler — receives a CreateOrderCommand, validates it,
creates an Order aggregate, and saves it via IOrderRepository.
```

#### Example 3: Request a code review

1. Open `.github/prompts/review-code.prompt.md`.
2. Paste the code to review in the "Code to Review" section.
3. Send to Copilot Chat or another assistant.
4. Receive structured feedback with severity levels.

#### Example 4: Create a MAUI page

1. Open `.github/prompts/create-maui-page.prompt.md`.
2. Fill in the "Page Description" section.
3. Send to the AI assistant.
4. Receive a complete XAML page, code-behind, and ViewModel following MVVM standards.

### Available Prompts

| Prompt | Use Case |
|--------|---------|
| `create-api.prompt.md` | Generate a complete .NET API with all layers |
| `create-service.prompt.md` | Generate a service with interface and DI registration |
| `create-endpoint.prompt.md` | Generate a Minimal API endpoint |
| `create-unit-tests.prompt.md` | Generate unit tests with Fixtures |
| `create-maui-page.prompt.md` | Generate a MAUI page with ViewModel |
| `create-angular-feature.prompt.md` | Generate a complete Angular feature |
| `create-react-feature.prompt.md` | Generate a complete React feature |
| `review-code.prompt.md` | Request a structured code review |
| `refactor-code.prompt.md` | Request standards-aligned refactoring |
| `improve-performance.prompt.md` | Request performance analysis and fixes |
| `explain-code.prompt.md` | Request a detailed code explanation |

---

## Architecture Standards

### Why These Standards Exist

Inconsistent architecture is one of the most costly problems in software
development. When different developers (or AI assistants) make different
architectural decisions, the codebase becomes harder to:

- **Understand** — developers cannot predict where logic lives.
- **Test** — tightly coupled layers cannot be tested independently.
- **Maintain** — changes cascade unpredictably across layers.
- **Scale** — different patterns in different areas require context switching.

These standards enforce a single, consistent architecture across all projects
so that any developer (or AI assistant) can navigate, contribute to, and extend
any codebase in the organisation.

### Why Consistency Is Important

Consistency reduces cognitive load. When every project follows the same patterns:

- New developers can contribute immediately without learning project-specific conventions.
- Code reviews focus on business logic rather than style disputes.
- AI assistants generate predictable, correct code because the patterns are explicit.
- Bugs are easier to find because code is always in the expected place.

### How AI Assistants Should Follow Them

AI coding assistants follow these standards when they are provided as context.
For GitHub Copilot, the `copilot-instructions.md` file provides this context
automatically. For other assistants, paste the relevant instruction files
directly into the conversation.

The more specific and explicit the context, the better the output. These
standards are designed to be machine-readable — they contain concrete examples,
explicit rules, and clear anti-patterns that AI assistants can reason about
reliably.

---

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for detailed contribution guidelines.

### Quick Start

To add a new technology or extend existing standards:

1. **Add a new instruction file**: Create `.github/instructions/{technology}.md`
   following the existing document structure.
2. **Reference it in copilot-instructions.md**: Add a row to the instruction
   table and a highlights section.
3. **Add a prompt file**: Create `.github/prompts/create-{feature}.prompt.md`
   for the most common generation task.
4. **Update this README**: Add the new document to the Instructions Folder section.

---

## Best Practices

### Keeping Standards Consistent

- Review the instruction files quarterly against the latest framework releases.
- When a framework deprecates a pattern (e.g., NgModules in Angular), update the
  instruction file and add the old pattern to the "Bad Practices" section.
- When a new package or tool is adopted, add it to the relevant instruction file.
- Keep examples in instruction files up to date — outdated examples cause
  AI assistants to generate outdated patterns.

### Using Standards Effectively

- Reference the specific instruction file when asking an AI assistant for help.
  For example: "Following the standards in `.github/instructions/dotnet.md`, generate..."
- Use the prompt files as a starting point — they are designed to produce
  production-ready code on the first attempt.
- When an AI assistant generates code that violates a standard, update the
  instruction file to make the rule more explicit.

### Avoiding Drift

- Define a standard before it is needed, not after problems arise.
- Document both what to do and what not to do — anti-patterns are as important
  as best practices.
- Keep checklists at the bottom of each instruction file — they serve as
  quick verification tools for both humans and AI assistants.

---

## Versioning

This repository follows [Semantic Versioning](https://semver.org/):

- **MAJOR** — breaking changes to existing standards (e.g., a fundamental
  architectural change that invalidates existing projects).
- **MINOR** — new instruction files, new prompt files, or significant additions
  to existing documents.
- **PATCH** — corrections, clarifications, example updates, and typo fixes.

### Managing Versions

Tag releases using GitHub Releases:

```
git tag -a v1.2.0 -m "Add Azure AI Foundry standards"
git push origin v1.2.0
```

Reference a specific version of this repository in projects that need stability:

```
# In a project's .github/copilot-instructions.md
# Adapted from ai-development-standards v1.2.0
```

When upgrading to a newer version, review the changelog for breaking changes
that might require codebase updates.

---

## Examples

### Using `copilot-instructions.md`

When a developer opens GitHub Copilot Chat in a repository containing
`.github/copilot-instructions.md`, Copilot automatically reads the file and
applies the instructions. No configuration is needed.

**Effect:** Asking Copilot to "create an order service" will produce a service
that uses primary constructors, `IAsyncCommandHandler<T>`, FluentValidation,
DTOs, and follows the DDD layer structure — without the developer needing to
specify each of these requirements.

### Using Prompt Files

**Scenario:** A developer needs to create a new orders API endpoint.

1. Open `.github/prompts/create-endpoint.prompt.md`.
2. Paste it into GitHub Copilot Chat.
3. Fill in the description: *"A POST endpoint at `/api/orders` that accepts a
   `CreateOrderRequestDto`, dispatches a `CreateOrderCommand`, and returns
   `201 Created`."*
4. Copilot generates the complete endpoint with all required metadata.

### Extending the Repository

**Scenario:** The team adopts Blazor for a new project.

1. Create `.github/instructions/blazor.md` following the existing document structure.
2. Add Blazor-specific standards: component lifecycle, state management, form handling, routing.
3. Add `.github/prompts/create-blazor-component.prompt.md`.
4. Update `.github/copilot-instructions.md` to reference the new file.
5. Update this README to document the new instruction file.

### Using with Non-Copilot Assistants

**Scenario:** A developer is using Claude to review a .NET service.

1. Open `.github/prompts/review-code.prompt.md`.
2. Open `.github/instructions/dotnet.md` and `.github/instructions/architecture.md`.
3. In a new Claude conversation, paste the instruction files as context.
4. Paste the prompt file with the code to review.
5. Claude reviews the code against the defined standards.

---

## Maintenance

### Keeping Standards Up to Date

Technology evolves constantly. Standards that are not maintained become stale
and produce outdated code. The following maintenance process ensures this
repository remains useful:

#### Quarterly Reviews

Every quarter, review each instruction file against:

- The latest stable release of the relevant framework or language.
- Any deprecated patterns or removed APIs.
- New best practices recommended by the framework authors.
- Any issues encountered in real projects.

#### Continuous Updates

Update instruction files immediately when:

- A new package version changes the recommended API.
- A security vulnerability is discovered in a recommended pattern.
- A new language or framework feature replaces an existing recommendation.
- An AI assistant consistently generates incorrect code despite the instructions.

#### Version Bumps

- Increment PATCH for clarifications and corrections.
- Increment MINOR for new instruction files or significant new content.
- Increment MAJOR for breaking changes that require updating existing projects.

### Feedback Loop

The best improvement to standards comes from real usage. When working on a
project:

- Note any cases where the AI assistant generated code that did not meet
  expectations.
- Identify whether the instruction file was unclear or missing a rule.
- Open a pull request with an improvement.

Standards improve incrementally through consistent use and feedback.

---

## License

This repository is licensed under the [MIT License](LICENSE).
