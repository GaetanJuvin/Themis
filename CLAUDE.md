# Themis

QA contract framework for protecting pages, features, and components during vibe coding.

## Project Structure

- `.claude-plugin/` — Claude Code plugin config (plugin.json, hooks.json)
- `skills/` — Themis skills (SKILL.md files)
- `commands/` — User-facing slash commands
- `templates/` — Template files used by /themis-init
- `docs/plans/` — Design docs and implementation plans

## Skills

Each skill lives in `skills/<skill-name>/SKILL.md`. Skills are the AI logic.

| Skill | Command | Purpose |
|-------|---------|---------|
| `themis-init` | `/themis-init` | Interactive project setup |
| `themis-generate` | `/themis-generate` | Create contracts from prompts/docs/screenshots |
| `themis-check` | `/themis-check` | Validate changes against locked contracts |
| `themis-amend` | `/themis-amend` | Propose contract updates after violations |
| `themis-scan` | `/themis-scan` | Auto-generate baseline contracts from existing code |

## Contract Format

Contracts are `.contract.md` files with YAML frontmatter + Visual/Behavior/Data/Rules sections. See `docs/plans/2026-03-15-themis-design.md` for full spec.

## Development

- Follow the design doc at `docs/plans/2026-03-15-themis-design.md`
- Skills must have `name` and `description` in frontmatter only (max 1024 chars total)
- Skill descriptions start with "Use when..."
- Commands reference skills with "Invoke the themis:<skill-name> skill"
