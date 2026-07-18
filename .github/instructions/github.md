# GitHub Standards

## Purpose

This document defines standards for using GitHub within this repository, including
branch conventions, pull request practices, commit messages, workflows, and
GitHub Actions CI/CD pipelines.

---

## Guidelines

### Branch Naming

| Type | Pattern | Example |
|------|---------|---------|
| Feature | `feature/{ticket}-{description}` | `feature/ORD-123-add-order-service` |
| Bug fix | `fix/{ticket}-{description}` | `fix/ORD-456-null-reference-in-handler` |
| Hot fix | `hotfix/{ticket}-{description}` | `hotfix/ORD-789-payment-failure` |
| Release | `release/{version}` | `release/1.2.0` |
| Chore | `chore/{description}` | `chore/update-dependencies` |
| Documentation | `docs/{description}` | `docs/update-readme` |

### Commit Messages

Follow the Conventional Commits specification:

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

Types:

| Type | Use |
|------|-----|
| `feat` | New feature |
| `fix` | Bug fix |
| `docs` | Documentation only |
| `style` | Formatting, no logic change |
| `refactor` | Code change without feature or fix |
| `test` | Adding or updating tests |
| `chore` | Build process, dependencies, tools |
| `perf` | Performance improvement |
| `ci` | CI/CD changes |
| `revert` | Reverts a previous commit |

Examples:

```
feat(orders): add create order endpoint
fix(auth): resolve token expiry not refreshing
docs(readme): update setup instructions
refactor(orders): extract validation to FluentValidation
test(orders): add unit tests for CreateOrderCommandHandler
chore(deps): update SimpleCqrs to 0.1.4
```

### Rules

- Commit messages must be in English.
- Subject line must be imperative, present tense: "add" not "added" or "adds".
- Subject line must not end with a period.
- Keep subject line to 72 characters or fewer.
- Separate subject from body with a blank line.
- Reference issue or ticket numbers in the footer when applicable.

---

## Pull Requests

### PR Title

Follow the same Conventional Commits format as commit messages:

```
feat(orders): add create order endpoint
fix(payments): handle declined card gracefully
```

### PR Description

Every PR must include:

- **Summary** — what the change does and why.
- **Changes** — list of significant changes made.
- **Testing** — how the change was tested.
- **Related issues** — reference to tickets or issues.

Template:

```markdown
## Summary

Describe what this PR does and why it is needed.

## Changes

- Added `CreateOrderCommandHandler`
- Added `CreateOrderCommand` record
- Added `CreateOrderCommandValidator`
- Added unit tests for all new classes

## Testing

- Unit tests added and passing
- Manually tested via Swagger

## Related Issues

Closes #123
```

### PR Rules

- PRs must be reviewed before merging.
- All CI checks must pass before merging.
- Keep PRs focused — one concern per PR.
- Avoid PRs with more than 500 lines of change where possible.
- Squash or rebase before merging to keep history clean.
- Delete the branch after merging.

---

## GitHub Actions

### Workflow Standards

Always define workflows for:

- **CI** — build, lint, test on every push and PR.
- **CD** — deploy on merge to `main` (or release branches).
- **Security** — dependency vulnerability scanning.

### CI Workflow Example (.NET)

```yaml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  build-and-test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Setup .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '9.x'

      - name: Restore dependencies
        run: dotnet restore

      - name: Build
        run: dotnet build --no-restore --configuration Release

      - name: Test
        run: dotnet test --no-build --configuration Release --verbosity normal

      - name: Check for vulnerable packages
        run: dotnet list package --vulnerable --include-transitive
```

### Workflow Rules

- Pin action versions using full SHA or major version tags (e.g., `@v4`).
- Never hardcode secrets in workflow files — use GitHub Secrets.
- Use environment-level approvals for production deployments.
- Cache dependencies to speed up builds.
- Fail fast — do not continue on test failures.

---

## GitHub Copilot Instructions

The `.github/copilot-instructions.md` file is automatically loaded by GitHub
Copilot and applies to all AI-assisted code generation in the repository.

When adding new technologies:

1. Create a new instruction file in `.github/instructions/`.
2. Reference it in `.github/copilot-instructions.md`.
3. Create any relevant prompt files in `.github/prompts/`.

---

## Repository Settings

Recommended repository settings:

- Require PR reviews before merging (at least 1 reviewer).
- Require status checks to pass before merging.
- Require branches to be up to date before merging.
- Automatically delete head branches after merging.
- Disallow force pushes to `main` and `develop`.
- Enable Dependabot for automated dependency updates.
- Enable GitHub Advanced Security (CodeQL scanning, secret scanning).

---

## Best Practices

- Write meaningful PR descriptions — reviewers should not need to read the code to understand the intent.
- Keep the default branch (`main`) always deployable.
- Use GitHub Environments for deployment gates.
- Use GitHub Secrets for all credentials — never in workflow files.
- Use CODEOWNERS to assign automatic reviewers.
- Add branch protection rules on all long-lived branches.

---

## Bad Practices

- Force-pushing to protected branches.
- Merging without a review.
- Committing directly to `main` or `develop`.
- Storing secrets in workflow files or repository settings as plain text.
- PRs that mix multiple unrelated concerns.
- Long-running feature branches that diverge significantly from `main`.

---

## Common Mistakes

- Not referencing the issue or ticket in the PR.
- Not deleting branches after merging.
- Using `actions/checkout` without specifying a version tag.
- Not caching dependencies in CI workflows.
- Not adding required status checks for PRs.

---

## Checklist

- [ ] Branch name follows the naming convention
- [ ] Commit messages follow Conventional Commits
- [ ] PR has a summary, changes list, and testing notes
- [ ] PR references related issues
- [ ] CI workflow builds and tests the code
- [ ] All secrets stored in GitHub Secrets
- [ ] Action versions are pinned
- [ ] Branch protection rules applied to `main`
- [ ] Dependabot enabled
- [ ] CodeQL scanning enabled
