# ${PROJECT_NAME}

## 项目背景

> ${PROJECT_DESCRIPTION}

- 历史文档全部包含在 `.claude/skills/${SKILLS_DIR}/` 中，**按需读取，禁止一次性全量加载**
- 新生成的各类文档写入 `.claude/skills/${SKILLS_DIR}/`，**禁止写在其他地方**

→ 技术栈详情：[`.claude/docs/tech-stack.md`](.claude/docs/tech-stack.md)  
→ 先决条件：[`.claude/docs/prerequisites.md`](.claude/docs/prerequisites.md)

## 知识索引

| 类型 | 路径 | 读取策略 |
|------|------|---------|
| 历史文档 / 设计产出 | `.claude/skills/${SKILLS_DIR}/` | 按需读取 |
| 自定义命令 | `.claude/commands/` | 按需读取 |
| Agent 定义 | `.claude/agents/` | 按需读取 |
| 工程文档 | `.claude/docs/` | 必读：tech-stack.md |
| 编码 / 流程规则 | `.claude/rules/` | 见下方规则索引 |

## 工作流

> 编排入口：`workflow-orchestrator` agent  
> 架构说明：[`.claude/docs/workflow-architecture.md`](.claude/docs/workflow-architecture.md)

| 阶段 | 描述 | 类型 | 入口 |
|------|------|------|------|
| 1 | 技术方案设计 | ECC 原生 | subagent-plan |
| 2 | 设计文档产出 | 自定义命令 | subagent-design-docs → `/design-docs` |
| 3 | 编码实现 | ECC 原生 | subagent-code |
| 4 | 单元测试 | ECC 原生 | subagent-test → `/tdd` |
| 5 | Sonar 扫描 | 自定义命令 | subagent-sonar → `/sonar` |
| 6 | Git 提交 | 自定义命令 | subagent-git → `/git-commit` |
| 7 | 构建 & 部署 | 自定义命令 | subagent-deploy → `/docker-build` + `/k8s-deploy` |

## 独立命令

| 命令 | 说明 |
|------|------|
| `/rd-plan <需求>` | 需求全流程入口（8 阶段） |
| `/bugfix <描述>` | Bug 修复（轻量工作流） |
| `/sonarfix [模块]` | Sonar 专项修复（最简流程） |
| `/sonar-my-bugs` | SonarQube Bug 查询与修复（交互式） |
| `/code-review` | 代码审查（ECC 原生） |
| `/tdd` | TDD 驱动实现（ECC 原生） |
| `/test-coverage` | 测试覆盖率分析（ECC 原生） |

> 首次使用？运行项目初始化引导：让 Claude 执行 `subagent-setup` agent。

## 强制规则

| # | 规则 |
|---|------|
| 1 | 每个阶段结束必须停止，输出阶段摘要，等待用户输入 `[继续]` |
| 2 | 禁止自动跳转到下一阶段 |
| 3 | Sonar 自动修复上限 3 轮，超出后停止汇报，不再循环 |
| 4 | Quality Gate 未通过禁止进入 Git 提交阶段 |
| 5 | 禁止硬编码任何密钥、密码、Token |
| 6 | 历史文档禁止一次性全量加载 |
| 7 | 新生成文档必须写入 skills 目录，禁止写在其他地方 |

## 规则索引

→ [`rules/common/`](.claude/rules/common/) — 通用原则（编码风格、测试、安全、Git 工作流、Agent 编排）  
→ [`rules/zh/`](.claude/rules/zh/) — 中文翻译版本  
→ 语言特定规则按项目技术栈选装（`typescript/` / `python/` / `golang/` / `java/` 等）

规则优先级：**语言特定 > 通用**
