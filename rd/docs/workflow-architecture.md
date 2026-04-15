# 工作流架构

## 概述

ear (everything-agentic) 的研发工作流采用 **Orchestrator + Subagent** 模式，由一个编排器串行驱动 7 个独立 subagent，每个 subagent 在独立的上下文窗口中执行，完成后向编排器汇报阶段摘要。

## 架构图

```
用户
 │
 ▼
workflow-orchestrator          ← 唯一用户入口，持有全局状态
 │
 ├──► Agent tool ──► subagent-plan          [阶段1] 技术方案设计
 │         ↓ 阶段摘要
 │    STOP + 等待 [继续]
 │
 ├──► Agent tool ──► subagent-design-docs   [阶段2] 设计文档产出
 │         ↓ 阶段摘要
 │    STOP + 等待 [继续]
 │
 ├──► Agent tool ──► subagent-code          [阶段3] 编码实现
 │         ↓ 阶段摘要
 │    STOP + 等待 [继续]
 │
 ├──► Agent tool ──► subagent-test          [阶段4] 单元测试
 │         ↓ 阶段摘要（Gate: 覆盖率≥80%）
 │    STOP + 等待 [继续]
 │
 ├──► Agent tool ──► subagent-sonar         [阶段5] Sonar 扫描（loop ≤3）
 │         ↓ 阶段摘要（Gate: Quality Gate PASS）
 │    STOP + 等待 [继续]
 │
 ├──► Agent tool ──► subagent-git           [阶段6] Git 提交
 │         ↓ 阶段摘要
 │    STOP + 等待 [继续]
 │
 └──► Agent tool ──► subagent-deploy        [阶段7] 构建 & 部署
           │
           ├── Step 1: Docker build + push
           ├── Step 2: K8s deploy
           └── Step 3: 验证 Pod 状态
           ↓ 阶段摘要（工作流结束）
```

## 设计决策

### 为什么用 Subagent 而非独立 Agent

| 维度 | 独立 Agent | Subagent（本方案） |
|------|-----------|----------------|
| 上下文隔离 | 共享主会话，随对话增长 | 独立窗口，每阶段干净启动 |
| 上下文传递 | 自动继承（可能噪声多） | 编排器精确注入所需信息 |
| 并行支持 | 手动协调 | orchestrator 可同时派发多个 |
| 防止 thinking 签名失效 | 无保护 | 阶段切换时上下文重置，规避问题 |
| 可复用性 | 高（用户可直接调用） | 中（设计为由 orchestrator 调用） |

### 为什么 Docker + K8s 合并为一个 Subagent

- 强依赖：K8s 部署的前置是镜像已推送，不应存在"镜像构建完但未部署"的中间状态
- 单一确认点：用户只需确认一次"是否部署到 {env}"，减少打断次数
- 原子性：build → push → deploy 作为一个事务，失败时统一回滚/重试

### Gate 机制

每个阶段设置 Gate 条件，编排器在派发下一阶段前验证：

| 阶段 → 下一阶段 | Gate 条件 |
|--------------|----------|
| 1 → 2 | 用户输入 [继续] |
| 2 → 3 | 用户输入 [继续] |
| 3 → 4 | 代码无编译错误 |
| 4 → 5 | 测试覆盖率 ≥ 80% |
| 5 → 6 | Sonar Quality Gate PASS |
| 6 → 7 | 用户输入 [继续] |

### 数据传递

Subagent 之间通过文件系统传递产出物：

```
.claude/skills/{项目}/
├── plan-{feature}-{date}.md      ← 阶段1 产出
├── design-{feature}-{date}.md    ← 阶段2 产出
└── api-{feature}-{date}.md       ← 阶段2 产出（可选）
```

编排器在 prompt 中传入上阶段产出物的文件路径，subagent 自行读取。

## 文件结构

```
rd/agents/
├── workflow-orchestrator.md     ← 编排器（用户入口）
├── subagent-plan.md             ← 阶段1
├── subagent-design-docs.md      ← 阶段2
├── subagent-code.md             ← 阶段3
├── subagent-test.md             ← 阶段4
├── subagent-sonar.md            ← 阶段5
├── subagent-git.md              ← 阶段6
└── subagent-deploy.md           ← 阶段7（Docker + K8s）
```

## 使用方式

### 在项目中安装

```bash
# 复制 agents 到项目的 .claude 目录
cp -r rd/agents/ .claude/agents/

# 复制模板到项目根
cp rd/templates/CLAUDE.md ./CLAUDE.md

# 按技术栈填写
cp rd/templates/tech-stack.md .claude/docs/tech-stack.md
```

### 启动工作流

```
用户：开始工作流，实现用户登录功能
Claude：（激活 workflow-orchestrator，从阶段1开始）
```

或从指定阶段继续：

```
用户：从阶段5继续，Sonar 扫描
Claude：（orchestrator 直接派发 subagent-sonar）
```
