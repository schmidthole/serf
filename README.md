# Serf

Serf is a spec-driven development plugin for Claude Code. It provides a set of conventions and skills that align users and agents during development by keeping feature specifications, work items, and code in sync.

## Installation

Add the marketplace and install the plugin:

```
/plugin marketplace add <github-user>/serf
/plugin install serf@serf
```

For local development:

```bash
claude --plugin-dir ./path/to/serf
```

## How it works

Serf uses a `specs/` directory structure to document features and track work. Features are the source of truth for what the project does. Work items are an append-only log of tasks undertaken to implement those features. Skills automate the creation, management, and validation of both.

See [skills/development-workflow.md](skills/development-workflow.md) for the full conventions, [user-guide.md](user-guide.md) for a step-by-step walkthrough of the workflow for humans

While Serf does not aim to dictate development lifecycle for teams, we do provide a suggested guide for how teams of humans and agents can user Serf together effectively at: [collaboration.md](collaboration.md).

### Typical workflow

```
/serf:spec-init                     # scaffold the specs directory
/serf:spec-new config "toml config" # create a new feature
/serf:spec-plan config              # generate work items from the spec
/serf:spec-implement                # implement the next work item
/serf:spec-status                   # check project status
/serf:spec-check                    # detect and fix spec drift
```

## Skills

### `/serf:spec-init`

Scaffold a new `specs/` directory or align an existing one with the expected structure. Supports monorepo layouts by targeting a specific service path.

```
/serf:spec-init
/serf:spec-init services/auth
```

### `/serf:spec-status`

Show a project status overview listing all features with their statuses and all active work items. Optionally filter to a single feature.

```
/serf:spec-status
/serf:spec-status configuration
```

### `/serf:spec-check`

Detect drift between feature specs and their code implementation. Verifies source paths exist, compares spec descriptions against actual code, and flags divergences. Offers to update the specs to match the current code if drift is found. Optionally target a single feature.

```
/serf:spec-check
/serf:spec-check configuration
```

### `/serf:spec-new`

Scaffold a new feature directory and draft its `proposal.md` and `spec.md` based on the provided description. Reads the existing codebase to infer project conventions and suggest concrete implementation details. The description can be as brief or detailed as you like.

```
/serf:spec-new configuration "toml-based config file loaded from ./serf.toml or /etc/serf/serf.toml with cli flag override"
```

### `/serf:spec-plan`

Analyze the gap between a feature's spec and the current codebase, then generate work items to close that gap. Reads the proposal and spec for the desired state, reads the code for the current state, identifies the delta, and proposes scoped work items. Presents the plan for approval before creating anything.

```
/serf:spec-plan configuration
```

### `/serf:spec-implement`

Pick up and implement the next ready work item, or specify one by name. Reads the work item, feature spec, and existing code, then works through each task — writing code, running tests, and marking tasks complete along the way. Logs decisions as it goes. Verifies the implementation against the spec before archiving. When all tasks are done, archives the work item and checks whether the feature is fully complete.

```
/serf:spec-implement
/serf:spec-implement 2026-03-08-implement-config-loader.md
```
