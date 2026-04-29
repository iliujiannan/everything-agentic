---
description: 需求全流程总控入口，启动技术方案设计并展示完整流水线
---

# 需求全流程规划

**需求描述：** $ARGUMENTS

## 流水线总览

| 阶段 | 命令 | 负责 Agent | 说明 |
|------|------|-----------|------|
| 1. 技术方案设计 | `/tech-design` | subagent-plan | 架构设计、核心流程分析、风险识别 |
| 2. 设计文档产出 | `/design-docs` | subagent-design-docs | 接口文档、数据库设计、技术方案文档 |
| 3. 编码实现 | `/code-impl` | subagent-code | 按设计实现代码，code-reviewer 自动介入 |
| 4. 单元测试 | `/unit-test` | subagent-test | TDD 补全单元测试，覆盖率 ≥ 80% |
| 5. Sonar 扫描 | `/sonar` | subagent-sonar | Quality Gate 扫描，最多 3 轮自动修复 |
| 6. Git 提交 | `/git-commit` | subagent-git | 生成 commit message，确认后提交 |
| 7. 构建 & 部署 | `/docker-build` + `/k8s-deploy` | subagent-deploy | 构建镜像并部署到 K8s |

## 执行规则

- 每个阶段结束**必须停止**，输出阶段摘要，等待用户输入 **[继续]**
- **禁止**自动跳转到下一阶段
- Sonar 自动修复上限 **3 轮**，超出停止汇报，不再循环
- Quality Gate 未通过**禁止**进入 Git 提交阶段
- 禁止硬编码任何密钥、密码、Token

---

## 立即执行：阶段 1 — 技术方案设计

使用 `subagent-plan` 分析以下需求，产出技术方案：

> $ARGUMENTS

调用 Agent 工具执行，subagent_type 为 `subagent-plan`，prompt 包含：
- 完整需求描述：$ARGUMENTS
- 项目技术栈：读取 `.claude/docs/tech-stack.md` 获取（按需加载，不要硬编码）
- 项目背景：读取 `.claude/skills/` 下对应项目的 `SKILL.md` 获取（按需加载，不要一次性全部读取）

阶段 1 完成后输出摘要，然后停止。
提示：✅ 阶段 1 完成。请 review 技术方案，确认后输入 **[继续]** 执行 `/design-docs $ARGUMENTS` 进入阶段 2。
