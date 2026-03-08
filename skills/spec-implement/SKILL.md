---
name: spec-implement
description: Implement the next active work item or a specified one. Use when ready to write code for a planned work item.
disable-model-invocation: true
---

## reference

read [development-workflow.md](../development-workflow.md) for the full conventions governing the specs directory structure, features, work items, and their formats.

## instructions

pick up and implement a work item, or resume one that is already in progress. `$ARGUMENTS` is an optional work item filename. if omitted, first look for an `in-progress` item to resume, then fall back to the next `ready` item (oldest by date).

```
/spec-implement
/spec-implement 2026-03-08-implement-config-loader.md
```

follow these steps:

### setup

1. **select the work item.** if a filename is provided, locate it in `specs/work/active/`. if not provided, glob for all files in `specs/work/active/`, read their frontmatter, and:
   - if any items have `status: in-progress`, select the oldest one (this is a resume)
   - otherwise, select the oldest item with `status: ready`
   - if no ready or in-progress items exist, inform the user

2. **read the work item.** understand the summary, tasks, and any existing log entries.

3. **read the feature context.** using the `feature` field from the frontmatter, read:
   - `specs/features/<feature>/proposal.md` — for the high-level intent
   - `specs/features/<feature>/spec.md` — for the technical details and source paths

4. **read the current code.** using the source paths from the spec, read existing code to understand what is already in place.

5. **mark the work item as in-progress.** update the frontmatter `status` to `in-progress` (if resuming, it will already be set).

6. **if resuming, review previous progress.** read the existing log entries and check which tasks are already marked `[x]`. understand what was previously done and pick up from the first incomplete task.

### implementation

7. **work through the tasks in order.** for each incomplete task in the tasks list:
   - implement the change described by the task
   - run any relevant tests, linters, or formatters as appropriate for the project
   - mark the task as complete (`[x]`) in the work item file
   - add a brief log entry noting what was done and any decisions made
   - if during implementation you discover additional work is required (e.g. a missing edge case, a needed refactor, an undocumented dependency), add new tasks to the tasks list in the work item file and log why they were added. this ensures the work item remains an accurate record of what was actually needed and helps future agents if the work is resumed.

8. **follow project conventions.** respect the coding style, directory structure, and patterns already established in the codebase. consult the project's `README.md` and any build/test/format commands it describes.

9. **update source paths.** if the implementation creates new files or directories that should be tracked by the feature spec, add them to the `## Source Paths` section in `spec.md`.

### verification

10. **verify all tasks are complete.** re-read the work item and confirm every task is marked `[x]`.

11. **run tests and formatting.** run the project's test suite and formatter to ensure everything passes.

12. **verify against the spec.** re-read the feature's `proposal.md` and `spec.md`. check the implementation against:
   - **use cases**: does the code satisfy each use case described in the proposal?
   - **failure states**: does the code handle the failure scenarios described in the proposal?
   - **technical details**: does the code match the data structures, interfaces, and behavior described in the spec?

   if any gaps are found, log them and either fix them (if within scope of the work item) or inform the user of the discrepancy.

### completion

13. **update the work item frontmatter.** set `status: done` and `archive_date` to today's date.

14. **move the work item** from `specs/work/active/` to `specs/work/archive/`.

15. **check feature completion.** glob for any remaining active work items targeting this feature. if none remain, ask the user whether the feature is fully implemented. if yes, update the feature's `proposal.md` status to `implemented`.

16. **summarize what was done.** output a brief summary of the implementation, any decisions logged, and the final state.

### guidelines

- if a task is ambiguous, consult the spec first. if still unclear, ask the user before guessing.
- if a task cannot be completed due to a blocker, log the issue, set the work item status to `blocked`, and inform the user rather than skipping it.
- keep log entries concise but useful — they serve as breadcrumbs for future readers.
- do not modify code outside the scope of the work item's tasks without asking the user.
