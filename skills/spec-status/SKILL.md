---
name: spec-status
description: Show project status overview including all features and active work items. Use when orienting on a project or checking current state.
disable-model-invocation: true
allowed-tools: Read, Glob, Grep
---

## reference

read [development-workflow.md](../development-workflow.md) for the full conventions governing the specs directory structure, features, work items, and their formats.

## instructions

provide a concise project status overview by reading the spec-driven development structure. follow these steps:

1. **locate specs directories.** glob for all `specs/features/features.md` files in the repo to find all spec roots (there may be multiple in a monorepo).

2. **for each specs root, list features.** read the `features.md` index. for each feature listed:
   - read its `proposal.md` frontmatter to get the `status` field
   - note the feature name, short description, and status
   also check for feature directories not listed in `features.md` that have `status: consolidated` or `status: retired` — report these separately if present, as they indicate historical features.

3. **list active work items.** glob for all files in `specs/work/active/*.md` under each specs root.
   - read the frontmatter of each to get `feature`, `status`, and `keywords`
   - note the filename (which contains the date and description)

4. **output a summary** in this format:

```
## features

| feature | status | description |
|---|---|---|
| configuration | in-progress | configuration loading and validation |

## active work items

| work item | feature | status |
|---|---|---|
| 2026-03-08-implement-config-loader | configuration | in-progress |
```

if no specs directory is found, inform the user that the project has not been scaffolded yet and suggest running `/spec-init` to get started.

$ARGUMENTS can optionally be a feature name to filter the output to a single feature and its associated work items.
