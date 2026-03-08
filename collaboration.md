# Team Collaboration

This guide describes how teams use serf and git together to spec, plan, and implement work across a shared repository. It covers role responsibilities, branching strategy, and conventions that minimize merge conflicts and duplication.

## Roles

### Product manager (PM)

PMs own the "what" and "why." Their primary artifacts are proposals and the feature index.

- create and maintain `proposal.md` files
- review and approve `spec.md` files
- review work item status via `specs/work/active/` or `/serf:spec-status`
- review PRs that update specs to ensure they still match intent

### Developer

Developers own the "how." Their primary artifacts are specs, work items, and code.

- co-author `spec.md` files (often drafted by an agent, reviewed by PM)
- create and implement work items
- update specs when implementation diverges from the original plan
- archive work items and update feature status on completion

### Agent (Claude Code with serf)

Agents act as developers. They follow the same conventions and touch the same files. The only distinction is that agents should always check for an `assignee` field in work item frontmatter before picking up work.

## Branching strategy

Use trunk-based development with short-lived branches. The guiding principle: **specs land on main before implementation branches off.**

### Branch naming

Branch names are **strictly derived** from the artifact they carry. This makes it possible for any human or agent to determine whether a work item is in-flight by checking if a corresponding branch exists on the remote.

| Branch | Naming convention | Example | Purpose |
|---|---|---|---|
| main | `main` | `main` | stable trunk, always deployable |
| spec branch | `spec/<feature-directory-name>` | `spec/configuration` | drafting a new feature's proposal and spec |
| work branch | `work/<work-item-filename-without-extension>` | `work/2026-03-08-implement-config-loader` | implementing a single work item |

The work branch name **must match** the work item filename (minus `.md`). This 1:1 mapping means anyone can:

- see a work item file and immediately know which branch to look for on the remote
- see a remote branch and immediately know which work item file it corresponds to
- list remote branches with `git branch -r --list 'origin/work/*'` to see all in-flight work items

### Lifecycle

```
1. PM creates spec/configuration branch
2. PM writes proposal.md, spec.md, updates features.md
3. PM opens PR → team reviews → merges to main
4. developer/agent runs /serf:spec-plan on main (or a short-lived branch)
5. work items land on main via PR
6. developer creates work/2026-03-08-implement-config-loader branch from main
7. developer implements, commits code + work item updates
8. developer opens PR → review → merge to main
9. repeat step 6 for next work item
```

### Why specs merge first

If specs and implementation live on the same branch, two problems emerge:

1. **review burden** — PRs mix intent (spec) with execution (code), making review harder
2. **conflict surface** — long-lived branches that touch specs create merge conflicts when other features also update `features.md` or shared specs

By landing specs on main first, implementation branches start from an agreed-upon foundation and only touch code files plus their own work item.

### When spec and plan can share a branch

For small features where the spec is trivial and unlikely to conflict, the PM and developer may combine the spec and plan into a single `spec/<feature-name>` branch. This is a pragmatic shortcut — not the default.

## Work item ownership

Add an `assignee` field to work item frontmatter to prevent two people (or agents) from picking up the same item:

```yaml
---
feature: configuration
status: ready
assignee:
start_date: 2026-03-08
archive_date:
keywords: "config,toml,loader"
---
```

When a developer picks up a work item:

1. set `status: in-progress`
2. set `assignee: <name or identifier>`
3. create the corresponding work branch (e.g. `work/2026-03-08-implement-config-loader`)
4. commit the frontmatter change and push to the remote immediately

This makes ownership visible in two places: the work item file (via `assignee`) and the git remote (via the branch name). Either signal is enough for others to know the item is claimed.

Agents should skip any work item that already has an assignee unless explicitly told to take it over. Agents can also check for an existing remote branch matching the work item name as a secondary signal.

## Avoiding merge conflicts

### High-conflict files

These files are touched by multiple people and are the most likely source of conflicts:

- `specs/features/features.md` — the feature index
- `specs/features/<name>/proposal.md` — status field changes
- `specs/features/<name>/spec.md` — source path updates

### Conventions

1. **keep feature index updates small.** add one row per PR. do not reformat the table.
2. **do not rewrite spec files during implementation.** only update the specific sections that diverged. use targeted edits, not full rewrites.
3. **one work item per branch.** this is the most important rule. a branch should never contain changes for multiple work items.
4. **commit spec updates in a dedicated commit.** within a work branch, separate spec update commits from code commits. this makes conflict resolution easier if it occurs.
5. **push to the remote often.** all parties — PMs, developers, and agents — should push their branches to the remote frequently, even if work is incomplete. this ensures progress is visible to the team, reduces the risk of lost work, and keeps remote branch state accurate for anyone checking what is in-flight. treat `git push` as part of every meaningful commit, not just a pre-PR step.
6. **rebase work branches before opening a PR.** this catches conflicts early when they are small.
7. **avoid long-lived branches.** if a work item takes more than a few days, consider splitting it.

## Reviewing work

### For PMs

PMs review at two points:

1. **spec PRs** — does the proposal capture the right problem? does the spec describe a reasonable solution?
2. **implementation PRs** — do spec updates (if any) still match intent? are work item logs clear? code review is optional depending on team norms.

PMs can check project status at any time by reading `specs/work/active/` or running `/serf:spec-status`. The feature index and proposal statuses give a high-level view. Work item logs give detailed progress.

### For developers

Developers review implementation PRs for code quality, test coverage, and spec adherence. Check that:

- work item tasks are all marked complete
- the log section explains decisions
- spec updates (if any) are accurate
- source paths are current

## Monorepo considerations

In monorepo layouts, each service has its own `specs/` directory. Cross-team coordination follows the same principles:

- spec branches are scoped to a single service (e.g. `spec/auth/oauth-flow`)
- work branches are scoped to a single work item within a service
- cross-service dependencies are noted in work item summaries with relative paths to the other service's spec
- the feature index is per-service — there is no global index

## Parallel feature development

When multiple features are in flight simultaneously:

1. each feature's spec merges to main independently
2. work items for different features can be implemented in parallel on separate branches
3. if two features share source paths, coordinate through the specs — one feature's spec should reference the other's
4. run `/serf:spec-check` after merging to detect drift caused by parallel changes

## Summary

| Principle | Rule |
|---|---|
| atomic unit | one branch = one work item = one PR |
| strict naming | branch names match artifact names — `spec/<feature>`, `work/<work-item-filename>` |
| spec before code | proposals and specs merge to main before implementation begins |
| ownership | work items have an assignee; remote branches signal in-flight work |
| push often | push to the remote after every meaningful commit, not just before PR |
| small changes | keep branches short-lived, keep edits targeted |
| separation | spec commits and code commits are separate, even on the same branch |
| sync | run spec-check after merges to catch drift |
