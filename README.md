# everything-agentic

> Enterprise Claude Code Harness Engineering Template

A production-ready template repository for integrating [Claude Code](https://claude.ai/code) into enterprise R&D workflows. Package your team's accumulated skills, agents, hooks, and rules into a reusable harness.

[中文文档](./README.zh-CN.md)

## What's Inside

| Directory | Purpose |
|-----------|---------|
| `agents/` | Specialized sub-agent definitions |
| `skills/` | Domain-specific skill bundles |
| `commands/` | Custom slash command definitions |
| `rules/` | Layered coding and workflow rules |
| `hooks/` | Event-driven automation scripts |
| `settings/` | Claude Code settings templates |
| `templates/` | Project-level CLAUDE.md starters |
| `enterprise/` | Governance, audit, and permission templates |
| `docs/` | Extended documentation |

## Quick Start

1. Browse the directories above and copy what fits your project
2. Place agents in `~/.claude/agents/`
3. Place rules in `~/.claude/rules/`
4. Merge settings into `~/.claude/settings.json`
5. Add a `CLAUDE.md` from `templates/` to your project root

## Philosophy

This harness follows a layered configuration model:

- **Rules** define standards and checklists (what to do)
- **Skills** provide deep domain knowledge (how to do it)
- **Agents** delegate specialized sub-tasks (who does it)
- **Hooks** enforce automation at lifecycle events (when to do it)
- **Commands** surface common workflows as slash commands (trigger it)

## Contributing

See [CONTRIBUTING.md](./CONTRIBUTING.md).

## License

MIT
