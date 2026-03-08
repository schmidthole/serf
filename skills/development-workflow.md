## Development Workflow

We use a "spec driven development" workflow to build the project. This allows both agents and users to understand the project, workstream, and current state. Any agent interacting with the spec files should be flexible and adaptable to the contents and structure as they may be working alongside human users.

* All specs are documented in a `./specs` directory.
* In monorepo or multi-service layouts, each service boundary may maintain its own `./specs` directory (i.e. `./services/auth/specs/`). The structure and conventions described below apply identically regardless of where the `specs/` directory lives. Cross-service references can be made via relative path in work item summaries or logs.

Any high level overview of a project or background should always be contained in a `README.md` file, which would be the canonical entrypoint to any repo/project.

### Sub Directories

Inside of the `./specs` directory will be the following sub-directories serving dedicated purposes:

* **features/**: Each sub-directory under the features directory describes a single feature/component (i.e. `./specs/features/configuration`). These features serve as the source of truth for documenting the project.
* **work/**: Describes the workstream performed to implement features. If features are the source of truth, the work stream is the log of development and decisions made to get there. This can be thought of as an append only diary of decisions and tasks.

### Features

The `./specs/features` directory serves as the source of truth documentation for a project.

* A top level `features.md` file is always present and contains a list of all sub-directory features along with a short description. This file can be used by an agent/user to search and discover features.
* Each feature is contained in its own sub-directory with a descriptive name describing the feature (i.e. `configuration/`). Features should be scoped to high level system components.

Each feature directory can contain the following files. All of these files are not required to be present.

* **proposal.md**: Describes the feature in high level detail. Will cover purpose, use cases, failure states, etc. This is used as a source of truth to build the technical specification.
* **spec.md**: This is the detailed technical specification, built from the `proposal.md`. This file covers detailed technical implementation details, such as code structure, libraries, logic, etc. It should be a natural extension of the `proposal.md` and should include enough context for a user or agent to build out the feature.

#### Feature Status

Each `proposal.md` should contain frontmatter with a `status` field indicating the feature's lifecycle stage:

```yaml
status: proposed | in-progress | implemented
```

* **proposed**: feature has been described but no implementation work has started.
* **in-progress**: active work items exist targeting this feature.
* **implemented**: feature is complete and reflected in code.

This allows agents and users to understand feature state without cross-referencing every work item.

#### Source Paths

Each `spec.md` should contain a `## Source Paths` section listing the directories and files in the codebase that implement the feature. This is the anchor that maps features to code.

```markdown
## Source Paths
- `internal/config/` — configuration loading and validation
- `cmd/serf/flags.go` — cli flag definitions
```

Source paths allow agents to navigate directly from a spec to the relevant code, and inversely, to identify which feature spec governs a given file. When modifying code under a feature's source paths, agents should consult and potentially update the corresponding spec.

### Work

The work directory can be thought of as an append only diary of the technical implementation for a project. This sits directly alongside the features and git history to serve as working documentation of the tasks undertaken to implement/amend features, design decisions made, issues encountered, etc. In a typical development team, the work directory serves as the "stories" or "tickets" assigned for development.

The work directory contains two sub-directories that each contain "work items":

* **active/**: Active work items currently ready for implementation or in progress.
* **archive/**: Work items that are completed/closed.

Keeping active and completed work items separate means that agents/users can understand what is currently relevant and reference historical work if needed.

#### Work Items

Each work item is simply a self contained markdown file. Each markdown file must follow a specific naming convention so that development order and timeline can be maintained:

```
YYYY-MM-DD-work-item-concise-description.md
```

* The date component of the file name is the date the work item is created.
* The rest of the file name should be a concise name that serves as a title or short description of the work.

This file naming structure allows agents and users to query work items by filename without reading each file. It also allows them to be ordered/indexed by implementation order so that changes can be understood.

#### Item Frontmatter

Each work item markdown file needs to contain some frontmatter in order to associate it correctly:

```yaml
feature: feature-directory # exact name of the feature directory this targets
status: ready | in-progress | blocked # current state of the work item
assignee: # name or identifier of the person/agent working on this item
start_date: YYYY-MM-DD # date matching the title date
archive_date: YYYY-MM-DD # date the work item was completed/archived
keywords: "comma,delimited,list,of,keywords" # comma delimited keywords for indexing/search
```

* **ready**: work item is defined and available for pickup.
* **in-progress**: actively being worked on by a user or agent.
* **blocked**: cannot proceed due to a dependency or issue (should be noted in the log section).

#### Item Content

Each work item markdown file should include the following sections. This is simply a guideline, as these files are meant to be flexible.

* **Summary**: Describe the work to be done, background, and why. Should be equivalent to a story description so an user/agent can pick up the item and have enough context to complete the tasks. References the feature where necessary and the required changes.
* **Tasks**: Markdown list of tasks, with adequate descriptions. The tasks should be in the form of `- [ ] X.X task description` and be marked with `[x]` after completion.
* **Log**: This is a free-form section describing the decisions made and issues encountered as the work is completed. The goal of this section is to provide enough background to hand off an in progress task to another user/agent and use as a reference later when investigating why certain approaches were taken for a given work item. Think of it as a breadcrumb for future readers.

### Keeping Specs In Sync

Specs are the source of truth, but code will inevitably diverge during implementation. The following conventions keep them aligned without adding overhead:

* **No code change without a work item, no work item without a feature reference.** This ensures every change traces back to a spec.
* **Source paths are the anchor.** When modifying files listed in a feature's source paths, agents and users should consult the relevant spec before making changes and note any divergences.
* **Specs are updated when work items are archived.** When a work item is completed and moved to `archive/`, the associated feature's `spec.md` should be updated to reflect any implementation decisions that diverged from the original specification. The work item's log should note what spec changes were made.
* **Feature status tracks lifecycle.** When the first work item targeting a feature moves to `in-progress`, the feature's status should be updated accordingly. When all work items are archived and the feature is complete, its status should move to `implemented`.
