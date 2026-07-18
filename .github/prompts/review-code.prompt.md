# Review Code Prompt

Perform a thorough code review of the code provided below, following the review
standards defined in this repository.

---

## Instructions

Review the code following all standards defined in:
- `.github/instructions/code-review.md`
- `.github/instructions/architecture.md`
- `.github/instructions/coding-style.md`
- `.github/instructions/naming.md`
- `.github/instructions/security.md`
- `.github/instructions/testing.md`

---

## Review Criteria

Evaluate the code across all of the following dimensions:

### 1. Readability

- Is the intent clear from the code and names?
- Are methods and classes short and focused?
- Is there unnecessary complexity?

### 2. Architecture

- Are dependencies flowing in the correct direction?
- Is business logic in the correct layer?
- Is the CQRS pattern applied correctly?
- Is ConfigurationLayer used as the Composition Root?

### 3. Naming

- Do names follow the conventions in `naming.md`?
- Are async methods suffixed with `Async`?
- Are interfaces prefixed with `I`?

### 4. Async Correctness

- Is `async/await` used consistently?
- Is `CancellationToken` propagated through all async calls?
- Are there any `.Result` or `.Wait()` calls?
- Are `async void` methods avoided?

### 5. Nullability

- Are Nullable Reference Types used correctly?
- Are null checks performed where required?
- Are guard clauses used to fail fast?

### 6. Performance

- Are database queries efficient?
- Is N+1 query problem avoided?
- Are large result sets paginated?
- Are resources properly disposed?

### 7. Security

- Are all inputs validated?
- Are parameterised queries / ORM used?
- Are secrets hardcoded?
- Are endpoints correctly authorised?
- Is sensitive data exposed in logs or errors?

### 8. Testability

- Is the code testable?
- Are dependencies injectable?
- Is there shared mutable state that makes testing difficult?

---

## Review Output Format

Provide feedback using this structure for each issue found:

```
**[SEVERITY]** File: `path/to/file.cs`, Line(s): N

**Issue:** Description of the problem.

**Why it matters:** Explanation of the risk or impact.

**Suggested fix:**
```code
// Fixed version here
```
```

Use these severity levels:
- `[BLOCKING]` — must be fixed before merging
- `[SUGGESTION]` — improvement recommended
- `[QUESTION]` — needs clarification
- `[NIT]` — minor style issue

---

## Summary

After individual issues, provide:

1. **Overall assessment** — is the code ready to merge, needs minor changes, or needs significant rework?
2. **Key strengths** — what is done well?
3. **Priority fixes** — the most important issues to address first.

---

## Code to Review

[Paste the code to review here, or describe the PR/files to review]
