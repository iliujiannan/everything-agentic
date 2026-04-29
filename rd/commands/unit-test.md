---
description: 阶段 4 — 单元测试，TDD 方式补全测试，覆盖率 ≥ 80%
---

## 执行步骤

调用 `subagent-test` subagent，传入目标模块：$ARGUMENTS

Agent 负责：

1. **扫描已实现的代码**，列出需要覆盖的类和方法

2. **按 TDD 方式编写测试**
   - 遵循项目测试框架配置（读取 `.claude/docs/tech-stack.md` 中的测试框架信息）
   - Service 层：Mock Repository，测试业务逻辑
   - Controller 层：MockMvc 测试 HTTP 行为
   - Repository 层（如有复杂查询）：使用项目配置的集成测试方式

3. **测试命名规范**
   ```
   void methodName_scenario_expectedBehavior()
   ```

4. **运行测试**
   ```bash
   <根据项目的构建工具运行测试，如 mvn test / gradle test>
   ```

5. **检查覆盖率**
   ```bash
   <根据项目的覆盖率工具运行，如 mvn jacoco:report / gradle jacocoTestReport>
   ```
   覆盖率 < 80% 时，补充缺失的测试用例

---

覆盖率达标后停止，输出测试报告摘要。
提示：✅ 阶段 4 完成。测试通过，覆盖率 {x}%。确认后输入 [继续] 执行 `/sonar` 进入阶段 5。
