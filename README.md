# agents

Shared agent platform repository for Claude Code, Codex, OpenCode, and Gemini.

## Structure

- `packs/core/instructions/` - shared base instruction files
- `packs/core/agents/` - reusable role prompts
- `packs/core/checklists/` - reusable process gates
- `packs/core/templates/` - reusable templates

## Consumption model

Projects consume this repository through a local `.agent-platform` path.

- Local dev: `.agent-platform` symlink to this checkout for instant updates
- Other machines/CI: clone this repo into `.agent-platform` via bootstrap script
