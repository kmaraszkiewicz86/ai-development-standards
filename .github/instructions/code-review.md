# Code Review Standards

## Purpose

This document defines what must be reviewed in every pull request, the criteria
for approval, and the expected quality bar for all code changes. Reviews exist to
ensure consistency, catch bugs early, improve security, and share knowledge.

---

## Guidelines

### What to Always Review

Every pull request must be reviewed for the following:

#### 1. Readability

- Is the intent of the code immediately clear?
- Are names descriptive and meaningful?
- Are methods and classes short and focused?
- Is there any unnecessary complexity?
- Are there any unnecessary abstractions or over-engineering?

#### 2. Architecture

- Does the change follow the established layer structure?
- Are dependencies flowing in the correct direction?
- Is business logic placed in the correct layer?
- Does the ConfigurationLayer remain the Composition Root?
- Is CQRS applied correctly?

#### 3. Naming

- Do names follow the conventions in `naming.md`?
- Are async methods suffixed with `Async`?
- Are commands and queries named correctly?
- Are interfaces prefixed with `I`?

#### 4. Async Correctness

- Is `async/await` used consistently?
- Is `CancellationToken` passed through all async calls?
- Are there any `.Result` or `.Wait()` calls that could deadlock?
- Are `async void` methods avoided?

#### 5. Nullability

- Are Nullable Reference Types enabled?
- Are null checks performed where required?
- Are null returns handled by the caller?
- Are guard clauses used to fail fast on invalid inputs?

#### 6. Performance

- Are database queries efficient?
- Is N+1 query problem avoided?
- Are large payloads paginated?
- Is caching applied where appropriate?
- Are resources properly disposed?

#### 7. Security

- Are all inputs validated?
- Is user input used in SQL queries parameterised?
- Are secrets hardcoded?
- Are all endpoints correctly authorised?
- Is sensitive data logged or returned in error responses?

#### 8. Testability

- Is the changed code covered by unit tests?
- Are happy path, validation, and exception cases tested?
- Are tests meaningful — not just checking for no exception?
- Do tests follow Arrange / Act / Assert?
- Are external dependencies mocked?

---

## Review Tone

- Be constructive — the goal is to improve the code, not criticise the author.
- Be specific — explain *why* a change is needed, not just *what* is wrong.
- Suggest minimal improvements — avoid requesting unnecessary rewrites.
- Distinguish between required changes (blocking) and suggestions (non-blocking).
- Acknowledge good decisions — reviews are not only for finding problems.

---

## Severity Levels

Use these labels when leaving comments:

| Label | Meaning |
|-------|---------|
| **[BLOCKING]** | Must be resolved before merging |
| **[SUGGESTION]** | Improvement recommended but not required |
| **[QUESTION]** | Needs clarification — may or may not need a change |
| **[NIT]** | Minor style issue — optional to address |

---

## What Not to Do in Reviews

- Do not nitpick formatting if automated formatters exist.
- Do not request changes based on personal preference when conventions are not violated.
- Do not request unnecessary rewrites of working code.
- Do not leave vague comments like "this is bad" without explanation.
- Do not approve code you have not actually read.

---

## Best Practices

- Review the architecture first, then the details.
- Read the tests before reading the implementation.
- Check the PR description — does it explain the *why*, not just the *what*?
- Look for patterns of issues rather than cataloguing every instance.
- Limit a single PR to a focused concern — reject PRs that mix unrelated changes.
- Reviews must be completed within the agreed SLA (e.g., one business day).

---

## Bad Practices

- Approving PRs without reading the code.
- Leaving only positive comments to avoid conflict.
- Focusing only on syntax and ignoring logic and architecture.
- Blocking PRs for trivial issues without justification.
- Writing overly long comments that are difficult to act on.

---

## Examples

### Async Correctness Comment

```
[BLOCKING] This method calls `.Result` on an async operation which can cause
deadlocks in ASP.NET Core. Replace with `await` or make the calling method async.
```

### Architecture Comment

```
[BLOCKING] Business logic must not be placed in the controller. Extract this
validation logic into a FluentValidation validator in the ApplicationLayer.
```

### Performance Comment

```
[SUGGESTION] This query loads all orders and filters in memory. Consider adding
a `.Where()` clause before `.ToListAsync()` to filter at the database level.
```

### Security Comment

```
[BLOCKING] The user ID is read directly from the request body. This allows
users to act as other users. Always read identity claims from the authenticated
token, not from user-supplied input.
```

### Nullable Comment

```
[BLOCKING] `GetByIdAsync` can return null but the result is used without a null
check. Either throw a domain exception or handle the null case explicitly.
```
