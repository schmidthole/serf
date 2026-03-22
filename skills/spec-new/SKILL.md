---
name: spec-new
description: Scaffold a new feature with proposal and spec templates. Use when adding a new feature to the project.
disable-model-invocation: true
allowed-tools: Read, Write, Edit, Glob
---

## reference

read [development-workflow.md](../development-workflow.md) for the full conventions governing the specs directory structure, features, work items, and their formats.

## instructions

scaffold a new feature directory and draft its proposal and spec based on the provided description. expects `$ARGUMENTS` in the format:

```
/spec-new <feature-name> <description of the feature>
```

the description can be as brief or detailed as the user wants. use whatever context is provided to draft meaningful content for the proposal and spec files.

follow these steps:

1. **validate the name.** the feature name should be lowercase kebab-case. if it contains spaces or uppercase letters, convert it.

2. **check for duplicates.** glob for `specs/features/<feature-name>/`. if it already exists, inform the user.

3. **understand the codebase.** read the project's `README.md` and any existing features in `specs/features/features.md` to understand the project context. this informs how the new feature fits into the existing architecture.

4. **check for overlap with existing features.** for each existing feature listed in `features.md`, read its `proposal.md` and `spec.md`. compare the new feature's described purpose, use cases, and likely source paths against existing features. look for:
   - the new feature's functionality is a subset of an existing feature
   - the new feature would share source paths with an existing feature
   - the new feature extends or modifies behavior already covered by an existing feature
   - the new feature's name or description is closely related to an existing feature

   if overlap is detected, inform the user with specifics:
   - which existing feature(s) overlap
   - what the overlap is (shared scope, shared source paths, extension of existing behavior)
   - a recommendation: would this be better as an amendment to the existing feature via `/spec-amend`?

   ask the user to choose:
   - **continue as new feature**: proceed with creating a standalone feature (the user may have good reasons for separation)
   - **convert to amendment**: stop the `/spec-new` flow and inform the user to run `/spec-amend <existing-feature> <description>` instead

   if the user chooses to convert, output the suggested `/spec-amend` command and stop. do not create any files.

   if no overlap is detected, or the user chooses to continue, proceed to the next step.

5. **ensure directory structure exists.** create `specs/features/`, `specs/work/active/`, and `specs/work/archive/` if they do not exist.

6. **create the feature directory** at `specs/features/<feature-name>/`.

7. **create `proposal.md`.** use the description provided in `$ARGUMENTS` and the project context to draft the content. follow this structure:

```markdown
---
status: proposed
---

# <Feature Name (title case)>

<description>

## purpose

<why this feature exists and what problem it solves>

## use cases

<concrete scenarios where this feature is used>

## failure states

<what can go wrong and how it should be handled>
```

fill in each section with substantive content derived from the description. if the description is too brief to cover a section meaningfully, write what can be inferred and leave a `<!-- TODO: expand -->` comment for the user.

8. **create `spec.md`.** use the proposal content and codebase context to draft a technical specification. follow this structure:

```markdown
# <Feature Name (title case)> — Technical Specification

## overview

<technical summary of how the feature works>

## source paths

<where the code for this feature should live, based on existing project conventions>

## implementation details

<data structures, interfaces, logic, libraries, and any other technical details>
```

infer source paths from existing project conventions (e.g. if the project uses `internal/` or `pkg/`, place new code accordingly). draft implementation details based on what the description implies — suggest concrete types, functions, and approaches where possible.

9. **update `specs/features/features.md`.** if the file does not exist, create it with a header. append a row for the new feature:

```markdown
| <feature-name> | <short description> |
```

10. **confirm creation** by outputting the paths of all files created and a brief summary of what was drafted. remind the user to review the proposal and spec, as the content is a starting point based on the provided description.
