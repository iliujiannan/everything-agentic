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

### Option A: Interactive Setup (Recommended)

1. Install the `ear` plugin from this repository
2. In your project, ask Claude to run `subagent-setup` agent
3. Answer the interactive questions (project name, tech stack, SonarQube, K8s, etc.)
4. All config files are generated automatically

### Option B: Manual Setup

1. Pick a domain directory (e.g., `rd/`)
2. Copy agents to `.claude/agents/` in your project
3. Copy rules to `~/.claude/rules/`
4. Copy `rd/templates/CLAUDE.md` to your project root and fill in the placeholders
5. Copy `rd/templates/tech-stack.md` to `.claude/docs/tech-stack.md` and describe your stack

### Workflow

The `rd/` harness ships a complete **Orchestrator + Subagent** R&D workflow:

```
workflow-orchestrator  →  subagent-plan
                       →  subagent-design-docs
                       →  subagent-code
                       →  subagent-test
                       →  subagent-sonar       (loop ≤ 3)
                       →  subagent-git
                       →  subagent-deploy      (Docker build + K8s deploy)
```

Each stage runs in an isolated context window, stops after completing, and waits for `[继续]` before the next stage starts. See [`rd/docs/workflow-architecture.md`](rd/docs/workflow-architecture.md) for the full design.

### Commands

| Command | Description |
|---------|-------------|
| `/rd-plan <feature>` | Full pipeline entry (8 stages) |
| `/tech-design` | Phase 1 — Technical design |
| `/design-docs` | Phase 2 — API & DB design documents |
| `/code-impl` | Phase 3 — Coding implementation |
| `/unit-test` | Phase 4 — Unit testing (≥80% coverage) |
| `/sonar` | Phase 5 — SonarQube scan (≤3 loops) |
| `/git-commit` | Phase 6 — Git commit |
| `/docker-build` | Phase 7a — Docker image build & push |
| `/k8s-deploy` | Phase 7b — K8s deployment |
| `/bugfix <desc>` | Lightweight bug fix workflow |
| `/sonarfix [module]` | Sonar-specific fix (minimal) |
| `/sonar-my-bugs` | SonarQube bug query & fix (interactive) |

### Skills

| Skill | Description |
|-------|-------------|
| `sonar-my-bugs` | Query & fix SonarQube bugs by author/project, with interactive config |
| `project-setup` | Interactive project onboarding (generates CLAUDE.md, tech-stack.md, etc.) |

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
