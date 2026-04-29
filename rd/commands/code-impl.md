---
description: 阶段 3 — 编码实现，按设计文档编写代码，code-reviewer 自动介入
---

## 执行步骤

调用 `subagent-code` subagent，传入实现目标：$ARGUMENTS

Agent 负责：

1. **读取设计文档**
   - 读取 `.claude/docs/output/$ARGUMENTS/technical-design.md`
   - 读取 `.claude/docs/output/$ARGUMENTS/api-design.md`
   - 读取 `.claude/docs/output/$ARGUMENTS/db-design.md`
   - 如果设计文档不存在，提示先执行 `/design-docs $ARGUMENTS`

2. **按顺序实现**（每层实现完毕后暂停，等待确认再继续）
   - DDL migration（根据 db-design.md）
   - Domain / Entity
   - Repository / Mapper
   - Service
   - Controller / API

3. **编码规范要求**（遵循 `.claude/rules/` 下的项目规范）
   - 构造器注入，禁止 Field 注入
   - 禁止 `SELECT *`，禁止 `${}` SQL 参数拼接
   - 异常交由 GlobalExceptionHandler 处理
   - 日志使用 SLF4J，禁止 System.out.println
   - 返回值使用统一包装类

4. **code-reviewer 介入**
   - 每个关键文件实现后，调用 `code-reviewer` agent 进行审查
   - CRITICAL / HIGH 问题立即修复后再继续

---

全部文件实现完毕后停止。
提示：✅ 阶段 3 完成。请 review 代码，确认后输入 [继续] 执行 `/unit-test $ARGUMENTS` 进入阶段 4。
