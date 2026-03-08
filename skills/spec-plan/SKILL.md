---
name: spec-plan
description: Analyze the gap between a feature spec and the current code, then create work items to close that gap. Use when planning implementation work for a feature.
disable-model-invocation: true
allowed-tools: Read, Write, Edit, Glob, Grep
---

## reference

read [development-workflow.md](../development-workflow.md) for the full conventions governing the specs directory structure, features, work items, and their formats.

## instructions

analyze the delta between a feature's desired state (its spec) and the current state of the codebase, then generate work items that describe how to close the gap. expects `$ARGUMENTS` as a feature name:

```
/spec-plan <feature-name>
```

follow these steps:

1. **read the feature spec.** load `specs/features/<feature-name>/proposal.md` and `specs/features/<feature-name>/spec.md`. if either does not exist, inform the user. both files are needed to understand the desired state.

2. **read the current code.** extract the `## Source Paths` section from `spec.md`. for each source path, read the relevant files. if no source paths exist yet, note that all implementation is net-new and use the spec's described structure to determine where code should live.

3. **read existing active work items.** glob for files in `specs/work/active/` with frontmatter referencing this feature. understand what is already planned or in progress to avoid duplicating work.

4. **identify the delta.** compare the spec's desired state against the current code and any existing work items. categorize the gaps:
   - **net-new**: functionality described in the spec with no corresponding code
   - **modification**: existing code that needs to change to match the spec
   - **removal**: existing code that the spec no longer describes or explicitly excludes

5. **present the plan.** before creating any files, output a summary of the identified gaps and the work items you propose to create. each proposed work item should include:
   - a concise title (which becomes part of the filename)
   - which gap(s) it addresses
   - a brief description of the implementation approach

   ask the user for approval or adjustments before proceeding.

6. **create work items.** after approval, for each work item:
   - ensure `specs/work/active/` and `specs/work/archive/` exist
   - create the file at `specs/work/active/YYYY-MM-DD-title.md` using today's date
   - populate frontmatter with `feature`, `status: ready`, `start_date`, empty `archive_date`, and relevant `keywords`
   - write a **summary** section that describes the gap, references the relevant parts of the spec, and explains why the change is needed
   - write a **tasks** section with a numbered checklist breaking the work into concrete steps
   - include an empty **log** section

7. **update feature status.** if the feature's `proposal.md` has `status: proposed`, update it to `status: in-progress`.

8. **confirm** by listing all created work items with their file paths.

### guidelines

- prefer fewer, well-scoped work items over many granular ones. each work item should represent a coherent unit of work that can be picked up and completed independently.
- order the work items logically — foundational changes first, dependent changes after.
- if the spec is incomplete or ambiguous, note the ambiguities in the plan summary and ask the user to clarify before creating work items.
- if existing active work items already cover part of the gap, reference them and only create items for the remaining delta.
