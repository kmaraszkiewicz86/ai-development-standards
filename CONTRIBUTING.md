# Contributing to AI Development Standards

Thank you for contributing to this repository. These standards help teams
produce consistent, high-quality code with AI coding assistants. Every
improvement benefits everyone who uses this repository.

---

## How to Contribute

### Prerequisites

- A GitHub account.
- Basic familiarity with Markdown.
- Familiarity with the technology area you are contributing to.

### Contribution Types

| Type | Description |
|------|-------------|
| New instruction file | Add standards for a new technology |
| Update existing standards | Improve, correct, or expand an existing file |
| New prompt file | Add a reusable prompt for a common task |
| Bug fix | Correct an incorrect example or broken code snippet |
| Documentation | Improve the README or other documentation |

---

## Adding a New Technology

When adding standards for a new technology (e.g., Blazor, FastAPI, Vue.js):

### 1. Create the Instruction File

Create `.github/instructions/{technology}.md` using the standard document structure:

```markdown
# {Technology} Standards

## Purpose

[Explain why this document exists and what it covers]

---

## Guidelines

[Core rules and conventions]

---

## Best Practices

[Recommended approaches]

---

## Bad Practices

[Patterns to avoid]

---

## Examples

[Concrete code examples]

---

## Common Mistakes

[Frequent pitfalls]

---

## Checklist

- [ ] Item 1
- [ ] Item 2
```

### 2. Update `copilot-instructions.md`

Add the new file to the instruction table:

```markdown
| [technology.md](instructions/technology.md) | Description of what this file covers |
```

Add a highlights section if the technology has specific global rules:

```markdown
### {Technology}

- Key rule 1
- Key rule 2
```

### 3. Add a Prompt File (if applicable)

If the new technology has a common generation task, add
`.github/prompts/create-{feature}.prompt.md` following the existing prompt structure.

### 4. Update the README

Add the new file to the "Instructions Folder" section in `README.md` with a
description of its purpose and when to use it.

---

## Modifying Existing Standards

When modifying an existing instruction file:

### For Corrections

- Fix incorrect examples — ensure they compile and follow the stated standards.
- Correct factual errors.
- Update outdated API references to the latest stable version.

### For Additions

- Add new rules to the "Guidelines" section.
- Add new examples to the "Examples" section.
- Add new anti-patterns to the "Bad Practices" section.
- Add new items to the "Checklist".

### For Removals

- Move removed/deprecated patterns to the "Bad Practices" section rather than
  simply deleting them. This ensures AI assistants know what to avoid, not
  just what to do.

---

## Creating New Prompts

When creating a new prompt file:

### Structure

Every prompt file must follow this structure:

```markdown
# {Task} Prompt

[One sentence description]

---

## Instructions

[What to follow and reference]

---

## What to generate

[Detailed specification of what to produce]

---

## Requirements

[Non-negotiable constraints as a bullet list]

---

## {Feature/Code/Class} Description

[Describe the [feature/code/class] here — e.g., "..."]
```

### Rules

- Always reference the relevant instruction files.
- Always provide concrete code examples for complex outputs.
- Always end with a placeholder section for the developer to fill in.
- Never include specific project names or business details.
- Keep prompts technology-agnostic within their domain where possible.

---

## Pull Request Process

### Branch Naming

Follow the conventions in `.github/instructions/github.md`:

```
docs/add-blazor-standards
docs/update-dotnet-ef-core
fix/correct-angular-example
```

### Commit Messages

Follow Conventional Commits:

```
docs(blazor): add Blazor instruction file
docs(dotnet): update EF Core entity configuration examples
fix(maui): correct BindingContext example in maui.md
```

### PR Description

Every PR must include:

- **Summary** — what the change does and why.
- **Changes** — list of files added or modified.
- **Validation** — how you verified the examples are correct (e.g., compiled and ran the code).

### Review Criteria

Pull requests to this repository are reviewed for:

- **Accuracy** — is the information technically correct?
- **Completeness** — does the document cover the most important aspects?
- **Clarity** — are instructions clear and unambiguous?
- **Examples** — do examples compile and follow the stated standards?
- **Consistency** — does the document follow the same structure as others?
- **Checklists** — does the checklist accurately reflect the document's rules?

---

## Quality Standards

### Code Examples

All code examples in instruction files must:

- Be syntactically correct and compilable.
- Follow the standards defined in the same document.
- Use the same package versions referenced in the document.
- Include both "Good" and "Bad" versions where anti-patterns are described.

### Language

- All documentation must be written in English.
- Use clear, direct language.
- Avoid marketing language or filler text.
- Prefer short sentences over complex compound sentences.
- Use imperative mood for rules: "Use", "Never", "Always", "Prefer".

### Checklists

Every instruction file must end with a checklist that:

- Contains one item per significant rule in the document.
- Uses concrete, verifiable items (not vague items like "follow best practices").
- Is short enough to use as a quick reference (aim for 10–15 items).

---

## Versioning Contributions

### Patch Contributions

Corrections, clarifications, and example fixes do not require a version bump.
They are included in the next PATCH release automatically.

### Minor Contributions

New instruction files and new prompt files are MINOR changes. When contributing
a new file, note in the PR description that it warrants a minor version bump.

### Major Contributions

Breaking changes — standards that fundamentally contradict existing guidance
and would require updating existing projects — require discussion before
implementation. Open an issue first to discuss the change.

---

## Questions and Discussions

If you are unsure whether a change is appropriate or want to discuss a potential
improvement before investing time in a PR:

1. Open a GitHub Issue with the label `discussion`.
2. Describe the improvement you are considering.
3. Explain the problem it solves or the improvement it makes.

---

## Code of Conduct

Contributions to this repository must be:

- Technically accurate and honest.
- Respectful of different experience levels.
- Free of vendor bias — recommend tools based on merit.
- Focused on the stated purpose of the repository.

---

## Thank You

Every contribution — large or small — improves the quality of AI-assisted
development for everyone who uses this repository.
