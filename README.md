# everything-agentic

> Enterprise Claude Code Harness Engineering Template

A production-ready template repository for integrating [Claude Code](https://claude.ai/code) into enterprise workflows. Package your team's accumulated skills, agents, hooks, and rules into reusable domain-specific harnesses.

[中文文档](./README.zh-CN.md)

## Repository Structure

This repository is organized by domain. Each domain is a self-contained harness directory.

```
everything-agentic/
├── rd/          # R&D / Software Engineering
└── ...          # More domains coming (ops, data, security, etc.)
```

## Domains

### `rd/` — Software Engineering

Claude Code harness for software R&D workflows.

| Directory | Purpose |
|-----------|---------|
| `rd/agents/` | Specialized sub-agent definitions |
| `rd/skills/` | Domain-specific skill bundles |
| `rd/commands/` | Custom slash command definitions |
| `rd/rules/` | Layered coding and workflow rules |
| `rd/hooks/` | Event-driven automation scripts |
| `rd/settings/` | Claude Code settings templates |
| `rd/templates/` | Project-level CLAUDE.md starters |
| `rd/enterprise/` | Governance, audit, and permission templates |
| `rd/docs/` | Extended documentation |

## Quick Start

1. Pick a domain directory (e.g., `rd/`)
2. Copy agents to `~/.claude/agents/`
3. Copy rules to `~/.claude/rules/`
4. Merge settings into `~/.claude/settings.json`
5. Add a `CLAUDE.md` from `rd/templates/` to your project root

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
