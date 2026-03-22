---
name: spec-check
description: Check for drift between feature specs and code. Use when verifying specs are in sync with implementation.
disable-model-invocation: true
allowed-tools: Read, Edit, Glob, Grep
---

## reference

read [development-workflow.md](../development-workflow.md) for the full conventions governing the specs directory structure, features, work items, and their formats.

## instructions

check for drift between feature specs and their implementation. `$ARGUMENTS` is an optional feature name to check a single feature. if omitted, check all features.

follow these steps:

1. **locate features.** read `specs/features/features.md` to get the list of features. if a specific feature was provided, filter to just that one.

2. **for each feature with status `in-progress` or `implemented`** (skip features with status `consolidated` or `retired`):

   a. read the `spec.md` and extract the `## Source Paths` section.

   b. if no source paths are listed, flag it as "missing source paths".

   c. for each source path listed, verify it exists in the codebase using glob. flag any paths that do not exist as "stale source path".

   d. read the spec's technical details (data structures, function signatures, config fields, behavior descriptions) and compare against the actual code at the source paths. flag any divergences as "spec drift".

3. **check for orphaned code.** look for source files that implement feature-related logic but are not listed in any feature's source paths. this is a best-effort heuristic check, not exhaustive.

4. **output a report** in this format:

```
## spec check results

### <feature-name> — <status>

- [x] source paths exist
- [ ] spec matches code — <brief description of drift>

### <feature-name> — <status>

- [ ] missing source paths
```

if everything is in sync, output a clean bill of health.

5. **offer to align.** if any drift was detected, ask the user whether they would like to update the feature specs to match the current code. if approved:
   - update `spec.md` to reflect the actual implementation, preserving existing structure and writing style
   - add or remove source paths as needed
   - do not remove spec context that is still accurate
   - present each proposed change before making it

only make changes the user explicitly approves.
