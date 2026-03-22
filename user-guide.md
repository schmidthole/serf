# Serf User Guide

This guide walks through the spec-driven development workflow that serf implements. Everything here can be done manually with a text editor and filesystem — the serf plugin simply automates these steps for Claude Code users.

## Core concepts

Serf organizes development around three ideas:

1. **Features** describe what the system does. They are the source of truth.
2. **Work items** describe what needs to be done to build or change features. They are an append-only log of development activity.
3. **Source paths** link features to the code that implements them.

All of this lives in plain markdown files inside a `specs/` directory. No database, no external tooling.

## Directory structure

```
your-project/
├── specs/
│   ├── features/
│   │   ├── features.md              # index of all features
│   │   ├── configuration/
│   │   │   ├── proposal.md          # what and why
│   │   │   └── spec.md              # how (technical details)
│   │   └── authentication/
│   │       ├── proposal.md
│   │       └── spec.md
│   └── work/
│       ├── active/                   # current work
│       │   └── 2026-03-08-add-config-loader.md
│       └── archive/                  # completed work
│           └── 2026-03-01-scaffold-project.md
└── ...
```

In monorepo or multi-service layouts, each service maintains its own `specs/` directory (e.g. `services/auth/specs/`). The structure is identical regardless of where it lives.

## Step-by-step workflow

### 1. Initialize the specs directory

Create the directory structure:

```bash
mkdir -p specs/features specs/work/active specs/work/archive
```

Create the feature index at `specs/features/features.md`:

```markdown
# Features

| Feature | Description |
|---|---|
```

This is the entrypoint for anyone (human or agent) orienting on the project.

> **Plugin equivalent:** `/serf:spec-init`

### 2. Define a new feature

Create a directory under `specs/features/` with a descriptive kebab-case name:

```bash
mkdir specs/features/configuration
```

#### Write the proposal

Create `specs/features/configuration/proposal.md`. This describes the feature at a high level — what it does, why it exists, and how it can fail.

```markdown
---
status: proposed
---

# Configuration

Serf uses toml for its configuration language. By default serf looks
for configuration files in `./serf.toml` and `/etc/serf/serf.toml`.

## purpose

Provide a centralized, file-based configuration that controls runtime
behavior without recompilation.

## use cases

- User places a `serf.toml` in the project root for project-specific settings.
- System administrator places a `serf.toml` in `/etc/serf/` for machine-wide defaults.
- User overrides the config path via the `-c` or `--config` cli flag.

## failure states

- No config file found at any location and none specified via flag — halt
  execution with a descriptive log message.
- Config file found but contains invalid toml — halt with a parse error
  indicating the line number.
- Config file found but missing required fields — halt with a message
  listing the missing fields.
```

The `status` field in the frontmatter tracks where the feature is in its lifecycle:

| Status | Meaning |
|---|---|
| `proposed` | described but no implementation work has started |
| `in-progress` | active work items exist targeting this feature |
| `implemented` | complete and reflected in code |

#### Write the technical spec

Create `specs/features/configuration/spec.md`. This is the technical blueprint built from the proposal — concrete enough to implement from.

```markdown
# Configuration — Technical Specification

## overview

Configuration is loaded at startup via the `internal/config` package.
The loader checks paths in priority order: cli flag, `./serf.toml`,
`/etc/serf/serf.toml`. The first file found is parsed and validated.

## source paths

- `internal/config/` — configuration loading, parsing, and validation
- `cmd/serf/main.go` — cli flag definition and config path passing

## implementation details

### config struct

The config is deserialized into a `Config` struct:

- `general.workspace_dir` (string, required) — sandboxed working directory
- `general.heartbeat_seconds` (int, default 600) — seconds between wakeups
- `llm.provider` (string, required) — the llm provider name
- `llm.api_key` (string, required) — provider api key
- `llm.high_model` (string, required) — model for complex tasks
- `llm.low_model` (string, required) — model for simple tasks

### loading logic

1. Check if `-c`/`--config` flag was provided. If so, use that path.
2. Check `./serf.toml`. If it exists, use it.
3. Check `/etc/serf/serf.toml`. If it exists, use it.
4. If no config found, log an error and exit.

### validation

After parsing, validate that all required fields are present and
non-empty. Return a structured error listing all missing fields.
```

The **source paths** section is critical — it maps this feature to the actual code. This is how anyone (human or agent) navigates from a spec to the implementation and back.

