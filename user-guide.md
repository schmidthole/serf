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

### 3. Plan the work

Read the proposal and spec, then look at the current codebase. Identify what exists and what needs to change to satisfy the spec. Create work items for each coherent unit of work.

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

### 4. Implement

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

### 5. Check for drift

Over time, code and specs can fall out of sync. Periodically review each feature:

1. Read the spec's source paths. Do they all still exist?
2. Read the code at those paths. Does it match what the spec describes?
3. Look for code related to the feature that isn't listed in the source paths.

If drift is found, update the spec to match the code. The spec should always reflect reality — it is the source of truth for what the system *does*, not what it was *planned* to do.

> **Plugin equivalent:** `/serf:spec-check`

## Quick reference

### Feature files

| File | Purpose |
|---|---|
| `specs/features/features.md` | index of all features with short descriptions |
| `specs/features/<name>/proposal.md` | high-level description: purpose, use cases, failure states |
| `specs/features/<name>/spec.md` | technical specification: source paths, data structures, logic |

### Work item files

| File | Purpose |
|---|---|
| `specs/work/active/<date>-<name>.md` | work in progress or ready to start |
| `specs/work/archive/<date>-<name>.md` | completed work |

### Status values

| Context | Values |
|---|---|
| feature (`proposal.md` frontmatter) | `proposed`, `in-progress`, `implemented` |
| work item (frontmatter) | `ready`, `in-progress`, `blocked`, `done` |

### Rules of thumb

- no code change without a work item, no work item without a feature reference
- source paths are the anchor between specs and code — keep them current
- update specs when archiving work items, not continuously
- the log section is a breadcrumb trail — write for the person who picks this up next
- when in doubt, check the spec before changing code
