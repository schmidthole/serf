# Plan: Use Native Plan Mode for Spec Creation

## Goal

Separate proposal creation from spec creation, and leverage Claude Code's native plan mode for the spec drafting step. This gives users a built-in review/approval flow when going from proposal.md → spec.md.

## Current Flow

```
/serf:spec-new config "description"   → creates BOTH proposal.md AND spec.md
/serf:spec-plan config                → reads spec, creates work items
/serf:spec-implement                  → implements work items
```

## New Flow

```
/serf:spec-new config "description"   → creates ONLY proposal.md + directory + features.md
  ↓ user reviews/edits proposal
/plan                                 → user enters plan mode
/serf:spec-draft config               → reads proposal + codebase, writes spec.md, exits plan mode for approval
  ↓ user reviews/approves spec via plan mode UI
/serf:spec-plan config                → reads approved spec, creates work items
/serf:spec-implement                  → implements work items
```

## Changes

### 1. Simplify `skills/spec-new/SKILL.md`

- Remove step 7 (spec.md creation) entirely
- Remove spec.md template/structure from the skill
- Add a closing note: "remind the user to review the proposal and then use plan mode with `/serf:spec-draft` to create the technical specification"
- Allowed tools stays: Read, Write, Edit, Glob (no change)

### 2. New `skills/spec-draft/SKILL.md`

Create a new plan-mode-oriented skill:

- **Input:** `$ARGUMENTS` = feature name
- **Allowed tools:** Read, Write, Edit, Glob, Grep
- **Steps:**
  1. Read `specs/features/<feature>/proposal.md` — fail if missing
  2. Read project README.md and existing features for context
  3. Read existing codebase to infer conventions (project structure, patterns)
  4. If spec.md already exists, read it and treat as a revision
  5. Write `specs/features/<feature>/spec.md` with the standard structure (overview, source paths, implementation details)
  6. Call ExitPlanMode to present the spec for user approval
- **Key point:** This skill is designed to be invoked during plan mode. The spec.md is the plan. ExitPlanMode triggers the review UI.
- If invoked outside plan mode, it still works — just writes spec.md and asks the user to review.

### 3. Minor update to `skills/spec-plan/SKILL.md`

- In step 1, add: if spec.md does not exist, inform the user to run `/serf:spec-draft` first.
- No other changes needed.

### 4. Update `skills/development-workflow.md`

- In the Features section, add a note that spec.md is created separately from proposal.md using plan mode for review.
- No structural changes to conventions.

### 5. Update `user-guide.md`

- Split "Step 2: Define a new feature" into two sub-steps:
  - 2a: Write the proposal (via `/serf:spec-new` or manually)
  - 2b: Draft the technical spec (via plan mode + `/serf:spec-draft` or manually)
- Update the plugin equivalent references.

### 6. Update `README.md`

- Add `/serf:spec-draft` to the typical workflow
- Add a skills section entry for spec-draft
- Update the spec-new description to say "proposal" instead of "proposal and spec"

## Files Changed

| File | Action |
|------|--------|
| `skills/spec-new/SKILL.md` | Edit — remove spec.md generation |
| `skills/spec-draft/SKILL.md` | Create — new plan-mode skill |
| `skills/spec-plan/SKILL.md` | Edit — add spec.md existence check |
| `skills/development-workflow.md` | Edit — add plan mode note |
| `user-guide.md` | Edit — update step 2 |
| `README.md` | Edit — add spec-draft, update descriptions |