#### Update the index

Add a row to `specs/features/features.md`:

```markdown
| configuration | toml-based runtime configuration with file discovery and cli override |
```

> **Plugin equivalent:** `/serf:spec-new configuration "toml-based runtime configuration..."`

### 3. Amend an existing feature

Features rarely stay static. When you need to add, change, or remove functionality from an existing feature, amend it rather than creating a new feature.

**When to amend instead of creating a new feature:**

- The change extends an existing feature's scope (e.g. adding hot-reload to configuration)
- The change modifies how an existing feature behaves (e.g. switching config format from toml to yaml)
- The change removes part of an existing feature (e.g. dropping support for a legacy auth method)
- The new functionality would share source paths with an existing feature

If you're unsure, amend. It is easier to split a feature later than to consolidate fragmented specs.

#### Update the proposal and spec

Read the existing `proposal.md` and `spec.md`, then update them in place to reflect the amended state. The proposal and spec should always represent the feature as it currently is (or will be once implemented), not as it was originally conceived.

For additions, add new use cases, failure states, and technical details alongside the existing ones. For modifications, update the affected sections. For removals, delete the relevant content.

#### Record the amendment

Create an `amendments/` directory under the feature (if it does not already exist) and add a record:

```
specs/features/configuration/amendments/2026-03-22-add-hot-reload.md
```

```markdown
---
date: 2026-03-22
type: addition
summary: add hot-reload support for configuration changes
---

## description

Added the ability for the application to detect configuration file
changes at runtime and reload without a restart. This was requested
to reduce downtime during configuration updates in production.

## changes

### proposal.md
- added hot-reload use case under use cases
- added failure state for reload with invalid config

### spec.md
- added `internal/config/watcher.go` to source paths
- added file watcher implementation details
- added reload validation logic
```

The amendment record is a historical breadcrumb. The canonical state is always `proposal.md` and `spec.md`.

After amending, create work items via the planning step if implementation is needed.

> **Plugin equivalent:** `/serf:spec-amend configuration "add hot-reload support for config changes"`

### 4. Plan the work

Read the proposal and spec (including any recent amendments), then look at the current codebase. Identify what exists and what needs to change to satisfy the spec. Create work items for each coherent unit of work.

Guidelines for scoping work items:

- Prefer fewer, well-scoped items over many granular ones.
- Each item should be independently completable.
- Order them logically — foundational work first, dependent work after.
- Check for existing active work items to avoid duplication.

#### Create a work item

Create a file in `specs/work/active/` following the naming convention:

```
YYYY-MM-DD-concise-description.md
```

Example: `specs/work/active/2026-03-08-implement-config-loader.md`

```markdown
---
feature: configuration
status: ready
assignee:
start_date: 2026-03-08
archive_date:
keywords: "config,toml,loader,validation"
---

## summary

Implement the configuration loading package as described in the
configuration spec. This includes the config struct, file discovery
logic, toml parsing, and validation. The cli flag for specifying
a config path is handled in a separate work item.

References: `specs/features/configuration/spec.md`

## tasks

- [ ] 1.0 create `internal/config/config.go` with the Config struct
- [ ] 2.0 implement file discovery logic (flag path, ./serf.toml, /etc/serf/serf.toml)
- [ ] 3.0 implement toml parsing into the Config struct
- [ ] 4.0 implement validation for required fields
- [ ] 5.0 write unit tests for loader, parser, and validator
- [ ] 6.0 add error handling for missing config, invalid toml, and missing fields

## log

```

Work item statuses:

| Status | Meaning |
|---|---|
| `ready` | defined and available for pickup |
| `in-progress` | actively being worked on |
| `blocked` | cannot proceed — reason should be noted in the log |
| `done` | complete, ready to archive |

When you create the first work item for a feature, update the feature's `proposal.md` status from `proposed` to `in-progress`.

> **Plugin equivalent:** `/serf:spec-plan configuration`

### 5. Implement

Pick up a work item and start building.

#### Start work

1. Choose a work item from `specs/work/active/`. Prefer items with status `ready`, oldest first.
2. Set its frontmatter `status` to `in-progress`.
3. Read the associated feature's proposal and spec to understand the full context.
4. Read the code at the spec's source paths to understand what exists.

#### Work through the tasks

For each task:

1. Implement the change.
2. Run tests, linters, and formatters.
3. Mark the task `[x]` in the work item.
4. Add a log entry noting what was done and any decisions made.

