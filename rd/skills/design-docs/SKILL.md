---
name: design-docs
description: 根据已确认的技术方案，产出接口设计文档（md+pdf）、数据库设计文档、技术方案详细设计文档
origin: ear
---

调用 `/ear:design-docs` 执行以下四个文档的生成流程：

1. **api-design.md** — 接口文档（必须先读取项目本地模板 `templates/api-doc-template.md`）
2. **api-design.pdf** — 接口文档 PDF 版本
3. **db-design.md** — 数据库设计文档（含 Flyway migration SQL）
4. **technical-design.md** — 技术方案详细设计（Mermaid 时序图、核心类设计、异常处理链路）

所有文档输出到 `.claude/docs/output/{功能名称}/` 目录。

> 规则参考：`rd/rules/` 下阿里 Java 规范、MySQL 5.7 兼容性要求（DATETIME 无毫秒精度）

文档生成完毕后停止，等待用户 review 确认后输入 [继续] 进入编码阶段。
