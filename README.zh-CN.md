# everything-agentic

> 企业级 Claude Code Harness 工程模板

一个生产就绪的模板仓库，用于将 [Claude Code](https://claude.ai/code) 集成到企业工作流中。将团队沉淀的 skills、agents、hooks 和 rules 打包为按领域划分的可复用 Harness。

[English](./README.md)

## 仓库结构

本仓库按领域组织，每个领域是一个独立的 Harness 目录。

```
everything-agentic/
├── rd/          # 研发 / 软件工程
└── ...          # 更多领域持续补充（运维、数据、安全等）
```

## 领域说明

### `rd/` — 软件工程研发

面向软件研发工作流的 Claude Code Harness。

| 目录 | 用途 |
|------|------|
| `rd/agents/` | 专用子 Agent 定义文件 |
| `rd/skills/` | 领域特定的 Skill 包 |
| `rd/commands/` | 自定义斜杠命令定义 |
| `rd/rules/` | 分层编码与工作流规则 |
| `rd/hooks/` | 事件驱动的自动化脚本 |
| `rd/settings/` | Claude Code 配置模板 |
| `rd/templates/` | 项目级 CLAUDE.md 起始模板 |
| `rd/enterprise/` | 治理、审计与权限模板 |
| `rd/docs/` | 扩展文档 |

## 快速开始

1. 选择一个领域目录（如 `rd/`）
2. 将 agents 复制到 `~/.claude/agents/`
3. 将 rules 复制到 `~/.claude/rules/`
4. 将 settings 合并到 `~/.claude/settings.json`
5. 从 `rd/templates/` 中选一个 `CLAUDE.md` 放到项目根目录

## 设计理念

本 Harness 遵循分层配置模型：

- **Rules** — 定义标准和检查清单（做什么）
- **Skills** — 提供深度领域知识（怎么做）
- **Agents** — 委派专项子任务（谁来做）
- **Hooks** — 在生命周期事件中强制执行自动化（什么时候做）
- **Commands** — 将常用工作流暴露为斜杠命令（怎么触发）

## 贡献

参见 [CONTRIBUTING.md](./CONTRIBUTING.md)。

## 许可证

MIT
