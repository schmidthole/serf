---
name: spec-amend
description: Amend an existing feature's proposal and spec to add, remove, or change functionality. Use when iterating on an existing feature rather than creating a new one.
disable-model-invocation: true
allowed-tools: Read, Write, Edit, Glob, Grep
---

## reference

read [development-workflow.md](../development-workflow.md) for the full conventions governing the specs directory structure, features, work items, and their formats.

## instructions

amend an existing feature by updating its proposal and spec files and recording the amendment for historical context. expects `$ARGUMENTS` in the format:

```
/spec-amend <feature-name> <description of what to add, remove, or change>
```

the description can be as brief or detailed as the user wants. use whatever context is provided to determine the scope of the amendment.

follow these steps:

1. **locate the feature.** glob for `specs/features/<feature-name>/`. if the feature does not exist, inform the user and suggest `/spec-new` instead.

2. **read the current state.** read the feature's `proposal.md` and `spec.md` in full. understand the existing purpose, use cases, failure states, and technical details before making any changes.

3. **read the codebase context.** if `spec.md` has a `## Source Paths` section, read the code at those paths to understand the current implementation. also read `specs/features/features.md` and the project's `README.md` for broader context.

4. **classify the amendment.** based on the description, determine what type of change this is:
   - **addition**: new functionality, use cases, or technical components being added to the feature
   - **modification**: existing functionality being changed in behavior, interface, or implementation
   - **removal**: functionality being dropped from the feature

5. **present a summary.** before making any changes, output:
   - the amendment type (addition, modification, removal)
   - what will change in `proposal.md` (new/updated use cases, failure states, purpose changes)
   - what will change in `spec.md` (new/updated source paths, data structures, logic)
   - any sections that will be removed or replaced

   ask the user for approval or adjustments before proceeding.

6. **update `proposal.md`.** apply the amendment to the proposal:
   - for **additions**: add new use cases and failure states. expand the purpose section if the feature's scope is growing. do not remove existing content that is still valid.
   - for **modifications**: update the affected use cases, failure states, or purpose description in place. preserve content that is unchanged.
   - for **removals**: remove the relevant use cases and failure states. update the purpose section if the feature's scope is shrinking. if a removed item had associated failure states, remove those too.
   - do not change the `status` frontmatter unless the user explicitly requests it.

7. **update `spec.md`.** apply the amendment to the technical specification:
   - for **additions**: add new source paths, data structures, interfaces, and implementation details. place new content in the appropriate existing sections rather than appending to the end.
   - for **modifications**: update the affected technical details in place. if an implementation approach is changing, update the relevant subsection rather than adding a parallel description.
   - for **removals**: remove the relevant technical details and source paths. if removing a component that other parts of the spec reference, update those references.
   - keep the spec internally consistent after edits.

8. **create the amendment record.** ensure `specs/features/<feature-name>/amendments/` exists. create a file at:

   ```
   specs/features/<feature-name>/amendments/YYYY-MM-DD-concise-description.md
   ```

   using today's date and a concise description of the amendment. the file should follow this format:

   ```markdown
   ---
   date: YYYY-MM-DD
   type: addition | modification | removal
   summary: one-line summary of what changed
   ---

   ## description

   <what was amended and why, written for someone reading this months later>

   ## changes

   ### proposal.md
   - <bullet list of what changed in the proposal>

   ### spec.md
   - <bullet list of what changed in the spec>
   ```

   the amendment record is a historical artifact. it does not need to be exhaustive, just enough to understand what changed and why without diffing the files.

9. **confirm the amendment** by outputting:
   - paths of all files modified and created
   - a brief summary of what changed
   - a reminder that work items should be created via `/spec-plan` if implementation is needed