If you discover additional work is needed during implementation, add new tasks to the list and log why. This keeps the work item accurate and helps anyone who picks it up later.

#### Resume interrupted work

If work is interrupted (end of day, context switch, agent session timeout), the state is preserved:

- The work item's `status: in-progress` signals it is active.
- Completed tasks are marked `[x]`.
- The log contains context on what was done and why.

To resume, read the log and task list to understand where things left off, then continue from the first incomplete task.

#### Verify against the spec

Before considering a work item complete, re-read the feature's proposal and spec. Check:

- Does the code satisfy each **use case** described in the proposal?
- Does the code handle each **failure state** described in the proposal?
- Does the code match the **technical details** described in the spec?

Fix any gaps that are within scope. Log any gaps that are out of scope.

#### Complete and archive

1. Confirm all tasks are marked `[x]`.
2. Run the full test suite and formatter.
3. Update the work item frontmatter: set `status: done` and `archive_date` to today.
4. If the implementation diverged from the spec, update the spec to match reality and note the changes in the log.
5. Move the file from `specs/work/active/` to `specs/work/archive/`.
6. Check if any active work items remain for this feature. If not, and the feature is complete, update its `proposal.md` status to `implemented`.

> **Plugin equivalent:** `/serf:spec-implement`

### 6. Check for drift

Over time, code and specs can fall out of sync. Periodically review each feature:

1. Read the spec's source paths. Do they all still exist?
2. Read the code at those paths. Does it match what the spec describes?
3. Look for code related to the feature that isn't listed in the source paths.

If drift is found, update the spec to match the code. The spec should always reflect reality — it is the source of truth for what the system *does*, not what it was *planned* to do.

> **Plugin equivalent:** `/serf:spec-check`

### 7. Consolidate fragmented specs

As a project grows, specs can fragment. Common patterns:

- A feature was amended by creating a new feature directory instead of updating the existing one (e.g. "configuration" and "config-hot-reload" as separate features)
- A feature was split across multiple directories that should be one (e.g. "auth-login" and "auth-session")
- A feature's source paths overlap with another feature
- A feature's source paths no longer exist in the codebase

Periodic consolidation merges these into unified features that reflect the actual product.

#### Analyze

Review all features, their source paths, and the codebase. Look for:

1. **Overlapping source paths** across features
2. **Subset features** whose scope is entirely within another feature
3. **Superseded features** replaced by newer ones
4. **Orphaned features** whose code no longer exists

#### Merge

For each consolidation:

1. Choose a target feature (the one with broader scope or the more established name).
2. Rewrite the target's `proposal.md` and `spec.md` to cover the full consolidated scope. This is a rewrite, not concatenation.
3. Move any amendment records from source features into the target's `amendments/` directory.
4. Create a consolidation amendment record in the target documenting what was merged.
5. Mark source features as consolidated:

```yaml
---
status: consolidated
consolidated_into: configuration
---
```

6. Remove consolidated features from `features.md`.
7. Update any active work items that referenced the source feature.

Source feature directories are preserved (not deleted) so that archived work item references remain valid.

#### Retire

For features whose code no longer exists, update their status to `retired` and remove them from `features.md`.

> **Plugin equivalent:** `/serf:spec-consolidate`

## Quick reference

### Feature files

| File | Purpose |
|---|---|
| `specs/features/features.md` | index of all features with short descriptions |
| `specs/features/<name>/proposal.md` | high-level description: purpose, use cases, failure states |
| `specs/features/<name>/spec.md` | technical specification: source paths, data structures, logic |
| `specs/features/<name>/amendments/<date>-<desc>.md` | historical record of changes to the feature |

### Work item files

| File | Purpose |
|---|---|
| `specs/work/active/<date>-<name>.md` | work in progress or ready to start |
| `specs/work/archive/<date>-<name>.md` | completed work |

### Status values

| Context | Values |
|---|---|
| feature (`proposal.md` frontmatter) | `proposed`, `in-progress`, `implemented`, `consolidated`, `retired` |
| work item (frontmatter) | `ready`, `in-progress`, `blocked`, `done` |

### Rules of thumb

- no code change without a work item, no work item without a feature reference
- source paths are the anchor between specs and code — keep them current
- update specs when archiving work items, not continuously
- the log section is a breadcrumb trail — write for the person who picks this up next
- when in doubt, check the spec before changing code
