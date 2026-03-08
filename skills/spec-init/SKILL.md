---
name: spec-init
description: Scaffold a new specs directory or align an existing one with the expected structure. Use when setting up spec-driven development in a project.
disable-model-invocation: true
allowed-tools: Read, Write, Edit, Glob, Bash
---

## reference

read [development-workflow.md](../development-workflow.md) for the full conventions governing the specs directory structure, features, work items, and their formats.

## instructions

initialize or align a specs directory with the spec-driven development structure defined in the development workflow. `$ARGUMENTS` is an optional path to a specific service directory in a monorepo (e.g. `/spec-init services/auth`). if omitted, targets the repo root.

follow these steps:

1. **determine the target root.** if `$ARGUMENTS` provides a path, use it. otherwise use the repository root.

2. **check for an existing specs directory.** glob for `<target>/specs/`. if it exists, switch to alignment mode. if not, switch to scaffold mode.

### scaffold mode (no existing specs directory)

3. **create the directory structure:**
   - `<target>/specs/features/`
   - `<target>/specs/work/active/`
   - `<target>/specs/work/archive/`

4. **create `specs/features/features.md`** with this template:

```markdown
# Features

| Feature | Description |
|---|---|
```

5. **confirm creation** by listing all directories and files created.

### alignment mode (existing specs directory)

3. **audit the current structure.** check for the presence of:
   - `specs/features/` directory
   - `specs/features/features.md` index
   - `specs/work/active/` directory
   - `specs/work/archive/` directory

4. **create any missing directories or files** using the same templates as scaffold mode.

5. **audit existing features.** glob for all subdirectories under `specs/features/`. for each:
   - check if it has a `proposal.md` — if missing, note it
   - check if `proposal.md` has `status` frontmatter — if missing, note it
   - check if it has a `spec.md` — if missing, note it
   - check if it has a `## Source Paths` section in `spec.md` — if missing, note it
   - check if the feature is listed in `features.md` — if missing, note it

6. **audit existing work items.** glob for all files in `specs/work/active/` and `specs/work/archive/`. for each:
   - check if the filename matches the `YYYY-MM-DD-description.md` convention — if not, note it
   - check if frontmatter contains `feature`, `status`, `start_date`, `archive_date`, and `keywords` fields — note any missing fields
   - check if the `feature` value references a valid feature directory — if not, note it

7. **check for orphaned spec files.** look for any files directly in `specs/` that should be under `specs/features/` (e.g. proposal or spec files at the wrong level).

8. **present a report** of all findings to the user, grouped as:
   - **created**: directories or files that were missing and have been created
   - **needs attention**: issues that require user decision (e.g. orphaned files to relocate, missing frontmatter fields to fill in, features not in the index)

9. **offer to fix automatically** where possible:
   - add missing features to `features.md`
   - add empty `status` frontmatter to proposals that lack it
   - add empty `## Source Paths` section to specs that lack it
   - add missing frontmatter fields to work items

only make changes the user approves.
