# Serf

Serf is a Claude Code plugin providing a spec-driven development framework. See [README.md](./README.md) for an overview and [user-guide.md](./user-guide.md) for the full workflow.

## Project Structure

This repo is a Claude Code plugin. It contains no application code — only skills, conventions, and documentation.

- `.claude-plugin/` — plugin manifest and marketplace config
- `skills/` — Claude Code skills (`/serf:spec-*` commands)
- `skills/development-workflow.md` — shared reference document used by all skills
- `user-guide.md` — human-readable walkthrough of the workflow

## Development

- all skills live in `skills/<skill-name>/SKILL.md`
- shared conventions live in `skills/development-workflow.md` and are linked from each skill via a `## reference` section
- when updating workflow conventions, edit `development-workflow.md` — do not duplicate the content into individual skills
- keep skills minimal and action-oriented — they are instructions, not documentation
- `user-guide.md` is the human-facing equivalent of the skills — keep it in sync when skills change
- test plugin changes locally with `claude --plugin-dir .`
