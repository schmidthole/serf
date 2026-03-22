---
name: spec-consolidate
description: Identify and merge fragmented or overlapping feature specs into unified features that reflect the actual product. Use when specs have grown disjointed over time.
disable-model-invocation: true
allowed-tools: Read, Write, Edit, Glob, Grep
---

## reference

read [development-workflow.md](../development-workflow.md) for the full conventions governing the specs directory structure, features, work items, and their formats.

## instructions

analyze the project's feature specs for fragmentation, overlap, and drift from the actual codebase, then consolidate them into unified features that accurately represent the product. `$ARGUMENTS` is an optional feature name to scope the analysis to a single feature and its related specs. if omitted, analyze all features.

```
/spec-consolidate
/spec-consolidate configuration
```

follow these steps:

### analysis

1. **read all features.** read `specs/features/features.md` and then read every feature's `proposal.md` and `spec.md`. build a mental map of:
   - each feature's purpose and scope
   - each feature's source paths
   - each feature's status

2. **read the codebase.** for every source path referenced across all features, verify the path exists and read enough of the code to understand what it implements. also look for source files that are not referenced by any feature.

3. **read archived work items.** glob for `specs/work/archive/*.md` and read their frontmatter and summaries. this reveals how features evolved over time and which work items amended existing features versus building new ones.

4. **identify consolidation candidates.** look for these patterns:

   - **overlapping source paths**: two or more features listing the same source paths or directories. this usually means one feature was created to amend another but was written as a separate feature instead.
   - **subset features**: a feature whose entire scope is a subset of another feature's scope (e.g. "config-hot-reload" is really part of "configuration").
   - **split features**: a single logical capability that was split across multiple feature directories (e.g. "auth-login" and "auth-session" that should be one "authentication" feature).
   - **superseded features**: a newer feature that replaces an older one without the older one being updated or removed.
   - **orphaned features**: features with status `implemented` whose source paths no longer exist, or features that no longer correspond to anything in the codebase.

5. **identify untracked code.** find source files or directories that implement meaningful functionality but are not covered by any feature's source paths. note these as candidates for new features or additions to existing features.

### proposal

6. **present findings.** output a report organized by category:

   ```
   ## consolidation report

   ### overlapping features
   - <feature-a> and <feature-b> both cover `internal/config/` — recommend merging into <feature-a>

   ### subset features
   - <feature-c> is a subset of <feature-d> — recommend merging into <feature-d>

   ### orphaned features
   - <feature-e> source paths no longer exist — recommend removal or update

   ### untracked code
   - `internal/middleware/` is not covered by any feature — recommend adding to <feature-f> or creating a new feature
   ```

   for each consolidation, propose a specific action:
   - **merge**: combine two or more features into one, with a recommended target feature name
   - **absorb**: fold a smaller feature into a larger one
   - **retire**: mark a feature as retired when it no longer applies
   - **update**: update a feature's spec to match current code without merging

7. **get approval.** ask the user to approve, adjust, or skip each proposed action. do not proceed without explicit approval for each consolidation.

### execution

8. **for each approved merge/absorb:**

   a. **choose the target feature.** this is the feature directory that will survive. prefer the feature with the broader scope or the more established name.

   b. **rewrite `proposal.md`.** combine the purpose, use cases, and failure states from all source features into the target feature's proposal. this is a rewrite, not a concatenation. the result should read as a single coherent proposal covering the full scope of the consolidated feature. preserve the target feature's status (use the most advanced status among the merged features).

   c. **rewrite `spec.md`.** combine the technical details from all source features into the target feature's spec. merge source paths (deduplicate), merge implementation details, and resolve any contradictions. the result should be a single coherent spec.

   d. **preserve amendment history.** if any source feature has an `amendments/` directory, move those amendment records into the target feature's `amendments/` directory. if the source feature itself was the result of an amendment to the target, create an amendment record in the target documenting the consolidation.

   e. **create a consolidation amendment record** in the target feature's `amendments/` directory:

   ```markdown
   ---
   date: YYYY-MM-DD
   type: modification
   summary: consolidated <source-features> into <target-feature>
   ---

   ## description

   Merged the following features into this feature as part of spec consolidation:
   - <source-feature-a>: <brief reason>
   - <source-feature-b>: <brief reason>

   ## changes

   ### proposal.md
   - <what was added/changed from each source feature>

   ### spec.md
   - <what was added/changed from each source feature>
   ```

   f. **mark source features as consolidated.** update each source feature's `proposal.md` frontmatter to:

   ```yaml
   status: consolidated
   consolidated_into: <target-feature-name>
   ```

   leave the source feature's files in place (do not delete them) so that historical references in archived work items remain valid. remove the source feature from `features.md`.

   g. **update `features.md`.** remove entries for consolidated source features. update the target feature's description if its scope has expanded.

   h. **update work item references.** glob for any active work items that reference a consolidated source feature. update their `feature` frontmatter to point to the target feature instead.

9. **for each approved retirement:**

   a. update the feature's `proposal.md` frontmatter to `status: retired`.
   b. remove the feature from `features.md`.
   c. check for any active work items targeting this feature and inform the user.

10. **for each approved update:**

    a. update the feature's `spec.md` to match the current code.
    b. create an amendment record documenting what changed.

### summary

11. **output a summary** listing:
    - all features that were consolidated, with their target
    - all features that were retired
    - all features that were updated
    - any untracked code that still needs a feature home
    - a reminder to run `/spec-check` afterward to verify alignment
