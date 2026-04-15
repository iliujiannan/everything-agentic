---
name: design-docs
description: 根据已确认的技术方案，产出接口设计文档（md+pdf）、数据库设计文档、技术方案详细设计文档
---

基于当前对话中已确认的技术方案，生成以下文档到 `.claude/docs/output/$ARGUMENTS/` 目录。
$ARGUMENTS 为功能名称，例如 `user-auth`。

## 接口文档模板

**生成 api-design.md 之前，必须先读取模板文件**（按优先级依次查找，读取第一个存在的）：
1. `.claude/templates/api-doc-template.md`（项目本地模板，优先）
2. `~/.claude/templates/api-doc-template.md`（用户全局模板）

以该文件作为格式参考，确保输出格式完全一致。

---

## 文档1: api-design.md

严格参考模板的结构与格式，包含以下章节：

### 文档结构
1. 标题 + 版本信息 + 核心文件引用
2. 目录
3. 背景与架构概述
4. 认证机制详解（如有）
5. 公共结构说明（标准响应基类、分页响应基类、通用分页请求参数）
6. 接口详情（见下方格式规范）
7. 分页查询机制（如涉及分页）
8. 接口调用依赖关系
9. 注意事项与已知问题

### 单个接口的固定写法

每个接口严格按以下格式输出，不得省略任何一项：

**`HTTP方法 /路径`**

> 如有接口级别的特殊说明（已知问题、注意事项），在此处用引用块标注。

**请求参数：**

| 参数名 | 位置 | 类型 | 必须 | 说明 |
|--------|------|------|------|------|
| ...    | Path/Query/Body/Header | String/Integer/... | 是/否 | ... |

**响应参数：**

| 字段名 | 类型 | 说明 |
|--------|------|------|
| ...    | ...  | ...  |

**响应示例：**

```json
{
  ...
}
```

### 格式规则
- 位置：`Path` / `Query` / `Body` / `Header`
- 类型：使用 Java 类型（`String`、`Integer`、`Long`、`Boolean`、`List<X>`）
- 必须：`是` / `否`
- 嵌套字段用 `parent[].child` 表示，如 `groupRuleList[].direction`
- 所有业务错误码及含义列在"注意事项"章节，不要分散在各接口中

---

## 文档2: api-design.pdf

基于 api-design.md 同步生成 PDF 版本，与 markdown 放在同一目录下，文件名相同仅后缀为 `.pdf`。

生成命令（使用 md-to-pdf，支持表格/代码块/标题完整格式）：

```bash
# 先查找 Playwright MCP 的 Chrome 路径
CHROME_PATH=$(find "$LOCALAPPDATA/ms-playwright" -name "chrome.exe" 2>/dev/null | head -1)
# Linux/Mac: CHROME_PATH=$(find ~/.local/share/ms-playwright -name "chrome" 2>/dev/null | head -1)

npx md-to-pdf ".claude/docs/output/$ARGUMENTS/api-design.md" \
  --launch-options "{\"executablePath\":\"$CHROME_PATH\",\"args\":[\"--no-sandbox\"]}"
```

如果 md-to-pdf 不可用，改用 pandoc：
```bash
pandoc ".claude/docs/output/$ARGUMENTS/api-design.md" \
  -o ".claude/docs/output/$ARGUMENTS/api-design.pdf" \
  --pdf-engine=xelatex -V CJKmainfont="SimSun" -V geometry:margin=2cm
```

如果两种方案均不可用，输出提示：「PDF 生成失败，请用 VS Code Markdown PDF 插件手动导出」，不要报错退出。

---

## 文档3: db-design.md

包含：
- 新增或修改的表清单
- 每张表的字段定义（字段名、类型、约束、说明）
- 数据库兼容性说明（如 DATETIME 毫秒精度、JSON 类型支持等）
- 索引设计及理由
- 完整的 Flyway migration SQL（命名格式：`V{N}__{描述}.sql`）

---

## 文档4: technical-design.md

包含：
- 核心流程时序图（Mermaid 格式）
- 关键类/方法设计
- 异常处理链路
- 与现有模块的集成方式
- 性能和安全注意事项

---

所有文档全部生成完毕后，输出文档路径清单，然后停止。
提示：✅ 设计文档生成完成。请 review 以上文档，确认后输入 [继续] 进入下一阶段。
