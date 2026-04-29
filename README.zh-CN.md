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

### 方式 A：交互式引导（推荐）

1. 安装本仓库的 `ear` 插件
2. 在项目中让 Claude 执行 `subagent-setup` agent
3. 回答交互式问题（项目名、技术栈、SonarQube、K8s 等）
4. 所有配置文件自动生成

### 方式 B：手动安装

1. 选择一个领域目录（如 `rd/`）
2. 将 agents 复制到项目的 `.claude/agents/`
3. 将 rules 复制到 `~/.claude/rules/`
4. 将 `rd/templates/CLAUDE.md` 复制到项目根目录并填写占位符
5. 将 `rd/templates/tech-stack.md` 复制到 `.claude/docs/tech-stack.md` 并填写技术栈

### 工作流

`rd/` Harness 提供完整的 **Orchestrator + Subagent** 研发工作流：

```
workflow-orchestrator  →  subagent-plan
                       →  subagent-design-docs
                       →  subagent-code
                       →  subagent-test
                       →  subagent-sonar       (循环 ≤ 3)
                       →  subagent-git
                       →  subagent-deploy      (Docker 构建 + K8s 部署)
```

每个阶段在独立的上下文窗口中运行，完成后停止等待 `[继续]` 再进入下一阶段。详见 [`rd/docs/workflow-architecture.md`](rd/docs/workflow-architecture.md)。

### 命令列表

| 命令 | 说明 |
|------|------|
| `/rd-plan <需求>` | 全流程入口（8 阶段） |
| `/tech-design` | 阶段 1 — 技术方案设计 |
| `/design-docs` | 阶段 2 — 接口与数据库设计文档 |
| `/code-impl` | 阶段 3 — 编码实现 |
| `/unit-test` | 阶段 4 — 单元测试（覆盖率 ≥80%） |
| `/sonar` | 阶段 5 — SonarQube 扫描（≤3 轮） |
| `/git-commit` | 阶段 6 — Git 提交 |
| `/docker-build` | 阶段 7a — Docker 镜像构建与推送 |
| `/k8s-deploy` | 阶段 7b — K8s 部署 |
| `/bugfix <描述>` | 轻量 Bug 修复工作流 |
| `/sonarfix [模块]` | Sonar 专项修复（最简流程） |
| `/sonar-my-bugs` | SonarQube Bug 查询与修复（交互式） |

### Skill 列表

| Skill | 说明 |
|-------|------|
| `sonar-my-bugs` | 按用户/项目查询并修复 SonarQube Bug，支持交互式配置 |
| `project-setup` | 项目初始化引导（自动生成 CLAUDE.md、tech-stack.md 等） |

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
