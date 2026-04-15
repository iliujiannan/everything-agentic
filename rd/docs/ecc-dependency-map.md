# ECC Dependency Map

`ear` 插件基于 [ECC (Everything Claude Code)](https://github.com/affaan-m/everything-claude-code) 构建。ECC 是运行时前置依赖，ear 不复制 ECC 的源代码。

## Version Pinned

- **ECC v1.10.0** — commit `62519f2`
- 安装：`claude plugin install ecc@ecc`
- 或在 `settings.json` 中启用：`"ecc@ecc": true`

---

## Agents Used

以下 ECC agents 由 `ear` 的 rules 和 workflow 直接引用，调用时通过 `ecc:agent-name` 触发：

| Agent | 引用位置 | 用途 |
|-------|---------|------|
| `planner` | `rules/common/agents.md` | 复杂特性实现规划 |
| `architect` | `rules/common/agents.md` | 架构决策 |
| `tdd-guide` | `rules/common/agents.md` | TDD 工作流 |
| `code-reviewer` | `rules/common/agents.md`, `rules/zh/code-review.md` | 代码审查 |
| `security-reviewer` | `rules/common/agents.md` | 安全分析 |
| `build-error-resolver` | `rules/common/agents.md` | 构建错误修复 |
| `e2e-runner` | `rules/common/agents.md` | E2E 测试执行 |
| `refactor-cleaner` | `rules/common/agents.md` | 死代码清理 |
| `doc-updater` | `rules/common/agents.md` | 文档维护 |
| `java-reviewer` | `rules/java/coding-style.md` | Java 规范审查 |
| `database-reviewer` | `rules/common/code-review.md` | SQL/Schema 审查 |

---

## Commands Used

| Command | 引用位置 | 用途 |
|---------|---------|------|
| `/plan` | `rules/common/agents.md` | 触发 planner agent |
| `/review` | `rules/common/code-review.md` | 触发 code-reviewer |
| `/tdd` | `rules/common/testing.md` | 触发 tdd-guide |

---

## Skills Used (Runtime)

以下 skills 在 `settings.json` 的 hooks 中被直接引用，必须通过 ECC 安装：

| Skill | 引用方式 | 用途 |
|-------|---------|------|
| `continuous-learning-v2` | `settings.json` PreToolUse/PostToolUse hook | observe.sh 持续学习 |

---

## Hooks Runtime Dependency (CRITICAL)

所有 hooks 通过 `run-with-flags.js` 分发，以下脚本是运行时依赖（**不可缺失**）：

```
~/.claude/scripts/
├── hooks/
│   ├── run-with-flags.js          ← 核心分发器，所有 hook 的入口
│   ├── run-with-flags-shell.sh    ← Shell 端等价分发器
│   ├── lib/
│   │   ├── hook-flags.js          ← Flag 解析
│   │   ├── utils.js               ← 工具函数
│   │   └── resolve-ecc-root.js    ← ECC 根路径解析
│   ├── post-bash-command-log.js   ← 审计日志 + 成本记录
│   ├── post-bash-pr-created.js
│   ├── post-bash-build-complete.js
│   ├── quality-gate.js            ← 质量卡点
│   ├── design-quality-check.js
│   ├── post-edit-accumulator.js
│   ├── post-edit-console-warn.js
│   ├── governance-capture.js
│   ├── pre-bash-tmux-reminder.js
│   ├── pre-bash-git-push-reminder.js
│   ├── pre-bash-commit-quality.js
│   ├── doc-file-warning.js
│   ├── suggest-compact.js
│   ├── pre-compact.js
│   ├── session-start-bootstrap.js
│   ├── session-end-marker.js
│   ├── session-end.js
│   ├── evaluate-session.js
│   ├── cost-tracker.js
│   ├── desktop-notify.js
│   └── mcp-health-check.js
└── lib/
```

**警告**：这些脚本由 ECC 安装和维护，不要手动修改。如需定制，在 `rd/hooks/` 中创建副本并更新 `settings.json` 路径。

---

## What ear DOES NOT Depend On

以下 ECC 内容未被 ear 使用，不会被引用：

- ECC `.cursor/` / `.codebuddy/` / `.codex/` 规则（Cursor 专用）
- ECC `agents/` 下面向 `.agents` 平台的 36 个 skills
- ECC healthcare/finance/supply-chain/gan 等领域专用 skills
- ECC frontend-slides/investor-materials 等业务无关 skills
- ECC 安装系统内部脚本

---

## Upstream Sync

当 ECC 更新版本时：

1. 更新 `plugin.json` 中的 `dependencies.ecc@ecc` 版本约束
2. 重新验证 hooks 仍能正常工作
3. 更新本文件的版本号
