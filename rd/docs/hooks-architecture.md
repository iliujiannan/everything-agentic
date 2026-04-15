# Hooks Architecture

## Overview

Hooks are JavaScript scripts that run at Claude Code lifecycle events (before/after tool calls, session start/end, etc.). The hook system is powered by ECC's `run-with-flags.js` dispatcher.

## How It Works

```
Claude Code lifecycle event
        ↓
settings.json hooks entry (matcher: "Bash|Edit|Write|...")
        ↓
run-with-flags.js <flag> <script> <profile>
        ↓
Target hook script executes
        ↓
Result passed back to Claude Code
```

**Flag**: Identifies which hook is firing (e.g., `post:quality-gate`, `pre:bash:commit-quality`)
**Script**: Path to the actual hook implementation (relative to ECC root)
**Profile**: `minimal | standard | strict` — controls whether the hook fires

## Profiles

| Profile | When it fires |
|---------|--------------|
| `minimal` | Always (for all users) |
| `standard` | Standard + minimal |
| `strict` | Strict + standard + minimal |

Profile is determined by `settings.json` hook config or the `STRICT_MODE` env var.

## Adding ear-Specific Hooks

When you need an ear-specific hook (e.g., Java build quality gate):

**Option A — Add to settings.json (recommended for global hooks)**

In `~/.claude/settings.json`, add a new hook entry:

```json
{
  "matcher": "Bash",
  "hooks": [{
    "type": "command",
    "command": "node \"~/.claude/scripts/hooks/run-with-flags.js\" \"post:bash:maven-build\" \"rd/hooks/ear-maven-check.js\" \"standard,strict\""
  }]
}
```

**Option B — Create a standalone script in `rd/hooks/`**

Place a `.js` file in `rd/hooks/`, then wire it in `settings.json` using the path above.

**Option C — Extend an existing ECC hook**

For minor customizations, copy the ECC script to `rd/hooks/`, modify it, and update the `settings.json` path to point to the ear copy.

## Current Wired Hooks

See `ecc-dependency-map.md` for the full list of hooks currently active in `settings.json`.

## Hook Guidelines

- Hooks must be fast (< 10s for sync, < 30s for async)
- Return non-zero exit code to signal failure (but note: most hooks log warnings rather than blocking)
- Use `process.exit(0)` to pass through unchanged
- Avoid blocking the main Claude Code thread — use `async: true` for long-running hooks
- Never hardcode absolute paths — use `path.join()` and `os.homedir()` for cross-platform compatibility
