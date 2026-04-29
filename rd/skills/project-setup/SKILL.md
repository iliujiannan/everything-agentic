---
name: project-setup
description: |
  项目初始化引导 Skill。配合 subagent-setup 使用，通过交互式问答收集项目特有信息，
  自动生成 CLAUDE.md、tech-stack.md、sonar-my-bugs/config.json 等配置文件。
  当用户首次引入 ear plugin、说"初始化项目"、"setup"、"引导"时触发。
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
  - AskUserQuestion
---

# 项目初始化引导

本 skill 定义引导问题清单和配置文件生成逻辑，由 `subagent-setup` agent 调用执行。

## 引导问题清单

分为**必填组**和**选填组**，必填组逐一询问，选填组整体询问是否需要配置。

### 必填组

| # | 问题 | 字段 | 写入文件 | 示例 |
|---|------|------|---------|------|
| 1 | "项目名称是什么？（如 `my-service`）" | `PROJECT_NAME` | CLAUDE.md | `my-service` |
| 2 | "请用一句话描述项目用途" | `PROJECT_DESCRIPTION` | CLAUDE.md | `用户认证服务` |
| 3 | "项目使用哪种后端语言/框架？（如 `Java 17 + Spring Boot 3.x`）" | `BACKEND_TECH` | tech-stack.md | `Java 17 + Spring Boot 3.x` |
| 4 | "构建工具是什么？（如 `Maven`、`Gradle`）" | `BUILD_TOOL` | tech-stack.md | `Maven` |
| 5 | "数据库类型和版本？（如 `MySQL 5.7`、`PostgreSQL 16`）" | `DB_TYPE_VERSION` | tech-stack.md | `MySQL 5.7` |
| 6 | "项目知识库（设计文档、PRD 等）存放路径？（如 `.claude/skills/${PROJECT_NAME}/`）" | `SKILLS_DIR` | CLAUDE.md | `my-service` |

### 选填组（整体询问）

使用 AskUserQuestion：

```
问题："以下配置项可选配置，需要配置哪些？"
选项（多选）：
  - SonarQube 连接信息
  - K8s 集群信息
  - CI/CD 信息
  - 测试框架信息
  - 前端技术栈
  - 全部跳过
```

#### 选填：SonarQube

| 问题 | 字段 | 写入文件 |
|------|------|---------|
| "SonarQube 地址？（如 `http://sonar.example.com:9000`）" | `sonarUrl` | sonar-my-bugs/config.json |
| "SonarQube 项目 Key？" | `project` | sonar-my-bugs/config.json |
| "默认分支名？（如 `main`）" | `branch` | sonar-my-bugs/config.json |
| "SonarQube 用户名？" | `author` | sonar-my-bugs/config.json |

#### 选填：K8s

| 问题 | 字段 | 写入文件 |
|------|------|---------|
| "K8s 默认命名空间？" | `K8S_NAMESPACE` | tech-stack.md |
| "容器 Registry 地址？" | `CONTAINER_REGISTRY` | tech-stack.md |

#### 选填：CI/CD

| 问题 | 字段 | 写入文件 |
|------|------|---------|
| "CI/CD 工具？（如 `Jenkins`、`GitLab CI`）" | `CICD_TOOL` | tech-stack.md |
| "触发策略？（如 `push to main`）" | `CICD_TRIGGER` | tech-stack.md |

#### 选填：测试框架

| 问题 | 字段 | 写入文件 |
|------|------|---------|
| "单元测试框架？（如 `JUnit 5 + Mockito`）" | `TEST_FRAMEWORK` | tech-stack.md |
| "集成测试方式？（如 `Testcontainers mysql:5.7`）" | `INTEGRATION_TEST` | tech-stack.md |

#### 选填：前端

| 问题 | 字段 | 写入文件 |
|------|------|---------|
| "前端框架？（如 `React 18`、`Vue 3`）" | `FRONTEND_FRAMEWORK` | tech-stack.md |
| "包管理工具？（如 `pnpm`、`npm`）" | `PKG_MANAGER` | tech-stack.md |

## 配置文件生成

引导完成后，自动生成/填充以下文件：

### 1. `CLAUDE.md`（项目根目录）

从 `rd/templates/CLAUDE.md` 复制模板，替换占位符：
- `${PROJECT_NAME}` → 用户输入
- `${PROJECT_DESCRIPTION}` → 用户输入
- `${SKILLS_DIR}` → 用户输入

### 2. `.claude/docs/tech-stack.md`

从 `rd/templates/tech-stack.md` 复制模板，填充用户提供的技术栈信息。

### 3. `.claude/skills/sonar-my-bugs/config.json`

如果用户配置了 SonarQube，生成配置文件；否则不生成。

### 4. `.claude/skills/${SKILLS_DIR}/` 目录

创建空的 skill 目录结构：
```
.claude/skills/${SKILLS_DIR}/
└── SKILL.md    ← 空模板，包含标题和占位符
```

## 生成的 SKILL.md 空模板

```markdown
---
name: ${PROJECT_NAME}
description: ${PROJECT_DESCRIPTION} 的项目知识库，包含历史设计文档和业务上下文
---

# ${PROJECT_NAME}

> 本文件由 project-setup skill 自动生成，请手动补充项目背景信息。

## 项目背景

（请补充项目背景、业务领域、核心实体等信息）

## 参考文档

（请将 PRD、设计文档等放入此目录，按需加载）
```
