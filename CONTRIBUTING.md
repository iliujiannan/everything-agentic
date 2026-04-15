# Contributing

## Adding a New Skill

1. Create a directory under `skills/` with a descriptive name (kebab-case)
2. Add a `SKILL.md` with the following frontmatter:

```markdown
---
name: skill-name
description: One-line description of what this skill does
tags: [java, spring, tdd]  # optional
---

# Skill content here
```

## Adding a New Agent

Create a `.md` file under `agents/` with frontmatter:

```markdown
---
name: agent-name
description: What this agent specializes in and when to use it
model: claude-sonnet-4-6
tools: [Read, Grep, Glob, Bash]
---

# Agent system prompt here
```

## Adding a New Command

Create a `.md` file under `commands/` with frontmatter:

```markdown
---
name: command-name
description: What this slash command does
---

# Command instructions here
```

## Adding Rules

- Language-agnostic rules go in `rules/common/`
- Chinese translations go in `rules/zh/` (must mirror `common/` file list)
- Language-specific overrides go in `rules/<language>/`
- Language-specific rules take precedence over common rules

## Commit Convention

```
feat: add springboot-security skill
fix: correct java rule constructor injection example
docs: update enterprise onboarding checklist
```
