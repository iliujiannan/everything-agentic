---
description: 从 SonarQube 拉取指定用户/项目的 Bug 列表，生成报告并逐一修复。Cookie 每次交互式索取，branch/author/project 首次配置后自动复用。
---

# SonarQube Bug 查询与修复

## 说明

本命令会：
1. 交互式索取 Cookie（每次必填）
2. 首次运行要求配置 sonarUrl / author / project / branch（后续可沿用）
3. 向 SonarQube 查询指定用户负责的所有未解决 Bug（CRITICAL + MAJOR）
4. 获取每个 Bug 对应的规则说明和代码位置
5. 输出一份简要 Bug 总结报告
6. 确认后逐一修复每个 Bug

**使用方式：**
```
/sonar-my-bugs
```

无需传入任何参数。命令运行后会交互式引导你提供 Cookie 和配置信息。

## 执行

调用 `sonar-my-bugs` skill 完成全部工作。
